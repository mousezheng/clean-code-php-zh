
# 比较

## 使用全等

PHP官方有对 [identical comparison](http://php.net/manual/en/language.operators.comparison.php) 的描述

**反例:**

 `string` 会被默认转为 `intager`

```php
$a = '42';
$b = 42;

if ($a == $b) {//作者原文有误
   // 表达式恒为 true
}
```

这个比较 `$a != $b` 返回 `FALSE` 实际上应该是 `TRUE`，`string` 类型的 `42` 与 intager 的 `42` 是不同的。

**正例：**

identical 比较器将会比较类型和值。

```php
$a = '42';
$b = 42;

if ($a !== $b) {
    // 验证通过
}
```

 `$a !== $b` 为 `TRUE`.

**[⬆ 返回顶部](#目录)**
