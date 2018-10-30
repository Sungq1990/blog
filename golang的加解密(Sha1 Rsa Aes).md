---
title: golang的加解密(Sha1 Rsa Aes)
date: 2018-08-03 16:45:00
tags: golang
---

公司的清算系统使用的是golang语言，支付需要对接第三方支付机构，第三方机构只有java，php，c#的demo，所以只能对着php和java的demo翻译成go，这家支付机构估计成立时间比较久，不同的支付终端有不同的加解密方式，这里记录一下使用到的golang加解密方法
<!--more-->

#### sha1
```
import (
  "crypto/sha1"
  "fmt"
  "io"
)
/**
 * sha1
 */
func Sha1String(str string) string {
  t := sha1.New()
  io.WriteString(t, str)
  return fmt.Sprintf("%x", t.Sum(nil))
}
```
#### Aes
```
import (
  "crypto/aes"
  "crypto/cipher"
  "bytes"
  "strconv"
  "bytes"
)
/**
 * Aes加密
 * @param []byte origData 待加密数据
 * @param []byte key Aeskey
 * @return string
 */
func AesEncrypt(origData, key []byte) (string, error) {
  block, err := aes.NewCipher(key)
  if err != nil {
    return "", err
  }
  blockSize := block.BlockSize()
  origData = PKCS5Padding(origData, blockSize)
  //这里iv为key[:blockSize]
  aes := cipher.NewCBCEncrypter(block, key[:blockSize])
  crypted := make([]byte, len(origData))
  aes.CryptBlocks(crypted, origData)

  return string(ByteToHex(crypted)), nil
}
/**
 * Aes解密
 * @param string crypted 待解密密数据
 * @param []byte key Aeskey
 * @return string
 */
func AesDecrypt(crypted string, key []byte) (string, error) {
  cryptedByte := HexToBye(crypted)
  block, err := aes.NewCipher(key)
  if err != nil {
    return "", err
  }
  blockMode := cipher.NewCBCDecrypter(block, key)
  origData := make([]byte, len(cryptedByte))
  blockMode.CryptBlocks(origData, cryptedByte)
  return string(origData), nil
}
/**
 * pkcs5填充
 */
func PKCS5Padding(ciphertext []byte, blockSize int) []byte {
  padding := blockSize - len(ciphertext) % blockSize
  padtext := bytes.Repeat([]byte{byte(padding)}, padding)
  return append(ciphertext, padtext...)
}
/**
 * 转16进制字符串
 */
func ByteToHex(data []byte) string {
  buffer := new(bytes.Buffer)
  for _, b := range data {
    s := strconv.FormatInt(int64(b & 0xff), 16)
    if len(s) == 1 {
      buffer.WriteString("0")
    }
    buffer.WriteString(s)
  }

  return buffer.String()
}
/**
 * 16进制字符串转[]byte
 */
func HexToBye(hex string) []byte {
  length := len(hex) / 2
  slice := make([]byte, length)
  rs := []rune(hex)

  for i := 0; i < length; i++ {
    s := string(rs[i * 2 : i * 2 + 2])
    value, _ := strconv.ParseInt(s, 16, 10)
    slice[i] = byte(value & 0xFF)
  }
  return slice
}
```
#### Rsa
```
import (
  "crypto"
  "crypto/rand"
  "crypto/rsa"
  "errors"
  "fmt"
  "math/big"
  "crypto/sha1"
)

// copy from crypt/rsa/pkcs1v5.go
var hashPrefixes = map[crypto.Hash][]byte{
  crypto.MD5:       {0x30, 0x20, 0x30, 0x0c, 0x06, 0x08, 0x2a, 0x86, 0x48, 0x86, 0xf7, 0x0d, 0x02, 0x05, 0x05, 0x00, 0x04, 0x10},
  crypto.SHA1:      {0x30, 0x21, 0x30, 0x09, 0x06, 0x05, 0x2b, 0x0e, 0x03, 0x02, 0x1a, 0x05, 0x00, 0x04, 0x14},
  crypto.SHA224:    {0x30, 0x2d, 0x30, 0x0d, 0x06, 0x09, 0x60, 0x86, 0x48, 0x01, 0x65, 0x03, 0x04, 0x02, 0x04, 0x05, 0x00, 0x04, 0x1c},
  crypto.SHA256:    {0x30, 0x31, 0x30, 0x0d, 0x06, 0x09, 0x60, 0x86, 0x48, 0x01, 0x65, 0x03, 0x04, 0x02, 0x01, 0x05, 0x00, 0x04, 0x20},
  crypto.SHA384:    {0x30, 0x41, 0x30, 0x0d, 0x06, 0x09, 0x60, 0x86, 0x48, 0x01, 0x65, 0x03, 0x04, 0x02, 0x02, 0x05, 0x00, 0x04, 0x30},
  crypto.SHA512:    {0x30, 0x51, 0x30, 0x0d, 0x06, 0x09, 0x60, 0x86, 0x48, 0x01, 0x65, 0x03, 0x04, 0x02, 0x03, 0x05, 0x00, 0x04, 0x40},
  crypto.MD5SHA1:   {}, // A special TLS case which doesn't use an ASN1 prefix.
  crypto.RIPEMD160: {0x30, 0x20, 0x30, 0x08, 0x06, 0x06, 0x28, 0xcf, 0x06, 0x03, 0x00, 0x31, 0x04, 0x14},
}

// copy from crypt/rsa/pkcs1v5.go
func encrypt(c *big.Int, pub *rsa.PublicKey, m *big.Int) *big.Int {
  e := big.NewInt(int64(pub.E))
  c.Exp(m, e, pub.N)
  return c
}

// copy from crypt/rsa/pkcs1v5.go
func pkcs1v15HashInfo(hash crypto.Hash, inLen int) (hashLen int, prefix []byte, err error) {
  // Special case: crypto.Hash(0) is used to indicate that the data is
  // signed directly.
  if hash == 0 {
    return inLen, nil, nil
  }

  hashLen = hash.Size()
  if inLen != hashLen {
    return 0, nil, errors.New("crypto/rsa: input must be hashed message")
  }
  prefix, ok := hashPrefixes[hash]
  if !ok {
    return 0, nil, errors.New("crypto/rsa: unsupported hash function")
  }
  return
}

// copy from crypt/rsa/pkcs1v5.go
func leftPad(input []byte, size int) (out []byte) {
  n := len(input)
  if n > size {
    n = size
  }
  out = make([]byte, size)
  copy(out[len(out) - n:], input)
  return
}
func unLeftPad(input []byte) (out []byte) {
  n := len(input)
  t := 2
  for i := 2; i < n; i++ {
    if input[i] == 0xff {
      t = t + 1
      if input[i + 1] == 0 {
        t += 1
      }
    } else {
      break
    }
  }
  out = make([]byte, n - t)
  copy(out, input[t:])
  return
}

// copy&modified from crypt/rsa/pkcs1v5.go
func publicDecrypt(pub *rsa.PublicKey, hash crypto.Hash, hashed []byte, sig []byte) (out []byte, err error) {
  hashLen, prefix, err := pkcs1v15HashInfo(hash, len(hashed))
  if err != nil {
    return nil, err
  }

  tLen := len(prefix) + hashLen
  k := (pub.N.BitLen() + 7) / 8
  if k < tLen + 11 {
    return nil, fmt.Errorf("length illegal")
  }
  c := new(big.Int).SetBytes(sig)
  m := encrypt(new(big.Int), pub, c)
  em := leftPad(m.Bytes(), k)

  out = unLeftPad(em)
  err = nil
  return
}
/**
 * 私钥加密
 */
func PrivateEncrypt(privt *rsa.PrivateKey, data []byte) ([]byte, error) {
  signData, err := rsa.SignPKCS1v15(nil, privt, crypto.Hash(0), data)
  if err != nil {
    return nil, err
  }
  return signData, nil
}
/**
 * 私钥加密(MD5withRSA)
 */
func PrivateEncryptMD5withRSA(privt *rsa.PrivateKey, data []byte) ([]byte, error) {
  hashMD5 := md5.New()
  hashMD5.Write(data)
  Digest := hashMD5.Sum(nil)
  signData, err := rsa.SignPKCS1v15(nil, privt, crypto.MD5, Digest)
  if err != nil {
    return nil, err
  }
  return signData, nil
}
/**
 * 公钥解密
 */
func PublicDecrypt(pub *rsa.PublicKey, data []byte) ([]byte, error) {
  decData, err := publicDecrypt(pub, crypto.Hash(0), nil, data)
  if err != nil {
    return nil, err
  }
  return decData, nil
}
/**
 * 公钥加密
 */
func PublicEncrypt(pub *rsa.PublicKey, data []byte) ([]byte, error) {
  signData, err := rsa.EncryptPKCS1v15(rand.Reader, pub, data)
  if err != nil {
    return nil, err
  }
  return signData, nil
}
/**
 * 私钥解密
 */
func PrivateDecrypt(privt *rsa.PrivateKey, data []byte) ([]byte, error) {
  signData, err := rsa.DecryptPKCS1v15(rand.Reader, privt, data)
  if err != nil {
    return nil, err
  }
  return signData, nil
}

/**
 * 私钥加签
 */
func PrivateSign(privt *rsa.PrivateKey, data []byte) ([]byte, error) {
  h := sha1.New()
  h.Write(data)
  hashed := h.Sum(nil)
  signData, err := rsa.SignPKCS1v15(rand.Reader, privt, crypto.SHA1, hashed)
  if err != nil {
    return nil, err
  }
  return signData, nil
}

/**
 * 公钥验签
 */
func PublicVerifySign(pub *rsa.PublicKey, src []byte, sign []byte) (error) {
  h := sha1.New()
  h.Write(src)
  hashed := h.Sum(nil)
  return rsa.VerifyPKCS1v15(pub, crypto.SHA1, hashed, sign)
}
```



