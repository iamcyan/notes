# 长度为k的最小字典子序列

给出一个长度为n(1<n<10000)的只有小写字母的字符串，然后找出一个长度为k的最小字典子序列。

思考点：

1、什么叫字典序列

2、保证得到的の字符串是最小的

```php
$str = "hklasdi";
$m = 3;
$strlen = strlen($str);
//遍历字符串，构建二维数组，统计每个数字出现的下标
$countArr = [];
$strArr = str_split($str);
foreach ($strArr as $key => $value) {
   if (isset($countArr[$value])) {
       array_push($countArr[$value], $value);
   } else {
       $countArr[$value][] = $key;
   }
}

//对数组按照key排序，排序之后key就是字典序列。
ksort($countArr);
$res = "";
$preIndex = 0;

for ($i =0; $i < $m; $i++) {
    $maxPos = $strlen - $m + $i;	//计算本次可取的最大index
    foreach ($countArr as $key => $value) {
        foreach ($value as $k => $index) {
            if ($index <= $maxPos && $index > $preIndex) {
                $res .= $key;
                $preIndex = $index;	//记录当前索引的位置，下一个字母的索引需要大于此值。
                unset($countArr[$key][$k]);
                break 2;
            }
        }
    }

}
echo $res;
```

