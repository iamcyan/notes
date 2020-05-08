# 最长回文字符串

使用中心扩展法计算回文字符串

时间复杂度O(n^2)，空间复杂度O(n)

```php
class Solution {
    public function longestPalindrome($s) {
        $strLen = strlen($s);
        if ($strLen < 1) {
            return "";
        } elseif ($strLen <= 2) {
            return $s{0} == $s{$strLen-1} ? $s : $s{0};
        }
        $start = $offset = 0;
        
        for ($i = 0; $i < $strLen; $i++) {
            $len1 = $this->extendArouncCenter($s, $i, $i);
            $len2 = $this->extendArouncCenter($s, $i, $i+1);	//会出现 abccba这种。

						//计算边界条件这个地方，多关注，第一次写的时候没搞太好。
						//这个边界条件计算能否统一一下呢。
            if ($len1 > $len2) {
                if ($len1 > $offset) {
                    $start = $i - ($len1 - 1)/2;
                    $offset = $len1;
                }
            } else {
                if ($len2 > $offset) {
                    $start = $i - $len2/2 + 1;
                    $offset = $len2;
                }
            }

        }
        return substr($s, $start, $offset);
    }

		//中心点想周围扩展。
    public function extendArouncCenter($s, $l, $r) {
        while ($l >= 0 && $r < strlen($s) && $s{$l} == $s{$r}) {
            $l--;
            $r++;
        }
        return $r - $l - 1 ;
    }
}
```

