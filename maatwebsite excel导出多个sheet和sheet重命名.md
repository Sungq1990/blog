---
title: maatwebsite excel导出多个sheet和sheet重命名
date: 2020-09-25 14:37:54
tags: maatwebsite
---

最近工作中需要用到Excel导出，用的是php的laravel框架，导出Excel就用到maatwebsite这个包。包的具体安装方法
这里就不具体介绍了哈，可以自行百度之。
这次主要说下实际使用过程中越到的2个问题：
①一个Excel包含多个sheet
②sheet重命名
<!--more-->

直接上代码先说①
主要是implements WithMultipleSheets，然后实现sheets()这个方法，数组有几个就有几个sheet

```
namespace App\Exports;

use Maatwebsite\Excel\Concerns\WithMultipleSheets;

class SheetExport implements WithMultipleSheets
{
    private $header;
    private $data;

    public function __construct($header, $data)
    {
        $this->header = $header;
        $this->data = $data;
    }


    public function sheets(): array
    {
        $sheets = [];

        foreach ($this->data as $sheet) {
            $sheets[] = new FromCollectionExport($this->header, $sheet['sheet_name'],  $sheet['data']);
        }
        return $sheets;
    }
}
```

②sheet重命名
主要是WithTitle这个，实现title()这个方法，返回的字符串就是这个sheet的名称

```
namespace App\Exports;

use Maatwebsite\Excel\Concerns\FromCollection;
use Maatwebsite\Excel\Concerns\WithTitle;

class FromCollectionExport implements FromCollection, WithTitle
{
    private $header;
    private $sheet_name;
    private $data;

    public function __construct($header, $sheet_name, $data)
    {
        $this->header = $header;
        $this->sheet_name = $sheet_name;
        $this->data = $data;
    }

    public function collection()
    {

        $key_arr = [];
        //设置表头
        foreach ($this->header[0] as $key => $value) {
            $key_arr[] = $key;
        }

        //输入数据
        foreach ($this->data as $key => &$value) {
            $js = [];
            for ($i = 0; $i < count($key_arr); $i++) {
                $js = array_merge($js, [$key_arr[$i] => $value[$key_arr[$i]]]);
            }
            array_push($this->header, $js);
            unset($val);
        }

        return collect($this->header);
    }

    public function title(): string
    {
        return $this->sheet_name;
    }
}
```