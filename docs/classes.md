
# 类

## 组合而不是继承

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

## 避免流式接口

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

## final

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
