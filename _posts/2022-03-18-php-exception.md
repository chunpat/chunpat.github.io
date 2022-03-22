---
layout: post 
title: PHP 异常处理 
date: 2022-03-18 
tags: exception
---

## PHP 错误处理

PHP因为便捷灵活，入门门槛低，严谨包容性大，如果使用不当，就容易写出很多难以入目与维护的代码。至此，如果想项目长久发展。
配置好相应的设置，加强把控至关重要。

## PHP 的错误级别种类

摘取了宝塔管理软件下的php.ini 配置下关于错误处理与记录的描述。

```
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
; Error handling and logging ;
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;

; This directive informs PHP of which errors, warnings and notices you would like
; it to take action for. The recommended way of setting values for this
; directive is through the use of the error level constants and bitwise
; operators. The error level constants are below here for convenience as well as
; some common settings and their meanings.
; By default, PHP is set to take action on all errors, notices and warnings EXCEPT
; those related to E_NOTICE and E_STRICT, which together cover best practices and
; recommended coding standards in PHP. For performance reasons, this is the
; recommend error reporting setting. Your production server shouldn't be wasting
; resources complaining about best practices and coding standards. That's what
; development servers and development settings are for.
; Note: The php.ini-development file has this setting as E_ALL. This
; means it pretty much reports everything which is exactly what you want during
; development and early testing.
;
; Error Level Constants:
; E_ALL             - All errors and warnings (includes E_STRICT as of PHP 5.4.0)
; E_ERROR           - fatal run-time errors
; E_RECOVERABLE_ERROR  - almost fatal run-time errors
; E_WARNING         - run-time warnings (non-fatal errors)
; E_PARSE           - compile-time parse errors
; E_NOTICE          - run-time notices (these are warnings which often result
;                     from a bug in your code, but it's possible that it was
;                     intentional (e.g., using an uninitialized variable and
;                     relying on the fact it is automatically initialized to an
;                     empty string)
; E_STRICT          - run-time notices, enable to have PHP suggest changes
;                     to your code which will ensure the best interoperability
;                     and forward compatibility of your code
; E_CORE_ERROR      - fatal errors that occur during PHP's initial startup
; E_CORE_WARNING    - warnings (non-fatal errors) that occur during PHP's
;                     initial startup
; E_COMPILE_ERROR   - fatal compile-time errors
; E_COMPILE_WARNING - compile-time warnings (non-fatal errors)
; E_USER_ERROR      - user-generated error message
; E_USER_WARNING    - user-generated warning message
; E_USER_NOTICE     - user-generated notice message
; E_DEPRECATED      - warn about code that will not work in future versions
;                     of PHP
; E_USER_DEPRECATED - user-generated deprecation warnings
;
; Common Values:
;   E_ALL (Show all errors, warnings and notices including coding standards.)
;   E_ALL & ~E_NOTICE  (Show all errors, except for notices)
;   E_ALL & ~E_NOTICE & ~E_STRICT  (Show all errors, except for notices and coding standards warnings.)
;   E_COMPILE_ERROR|E_RECOVERABLE_ERROR|E_ERROR|E_CORE_ERROR  (Show only errors)
; Default Value: E_ALL & ~E_NOTICE & ~E_STRICT & ~E_DEPRECATED
; Development Value: E_ALL
; Production Value: E_ALL & ~E_DEPRECATED & ~E_STRICT
; http://php.net/error-reporting
error_reporting = E_ALL & ~E_NOTICE

; This directive controls whether or not and where PHP will output errors,
; notices and warnings too. Error output is very useful during development, but
; it could be very dangerous in production environments. Depending on the code
; which is triggering the error, sensitive information could potentially leak
; out of your application such as database usernames and passwords or worse.
; For production environments, we recommend logging errors rather than
; sending them to STDOUT.
; Possible Values:
;   Off = Do not display any errors
;   stderr = Display errors to STDERR (affects only CGI/CLI binaries!)
;   On or stdout = Display errors to STDOUT
; Default Value: On
; Development Value: On
; Production Value: Off
; http://php.net/display-errors
display_errors = On

; The display of errors which occur during PHP's startup sequence are handled
; separately from display_errors. PHP's default behavior is to suppress those
; errors from clients. Turning the display of startup errors on can be useful in
; debugging configuration problems. We strongly recommend you
; set this to 'off' for production servers.
; Default Value: Off
; Development Value: On
; Production Value: Off
; http://php.net/display-startup-errors
display_startup_errors = Off

; Besides displaying errors, PHP can also log errors to locations such as a
; server-specific log, STDERR, or a location specified by the error_log
; directive found below. While errors should not be displayed on productions
; servers they should still be monitored and logging is a great way to do that.
; Default Value: Off
; Development Value: On
; Production Value: On
; http://php.net/log-errors
log_errors = On

; Set maximum length of log_errors. In error_log information about the source is
; added. The default is 1024 and 0 allows to not apply any maximum length at all.
; http://php.net/log-errors-max-len
log_errors_max_len = 1024

; Do not log repeated messages. Repeated errors must occur in same file on same
; line unless ignore_repeated_source is set true.
; http://php.net/ignore-repeated-errors
ignore_repeated_errors = Off

; Ignore source of message when ignoring repeated messages. When this setting
; is On you will not log errors with repeated messages from different files or
; source lines.
; http://php.net/ignore-repeated-source
ignore_repeated_source = Off

; If this parameter is set to Off, then memory leaks will not be shown (on
; stdout or in the log). This has only effect in a debug compile, and if
; error reporting includes E_WARNING in the allowed list
; http://php.net/report-memleaks
report_memleaks = On

; This setting is on by default.
;report_zend_debug = 0

; Store the last error/warning message in $php_errormsg (boolean). Setting this value
; to On can assist in debugging and is appropriate for development servers. It should
; however be disabled on production servers.
; This directive is DEPRECATED.
; Default Value: Off
; Development Value: Off
; Production Value: Off
; http://php.net/track-errors
;track_errors = Off

; Turn off normal error reporting and emit XML-RPC error XML
; http://php.net/xmlrpc-errors
;xmlrpc_errors = 0

; An XML-RPC faultCode
;xmlrpc_error_number = 0

; When PHP displays or logs an error, it has the capability of formatting the
; error message as HTML for easier reading. This directive controls whether
; the error message is formatted as HTML or not.
; Note: This directive is hardcoded to Off for the CLI SAPI
; Default Value: On
; Development Value: On
; Production value: On
; http://php.net/html-errors
html_errors = On

; If html_errors is set to On *and* docref_root is not empty, then PHP
; produces clickable error messages that direct to a page describing the error
; or function causing the error in detail.
; You can download a copy of the PHP manual from http://php.net/docs
; and change docref_root to the base URL of your local copy including the
; leading '/'. You must also specify the file extension being used including
; the dot. PHP's default behavior is to leave these settings empty, in which
; case no links to documentation are generated.
; Note: Never use this feature for production boxes.
; http://php.net/docref-root
; Examples
;docref_root = "/phpmanual/"

; http://php.net/docref-ext
;docref_ext = .html

; String to output before an error message. PHP's default behavior is to leave
; this setting blank.
; http://php.net/error-prepend-string
; Example:
;error_prepend_string = "<span style='color: #ff0000'>"

; String to output after an error message. PHP's default behavior is to leave
; this setting blank.
; http://php.net/error-append-string
; Example:
;error_append_string = "</span>"

; Log errors to specified file. PHP's default behavior is to leave this value
; empty.
; http://php.net/error-log
; Example:
error_log = /tmp/php_errors.log
; Log errors to syslog (Event Log on Windows).
;error_log = syslog

;windows.show_crt_warning
; Default value: 0
; Development value: 0
; Production value: 0

```

下面是官方中文版的解释，链接：[https://www.php.net/manual/zh/errorfunc.constants.php](https://www.php.net/manual/zh/errorfunc.constants.php)

| 值    | 常量    | 说明    | 备注| 
|  ----  | ----  |  ----  | ----  |
|1    |E_ERROR (int)    |致命的运行时错误。这类错误一般是不可恢复的情况，例如内存分配导致的问题。后果是导致脚本终止不再继续运行。    |  | 
|2    |E_WARNING (int)    |运行时警告 (非致命错误)。仅给出提示信息，但是脚本不会终止运行。     |  | 
|4    |E_PARSE (int)|    编译时语法解析错误。解析错误仅仅由分析器产生。     |  | 
|8    |E_NOTICE (int)    |运行时通知。表示脚本遇到可能会表现为错误的情况，但是在可以正常运行的脚本里面也可能会有类似的通知。     |  | 
|16    |E_CORE_ERROR (int)    |在 PHP 初始化启动过程中发生的致命错误。该错误类似 E_ERROR，但是是由 PHP 引擎核心产生的。     |  | 
|32    |E_CORE_WARNING (int)|    PHP 初始化启动过程中发生的警告 (非致命错误) 。类似 E_WARNING，但是是由 PHP 引擎核心产生的。     |  | 
|64|    E_COMPILE_ERROR (int)|    致命编译时错误。类似 E_ERROR，但是是由 Zend 脚本引擎产生的。     |  | 
|128    |E_COMPILE_WARNING (int)|    编译时警告 (非致命错误)。类似 E_WARNING，但是是由 Zend 脚本引擎产生的。     |  | 
|256    |E_USER_ERROR (int)|    用户产生的错误信息。类似 E_ERROR，但是是由用户自己在代码中使用 PHP 函数 trigger_error()来产生的。     |  | 
|512    |E_USER_WARNING (int)|    用户产生的警告信息。类似 E_WARNING，但是是由用户自己在代码中使用 PHP 函数 trigger_error()来产生的。    |  |  
|1024|    E_USER_NOTICE (int)    |用户产生的通知信息。类似 E_NOTICE，但是是由用户自己在代码中使用 PHP 函数 trigger_error()来产生的。     |  | 
|2048|    E_STRICT (int)    |启用 PHP 对代码的修改建议，以确保代码具有最佳的互操作性和向前兼容性。 |    PHP 5.4.0 之前的版本中不包含 E_ALL  | 
|4096    |E_RECOVERABLE_ERROR |(int)    可被捕捉的致命错误。 它表示发生了一个可能非常危险的错误，但是还没有导致PHP引擎处于不稳定的状态。 如果该错误没有被用户自定义句柄捕获 (参见 set_error_handler())，将成为一个 E_ERROR　从而脚本会终止运行。|    自 PHP 5.2.0 起 | 
|8192    |E_DEPRECATED (int)|    运行时通知。启用后将会对在未来版本中可能无法正常工作的代码给出警告。    |自 PHP 5.3.0 起|
|16384|    E_USER_DEPRECATED |(int)    用户产生的警告信息。 类似 E_DEPRECATED, 但是是由用户自己在代码中使用PHP函数 trigger_error()来产生的。|    自 PHP 5.3.0 起|
|32767|    E_ALL (int)|    PHP 5.4.0 之前为 E_STRICT 除外的所有错误和警告信息。 PHP 5.4.x 中为 32767, PHP 5.3.x 中为 30719, PHP 5.2.x 中为 6143, 更早之前的 PHP 版本中为 2047。|

我截取配置，把注释去掉，并在对应下方加上自己的理解，来源：https://www.php.net/manual/zh/errorfunc.configuration.php
```
error_reporting = E_ALL & ~E_NOTICE
//错误显示级别
display_errors = On
//错误显示开关
display_startup_errors = Off
//启动php显示错误，这个在开发环境是开启，生产关闭。
log_errors = On
//错误日志记录开关
log_errors_max_len = 1024
//不记录重复的信息。重复的错误必须出现在同一个文件中的同一行代码上，除非 ignore_repeated_source 设置为true。
ignore_repeated_errors = Off
//忽略重复消息时，也忽略消息的来源。当该设置开启时，重复信息将不会记录它是由不同的文件还是不同的源代码行产生的。
ignore_repeated_source = Off
忽略重复消息时，也忽略消息的来源。当该设置开启时，重复信息将不会记录它是由不同的文件还是不同的源代码行产生的。
report_memleaks = On
如果这个参数设置为Off，则内存泄露信息不会显示 (在 stdout 或者日志中)。
html_errors = On
在错误信息中关闭HTML标签。这种新的HTML格式的错误信息是可以点击，它引导用户前往描述该错误或者导致该错误发生的函数的参考信息页面。 
error_log = /tmp/php_errors.log
设置脚本错误将被记录到的文件。该文件必须是web服务器用户可写的。
```

## 原生代码演示

这里演示日常开发常常遇到的，分别是
* 1、E_NOTICE (int)
* 2、E_PARSE (int)
* 3、E_STRICT (int)
* 4、E_DEPRECATED (int)
* 5、E_WARNING (int)
* 6、E_ERROR (int)

首先默认配置如下，error_reporting = E_ALL & ~E_NOTICE，表示所有错误都显示出来，除了E_NOTICE，这里为了测试
E_NOTICE是什么错误，我配置修改为如下：
```
error_reporting = E_ALL
//错误显示级别
display_errors = On
//错误显示开关
```

针对上面日常遇到的错误，进行代码实操：

> E_NOTICE 

实例代码
```
<?php
echo $aa;

// Notice: Undefined variable: aa in /Volumes/work/www/onehour/onehour_docker/www/sima2/test.php on line 3
```

> E_WARNING

实例代码
```
<?php
echo aa;

// Warning: Use of undefined constant aa - assumed 'aa' (this will throw an Error in a future version of PHP) in /Volumes/work/www/onehour/onehour_docker/www/sima2/test.php on line 2
string(1) "a"

<?php
$person->name = 'Rasmus Lerdorf';
//Warning: Creating default object from empty value in /Volumes/work/www/onehour/onehour_docker/www/sima2/test.php on line 2
```

> E_PARSE (int)

实例代码
```
z = 1;
// Parse error: syntax error, unexpected '=' in /Volumes/work/www/onehour/onehour_docker/www/sima2/test.php on line 2
```

> E_DEPRECATED (int)

实例代码
```
<?php
$size = mcrypt_get_block_size(MCRYPT_RIJNDAEL_128, MCRYPT_MODE_ECB);

// Deprecated: Function mcrypt_get_block_size() is deprecated in /Volumes/work/www/onehour/onehour_docker/www/sima2/test.php on line 2
```
> E_STRICT (int)

**抱歉，PHP7.0以后就没有这个分类了，合并到其他分类去了** ，来源链接: [https://wiki.php.net/rfc/reclassify_e_strict](https://wiki.php.net/rfc/reclassify_e_strict)

下面演示的是旧版本，PHP7.0前版本。
```
<?php
class Person {
    var $name;

    function __construct($name) {
        $this->name = $name;
    }

    function Person($name) {
        $this->name = $name;
    }
}

//PHP Strict Standards:  var: Deprecated. Please use the public/private/protected modifiers
                  PHP Strict Standards:  Redefining already defined constructor for class Person
```

> E_ERROR (int)

实例代码
```
<?php
$aa = null;
var_dump( $aa->toArray());  //这操作在ORM 对象操作经常遇到
// Fatal error: Uncaught Error: Call to a member function toArray() on null in /Volumes/work/www/onehour/onehour_docker/www/sima2/test.php:3 Stack trace: #0 {main} thrown in /Volumes/work/www/onehour/onehour_docker/www/sima2/test.php on line 3

class Person {
    var $name;

    function __construct($name) {
        $this->name = $name;
    }

    function Person($name) {
        $this->name = $name;
    }
}

Person::Person();

// Fatal error: Uncaught Error: Non-static method Person::Person() cannot be called statically in /Volumes/work/www/onehour/onehour_docker/www/sima2/test.php:14 Stack trace: #0 {main} thrown in /Volumes/work/www/onehour/onehour_docker/www/sima2/test.php on line 14

function test($test){
}
test();

//Fatal error: Uncaught ArgumentCountError: Too few arguments to function test(), 0 passed in /Volumes/work/www/onehour/onehour_docker/www/sima2/test.php on line 6 and exactly 1 expected in /Volumes/work/www/onehour/onehour_docker/www/sima2/test.php:2 Stack trace: #0 /Volumes/work/www/onehour/onehour_docker/www/sima2/test.php(6): test() #1 {main} thrown in /Volumes/work/www/onehour/onehour_docker/www/sima2/test.php on line 2

throw new \Exception('aaa');
//Fatal error: Uncaught Exception: aaa in /Volumes/work/www/onehour/onehour_docker/www/sima2/test.php:2 Stack trace: #0 {main} thrown in /Volumes/work/www/onehour/onehour_docker/www/sima2/test.php on line 2
```

## 处理异常

日常开发中可以直接抛出错误，但是在生产环境中，有些警告类非中断的错误是不能影响正常业务流程的。如除了E_ERROR会中断。
那怎么办呢？可以使用set_error_handler处理并且记录异常，而不影响正常业务。 set_error_handler 不会捕抓 E_ERROR,
E_PARSE, E_CORE_ERROR, E_CORE_WARNING, E_COMPILE_ERROR, E_COMPILE_WARNING。下面是原文：

```
//https://www.php.net/manual/en/function.set-error-handler
The following error types cannot be handled with a user defined function: E_ERROR, E_PARSE,
E_CORE_ERROR, E_CORE_WARNING, E_COMPILE_ERROR, E_COMPILE_WARNING independent of where they
were raised, and most of E_STRICT raised in the file where set_error_handler() is called.
```

## ThinkPHP异常处理

## Laravel异常处理

## 迭代

* 2022年03月18日 23:59:59 初稿

## 参考

1、