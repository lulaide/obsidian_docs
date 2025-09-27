---
source: https://xz.aliyun.com/news/18630
tags:
  - js逆向
---

这篇文章主要是给没有学习过js逆向的师傅学习的，分享一些js逆向基础知识，js实战断点调试技巧以及后面分享js逆向靶场搭建以及js逆向前端加密对抗，拿微信小程序常用的AES、RSA和明文Sign 签名校验绕过几个方面给师傅们分享下操作技巧。最后再分享一个企业SRC实战案例。

## 0x1 前言

哈咯，师傅们！最近在学习js逆向相关的知识点，跟着网上的师傅的课程已经很多相关文章探索学习，今天想着写一篇js逆向断点调试&js逆向前端加密对抗相关的文章出来，给师傅们分享下，有不正确的地方，希望大佬勿喷。

这篇文章主要是给没有学习过js逆向的师傅学习的，分享一些js逆向基础知识，js实战断点调试技巧以及后面分享js逆向靶场搭建以及js逆向前端加密对抗，拿微信小程序常用的AES、RSA和明文Sign 签名校验绕过几个方面给师傅们分享下操作技巧。

最后面给师傅们分享一个前段时间搞的一个企业src的商城优惠卷并发漏洞，也是拿到了一千块的赏金，漏洞都很详细的给师傅们分享了这个案例，师傅们看完我上面的js断点调试和js前端加解密靶场打法等，可以去尝试玩下，要是有地方写的有问题，大佬勿喷！

![](https://cdn.nlark.com/yuque/0/2025/png/55850010/1755231205435-c41b1d85-dcba-49c6-a208-6b392133b4dc.png "null")

## 0x2 如何找到加密算法

这里我直接拿Google浏览器控制面板来给师傅们演示下这个流程，主要是通过F12调试控制js前端代码

![](https://cdn.nlark.com/yuque/0/2025/png/55850010/1755231205580-b7e3afbd-077d-45ba-8318-9462f6b2fc30.png "null")

其中里面的作用域，调用堆栈，XHR断点这三个功能需要了解认识下

![](https://cdn.nlark.com/yuque/0/2025/png/55850010/1755231205647-dfd3b445-5329-4118-aa13-56b364cead14.png "null")

### 一、作用域（Scope）

作用域是指变量、函数和对象在代码中可访问的范围，决定了标识符（变量名、函数名）的可见性。

**主要类型：**

- **全局作用域** ：在所有函数和代码块之外声明的变量，在整个程序中都可访问。
```php
const globalVar = "我是全局变量";
function test() {
  console.log(globalVar); // 可访问
}
```
- **函数作用域** ：在函数内部声明的变量，仅在该函数内部可访问。
```php
function test() {
  const funcVar = "我是函数内变量";
  console.log(funcVar); // 可访问
}
console.log(funcVar); // 报错：未定义
```
- **块级作用域** ：由 `{}` 包裹的代码块（如 `if` 、 `for` 、 `while` ）中用 `let` / `const` 声明的变量，仅在块内可访问。
```php
if (true) {
  const blockVar = "我是块内变量";
  console.log(blockVar); // 可访问
}
console.log(blockVar); // 报错：未定义
```

**作用：** 避免变量名冲突，控制内存使用（离开作用域后变量可能被垃圾回收）。

### 二、调用堆栈（Call Stack）

调用堆栈是JavaScript引擎用于管理函数调用顺序的一种数据结构（遵循“后进先出”原则）。

**工作原理：**

- 当函数被调用时，引擎会为其创建一个“执行上下文”并压入栈顶。
- 函数执行完毕后，其执行上下文从栈顶弹出，控制权回到之前的函数。
- 栈顶始终是当前正在执行的函数。

**示例：**

```php
function a() {
  console.log("a开始");
  b();
  console.log("a结束");
}

function b() {
  console.log("b开始");
  c();
  console.log("b结束");
}

function c() {
  console.log("c执行");
}

a();
```

**调用堆栈过程：**

1. 执行 `a()` → `a` 入栈 → 栈： `[a]`
2. `a` 中调用 `b()` → `b` 入栈 → 栈： `[a, b]`
3. `b` 中调用 `c()` → `c` 入栈 → 栈： `[a, b, c]`
4. `c` 执行完毕 → `c` 出栈 → 栈： `[a, b]`
5. `b` 执行完毕 → `b` 出栈 → 栈： `[a]`
6. `a` 执行完毕 → `a` 出栈 → 栈空

**作用：** 追踪函数执行顺序，调试时可通过堆栈信息定位代码执行路径（如报错时的“堆栈跟踪”）。

### 三 、XHR断点（XHR Breakpoint）

XHR断点是浏览器开发者工具中的一种调试功能，用于在发送AJAX请求（XMLHttpRequest 或 Fetch）时暂停代码执行，方便调试网络请求相关逻辑。

**使用场景：**

- 调试接口请求参数是否正确
- 查看请求发送时机和触发条件
- 分析请求被拦截或修改的逻辑

### 四、 js基础断点调试

我们这里随便输入一个电话号码以及密码，直接看这个网络这里，可以看到账户输入的账户、密码都被进行了加密

其中我们常见的加密内容是md5、base64加密的，但是下面这个系统加密的一看就不是常见的加密方式

![](https://cdn.nlark.com/yuque/0/2025/png/55850010/1755231205708-58848d08-8ef4-4e24-94a5-4b04d7f1c792.png "null")

像这个，我们要是想要在我们输入账户密码的后，在传输到服务器端中可以将其加密的字段截取，然后进行分析，看看这个网站是使用什么类型进行加密的，就可以进行破解了，这就是后面我需要讲的js断点调试。

![](https://cdn.nlark.com/yuque/0/2025/png/55850010/1755231205785-2a1b2944-4dd3-4215-abcc-547b4432da0f.png "null")

我们输入账号密码-> 浏览器接受到我们的账号密码-> 根据js进行加密-> 发送到对方的服务器（网页）

服务器接收数据-> 对我们的数据进行解密-> 判断我们的数据解密后是否正常-> 正常就返回正常结果（否则返回报错）

## 0x3 实战断点调试-找加密算法

### 一、案例一

在讲这个案例断点调试之前，先带师傅们认识下这几个按钮工具作用

工具栏作为断点调试的操作工具，包含了 6 个按钮：

- 按钮 1：让代码继续执行，运行到下一个断点会中断执行，如果没有设置断点会直接运行完代码
- 按钮 2：跳过下一个函数调用。即不遇到函数时，执行下一步；遇到函数时，不进入函数直接执行下一步
- 按钮 3：跳进下一个函数上下文。即不遇到函数时，执行下一步；遇到函数时，进入函数上下文，查看函数具体内容
- 按钮 4：跳出当前函数调用
- 按钮 5：单步调试，当前断点的下一步
- 按钮 6：停用/激活全部断点

![](https://cdn.nlark.com/yuque/0/2025/png/55850010/1755231205854-9a2243eb-3026-48d5-9bb8-a4a349d2d43d.png "null")

这里直接搜索Vip/LoginResult接口关键字，因为我们断点调试的话需要点击web端端某个个功能点，出发我们后面打的断点才可以成功，这里我们开始点击登陆，直接请求的是这个接口，所有这里我们可以先搜索这个Vip/LoginResult关键词看看

![](https://cdn.nlark.com/yuque/0/2025/png/55850010/1755231205956-6df8b16f-4015-4512-a2c5-9e335c89becf.png "null")

这里直接选择最上面的top文件，右击选择在所有文件中搜索

![](https://cdn.nlark.com/yuque/0/2025/png/55850010/1755231206037-928d540d-c154-44e6-a898-9b64a1fd9b85.png "null")

![](https://cdn.nlark.com/yuque/0/2025/png/55850010/1755231206120-7bbf2607-966f-4bc4-aa4c-2df624eb9368.png "null")

找到这个有关登陆功能处的代码，然后打断点

![](https://cdn.nlark.com/yuque/0/2025/png/55850010/1755231206203-3bdd3a93-bac7-4763-8851-a242587995f2.png "null")

可以尝试多打几个断点，然后挨个阶段运行调试看看

![](https://cdn.nlark.com/yuque/0/2025/png/55850010/1755231206258-bcded4f6-478d-4b17-bb60-faafef632a81.png "null")

点击登陆按钮，就可以成功执行断点了，右边那个按钮就是执行到下一个断点

![](https://cdn.nlark.com/yuque/0/2025/png/55850010/1755231206341-1dac1f0d-6c4e-4e31-97ae-b4455326acce.png "null")

可以看到图中，代码断点运行成功了，在控制台输入logindata，就可以显示对应的加密数据了（手机号、密码）

![](https://cdn.nlark.com/yuque/0/2025/png/55850010/1755231206413-b02d3dd2-6d02-45a7-97e0-6baf08698c21.png "null")

### 二、案例二

老规矩还是直接输入手机号和密码，然后看F12中的网络数据包，可以看到这里的密码也是进行了加密

![](https://cdn.nlark.com/yuque/0/2025/png/55850010/1755231206486-9859ddfc-5507-4301-b1a8-e1ff06b0db29.png "null")

找到登陆口附近的js代码，寻找publickey关键字

![](https://cdn.nlark.com/yuque/0/2025/png/55850010/1755231206557-e9ce8747-8edc-41ea-a00a-0a96ba0d652b.png "null")

这里先随机设置一个断点

设置好断点后，点击登录，即可触发断点，进行 js 断点的调试。

![](https://cdn.nlark.com/yuque/0/2025/png/55850010/1755231206625-b681cca9-d87a-4a11-bf90-36ed7bd548fd.png "null")

可以看到点击完登陆，出现了这个功能

![](https://cdn.nlark.com/yuque/0/2025/png/55850010/1755231206690-5d109229-9005-4e0b-8ec6-e666b1a0fb91.png "null")

常见的js逆向加密搜索关键字：

```php
encrypt
encryptedData
setPublicKey
publicKey
```

直接进行js断点调试，在控制台输入dataJson.password，就可以看到密码的js加密数据

![](https://cdn.nlark.com/yuque/0/2025/png/55850010/1755231206767-35f20787-3da8-4f62-9d92-80f7aecf9c5e.png "null")

### 三、案例三

这里再给师傅们分享一个标签断点法，这里刚好有个网上实际的站点，拿出来给师傅们分享下

![](https://cdn.nlark.com/yuque/0/2025/png/55850010/1755231206840-d16b7f40-6c45-4b4e-b112-64b9f780c013.png "null")

线F12，然后点击1，然后把鼠标选中2（登陆功能上）

![](https://cdn.nlark.com/yuque/0/2025/png/55850010/1755231206922-5003d749-6974-47ca-9f62-9bd23957c628.png "null")

然后右键选中“登陆”标签，将子树修改，和属性修改都勾选上。然后我们随便输入一个账号密码点击登录

![](https://cdn.nlark.com/yuque/0/2025/png/55850010/1755231206991-f06f51e1-a996-42b5-b0f0-c4cee6dd1604.png "null")

这样就可以成功 把运行的js代码给断下来了，但是目前这个网站今天写文章的时候已经找不到加密函数了，可能是已经修复了这个系统

![](https://cdn.nlark.com/yuque/0/2025/png/55850010/1755231207081-b01cd669-c7c4-4bb6-98a3-9b4baec35d77.png "null")

## 0x4 js逆向靶场安装

前端加密对抗练习靶场——encrypt-labs下载链接： [https://github.com/SwagXz/encrypt-labs](https://github.com/SwagXz/encrypt-labs)

1、直接下载这个压缩包即可

![](https://cdn.nlark.com/yuque/0/2025/png/55850010/1755231207158-27fee3fd-2712-497e-9bb3-a6c08fb62ba4.png "null")

2、然后解压之后直接放在phpstudy的www网站目录下

![](https://cdn.nlark.com/yuque/0/2025/png/55850010/1755231207227-bef8c70a-4dfa-4957-858d-7f558fd7ead1.png "null")

3、这里还需要把这个文件下的数据库的用户名和密码都修改成phpstudy默认的root

要不然会提示连接数据库账号密码错误

![](https://cdn.nlark.com/yuque/0/2025/png/55850010/1755231207288-3de25a3f-f7ab-4302-b666-90ffdb873087.png "null")

4、还需要注意的一点就是php的版本需要大于7.3.4版本

要不然会有提示openssl等服务启动不了等报错

![](https://cdn.nlark.com/yuque/0/2025/png/55850010/1755231207365-09108b21-b7e6-4e6a-bd2e-41bbe0bafcff.png "null")

5、使用命令行创建encryptDB数据库

```php
mysql -u root -p

create database encryptDB
```

![](https://cdn.nlark.com/yuque/0/2025/png/55850010/1755231207448-14525649-4475-4a70-acbc-a1d07b3a8bf8.png "null")

6、导入sql数据

```php
mysql> use encryptdb;
Database changed
mysql> source C:\phpstudy_pro\WWW\encrypt-labs-main\encryptDB.sql
```

![](https://cdn.nlark.com/yuque/0/2025/png/55850010/1755231207540-a3a3f3a7-1308-41e0-b6f2-44566f33f927.png "null")

账号密码是：admin:123456，直接登陆即可完成靶场搭建了

![](https://cdn.nlark.com/yuque/0/2025/png/55850010/1755231207621-7989bf65-f36b-4213-bd50-2399fb578899.png "null")

## 0x5 解密工具下载

### 一、autoDecoder

工具下载链接： [https://github.com/f0ng/autoDecoder](https://github.com/f0ng/autoDecoder)

![](https://cdn.nlark.com/yuque/0/2025/png/55850010/1755231207692-475e81d0-fce3-4c5b-b525-36710e9c11b3.png "null")

下载jar文件那个，直接导入burpsuit插件即可使用

![](https://cdn.nlark.com/yuque/0/2025/png/55850010/1755231207755-03e03806-ce46-4471-b87c-c3e546c9c9da.png "null")

具体的使用和操作方法，师傅们可以看这个文章，写的很详细：

[https://blog.csdn.net/2202\_75361164/article/details/144360050](https://blog.csdn.net/2202_75361164/article/details/144360050)

### 二、Js-Forward脚本工具

Js-Forward是为了解决在渗透测试过程中所遇到的WEB参数加密而开发出的脚本工具

工具下载链接： [https://github.com/G-Security-Team/JS-Forward](https://github.com/G-Security-Team/JS-Forward)

![](https://cdn.nlark.com/yuque/0/2025/png/55850010/1755231207840-7c2c782e-d703-4e7c-ac7f-17ba91e4796a.png "null")

### 三、JsRpc工具

工具下载链接：

![](https://cdn.nlark.com/yuque/0/2025/png/55850010/1755231207926-f770a213-450d-4e43-8d16-b1443ee94142.png "null")

具体操作使用方法如下： [https://github.com/jxhczhl/JsRpc](https://github.com/jxhczhl/JsRpc)

![](https://cdn.nlark.com/yuque/0/2025/png/55850010/1755231208006-317fd8c9-cbb9-4a6d-9d73-844cd235a335.png "null")

### 四、ctool浏览器插件加解密

工具下载链接：

这里直接拿Google浏览器演示，下载下面的压缩包

![](https://cdn.nlark.com/yuque/0/2025/png/55850010/1755231208077-1e32ed75-5786-486c-a8c7-d5aa49adcd23.png "null")

然后解压，Google浏览器插件点击加载这个工具文件夹即可

![](https://cdn.nlark.com/yuque/0/2025/png/55850010/1755231208158-d4df7a98-b8e7-49b0-a084-d6be49f1b919.png "null")

![](https://cdn.nlark.com/yuque/0/2025/png/55850010/1755231208211-7637937c-53f5-440f-a624-a53b19f4d896.png "null")

## 0x6 对称加密-AES 加解密

### 一、AES 加解密简介

高级加密标准(AES,Advanced Encryption Standard)为最常见的对称加密算法（微信小程序）加密传输就是用这个加密算法的)。对称加密算法也就是加密和解密用相同的密钥，具体的加密流程如下图：

![](https://cdn.nlark.com/yuque/0/2025/png/55850010/1755231208284-261432df-3569-41e3-94db-f82ea2f837d8.png "null")

AES 加解密：常用的微信小程序加解密方法

常见的搜索关键词：

```php
encrypt、encryptedData、setpublickey
```

### 二、AES固定Key

第一种比较简单，key和iv写死，抓包发现数据传输被加密了

![](https://cdn.nlark.com/yuque/0/2025/png/55850010/1755231208340-2b429114-e3ba-42b3-a945-a8323f4b20c1.png "null")

这里直接定位搜索“encryptedData”加密字段定位到算法位置

![](https://cdn.nlark.com/yuque/0/2025/png/55850010/1755231208393-c9712813-4fb2-4b5b-9692-24669e6d9b64.png "null")

简单的分析，就是一个固定key 和iv 的aes加密，直接还原明文数据

```php
{"username":"admin","password":"123456"}
```

![](https://cdn.nlark.com/yuque/0/2025/png/55850010/1755231208471-4bb60afe-57e5-4222-9b2e-4cae37158af3.png "null")

### 三、AES服务端获取key

key和iv没有写在前端，直接使用bp抓包即可

![](https://cdn.nlark.com/yuque/0/2025/png/55850010/1755231208543-ec8694f2-5399-4055-b52f-d54a1c90ed71.png "null")

![](https://cdn.nlark.com/yuque/0/2025/png/55850010/1755231208603-dfcd2903-922b-4597-b9aa-75454fa3d8b2.png "null")

这里使用bp插件autoDecoder，先配置自带的方法，红框的配置要注意配置好

![](https://cdn.nlark.com/yuque/0/2025/png/55850010/1755231208657-dc84e5ce-a2dd-453f-8c96-749abb200171.png "null")

主页面配置，仅登录所以关键字写了password

![](https://cdn.nlark.com/yuque/0/2025/png/55850010/1755231208723-72f263a1-3e5d-4cb1-9afb-7e317ba774f7.png "null")

然后进入intruder进行爆破登录尝试

![](https://cdn.nlark.com/yuque/0/2025/png/55850010/1755231208788-69fedd80-5e6d-48ef-8bbd-59c6d01b6c7c.png "null")

成功爆破

![](https://cdn.nlark.com/yuque/0/2025/png/55850010/1755231208853-720f827a-cd5c-4737-94ac-f5b4325723e3.png "null")

## 0x7 非对称加密-RSA 加解密

### 一、如何快速判定 RSA 呢？

RSA 只能加密短小的数据，如果数据太大，会直接报错，因此可以入超长数值，看看是否报错！

![](https://cdn.nlark.com/yuque/0/2025/png/55850010/1755231208916-7c056649-c204-44a2-9daa-1b908e372ef4.png "null")

提示了这个错误，显示加密失败，说明就是非对称加密- RSA加密了

![](https://cdn.nlark.com/yuque/0/2025/png/55850010/1755231208992-db3c47d2-bb1b-4b7a-a82c-a819c8cfcedf.png "null")

对于非对称加密，他需要设置公钥，因此一般全局搜索：

```php
setpublickey、encrypt
```

### 二、RSA 加密

![](https://cdn.nlark.com/yuque/0/2025/png/55850010/1755231209082-17b24e2b-52cb-43dd-be31-b9969dd37370.png "null")

这个关卡不难，搜索非对称加密RSA常用的加密关键字：setpublickey，通过前端js找到公钥

![](https://cdn.nlark.com/yuque/0/2025/png/55850010/1755231209154-fe817726-4286-4618-9d69-51b0184160ba.png "null")

然后通过断点调试，进行找到没有加密的原始数据

![](https://cdn.nlark.com/yuque/0/2025/png/55850010/1755231209228-1fa54749-f7a0-43c6-9eb2-a52979421aef.png "null")

跟上面一样使用bp的插件autoDeceder

![](https://cdn.nlark.com/yuque/0/2025/png/55850010/1755231209336-e13af1e5-7b51-4cc5-bc6d-b69b1334ce40.png "null")

## 0x8 Sign 签名校验绕过

直接使用bp抓包，数据包的参数如下：username、password、nonce、timestamp、signature

![](https://cdn.nlark.com/yuque/0/2025/png/55850010/1755231209408-aa2b6b49-af78-4c46-abea-c9930a8d0bad.png "null")

我们这里来给师傅们演示下这个signature参数的由来

登陆界面，密码输入1234567，返回包提示密码错误

![](https://cdn.nlark.com/yuque/0/2025/png/55850010/1755231209468-fc0c432a-5204-4151-ae28-92c5e23571e9.png "null")

这个靶场的密码是123456，那么在bp数据包中，直接这里就把密码修改成123456，看看返回包

显示signature校验不正确，因为前面username、password、nonce、timestamp参数会生成signature传入到后台，后台就回和我们这里输入到signature进行匹配，要是不一样，就会进行报错

![](https://cdn.nlark.com/yuque/0/2025/png/55850010/1755231209574-806ca8f6-43ca-4ac1-8ed8-4b19757c8352.png "null")

搜索关键字：signature，去断点看js代码

```php
function sendDataWithNonce(url) {
  const username = document.getElementById("username")
    .value;
  const password = document.getElementById("password")
    .value;

  const nonce = Math.random()
    .toString(36)
    .substring(2);
  const timestamp = Math.floor(Date.now() / 1000);

  const secretKey = "be56e057f20f883e";

  const dataToSign = username + password + nonce + timestamp;
  const signature = CryptoJS.HmacSHA256(dataToSign, secretKey)
    .toString(CryptoJS.enc.Hex);
```

打完断点，点击明文加签，进行断点调试

![](https://cdn.nlark.com/yuque/0/2025/png/55850010/1755231209634-ef1c8c83-c4d5-43fd-bce3-38f239c1a3b2.png "null")

1. 首先将 `username` 、 `password` 、 `nonce` （随机数）和 `timestamp` （时间戳）拼接成一个字符串 `dataToSign` ，作为待签名的数据
```php
拼接：admin123456drzeh01d5sn1755059730
```
1. 使用 `CryptoJS` 库的 `HmacSHA256` 方法，用 `secretKey` （be56e057f20f883e）对拼接后的字符串进行 HMAC-SHA256 加密，生成 `signature` （签名）
```php
cd5e4af5343372cc5945539cddb67c8d33ae0f9be361c60e935e80b4c5a5f719
```

![](https://cdn.nlark.com/yuque/0/2025/png/55850010/1755231209696-90974581-fd5a-448d-9ca4-b3eca3c41d61.png "null")

1. 这样再替换回到数据包中即可成功登陆了

![](https://cdn.nlark.com/yuque/0/2025/png/55850010/1755231209759-7e139160-b4ef-497e-a906-c8954c4f2e11.png "null")

## 0x9 企业SRC实战分享—优惠卷

这个企业SRC是个商城的web资产应用，一进去，直接使用手机号登陆，因为像这样的可以直接使用手机号登陆（有账号），方便后续的渗透测试，很多功能点可能得登陆进去才可以测试

![](https://cdn.nlark.com/yuque/0/2025/png/55850010/1755231209818-089ebadb-ae23-4481-b9c1-1ecb7470e91b.png "null")

上面图片中，登陆到后台，有好几个功能点跟会员优惠卷有关系，商城优惠卷并发领取，必须得测下

![](https://cdn.nlark.com/yuque/0/2025/png/55850010/1755231209871-d2fa4d7a-9c28-436e-891f-ef8b835671c8.png "null")

点击领取，然后进行抓包测试，可以看到优惠卷的ID，为什么这里我要提下这个ID值呢，因为在后面我们还可以进行优惠卷横向并发，把不同金额的优惠卷ID进行便利并发领取，比如说你新人只能领取100元的，但是你通过ID横向并发领取到了200元的客户回归优惠卷，那么这个对商城是不是就有一定的危害了

![](https://cdn.nlark.com/yuque/0/2025/png/55850010/1755231209956-87a75315-6db7-42ab-8e07-77b72cfad4cf.png "null")

然后可以使用这个并发插件脚本进行爆破，数量可以设置20个即可

```php
def queueRequests(target, wordlists):
    engine = RequestEngine(endpoint=target.endpoint,
         concurrentConnections=20,
         requestsPerConnection=100,
         pipeline=False
               )
    for i in range(20):
  engine.queue(target.req, target.baseInput, gate='race1')
    engine.openGate('race1')
    engine.complete(timeout=60)

def handleResponse(req, interesting):
  table.add(req)
```

![](https://cdn.nlark.com/yuque/0/2025/png/55850010/1755231210026-15f13650-fca0-4670-9ba0-f340b7c28500.png "null")

可以看到数据包的状态200和长度都是一样的，那就说明我们成功进行了并发操作

![](https://cdn.nlark.com/yuque/0/2025/png/55850010/1755231210091-5bf4e41a-70c4-47af-9b6f-be5aded9920c.png "null")

然后返回账号的卡卷中心，发现领取了24张优惠卷

其中里面有20元、50元、100元优惠卷，并发操作都一样，都可以进行无限并发领取优惠卷

![](https://cdn.nlark.com/yuque/0/2025/png/55850010/1755231210190-d645c9cd-5a22-4ff9-b8c1-676273774761.png "null")

返回会员支付界面，可以看到支付的优惠卷都是可以直接使用叠加的，漏洞提交平台，一千块大洋到手

![](https://cdn.nlark.com/yuque/0/2025/png/55850010/1755231210249-821e20da-7966-416e-87ad-fa836c03d811.png "null")

## 0x10 总结

这篇文章到这里就结束了，希望看完这篇文章对师傅们有帮助，该文章主要是记录和分享我的学习过程和一些漏洞挖掘的案例和手法等，本人技术水平一般，有问题欢迎师傅们加我微信交流学习！

最后祝愿看了这篇文章的师傅们，天天有漏洞产出，天天有src赏金！