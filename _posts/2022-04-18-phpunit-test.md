---
layout: post 
title: PHP 单元测试 与 Gitlab-CI
date: 2022-04-18 
tags: PHPUnit,Gitlab-Ci
---

## 简介

不做单元测试，你对你的代码有信心吗？

## 安装

> 1、composer 组件安装

> 2、composer.phar指令

## 官方案例

这里结合ThinkPHP6 做案例，PHPUnit 8 的版本。

> 安装组件包

```
//直接组件安装
composer require --dev phpunit/phpunit ^8

//查看版本
➜  tp6 git:(master) ✗ ./vendor/bin/phpunit --version
PHPUnit 8.5.26 #StandWithUkraine
```

> 修改composer.json的自动加载目录

```
    ...
    "autoload": {
        "psr-4": {
            "app\\": "app",
            "tests\\": "tests"
        },
        "psr-0": {
            "": "extend/"
        }
    },
    ...
```

> 案例代码

```
//
<?php

namespace tests\src;

use think\exception\InvalidArgumentException;

final class Email
{
    private $email;

    private function __construct(string $email)
    {
        $this->ensureIsValidEmail($email);

        $this->email = $email;
    }

    public static function fromString(string $email): self
    {
        return new self($email);
    }

    public function __toString(): string
    {
        return $this->email;
    }

    private function ensureIsValidEmail(string $email): void
    {
        if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
            throw new InvalidArgumentException(
                sprintf(
                    '"%s" is not a valid email address',
                    $email
                )
            );
        }
    }
}
```

> 测试案例代码

```
<?php

namespace tests;

use PHPUnit\Framework\TestCase;
use tests\src\Email;
use think\exception\InvalidArgumentException;

final class EmailTest extends TestCase
{
    public function testCanBeCreatedFromValidEmailAddress(): void
    {
        $this->assertInstanceOf(
            Email::class,
            Email::fromString('user@example.com')
        );
    }

    public function testCannotBeCreatedFromInvalidEmailAddress(): void
    {
        $this->expectException(InvalidArgumentException::class);

        Email::fromString('invalid');
    }

    public function testCanBeUsedAsString(): void
    {
        $this->assertEquals(
            'user@example.com',
            Email::fromString('user@example.com')
        );
    }
}
```

> 使用PHPSTORM的自带命令测试

phpstorm测试结果如下
```
/usr/local/Cellar/php/7.3.9_1/bin/php /Volumes/work/www/zzhpeng/tp6/vendor/phpunit/phpunit/phpunit --no-configuration --filter tests\\EmailTest --test-suffix EmailTest.php /Volumes/work/www/zzhpeng/tp6/tests --teamcity --cache-result-file=/Volumes/work/www/zzhpeng/tp6/.phpunit.result.cache
Testing started at 12:10 AM ...
PHPUnit 8.5.26 #StandWithUkraine



Time: 104 ms, Memory: 6.00 MB

OK (3 tests, 3 assertions)

Process finished with exit code 0
```

## 迭代

* 2022年04月18日 23:59:59 初稿

## 参考

1、