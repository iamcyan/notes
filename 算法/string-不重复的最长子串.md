# 给定一个字符串，求不重复的最长字串长度

```php
class Solution {

    /**
     * @param String $s
     * @return Integer
     */
    function lengthOfLongestSubstring($s) {
        $size = strlen($s);
        $start = $end = $length = $res = 0;
        while ($end < $size) {
            $tmpChar = $s{$end};
            for ($index = $start; $index < $end; $index++) {
                if ($s{$index} == $tmpChar) {
                    $start = $index+1;
                    $length = $end - $start;
                    break;
                }
            }

            $end++;
            $length++;
            $res = max($length, $res);
        }

        return $res;
    }
}
```



