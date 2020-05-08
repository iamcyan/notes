# PHP-文件下载、导出专题

## 导出为excel

> 以前导出excel的时候总是依赖于别人写好的代码，或者laravel-excel扩展包，并不清楚是如何导出为excel的。

##### 1、使用laravel-excel扩展包导出

>  扩展包的3.0的版本和2.0相比做了很多的改动，个人感觉更容易使用了。扩展包给出了很多基于query的导出，视图的导出。下面例子为基于array的导出，其他的查看文档即可。

导出类：

```php
use Maatwebsite\Excel\Concerns\FromCollection;
use Maatwebsite\Excel\Concerns\Exportable;
use Maatwebsite\Excel\Concerns\WithHeadings;
class UsersExport implements FromCollection, WithHeadings
{
    use Exportable;

    private $data;
    private $headings;
    
	//数据注入
    public function __construct($data, $headings)
    {
        $this->data = $data;
        $this->headings = $headings;
    }

	//实现FromCollection接口
    public function collection()
    {
        return collect($this->data);
    }

	//实现WithHeadings接口
    public function headings(): array
    {
        return $this->headings;
    }

}
```

控制器中导出

```php
namespace App\Http\Controllers;

use Maatwebsite\Excel\Facades\Excel;
use App\Exports\UsersExport;
class UsersController extends Controller
{
    public function export()
    {
        $data = [
            [
                'name' => 'cheng',
                'email' => 'cheng111'
            ],
            [
                'name' => 'cheng',
                'email' => 'cheng111'
            ],
        ];

        $headings = [
            'name',
            'email'
        ];
        return Excel::download(new UsersExport($data, $headings), 'users.csv');

    }
}
```



##### 2、使用流数据导出—简单的echo方法

```php
Route::get('/iamcyan', function() {
    //生命文件类型为字节流，浏览器处理字节流的形式默认为下载
    header("Content-type: application/octet-stream");
    
    //提供一个文件名，强制浏览器显示文件下载的对话框
    header("Content-Disposition: attachment; filename=test.csv");
    
    
    echo iconv('utf8', 'gbk','标题1, 标题2') . "\n";
    $output = [1,2];
    echo iconv("UTF-8", "gbk",  implode(',', $output)) . "\n";

    exit;
});
```



##### 3、使用流数据导出--fopen，开启php输出流

```php
public function exportBigData($params)
    {
        set_time_limit(0);

        $columns = ['字段名'];

        $fileName = 'GPS管理明细' . '.csv';
        //设置好告诉浏览器要下载excel文件的headers
        header('Content-Description: File Transfer');
        header('Content-Type: application/vnd.ms-excel');
        header('Content-Disposition: attachment; filename="'. $fileName .'"');
        header('Expires: 0');
        header('Cache-Control: must-revalidate');
        header('Pragma: public');

        $fp = fopen('php://output', 'a');//打开output流
        mb_convert_variables('GBK', 'UTF-8', $columns);
        fputcsv($fp, $columns);     //将数据格式化为CSV格式并写入到output流中

        $num = $this->getExportNum($params);
        $perSize = 2000;//每次查询的条数
        $pages   = ceil($num / $perSize);

        for($i = 1; $i <= $pages; $i++) {
            $list = $this->getUnitExportData($params, $i, $perSize);

            foreach($list as $item) {
                $rowData[] = $this->switchDeviceMakerToDesc($item['device_maker']);
                .
                .
                .
                mb_convert_variables('GBK', 'UTF-8', $rowData);
                fputcsv($fp, $rowData);
                unset($rowData);
            }
            unset($accessLog);//释放变量的内存
            
            ob_flush();		//刷新输出缓冲到浏览器
            flush();		//必须同时使用 ob_flush() 和flush() 函数来刷新输出缓冲。
        }
        fclose($fp);
        exit();
    }
```



##### 4、在laravel-china上面发现了一个导出excel的扩展

```
github地址：https://github.com/viest/php-ext-excel-export
```

##### 5、多个文件打包下载

```
//创建一个zip文件
$zip = new \ZipArchive();
$zip->open('test.zip', \ZipArchive::CREATE|\ZipArchive::OVERWRITE);
$zip->addFile('test1.csv');
$zip->addFile('test2.csv');
$zip->close();

//下载文件

header('Content-Type:application/octet-stream');
header('Content-Disposition: attachment; filename=iamcyan.zip');
readfile('iamcyan.zip');
exit();
```

