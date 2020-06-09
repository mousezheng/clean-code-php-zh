# 变量

## 见名知意

**反例:**

```php
$ymdstr = $moment->format('y-m-d');
```

**正例:**

```php
$currentDate = $moment->format('y-m-d');
```



## 相同变量相同含义

同样含义的变量，尽量使用统一命名，避免不同命名相同含义而产生歧义。

**反例：**

```php
getUserInfo();
getUserData();
getUserRecord();
getUserProfile();
```

**正例：**

```php
getUser();
```



## 可理解的命名(上)

更多时候我们在读代码而不是写代码，所以编写可阅读可理解的代码是非常重要的，使用无意义的变量会使项目难以阅读，尽量使得项目的变量可理解（*译者注：不得不说给一个变量起一个合适的名字真的很难*）。

**反例：**

```php
//  448 什么东东?
$result = $serializer->serialize($data, 448);
```

**正例：**

```php
$json = $serializer->serialize($data, JSON_UNESCAPED_SLASHES | JSON_PRETTY_PRINT | JSON_UNESCAPED_UNICODE);
```

## 可理解的命名(下)

**反例：**

```php
class User
{
    // 7 是什么东东?
    public $access = 7;
}

// 4 是什么东东?
if ($user->access & 4) {
    // ...
}

// 为什么要这样处理?
$user->access ^= 2;
```

**正例：**

```php
class User
{
    public const ACCESS_READ = 1;
    public const ACCESS_CREATE = 2;
    public const ACCESS_UPDATE = 4;
    public const ACCESS_DELETE = 8;

    // 使用可读的值
    public $access = self::ACCESS_READ | self::ACCESS_CREATE | self::ACCESS_UPDATE;
}

if ($user->access & User::ACCESS_UPDATE) {
}

$user->access ^= User::ACCESS_CREATE;
```



## 有含义的命名

**反例：**

```php
$address = 'One Infinite Loop, Cupertino 95014';
$cityZipCodeRegex = '/^[^,]+,\s*(.+?)\s*(\d{5})$/';
preg_match($cityZipCodeRegex, $address, $matches);

saveCityZipCode($matches[1], $matches[2]);
```

**反例:**

此例稍微好点，但是大量使用正则时也会多有不便。

```php
$address = 'One Infinite Loop, Cupertino 95014';
$cityZipCodeRegex = '/^[^,]+,\s*(.+?)\s*(\d{5})$/';
preg_match($cityZipCodeRegex, $address, $matches);

[, $city, $zipCode] = $matches;
saveCityZipCode($city, $zipCode);
```

**正例：**

通过正则**子模式**的方式更利于阅读。

```php
$address = 'One Infinite Loop, Cupertino 95014';
$cityZipCodeRegex = '/^[^,]+,\s*(?<city>.+?)\s*(?<zipCode>\d{5})$/';
preg_match($cityZipCodeRegex, $address, $matches);

saveCityZipCode($matches['city'], $matches['zipCode']);
```



## 避免嵌套中提前返回(上)

if-else 语句多层嵌套使得代码很难再追加，简洁明了会更好。

**反例：**

```php
function isShopOpen($day): bool
{
    if ($day) {
        if (is_string($day)) {
            $day = strtolower($day);
            if ($day === 'friday') {
                return true;
            } elseif ($day === 'saturday') {
                return true;
            } elseif ($day === 'sunday') {
                return true;
            } else {
                return false;
            }
        } else {
            return false;
        }
    } else {
        return false;
    }
}
```

**正例：**

```php
function isShopOpen(string $day): bool
{
    if (empty($day)) {
        return false;
    }

    $openingDays = [
        'friday', 'saturday', 'sunday'
    ];

    return in_array(strtolower($day), $openingDays, true);
}
```



## 避免嵌套中提前返回(下)

**反例：**

```php
function fibonacci(int $n)
{
    if ($n < 50) {
        if ($n !== 0) {
            if ($n !== 1) {
                return fibonacci($n - 1) + fibonacci($n - 2);
            } else {
                return 1;
            }
        } else {
            return 0;
        }
    } else {
        return 'Not supported';
    }
}
```

**正例：**

```php
function fibonacci(int $n): int
{
    if ($n === 0 || $n === 1) {
        return $n;
    }

    if ($n >= 50) {
        throw new \Exception('Not supported');
    }

    return fibonacci($n - 1) + fibonacci($n - 2);
}
```



## 避免奇怪的映射

避免使读者疑惑或产生歧义。简洁明了会更好。

**反例：**

```php
$l = ['Austin', 'New York', 'San Francisco'];

for ($i = 0; $i < count($l); $i++) {
    $li = $l[$i];
    doStuff();
    doSomeOtherStuff();
    // ...
    // ...
    // ...
    // 怎么又是 $li?
    dispatch($li);
}
```

**正例：**

```php
$locations = ['Austin', 'New York', 'San Francisco'];

foreach ($locations as $location) {
    doStuff();
    doSomeOtherStuff();
    // ...
    // ...
    // ...
    dispatch($location);
}
```



## 不必要的上下文

如果 `class/object` 名已经明确背景，不需要在变量名中再次重复。

**反例：**

```php
class Car
{
    public $carMake;
    public $carModel;
    public $carColor;

    //...
}
```

**正例：**

```php
class Car
{
    public $make;
    public $model;
    public $color;

    //...
}
```



## 默认参数

**反例:**

`$breweryName` 允许为 `NULL` ,无法明确 `$breweryName` 类型。

```php
function createMicrobrewery($breweryName = 'Hipster Brew Co.'): void
{
    // ...
}
```

**反例:**

这种写法好理解，并且可以更好的控制变量。

```php
function createMicrobrewery($name = null): void
{
    $breweryName = $name ?: 'Hipster Brew Co.';
    // ...
}
```

**正例：**

可以使用 [type hinting](http://php.net/manual/en/functions.arguments.php#functions.arguments.type-declaration)，明确 `$breweryName` 不允许为 `NULL`

```php
function createMicrobrewery(string $breweryName = 'Hipster Brew Co.'): void
{
    // ...
}
```


