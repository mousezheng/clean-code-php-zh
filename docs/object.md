
# 对象和数据结构

## 封装对象

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



## private/protected

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


