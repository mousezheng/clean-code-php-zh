
# SOLID

**SQLID** 是 Michael Feathers 对 Robert Martin 提出的面向对象五大基本设计原则命名的缩写。

* [S: 单一职责原则 Single Responsibility Principle (SRP)](#单一职责原则)
* [O: 开闭原则 Open/Closed Principle (OCP)](#开闭原则)
* [L: 里氏代换原则 Liskov Substitution Principle (LSP)](#里氏代换原则)
* [I: 接口隔离原则 Interface Segregation Principle (ISP)](#接口隔离原则)
* [D: 依赖倒转原则 Dependency Inversion Principle (DIP)](#依赖倒转原则)

## 单一职责原则

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

## 开闭原则

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

## 里氏代换原则

这个看似复杂的术语是非常简单的。它正式的定义了作为 ”如果 S 是 T 的父类（或者父接口或者抽象），
然后对象的类型是 T 可以被类型 S 的对象取代，而不修改项目中任何可取代属性（正确性，执行的任务等等）
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

## 接口隔离原则

接口隔离原则是客户端不需要被迫依赖用不到的接口。

一个很好的例子证明这个原理，需要设置很大的对象时，最好不让客户端设置大量选项，因为大多数情况下，客户端不需要所有的设置。将其设置为可选，有利于防止”胖接口“

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

## 依赖倒转原则

这个原则有两件重要的事：

1. ”高级模块“ 不应该依赖 ”低级模块“。都应该依赖于抽象。
2. 抽象不应该依赖细节，细节应该依赖于抽象。

一开始有点难以理解，如果使用 PHP 框架（例如 Symfony）,你已经看过这个原理的实现-依赖注入（DI）。
虽然他们概念不一致，但是依赖倒转保证 ”高级模块“ 不了解 "低级模块" 的细节并设置他们。他们可以通过依赖注入完成
代码耦合非常不利于重构是一个坏的开发模式。

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
