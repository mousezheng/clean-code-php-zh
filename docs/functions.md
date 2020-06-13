
# 方法

## 方法参数少于两个

测试中当方法超过三个参数时，就需要穷举非常多的测试用例，限制方法参数对测试十分有利。没有参数是最理想的情况，一个或者两个参数也是可以的，避免超过三个。通常，如果方法超过两个参数，想办法将超出的参数组合起来。大多数时候一个 “高级” 的方法只会有一个参数。

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



## 方法见名知意

方法应当尽可能见明知意，告诉读者其主要用途。

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



## 只做一件事

尽可能避免一个方法中做太多操作，合理对方法进行拆分，可以更好的重用同时方便测试。

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



## 不使用标志作参数

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



## 避免副作用

“副作用”是指在一个方法中做其他不必要的事。例如写入一个文件、修改一些全局变量或不小心把钱汇给陌生人都是所谓的副作用。

要避免常见的陷阱，例如在对象之间共享状态，使用可被读写的全局变量。

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



## 不用全局函数

很多语言里**全局污染**是一个不好的情况，因为这样会与其他库产生冲突，并且使用这样的 API 是非常不明智的，很有可能在生产中产生一些意料之外的状况。思考一个例子，如果想做一个配置数组该干什么？编写一个全局的方法 `config()`,但是这个造成**全局污染**

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



## 不使用单例模式

单例是一个 [反面模式](https://baike.baidu.com/item/%E5%8F%8D%E9%9D%A2%E6%A8%A1%E5%BC%8F/3703771?fr=aladdin).下面是 **Brian Button** 的解释：

 1. 通常使用一个 **全局实例**是很糟糕的，因为在应用的代码中**隐藏依赖**，实例仅暴漏它们通过某些接口实现。
 2. 它们违背了 [单一职责原则](#单一职责原则)：通过**他们控制他们自己的创建和生命周期**。
 3. 它们会使得代码 [耦合性](https://baike.baidu.com/item/%E8%80%A6%E5%90%88%E6%80%A7) 更高。它们通过伪装使得**测试更加困难**
 4. 他们关心状态在应用程序的整个生命周期中。 **在需要结束的时候，需要通过一个命令**，这个对于测试来说是很不理想的。因为每个单元测试都应该互相独立。

这里也有许多很好的资料可做参考，作者 [Misko Hevery](http://misko.hevery.com/about/) 关于 [root of problem](http://misko.hevery.com/2008/08/25/root-cause-of-singletons/) .

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



## 封装条件

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



## 避免非条件

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



## 避免条件

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



## 避免类型检查(上)

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



## 避免类型检查(下)

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



## 删除无用代码

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



