
# 避免重复代码

查看相关 [DRY](https://verraes.net/2014/08/dry-is-about-knowledge/) 原则.

重复代码是不好的，尽可能避免重复代码，如果需要改变一些逻辑就需要改变很多地方。

通常重复的代码，因为两个或多个稍有不同的事物，它们有很多共同点，但是它们之间的差异迫使需要两个或多个独立的函数来执行许多相同的事情。
删除重复代码，创建一个可以执行这些不同的事仅仅通过一个方法、模型或者类。

确保正确的抽象是至关重要的，这就是为什么应该遵循 [类](＃类) 中列出部分 SOLID 原理的原因。
错误的抽象可能比重复的代码更糟糕，因此请小心！话虽如此，如果您可以做一个好的抽象，那就去做吧！
避免重复代码，否则会发现想更改一件事情时，需要更新多个位置

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

**正例:**

使用简洁的代码会更好。

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


