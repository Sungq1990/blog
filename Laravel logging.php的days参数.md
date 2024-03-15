---
title: Laravel logging.php的days参数
date: 2024-03-15 13:47:00
tags: Laravel
---

laravel项目设置了日志按日分隔，然后days参数设置了30，但是过了一阵子去看30前的日志还在没被删除，
然后各种排查，最后发现被days这个配置的命名误导了，虽然叫days，但是实际上是max的意思。
直白的说配了这个看的不是日期，然后最多保留这个数量的日志文件。

具体可以看下面的代码实习

LogManager.php

```
protected function createDailyDriver(array $config)
{
    return new Monolog($this->parseChannel($config), [
        $this->prepareHandler(new RotatingFileHandler(
            $config['path'], $config['days'] ?? 7, $this->level($config),
            $config['bubble'] ?? true, $config['permission'] ?? null, $config['locking'] ?? false
        ), $config),
    ]);
}
```

RotatingFileHandler.php

```
public function __construct(string $filename, int $maxFiles = 0, $level = Logger::DEBUG, bool $bubble = true, ?int $filePermission = null, bool $useLocking = false)
{
    $this->filename = Utils::canonicalizePath($filename);
    $this->maxFiles = $maxFiles;
    $this->nextRotation = new \DateTimeImmutable('tomorrow');
    $this->filenameFormat = '{filename}-{date}';
    $this->dateFormat = static::FILE_PER_DAY;

    parent::__construct($this->getTimedFilename(), $level, $bubble, $filePermission, $useLocking);
}


protected function rotate(): void
{
    // update filename
    $this->url = $this->getTimedFilename();
    $this->nextRotation = new \DateTimeImmutable('tomorrow');

    // skip GC of old logs if files are unlimited
    if (0 === $this->maxFiles) {
        return;
    }

    $logFiles = glob($this->getGlobPattern());
    if (false === $logFiles) {
        // failed to glob
        return;
    }

    if ($this->maxFiles >= count($logFiles)) {
        // no files to remove
        return;
    }

    // Sorting the files by name to remove the older ones
    usort($logFiles, function ($a, $b) {
        return strcmp($b, $a);
    });

    foreach (array_slice($logFiles, $this->maxFiles) as $file) {
        if (is_writable($file)) {
            // suppress errors here as unlink() might fail if two processes
            // are cleaning up/rotating at the same time
            set_error_handler(function (int $errno, string $errstr, string $errfile, int $errline): bool {
                return false;
            });
            unlink($file);
            restore_error_handler();
        }
    }

    $this->mustRotate = false;
}
```
