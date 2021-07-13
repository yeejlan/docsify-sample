# 单元测试
### 为什么要做单元测试？
一句话，求安心。

合格的代码是什么样子的？容易读懂，容易改动。易读易维护。

单元测试会让代码易维护，举例来说：

* 代码重构

  把源代码从service/libs/GetRemoteMediaService.php移动到 libs/support

  把类名从GetRemoteMediaService变成GetRemoteFileService

  把命名空间从service\libs改成libs\support

  如果没有单元测试，代码重构之后需要手工测试，假设这个服务被3个地方调用，那么需要测试3次，假设有5个类文件需要做同样的变更，那么需要做3x5=15次测试。

* 调用他人的服务接口

  GetRemoteMediaService::fetch($url, $timeout, $options)

  没单元测试，$timeout单位是什么？秒/毫秒？$options有哪些选项？

  有单元测试，看对应的测试用例，单元测试相当于代码调用的example

* 改动关键代码

  PaymentGatewayService::antiFraud($creditCard, $ipAddr)  需要根据不同用户的ip调用不同的反欺诈API，比如北美ip调用A服务，北美以外调用B服务，现在需要新增俄罗斯ip调用C服务

  没单元测试，万一改动出错，合法的信用卡被拒绝，或者是非法的信用卡被通过，线上业务损失大量金钱。

  有单元测试，跑完测试就行。

* 紧急代码更改
  
  快下班了，突然接到紧急通知，客户要求线上的API新增一个输出项。
  
  没单元测试，改好了代码，因为客户着急，紧急上线，结果上线后发现输出的项有点问题。匆忙重复上述过程，很容易出错。
  
  有单元测试，对新增的输出项，添加对应的数据校验测试，跑单元测试，上线前就能发现问题。
### 单元测试要怎么做？
* 覆盖一个函数的所有路径
```php
function getGreeting($currTime = -1) {
    $t_now = time();
    if($currTime > 0) {
        $t_now = $currTime;
    }
    $t_6am = strtotime('6am');
    $t_12pm = strtotime('12pm');
    $t_6pm = strtotime('6pm');

    $greeting = 'Good morning';
    if($t_now>$t_12pm) {
        $greeting = 'Good afternoon';
    }
    if($t_now>$t_6pm) {
        $greeting = 'Good evening';
    }
    if($t_now<=$t_6am) {
        $greeting = 'Good evening';
    }
    return $greeting;
}
    
function getGreetingProvider() {
    return [
        [strtotime('6am'), 'Good evening'],
        [strtotime('7am'), 'Good morning'],
        [strtotime('12pm'), 'Good morning'],
        [strtotime('1pm'), 'Good afternoon'],
        [strtotime('6pm'), 'Good afternoon'],
        [strtotime('7pm'), 'Good evening'],
    ];
}
```
* 覆盖尽可能多的函数

* 注意

  单元测试不是银弹，哪怕100%覆盖，也无法保证整个系统的正确性。

* 缺点

  需要花费额外的时间来编写。养成用单元测试做代码自测的习惯，养成用单元测试编写调用demo的习惯。

* 优点

  一次编写，终身有效。

### 单元测试入门

* 安装 PHPUNIT

推荐全局安装，因为所有项目都要用
```code
$ wget https://phar.phpunit.de/phpunit-9.5.phar
$ chmod +x phpunit-9.5.phar
$ ./phpunit-9.5.phar --version
```

windows下创建 phpunit.bat
```code
@ECHO OFF
php "%~dp0phpunit-9.5.4.phar" %*
```
* 配置PHPUNIT

1.建立tests目录，一般建议跟src目录平级
2.在和tests平级的地方建立phpunit.xml.dist配置文件
3.当前目录运行phpunit

```xml
<?xml version="1.0" encoding="UTF-8"?>
<phpunit xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" backupGlobals="false" backupStaticAttributes="false" beStrictAboutTestsThatDoNotTestAnything="true" beStrictAboutOutputDuringTests="true" bootstrap="vendor/autoload.php" colors="true" convertErrorsToExceptions="true" convertNoticesToExceptions="false" convertWarningsToExceptions="true" failOnRisky="true" failOnWarning="true" processIsolation="false" stopOnError="false" stopOnFailure="false" verbose="true" xsi:noNamespaceSchemaLocation="https://schema.phpunit.de/9.3/phpunit.xsd">
  <coverage processUncoveredFiles="true">
    <include>
      <directory suffix=".php">./src</directory>
    </include>
  </coverage>
  <testsuites>
    <testsuite name="My PHP Test Suite">
      <directory suffix="Test.php">./tests</directory>
    </testsuite>
  </testsuites>
</phpunit>
```
注意 bootstrap="vendor/autoload.php"，testsuites

* 多看官方文档

https://phpunit.readthedocs.io/zh_CN/latest/

https://phpunit.readthedocs.io/en/9.5/

### 单元测试例子
* 基本例子
```php
use PHPUnit\Framework\TestCase;
class AddTest extends TestCase
{
    public function testAdd(int $a, int $b)
    {
        $a = 1;
        $b = 2;
        $actual = add($a, $b);
        $expected = 3;
        $this->assertSame($expected, $actual);
    }    
}
```
*  Data Providers
```php
class DataTest extends TestCase
{
    /**
     * @dataProvider additionProvider
     */
    public function testAdd(int $a, int $b, int $expected): void
    {
        $this->assertSame($expected, $a + $b);
    }

    public function additionProvider(): array
    {
        return [
            [0, 0, 0],
            [0, 1, 1],
            [1, 0, 1],
            [1, 1, 3]
        ];
    }
}
```
*  Testing Exceptions
```php
class ExceptionTest extends TestCase
{
    public function testException(): void
    {
        $this->expectException(InvalidArgumentException::class);
        throw new InvalidArgumentException('This is a test.');
    }
}
```
* Incomplete and Skipped Tests
```php
class SampleTest extends TestCase
{
    public function testSomething(): void
    {
        // Stop here and mark this test as incomplete.
        $this->markTestIncomplete(
          'This test has not been implemented yet.'
        );
		$this->markTestSkipped(
          'This test is skipped for some reason.'
         );        
    }
}
```
* Mock
```php
class MyMathLib
{
    public function __construct($handler) 
    {
        $this->handler = $handler;
    }
    public function doAdd($a, $b)
    {
        if ($a < 1e15 || $b < 1e15) { //should change || to &&
            return $a + $b;
        }
        return $this->handler->add($a, $b);
    }
}
class BigIntHandler
{
    public function add($a, $b)
    {
        $a = new Math_BigInteger($a);
        $b = new Math_BigInteger($b);
        return $a + $b;
    }
}
class SampleTest extends TestCase
{
    public function testBigIntAdd()
    {
        $a             = 1e10;
        $b             = 1e20;
        $morkBigIntAdd = $this->createMock(BigIntHandler::class);
        $morkBigIntAdd->expects($this->once())
            ->method('add')
            ->with(
                $this->equalTo($a),
                $this->equalTo($b),
            );
        (new MyMathLib($morkBigIntAdd))->doAdd($a, $b);
    }
}
```
* Stub
```php
class SampleTest extends TestCase
{
    public function testSomething()
    {
        $couponCode        = '123';
        $stubCouponService = $this->createStub(CouponService::class);
        $stubCouponService->method('verifyCouponCode')
            ->willReturn(true);

        $giftService = new GiftService($stubCouponService);
        $isSuccess   = $giftService->sendByCouponCode($couponCode);
        $this->assertTrue($isSuccess);
    }
}
class CouponService
{
    public function verifyCouponCode($couponCode)
    {
        if (sha1("someTokenCalculation") == $couponCode) {
            return true;
        }
        return false;
    }
}
class GiftService
{
    public function __construct($couponSercice)
    {
        $this->couponSerice = $couponSercice;
    }
    public function sendByCouponCode($couponCode)
    {
        if ($this->couponSerice->verifyCouponCode($couponCode)) {
            //gift sending process
            return true;
        }
        return false;
    }
}
```

* Fixtures

```php
class DatabaseTest extends TestCase
{
    private static $dbh;

    public static function setUpBeforeClass(): void   //setUp()
    {
        self::$dbh = new PDO('sqlite::memory:');
    }

    public static function tearDownAfterClass(): void //tearDown()
    {
        self::$dbh = null;
    }
}
```



* 项目代码参考

https://git.lingmou.ai:8000/lingmou-composer/ir-result/-/tree/first_add_0617

https://git.lingmou.ai:8000/rea/rea/-/tree/dev

rea\common\tests\unit\service\StorageServiceTest.php
rea\common\tests\unit\libs\storage\TencentCosStsTest.php

### codeception还是phpunit

* codeception
Yii自带功能，除了做单元测试还能做整体功能测试，验收测试，不过后两者一般由专门的QA人员编写。

https://codeception.com/docs/01-Introduction

https://github.com/Codeception/Codeception

缺点：虽然集成的是phpunit，但是无法对phpunit的所有配置参数进行控制，目前看来有2个待解决的问题：
1) 测试函数里面echo输出，不会输出到控制台下，不会报警
2) 测试函数里没写任何断言，不会报警

```php
class LoginFormTest extends \Codeception\Test\Unit
{

    public function testAlwaysTrue()
    {
        echo 123;
        //$this->assertSame(1, 1);
    }
}

//output: + LoginFormTest: Always true (0.00s)
```
* phpunit

php单元测试的既定事实标准。
https://phpunit.readthedocs.io/

有控制台输出会报警
```code
There was 1 risky test:

1) IrResult\Tests\LoginFormTest::testAlwaysTrue
This test printed output: 123

OK, but incomplete, skipped, or risky tests!
Tests: 27, Assertions: 29, Risky: 1.
```
测试函数没断言会报警
```code
There was 1 risky test:

1) IrResult\Tests\LoginFormTest::testAlwaysTrue
This test did not perform any assertions

D:\work\source\ir-result\tests\IrResult\LoginFormTest.php:10

OK, but incomplete, skipped, or risky tests!
Tests: 27, Assertions: 28, Risky: 1.
```



That's all. Thank you.
