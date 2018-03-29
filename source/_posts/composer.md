---
title: PHP依赖管理Composer
date: 2017-10-26 11:27:23
categories:
- PHP
tags:
- Composer
---

对于现代语言而言，包管理器基本上是标配。Java 有 Maven，Python 有 pip，Ruby 有 gem，Nodejs 有 npm。PHP 的则是 PEAR，不过 PEAR 坑不少：

- 依赖处理容易出问题
- 配置非常复杂
- 难用的命令行接口

好在我们有 Composer，PHP依赖管理的利器。它是开源的，使用起来也很简单，提交自己的包也很容易。

# 安装 Composer
Composer 需要 PHP 5.3.2+ 才能运行。
``` bash
$ curl -sS https://getcomposer.org/installer | php
```
这个命令会将 composer.phar 下载到当前目录。PHAR（PHP 压缩包）是一个压缩格式，可以在命令行下直接运行。

你可以使用 --install-dir 选项将 Composer 安装到指定的目录，例如：
``` bash
$ curl -sS https://getcomposer.org/installer | php -- --install-dir=bin
```
当然也可以进行全局安装：
``` bash
$ curl -sS https://getcomposer.org/installer | php
$ mv composer.phar /usr/local/bin/composer
```
在 Mac OS X 下也可以使用 homebrew 安装：
``` bash
$ brew tap josegonzalez/homebrew-php
$ brew install josegonzalez/php/composer
```
不过通常情况下只需将 composer.phar 的位置加入到 PATH 环境变量就可以，不一定要全局安装。

# 声明依赖
在项目目录下创建一个 composer.json 文件，指明依赖，比如，你的项目依赖 monolog：
``` js
{
    "require": {
        "monolog/monolog": "1.2.*"
    }
}
```
# 安装依赖

安装依赖非常简单，只需在项目目录下运行：
``` bash
composer install
```
如果没有全局安装的话，则运行：
``` bash
php composer.phar install
```
# 自动加载
Composer 提供了自动加载的特性，只需在你的代码的初始化部分中加入下面一行：
```php
require 'vendor/autoload.php';
```
# 模块仓库
packagist.org 是Composer的仓库，很多著名的 PHP 库都能在其中找到。你也可以提交你自己的作品。

原文地址：http://www.phpcomposer.com/composer-the-new-age-of-dependency-manager-for-php/