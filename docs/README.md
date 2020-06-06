# Clean Code PHP

## 目录

  1. [简介](#简介)
  2. [变量](#变量)
     * [见名知意](#见名知意)
     * [相同变量相同含义](#相同变量相同含义)
     * [可理解的命名 (上)](#可理解的命名(上))
     * [可理解的命名 (下)](#可理解的命名(下))
     * [有含义的命名](#有含义的命名)
     * [避免嵌套中提前返回(上)](#避免嵌套中提前返回(上))
     * [避免嵌套中提前返回(下)](#避免嵌套中提前返回(上))
     * [避免奇怪的映射](#避免奇怪的映射)
     * [不必要的上下文](#不必要的上下文)
     * [默认参数避](#默认参数避)
  3. [比较](#比较)
     * [使用全等](#使用全等)
  4. [方法](#方法)
     * [方法参数少于两个](#方法参数少于两个)
     * [见名知意](#方法见名知意)
     * [只被一个水平抽象](#只被一个水平抽象)
     * [不使用标志作参数](#不使用标志作参数)
     * [避免副作用](#避免副作用)
     * [不用全局函数](#不用全局函数)
     * [不使用单例模式](#不使用单例模式)
     * [封装条件](#封装条件)
     * [避免非条件](#避免非条件)
     * [避免条件](#避免条件)
     * [避免类型检查(上)](#避免类型检查(上))
     * [避免类型检查(下)](#避免类型检查(下))
     * [删除无用代码](#删除无用代码)
  5. [对象和数据结构](#对象和数据结构)
     * [封装对象](#封装对象)
     * [private/protected](#privateprotected)
  6. [类](#类)
     * [组合而不是继承](#组合而不是继承)
     * [避免流式接口](#避免流式接口)
     * [Prefer final classes](#prefer-final-classes)
  7. [SOLID](#solid)
     * [Single Responsibility Principle (SRP)](#single-responsibility-principle-srp)
     * [Open/Closed Principle (OCP)](#openclosed-principle-ocp)
     * [Liskov Substitution Principle (LSP)](#liskov-substitution-principle-lsp)
     * [Interface Segregation Principle (ISP)](#interface-segregation-principle-isp)
     * [Dependency Inversion Principle (DIP)](#dependency-inversion-principle-dip)
  8. [Don’t repeat yourself (DRY)](#dont-repeat-yourself-dry)
  9. [Translations](#translations)

## 简介

PHP版本的软件工程原则, 改编自 C. Martin 的书
[*Clean Code*](https://www.amazon.com/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882),
. 本篇不是风格指南.而是一个用于开发可读、可复用并且可重构PHP软件的指南.

不是所有的原则都需要被遵守，并且少数原则还未被普遍认可。很多指南和其他东西都是一些通过*Clean Code*作者多年积攒下的经验改编而成的。深受 [clean-code-javascript](https://github.com/ryanmcdermott/clean-code-javascript) 的启发.

即使很多开发者依旧使用 PHP5，本篇文章中更多的离职仍然使用 PHP7.1+ 。(*译者注：改变是趋势，拥抱变化*)

## 变量

### 见名知意

**反例:**

```php
$ymdstr = $moment->format('y-m-d');
```

**正例:**

```php
$currentDate = $moment->format('y-m-d');
```

**[⬆ 返回顶部](#目录)**

### 相同变量相同含义

同样含义的变量，尽量使用同一变量，避免不同变量相同含义而产生歧义。

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

**[⬆ 返回顶部](#目录)**

### 可理解的命名(上)

更多时候我们在读代码而不是写代码，所以编写可阅读可理解的代码是非常重要的，使用无意义的变量会使项目难以阅读，尽量项目的变量可理解（*译者注：不得不说给一个变量起一个合适的名字真的很难*）。

**反例：**

```php
//  448 什么东东?
$result = $serializer->serialize($data, 448);
```

**正例：**

```php
$json = $serializer->serialize($data, JSON_UNESCAPED_SLASHES | JSON_PRETTY_PRINT | JSON_UNESCAPED_UNICODE);
```

### 可理解的命名(下)

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

**[⬆ 返回顶部](#目录)**

### 有含义的命名

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

**[⬆ 返回顶部](#目录)**

### 避免嵌套中提前返回(上)

许多 if-else 语句可以使得代码很难再追加，明确会比隐含更理想。

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

**[⬆ 返回顶部](#目录)**

### 避免嵌套中提前返回(下)

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

**[⬆ 返回顶部](#目录)**

### 避免奇怪的映射

不要高估读者能够理准确理解代码的含义。明确会比隐含更理想。

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
    // Wait, what is `$li` for again?
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

**[⬆ 返回顶部](#目录)**

### 不必要的上下文

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

**[⬆ 返回顶部](#目录)**

### 默认参数避

**反例:**

`$breweryName` 允许为 `NULL` ,无法明确 `$breweryName` 类型。

```php
function createMicrobrewery($breweryName = 'Hipster Brew Co.'): void
{
    // ...
}
```

**反例:**


这种写法比更好理解，并且可以更好的控制变量。

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

**[⬆ 返回顶部](#目录)**

## 比较

### 使用全等

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
    // The expression is verified
}
```

 `$a !== $b` 为 `TRUE`.

**[⬆ 返回顶部](#目录)**


## 方法

### 方法参数少于两个

为了使方法更方便测试，所以要限制方法参数个数。如果有超过三个参数，自由组合的情况将会爆炸增长。没有参数是最理想的情况，一个或者两个参数也是可以的，但是避免超过三个。通常，如果超过两个参数方法就需要很多种测试用例，想办法将超出的参数组合起来。大多数时候一个 “高级” 的（higher-level）Object 只会有一个参数。

**反例：**

```php
class Questionnaire
{
    public function __construct(
        string $firstname,
        string $lastname,
        string $patronymic,
        string $region,
        string $district,
        string $city,
        string $phone,
        string $email
    ) {
        // ...
    }
}
```

**正例：**

```php
class Name
{
    private $firstname;
    private $lastname;
    private $patronymic;

    public function __construct(string $firstname, string $lastname, string $patronymic)
    {
        $this->firstname = $firstname;
        $this->lastname = $lastname;
        $this->patronymic = $patronymic;
    }

    // getters ...
}

class City
{
    private $region;
    private $district;
    private $city;

    public function __construct(string $region, string $district, string $city)
    {
        $this->region = $region;
        $this->district = $district;
        $this->city = $city;
    }

    // getters ...
}

class Contact
{
    private $phone;
    private $email;

    public function __construct(string $phone, string $email)
    {
        $this->phone = $phone;
        $this->email = $email;
    }

    // getters ...
}

class Questionnaire
{
    public function __construct(Name $name, City $city, Contact $contact)
    {
        // ...
    }
}
```

**[⬆ 返回顶部](#目录)**

### 方法见名知意

一个方法的名字应该告诉读者，其主要目的做了件什么事。

**反例：**

```php
class Email
{
    //...

    public function handle(): void
    {
        mail($this->to, $this->subject, $this->body);
    }
}

$message = new Email(...);
// 这是什么？ 一个消息的执行？具体执行了什么？
$message->handle();
```

**正例：**

```php
class Email
{
    //...

    public function send(): void
    {
        mail($this->to, $this->subject, $this->body);
    }
}

$message = new Email(...);
// 清楚明白的做法
$message->send();
```

**[⬆ 返回顶部](#目录)**

### 只被一个水平抽象

当一个方法超过一个水平的抽象时它做的事太多了。合理水平拆分方法使其可重用并方便测试。


**反例：**

```php
function parseBetterPHPAlternative(string $code): void
{
    $regexes = [
        // ...
    ];

    $statements = explode(' ', $code);
    $tokens = [];
    foreach ($regexes as $regex) {
        foreach ($statements as $statement) {
            // ...
        }
    }

    $ast = [];
    foreach ($tokens as $token) {
        // lex...
    }

    foreach ($ast as $node) {
        // parse...
    }
}
```

**反例：**

我们已经做了一定的功能性拆分，但是 `parseBetterPHPAlternative()` 方法仍然非常复杂并且不利于测试。

```php
function tokenize(string $code): array
{
    $regexes = [
        // ...
    ];

    $statements = explode(' ', $code);
    $tokens = [];
    foreach ($regexes as $regex) {
        foreach ($statements as $statement) {
            $tokens[] = /* ... */;
        }
    }

    return $tokens;
}

function lexer(array $tokens): array
{
    $ast = [];
    foreach ($tokens as $token) {
        $ast[] = /* ... */;
    }

    return $ast;
}

function parseBetterPHPAlternative(string $code): void
{
    $tokens = tokenize($code);
    $ast = lexer($tokens);
    foreach ($ast as $node) {
        // parse...
    }
}
```

**正例：**

最好的方案是将 `parseBetterPHPAlternative()` 方法单独拆分出去，独立封装。

```php
class Tokenizer
{
    public function tokenize(string $code): array
    {
        $regexes = [
            // ...
        ];

        $statements = explode(' ', $code);
        $tokens = [];
        foreach ($regexes as $regex) {
            foreach ($statements as $statement) {
                $tokens[] = /* ... */;
            }
        }

        return $tokens;
    }
}

class Lexer
{
    public function lexify(array $tokens): array
    {
        $ast = [];
        foreach ($tokens as $token) {
            $ast[] = /* ... */;
        }

        return $ast;
    }
}

class BetterPHPAlternative
{
    private $tokenizer;
    private $lexer;

    public function __construct(Tokenizer $tokenizer, Lexer $lexer)
    {
        $this->tokenizer = $tokenizer;
        $this->lexer = $lexer;
    }

    public function parse(string $code): void
    {
        $tokens = $this->tokenizer->tokenize($code);
        $ast = $this->lexer->lexify($tokens);
        foreach ($ast as $node) {
            // parse...
        }
    }
}
```

**[⬆ 返回顶部](#目录)**

### 不使用标志作参数

标志会让使用者觉得方法做了不止一件事，方法应该做一件事。如果代码是通过boolean做不同的操作，那么就需要做拆分。

**反例：**

```php
function createFile(string $name, bool $temp = false): void
{
    if ($temp) {
        touch('./temp/'.$name);
    } else {
        touch($name);
    }
}
```

**正例：**

```php
function createFile(string $name): void
{
    touch($name);
}

function createTempFile(string $name): void
{
    touch('./temp/'.$name);
}
```

**[⬆ 返回顶部](#目录)**

### 避免副作用

如果函数执行其他操作，而不是将一个值带入并返回另一个值或多个值，则会产生副作用。写入一个文件、修改一些全局变量或者不小心把所有钱汇给陌生人都是所谓的副作用。

要点是要避免常见的陷阱，例如在对象之间共享状态任何结构，使用可以被任何东西写入的可变数据类型，而不是
集中发生副作用的地方。 

**反例：**

```php
// 在下面这个方法中引入一个全局变量
// 如果我们其他方法使用这个 $name, $name 是个数组并且修改其内容
$name = 'Ryan McDermott';

function splitIntoFirstAndLastName(): void
{
    global $name;

    $name = explode(' ', $name);
}

splitIntoFirstAndLastName();

var_dump($name); // ['Ryan', 'McDermott'];
```

**正例：**

```php
function splitIntoFirstAndLastName(string $name): array
{
    return explode(' ', $name);
}

$name = 'Ryan McDermott';
$newName = splitIntoFirstAndLastName($name);

var_dump($name); // 'Ryan McDermott';
var_dump($newName); // ['Ryan', 'McDermott'];
```

**[⬆ 返回顶部](#目录)**

### 不用全局函数

很多语言里**全局污染**是一个不好的情况，因为这样会与其他库产生冲突，并且使用这样的 API 是非常不明智的，很有可能在生产中产生一些意料之外的状况。思考一个例子，如果香做一个配置数组该干什么？编写一个全局的方法 `config()`,但是这个造成**全局污染**

**反例：**

```php
function config(): array
{
    return  [
        'foo' => 'bar',
    ]
}
```

**正例：**

```php
class Configuration
{
    private $configuration = [];

    public function __construct(array $configuration)
    {
        $this->configuration = $configuration;
    }

    public function get(string $key): ?string
    {
        return isset($this->configuration[$key]) ? $this->configuration[$key] : null;
    }
}
```

加载配置并且创建一个 `Configuration` 类的实例。

```php
$configuration = new Configuration([
    'foo' => 'bar',
]);
```

并且必须使用 `Configuration` 的实例到应用中。

**[⬆ 返回顶部](#目录)**

### 不使用单例模式

单例是一个 [反面模式](https://baike.baidu.com/item/%E5%8F%8D%E9%9D%A2%E6%A8%A1%E5%BC%8F/3703771?fr=aladdin).下面是 **Brian Button** 的解释：

 1. 通常使用一个 **全局实例**，为什么这个很糟糕？因为在应用的代码中**隐藏依赖**，实例仅暴漏它们通过某些接口实现。
 2. 它们违背了 [单一职责原则](#单一职责原则)：通过**他们控制他们自己的创建和生命周期**。
 3. 它们会使得代码 [耦合性](https://baike.baidu.com/item/%E8%80%A6%E5%90%88%E6%80%A7) 更高。它们通过伪装使得**测试更加困难**
 4. 他们关心状态在应用程序的整个生命周期中。 **在需要结束的时候，需要通过一个命令**，这个对于测试来说是很不理想的。为什么？因为每个单元测试都应该互相独立。

这里也有许多很好的资料可做参考，作者[Misko Hevery](http://misko.hevery.com/about/) 关于 [root of problem](http://misko.hevery.com/2008/08/25/root-cause-of-singletons/) .

**反例：**

```php
class DBConnection
{
    private static $instance;

    private function __construct(string $dsn)
    {
        // ...
    }

    public static function getInstance(): DBConnection
    {
        if (self::$instance === null) {
            self::$instance = new self();
        }

        return self::$instance;
    }

    // ...
}

$singleton = DBConnection::getInstance();
```

**正例：**

```php
class DBConnection
{
    public function __construct(string $dsn)
    {
        // ...
    }

     // ...
}
```

创建一个 `DBConnection` 实例并且配置 [DSN](http://php.net/manual/en/pdo.construct.php#refsect1-pdo.construct-parameters)。

```php
$connection = new DBConnection($dsn);
```

在应用中使用 `DBConnection`

**[⬆ 返回顶部](#目录)**

### 封装条件

**反例：**

```php
if ($article->state === 'published') {
    // ...
}
```

**正例：**

```php
if ($article->isPublished()) {
    // ...
}
```

**[⬆ 返回顶部](#目录)**

### 避免非条件

**反例：**

```php
function isDOMNodeNotPresent(\DOMNode $node): bool
{
    // ...
}

if (!isDOMNodeNotPresent($node))
{
    // ...
}
```

**正例：**

```php
function isDOMNodePresent(\DOMNode $node): bool
{
    // ...
}

if (isDOMNodePresent($node)) {
    // ...
}
```

**[⬆ 返回顶部](#目录)**

### 避免条件

这个似乎是个不可能的任务。
初次听到，很多的人说，“我应该怎么做才能不是使用 `if` 语句？”。
回答是，可以使用多态在很多方案中实现这些任务。
问题通常是，“看似大而空，为什么要这样做呢”。
这个回答是 clean code 中的一个概念，一个方法应该仅仅做一件事。当我们有一个类并且有 `if` 语句方法，应该告诉使用者方法做了不知一件事。

切记 **单一职责（just do one thing）**

**反例：**

```php
class Airplane
{
    // ...

    public function getCruisingAltitude(): int
    {
        switch ($this->type) {
            case '777':
                return $this->getMaxAltitude() - $this->getPassengerCount();
            case 'Air Force One':
                return $this->getMaxAltitude();
            case 'Cessna':
                return $this->getMaxAltitude() - $this->getFuelExpenditure();
        }
    }
}
```

**正例：**

```php
interface Airplane
{
    // ...

    public function getCruisingAltitude(): int;
}

class Boeing777 implements Airplane
{
    // ...

    public function getCruisingAltitude(): int
    {
        return $this->getMaxAltitude() - $this->getPassengerCount();
    }
}

class AirForceOne implements Airplane
{
    // ...

    public function getCruisingAltitude(): int
    {
        return $this->getMaxAltitude();
    }
}

class Cessna implements Airplane
{
    // ...

    public function getCruisingAltitude(): int
    {
        return $this->getMaxAltitude() - $this->getFuelExpenditure();
    }
}
```

**[⬆ 返回顶部](#目录)**

### 避免类型检查(上)

PHP 是无类型语言，意思就是一个变量可以作为任何类型。有时候会因为这个灵活而困扰，让人很希望在方法中做类型检查。这里有很多方法去避免，第一件事是去考虑一致 APIs。

**反例：**

```php
function travelToTexas($vehicle): void
{
    if ($vehicle instanceof Bicycle) {
        $vehicle->pedalTo(new Location('texas'));
    } elseif ($vehicle instanceof Car) {
        $vehicle->driveTo(new Location('texas'));
    }
}
```

**正例：**

```php
function travelToTexas(Vehicle $vehicle): void
{
    $vehicle->travelTo(new Location('texas'));
}
```

**[⬆ 返回顶部](#目录)**

### 避免类型检查(下)

如果你使用基础的原始的数据例如 string、integer 或者 array 等，即使使用 PHP 7+ 并且没使用多态，仍然需要做类型检查。可以考虑 [类型声明](http://php.net/manual/en/functions.arguments.php#functions.arguments.type-declaration) 或者严格的模式。它在标准 PHP 基础上提供了静态类型。手动检查类型的话可以做到 “类型安全”，但是这样会做很多的是，造成代码可读性差。保持 PHP 简洁干净，写好的测试，并且做好代码检查。除此以外，尽可能严格的使用 PHP 类型生命或者严格模式。

**反例：**

```php
function combine($val1, $val2): int
{
    if (!is_numeric($val1) || !is_numeric($val2)) {
        throw new \Exception('Must be of type Number');
    }

    return $val1 + $val2;
}
```

**正例：**

```php
function combine(int $val1, int $val2): int
{
    return $val1 + $val2;
}
```

**[⬆ 返回顶部](#目录)**

### 删除无用代码

无用代码仅仅是重复代码，没有理由再将它维护在代码库中。如果不用关心它，那就清理掉。如果仍然需要，依旧可以从历史版本中获取。

**反例：**

```php
function oldRequestModule(string $url): void
{
    // ...
}

function newRequestModule(string $url): void
{
    // ...
}

$request = newRequestModule($requestUrl);
inventoryTracker('apples', $request, 'www.inventory-awesome.io');
```

**正例：**

```php
function requestModule(string $url): void
{
    // ...
}

$request = requestModule($requestUrl);
inventoryTracker('apples', $request, 'www.inventory-awesome.io');
```

**[⬆ 返回顶部](#目录)**


## 对象和数据结构

### 封装对象

在 PHP 中，可以给方法设置 `public`, `protected` 和 `private` 关键字。使用它可以控制一个对象属性的修改。

* 除了获取对象属性还想做其他操作时，可以不去查看并且修改代码库中每一个存取器
* 在 `set` 中做简单验证
* 更好的封装内部表现
* 更简单的在 `getting` 和 `setting` 中添加日志和错误控制
* 这个类内部，可以覆盖默认方法
* 可以懒加载对象的属性，从服务器获取

另外，这里是一部分原理 [开闭原则](#开闭原则(OCP))

**反例：**

```php
class BankAccount
{
    public $balance = 1000;
}

$bankAccount = new BankAccount();

// Buy shoes...
$bankAccount->balance -= 100;
```

**正例：**

```php
class BankAccount
{
    private $balance;

    public function __construct(int $balance = 1000)
    {
      $this->balance = $balance;
    }

    public function withdraw(int $amount): void
    {
        if ($amount > $this->balance) {
            throw new \Exception('Amount greater than available balance.');
        }

        $this->balance -= $amount;
    }

    public function deposit(int $amount): void
    {
        $this->balance += $amount;
    }

    public function getBalance(): int
    {
        return $this->balance;
    }
}

$bankAccount = new BankAccount();

// Buy shoes...
$bankAccount->withdraw($shoesPrice);

// Get balance
$balance = $bankAccount->getBalance();
```

**[⬆ 返回顶部](#目录)**

### private/protected

* `public` 方法和属性是会造成更多的修改危险。因为外部的代码很容易依赖他们并且无法控制。**修改类会影响到所有这个类的使用者**
* `protected` 修饰符和 `public` 一样危险，因为他允许在子类范围内可用。`public` 与 `protected` 的不同仅仅在访问机制上，但是封装保证是一样的。**修改类会影响到所有继承类。**
* `private` 修饰符保证 **修改仅仅会影响到一个类**

因此，默认使用 `private` ，当需要提供外部类使用时用`public/protected`。

更多的信息可以阅读出自 [Fabien Potencier](https://github.com/fabpot) 的 [Protected vs Private](http://fabien.potencier.org/pragmatism-over-theory-protected-vs-private.html).

**反例：**

```php
class Employee
{
    public $name;

    public function __construct(string $name)
    {
        $this->name = $name;
    }
}

$employee = new Employee('John Doe');
echo 'Employee name: '.$employee->name; // Employee name: John Doe
```

**正例：**

```php
class Employee
{
    private $name;

    public function __construct(string $name)
    {
        $this->name = $name;
    }

    public function getName(): string
    {
        return $this->name;
    }
}

$employee = new Employee('John Doe');
echo 'Employee name: '.$employee->getName(); // Employee name: John Doe
```

**[⬆ 返回顶部](#目录)**

## 类

### 组合而不是继承

Gang of Four 的 [*设计模式*](https://baike.baidu.com/item/%E8%BD%AF%E4%BB%B6%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F/2117635?fromtitle=%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F&fromid=1212549) 中一个很著名的说法，尽可能使用组合而不是继承。有很多支持继承原因也有很多支持组合原因。
这句话主要的意思是，如果本能的想使用继承是，尝试考虑下使用组合解决问题会不会更好。

你可能有疑虑，什么时候应该使用继承？，这个取决于业务问题，这事一份不错的清单，说明组合比继承更有意义：

1. 继承代表的是 ”是一个“ 的关系，而不是一个 ”有一个“ 的关系。（例如 帆船是一个船 VS 帆船有一个帆）
2. 可以重用代码通过基础的类（帆船可以像船一样行驶在水上）
3. 可以通过改变一个基础类，从而做到全局改变。（改变所有船的行驶速度）

**反例：**

```php
class Employee
{
    private $name;
    private $email;

    public function __construct(string $name, string $email)
    {
        $this->name = $name;
        $this->email = $email;
    }

    // ...
}

// Bad because Employees "have" tax data.
// EmployeeTaxData is not a type of Employee

class EmployeeTaxData extends Employee
{
    private $ssn;
    private $salary;

    public function __construct(string $name, string $email, string $ssn, string $salary)
    {
        parent::__construct($name, $email);

        $this->ssn = $ssn;
        $this->salary = $salary;
    }

    // ...
}
```

**正例：**

```php
class EmployeeTaxData
{
    private $ssn;
    private $salary;

    public function __construct(string $ssn, string $salary)
    {
        $this->ssn = $ssn;
        $this->salary = $salary;
    }

    // ...
}

class Employee
{
    private $name;
    private $email;
    private $taxData;

    public function __construct(string $name, string $email)
    {
        $this->name = $name;
        $this->email = $email;
    }

    public function setTaxData(string $ssn, string $salary)
    {
        $this->taxData = new EmployeeTaxData($ssn, $salary);
    }

    // ...
}
```

**[⬆ 返回顶部](#目录)**

### 避免流式接口



一个 [流式接口](https://baike.baidu.com/item/%E6%B5%81%E5%BC%8F%E6%8E%A5%E5%8F%A3) 是一个对象定向的API 目的是提高源码使用使用 [方法链](https://baike.baidu.com/item/%E6%96%B9%E6%B3%95%E9%93%BE) 的可读性。

可以在很多语境中使用，通常构用来建对象，这种方式减少了代码的赘余（例如 [PHPUnit Mock Builder](https://phpunit.de/manual/current/en/test-doubles.html) 或者 [Doctrine Query Builder](http://docs.doctrine-project.org/projects/doctrine-dbal/en/latest/reference/query-builder.html))，更多时候需要付出一些代价：


1. 破坏 [封装](https://baike.baidu.com/item/%E5%B0%81%E8%A3%85/13027517).
2. 破坏 [装饰模式](https://baike.baidu.com/item/%E8%A3%85%E9%A5%B0%E6%A8%A1%E5%BC%8F).
3. 测试中就更难 [模拟对象](https://baike.baidu.com/item/%E6%A8%A1%E6%8B%9F%E5%AF%B9%E8%B1%A1) 。
4. 使得提交变更更难阅读.

更多信息可以阅读 [Marco Pivetta](https://github.com/Ocramius) 的 [博客](https://ocramius.github.io/blog/fluent-interfaces-are-evil/)

**反例：**

```php
class Car
{
    private $make = 'Honda';
    private $model = 'Accord';
    private $color = 'white';

    public function setMake(string $make): self
    {
        $this->make = $make;

        // NOTE: Returning this for chaining
        return $this;
    }

    public function setModel(string $model): self
    {
        $this->model = $model;

        // NOTE: Returning this for chaining
        return $this;
    }

    public function setColor(string $color): self
    {
        $this->color = $color;

        // NOTE: Returning this for chaining
        return $this;
    }

    public function dump(): void
    {
        var_dump($this->make, $this->model, $this->color);
    }
}

$car = (new Car())
  ->setColor('pink')
  ->setMake('Ford')
  ->setModel('F-150')
  ->dump();
```

**正例：**

```php
class Car
{
    private $make = 'Honda';
    private $model = 'Accord';
    private $color = 'white';

    public function setMake(string $make): void
    {
        $this->make = $make;
    }

    public function setModel(string $model): void
    {
        $this->model = $model;
    }

    public function setColor(string $color): void
    {
        $this->color = $color;
    }

    public function dump(): void
    {
        var_dump($this->make, $this->model, $this->color);
    }
}

$car = new Car();
$car->setColor('pink');
$car->setMake('Ford');
$car->setModel('F-150');
$car->dump();
```

**[⬆ 返回顶部](#目录)**

### final

`final` 应该被尽可能的使用:

1. 防止不受控制的继承链
2. 鼓励使用 [组合](#组合而不是继承)
3. 鼓励使用 [单一职责](#单一职责)
4. 鼓励开发者使用 `public` 方法代替扩展类中使用 `protected`
5. 允许改变代码而不会破坏使用该类的应用

唯一的条件是类应该实现一个接口并且没有定义公共方法。

更多的信息可以阅读这篇 [Marco Pivetta (Ocramius)](https://ocramius.github.io/) 写的 [博客](https://ocramius.github.io/blog/when-to-declare-classes-final/) 。

**反例：**

```php
final class Car
{
    private $color;

    public function __construct($color)
    {
        $this->color = $color;
    }

    /**
     * @return string The color of the vehicle
     */
    public function getColor()
    {
        return $this->color;
    }
}
```

**正例：**

```php
interface Vehicle
{
    /**
     * @return string The color of the vehicle
     */
    public function getColor();
}

final class Car implements Vehicle
{
    private $color;

    public function __construct($color)
    {
        $this->color = $color;
    }

    /**
     * {@inheritdoc}
     */
    public function getColor()
    {
        return $this->color;
    }
}
```

**[⬆ 返回顶部](#目录)**

## SOLID

**SQLID** 是 Michael Feathers 对 Robert Martin 提出的面向对象五大基本设计原则命名的缩写。

 * [S: 单一职责原则 Single Responsibility Principle (SRP)](#单一职责原则)
 * [O: 开闭原则 Open/Closed Principle (OCP)](#开闭原则)
 * [L: 里氏代换原则 Liskov Substitution Principle (LSP)](#里氏代换原则)
 * [I: 接口隔离原则 Interface Segregation Principle (ISP)](#接口隔离原则)
 * [D: 依赖倒转原则 Dependency Inversion Principle (DIP)](#依赖倒转原则)

### 单一职责原则

Clean Code 中这样说：”改变一个类的原因不应该有一个以上“。都喜欢给一个类里塞很多功能，
就像坐飞机只能提一个行李箱于是把所有东西都塞进去。问题是这各类在概念上是不耦合的并且他将会因为各种原因而被改动
将修改一个类的情况降到最小是很重要的。如果太多方法在一个类里改变其中一个，对于理解代码库中其他依赖将会很困难。

**反例：**

```php
class UserSettings
{
    private $user;

    public function __construct(User $user)
    {
        $this->user = $user;
    }

    public function changeSettings(array $settings): void
    {
        if ($this->verifyCredentials()) {
            // ...
        }
    }

    private function verifyCredentials(): bool
    {
        // ...
    }
}
```

**正例：**

```php
class UserAuth
{
    private $user;

    public function __construct(User $user)
    {
        $this->user = $user;
    }

    public function verifyCredentials(): bool
    {
        // ...
    }
}

class UserSettings
{
    private $user;
    private $auth;

    public function __construct(User $user)
    {
        $this->user = $user;
        $this->auth = new UserAuth($user);
    }

    public function changeSettings(array $settings): void
    {
        if ($this->auth->verifyCredentials()) {
            // ...
        }
    }
}
```

**[⬆ 返回顶部](#目录)**

### 开闭原则

Bertrand Meyer 这样说，”软件实体（类、模块、方法等）应该对扩展开放，但是对修改关闭。“什么意思呢？这个原则说明了，应该使用增加一个方法而不是修改代码

**反例：**

```php
abstract class Adapter
{
    protected $name;

    public function getName(): string
    {
        return $this->name;
    }
}

class AjaxAdapter extends Adapter
{
    public function __construct()
    {
        parent::__construct();

        $this->name = 'ajaxAdapter';
    }
}

class NodeAdapter extends Adapter
{
    public function __construct()
    {
        parent::__construct();

        $this->name = 'nodeAdapter';
    }
}

class HttpRequester
{
    private $adapter;

    public function __construct(Adapter $adapter)
    {
        $this->adapter = $adapter;
    }

    public function fetch(string $url): Promise
    {
        $adapterName = $this->adapter->getName();

        if ($adapterName === 'ajaxAdapter') {
            return $this->makeAjaxCall($url);
        } elseif ($adapterName === 'httpNodeAdapter') {
            return $this->makeHttpCall($url);
        }
    }

    private function makeAjaxCall(string $url): Promise
    {
        // request and return promise
    }

    private function makeHttpCall(string $url): Promise
    {
        // request and return promise
    }
}
```

**正例：**

```php
interface Adapter
{
    public function request(string $url): Promise;
}

class AjaxAdapter implements Adapter
{
    public function request(string $url): Promise
    {
        // request and return promise
    }
}

class NodeAdapter implements Adapter
{
    public function request(string $url): Promise
    {
        // request and return promise
    }
}

class HttpRequester
{
    private $adapter;

    public function __construct(Adapter $adapter)
    {
        $this->adapter = $adapter;
    }

    public function fetch(string $url): Promise
    {
        return $this->adapter->request($url);
    }
}
```

**[⬆ 返回顶部](#目录)**

### 里氏代换原则

这个看似复杂的术语是非常简单的。它正式的定义了作为 ”如果 S 是 T 的父类（或者父接口或者抽象），
然后对象的类型是 T 可以被类型 S 的对象取代，而不修改项目中任何可取代属性（正确性，执行的任务等等）“ 
这个好像看起来更复杂。
 
最好的解释是  如果有一个父类和一个子类 然后基础类和子类可以互换不会造成错误的结果。这样解释仍然让人疑惑，
让我们看一个 Square-Rectangle 的例子。数学上 一个正方形是一个长方形 但是如果模使用通过继承实现 ”是一个“ 的关系，将会陷入麻烦。

**反例：**

```php
class Rectangle
{
    protected $width = 0;
    protected $height = 0;

    public function setWidth(int $width): void
    {
        $this->width = $width;
    }

    public function setHeight(int $height): void
    {
        $this->height = $height;
    }

    public function getArea(): int
    {
        return $this->width * $this->height;
    }
}

class Square extends Rectangle
{
    public function setWidth(int $width): void
    {
        $this->width = $this->height = $width;
    }

    public function setHeight(int $height): void
    {
        $this->width = $this->height = $height;
    }
}

function printArea(Rectangle $rectangle): void
{
    $rectangle->setWidth(4);
    $rectangle->setHeight(5);

    // BAD: Will return 25 for Square. Should be 20.
    echo sprintf('%s has area %d.', get_class($rectangle), $rectangle->getArea()).PHP_EOL;
}

$rectangles = [new Rectangle(), new Square()];

foreach ($rectangles as $rectangle) {
    printArea($rectangle);
}
```

**正例：**

最好的方式是分离四边形并且分配多个抽象给两种图形。

尽管正方形和长方形有很多相似的地方，但是他们都是不同的。
正方形和菱形有很多相似点，长方形和平行四边形也是，但是他们都不是抽象。
正方形、矩形、菱形和平行四边形都具有各自的属性，尽管很相似。

```php
interface Shape
{
    public function getArea(): int;
}

class Rectangle implements Shape
{
    private $width = 0;
    private $height = 0;

    public function __construct(int $width, int $height)
    {
        $this->width = $width;
        $this->height = $height;
    }

    public function getArea(): int
    {
        return $this->width * $this->height;
    }
}

class Square implements Shape
{
    private $length = 0;

    public function __construct(int $length)
    {
        $this->length = $length;
    }

    public function getArea(): int
    {
        return $this->length ** 2;
    }
}

function printArea(Shape $shape): void
{
    echo sprintf('%s has area %d.', get_class($shape), $shape->getArea()).PHP_EOL;
}

$shapes = [new Rectangle(4, 5), new Square(5)];

foreach ($shapes as $shape) {
    printArea($shape);
}
```

**[⬆ 返回顶部](#目录)**

### Interface Segregation Principle (ISP)

ISP states that "Clients should not be forced to depend upon interfaces that
they do not use."

A good example to look at that demonstrates this principle is for
classes that require large settings objects. Not requiring clients to set up
huge amounts of options is beneficial, because most of the time they won't need
all of the settings. Making them optional helps prevent having a "fat interface".

**反例：**

```php
interface Employee
{
    public function work(): void;

    public function eat(): void;
}

class HumanEmployee implements Employee
{
    public function work(): void
    {
        // ....working
    }

    public function eat(): void
    {
        // ...... eating in lunch break
    }
}

class RobotEmployee implements Employee
{
    public function work(): void
    {
        //.... working much more
    }

    public function eat(): void
    {
        //.... robot can't eat, but it must implement this method
    }
}
```

**正例：**

Not every worker is an employee, but every employee is a worker.

```php
interface Workable
{
    public function work(): void;
}

interface Feedable
{
    public function eat(): void;
}

interface Employee extends Feedable, Workable
{
}

class HumanEmployee implements Employee
{
    public function work(): void
    {
        // ....working
    }

    public function eat(): void
    {
        //.... eating in lunch break
    }
}

// robot can only work
class RobotEmployee implements Workable
{
    public function work(): void
    {
        // ....working
    }
}
```

**[⬆ 返回顶部](#目录)**

### Dependency Inversion Principle (DIP)

This principle states two essential things:
1. High-level modules should not depend on low-level modules. Both should
depend on abstractions.
2. Abstractions should not depend upon details. Details should depend on
abstractions.

This can be hard to understand at first, but if you've worked with PHP frameworks (like Symfony), you've seen an implementation of this principle in the form of Dependency
Injection (DI). While they are not identical concepts, DIP keeps high-level
modules from knowing the details of its low-level modules and setting them up.
It can accomplish this through DI. A huge benefit of this is that it reduces
the coupling between modules. Coupling is a very bad development pattern because
it makes your code hard to refactor.

**反例：**

```php
class Employee
{
    public function work(): void
    {
        // ....working
    }
}

class Robot extends Employee
{
    public function work(): void
    {
        //.... working much more
    }
}

class Manager
{
    private $employee;

    public function __construct(Employee $employee)
    {
        $this->employee = $employee;
    }

    public function manage(): void
    {
        $this->employee->work();
    }
}
```

**正例：**

```php
interface Employee
{
    public function work(): void;
}

class Human implements Employee
{
    public function work(): void
    {
        // ....working
    }
}

class Robot implements Employee
{
    public function work(): void
    {
        //.... working much more
    }
}

class Manager
{
    private $employee;

    public function __construct(Employee $employee)
    {
        $this->employee = $employee;
    }

    public function manage(): void
    {
        $this->employee->work();
    }
}
```

**[⬆ 返回顶部](#目录)**

## Don’t repeat yourself (DRY)

Try to observe the [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself) principle.

Do your absolute best to avoid duplicate code. Duplicate code is bad because
it means that there's more than one place to alter something if you need to
change some logic.

Imagine if you run a restaurant and you keep track of your inventory: all your
tomatoes, onions, garlic, spices, etc. If you have multiple lists that
you keep this on, then all have to be updated when you serve a dish with
tomatoes in them. If you only have one list, there's only one place to update!

Often you have duplicate code because you have two or more slightly
different things, that share a lot in common, but their differences force you
to have two or more separate functions that do much of the same things. Removing
duplicate code means creating an abstraction that can handle this set of different
things with just one function/module/class.

Getting the abstraction right is critical, that's why you should follow the
SOLID principles laid out in the [Classes](#classes) section. Bad abstractions can be
worse than duplicate code, so be careful! Having said this, if you can make
a good abstraction, do it! Don't repeat yourself, otherwise you'll find yourself
updating multiple places any time you want to change one thing.

**反例：**

```php
function showDeveloperList(array $developers): void
{
    foreach ($developers as $developer) {
        $expectedSalary = $developer->calculateExpectedSalary();
        $experience = $developer->getExperience();
        $githubLink = $developer->getGithubLink();
        $data = [
            $expectedSalary,
            $experience,
            $githubLink
        ];

        render($data);
    }
}

function showManagerList(array $managers): void
{
    foreach ($managers as $manager) {
        $expectedSalary = $manager->calculateExpectedSalary();
        $experience = $manager->getExperience();
        $githubLink = $manager->getGithubLink();
        $data = [
            $expectedSalary,
            $experience,
            $githubLink
        ];

        render($data);
    }
}
```

**正例：**

```php
function showList(array $employees): void
{
    foreach ($employees as $employee) {
        $expectedSalary = $employee->calculateExpectedSalary();
        $experience = $employee->getExperience();
        $githubLink = $employee->getGithubLink();
        $data = [
            $expectedSalary,
            $experience,
            $githubLink
        ];

        render($data);
    }
}
```

**Very good:**

It is better to use a compact version of the code.

```php
function showList(array $employees): void
{
    foreach ($employees as $employee) {
        render([
            $employee->calculateExpectedSalary(),
            $employee->getExperience(),
            $employee->getGithubLink()
        ]);
    }
}
```

**[⬆ 返回顶部](#目录)**

## Translations

This is also available in other languages:

* :cn: **Chinese:**
   * [php-cpm/clean-code-php](https://github.com/php-cpm/clean-code-php)
* :ru: **Russian:**
   * [peter-gribanov/clean-code-php](https://github.com/peter-gribanov/clean-code-php)
* :es: **Spanish:**
   * [fikoborquez/clean-code-php](https://github.com/fikoborquez/clean-code-php)
* :brazil: **Portuguese:**
   * [fabioars/clean-code-php](https://github.com/fabioars/clean-code-php)
   * [jeanjar/clean-code-php](https://github.com/jeanjar/clean-code-php/tree/pt-br)
* :thailand: **Thai:**
   * [panuwizzle/clean-code-php](https://github.com/panuwizzle/clean-code-php)
* :fr: **French:**
   * [errorname/clean-code-php](https://github.com/errorname/clean-code-php)
* :vietnam: **Vietnamese**
   * [viethuongdev/clean-code-php](https://github.com/viethuongdev/clean-code-php)
* :kr: **Korean:**
   * [yujineeee/clean-code-php](https://github.com/yujineeee/clean-code-php)
* :tr: **Turkish:**
   * [anilozmen/clean-code-php](https://github.com/anilozmen/clean-code-php)

**[⬆ 返回顶部](#目录)**
