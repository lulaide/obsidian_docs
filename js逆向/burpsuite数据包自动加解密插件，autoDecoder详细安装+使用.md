---
source: "https://blog.csdn.net/2202_75361164/article/details/144360050"
tags:
  - "js逆向"
---
> 公众号：泷羽Sec-尘宇安全

文章首发于先知社区： [burpsuite数据包自动加解密插件，autoDecoder详细安装+使用](https://xz.aliyun.com/t/16564)

## 前言

在我们进行 [渗透测试](https://so.csdn.net/so/search?q=%E6%B8%97%E9%80%8F%E6%B5%8B%E8%AF%95&spm=1001.2101.3001.7020) 的过程中，往往会遇到数据包、响应包是加密的情况。如果选择放弃，就失去了一次可能渗透成功的机会。不过burpsuite里面有一个插件可以完美解决这个问题，就是autoDecoder，他可以实现内置的自动加解密，和自定义的加解密，已经给出框架，我们写加解密实现即可。autoDecoder使用门槛比其他一些的bp插件要高一些，我这里会详细讲解autoDecode的安装+使用

正常 流程图 和autoDecoder处理明文和密文的流程图对比

![image.png](https://i-blog.csdnimg.cn/img_convert/49ed56e5f556c3809bcc420be283031b.png)  
项目地址：  
https://github.com/f0ng/autoDecoder

## 安装

### 配置插件扩展环境

在扩展设置下面，配置插件环境。因为我选择的是java8的插件，所以设置java8的环境，其他java版本的类似  
![image.png](https://i-blog.csdnimg.cn/img_convert/a62c57451b737580f40f9f3a398f8f95.png)

![image.png](https://i-blog.csdnimg.cn/img_convert/8b1a1b2a61b1fdde9e403f8ab9035ba5.png)

### 加载插件

![](https://i-blog.csdnimg.cn/img_convert/b256bd08e5cbb18b641b579b1a5dfffc.png)

![](https://i-blog.csdnimg.cn/img_convert/9b594cf84af8adac40cd9d580083076b.png)

### 测试使用是否正常

首先安装flask框架

```
pip3 install flask
```

![image.png](https://i-blog.csdnimg.cn/img_convert/97551bca0a68feb05b12da89135c9a29.png)  
创建一个文件，内容为

```
# -*- coding:utf-8 -*-  
# author:f0ngf0ng  
  
from flask import Flask, Response, request  
from pyDes import *  
import base64  
  
  
def des_encrypt(s):  
    """  
    DES 加密    :param s: 原始字符串    :return: 加密后字符串，16进制  
    """    secret_key = "f0ngtest"  
    iv = "f0ngf0ng"  
    k = des(secret_key, CBC, iv, pad=None, padmode=PAD_PKCS5)  
    en = k.encrypt(s, padmode=PAD_PKCS5)  
    return base64.encodebytes(en).decode()  
  
  
def des_decrypt(s):  
    """  
    DES 解密    :param s: 加密后的字符串，16进制    :return:  解密后的字符串  
    """    secret_key = "f0ngtest"  
    iv = "f0ngf0ng"  
    k = des(secret_key, CBC, iv, pad=None, padmode=PAD_PKCS5)  
    de = k.decrypt(base64.decodebytes(bytes(s, encoding="utf-8")), padmode=PAD_PKCS5)  
    return de.decode()  
  
  
app = Flask(__name__)  
  
  
@app.route('/encode', methods=["POST"])  
def encrypt():  
    param = request.form.get('dataBody')  # 获取  post 参数  
    param_headers = request.form.get('dataHeaders')  # 获取  post 参数  
    param_requestorresponse = request.form.get('requestorresponse')  # 获取  post 参数  
    encry_param = des_encrypt(param.strip("\n"))  
    print(param)  
    print(encry_param)  
    if param_requestorresponse == "request":  
        return param_headers + "\r\n\r\n\r\n\r\n" + encry_param  
    return encry_param  
  
  
@app.route('/decode', methods=["POST"])  
def decrypt():  
    print(request.form)  
    param = request.form.get('dataBody')  # 获取  post 参数  
    param_headers = request.form.get('dataHeaders')  # 获取  post 参数  
    param_requestorresponse = request.form.get('requestorresponse')  # 获取  post 参数  
    decrypt_param = des_decrypt(param.strip("\n"))  
    print(decrypt_param)  
    print(param_headers)  
    print(param_requestorresponse)  
    if param_requestorresponse == "request":  
        return param_headers + "\r\n\r\n\r\n\r\n" + decrypt_param  
    else:  
        return decrypt_param  
  
  
if __name__ == '__main__':  
    app.debug = True  # 设置调试模式，生产模式的时候要关掉debug  
    app.run(host="0.0.0.0", port="8888")
```

然后，运行 app.py

```
python3 ./app.py
```

![image.png](https://i-blog.csdnimg.cn/img_convert/8bb77ff9cbcaca896ed9f9c62371930f.png)  
然后将测试的 [数据包](https://so.csdn.net/so/search?q=%E6%95%B0%E6%8D%AE%E5%8C%85&spm=1001.2101.3001.7020) 放入，点击解密和加密，发现解码、编码都成功，证明可以正常使用  
![image.png](https://i-blog.csdnimg.cn/img_convert/22242dbb2f9f0421bcc03ee6bb132f44.png)

![image.png](https://i-blog.csdnimg.cn/img_convert/e14c90241254816d14a1273752e9c3d2.png)

## 自带加解密算法使用

### 请求包解密，响应包不解密 和 全文本加密

算法为DES/CBC/PKCS5Padding算法，密钥为f0ngtest，iv为f0ngf0ng

请求体为

```
I9z1fsH5QQ2NUbJi/7a8lw==
```

#### 自带算法加解密模块设置

加解密如下，需要在响应包哪里设置null，表示为空，即不使用解密。如果不为空则就是全文本加密，这里演示请求包解密，响应包不解密，全文本加密就不重复演示了  
![image.png](https://i-blog.csdnimg.cn/img_convert/71502050c73e5a68ef0b933c53916c1b.png)  
注意，一定要保存配置不然是不会生效的， 文件名 和后缀都没有要求  
![image.png](https://i-blog.csdnimg.cn/img_convert/21cbfccc7024807fd776e52974dcdf8f.png)  
显示这样就行了  
![image.png](https://i-blog.csdnimg.cn/img_convert/5dae19bfa3db365f993446dfd0d0d07d.png)

#### 选项模块设置

同样的，选项这里，也需要设置：

- 加密选项为自带算法加解密
- 域名选择需要加解密的网站
- 设置明文关键字，如果检测出有该关键字，则认为是明文，这里选择"，因为前面解密出来是：{“id”:“1”}，明文中含有 " ，并且根据DES/CBC/PKCS5Padding算法可以知道密文中的字符是不会含有 " ，所以可以选择 "  
	![image.png](https://i-blog.csdnimg.cn/img_convert/4c65e7d7db74fd33ef083b3aea997c76.png)  
	同样要保存，不然不能使用，可以和之前的文件一样，也可以不一样  
	![image.png](https://i-blog.csdnimg.cn/img_convert/e0733e5bd173d0622c5bccfd7e45d257.png)

#### 自带加解密算法使用

当上面的配置保存之后，就可以在请求模块和响应模块这里看到多出来了autoDecoder和autoDecoder选项  
![image.png](https://i-blog.csdnimg.cn/img_convert/1f1b63eacd0dfe12318433f2fd17ea86.png)  
可以在autoDecoder这里查看到明文  
![image.png](https://i-blog.csdnimg.cn/img_convert/ec6a7b4def47a06e91270908380fe654.png)

### 指定文本加密+multipart提交方式进行加密

#### 应用场景

比如下面这个，他对passwd的输入进行了加密，导致我们不能直接进行爆破，这里我们就可以指定加密passwd的值  
例如： `123456` 加密成 `CWppt9RAuoY=`

```
{"username":"admin","passwd":"CWppt9RAuoY="}
```

![image.png](https://i-blog.csdnimg.cn/img_convert/1bc32329709a4d5531b9fb57f6de12c9.png)

#### 自带算法加解密模块设置

- 这里还是只加密请求包，所以响应包设置为null
- 注意，这里首先是DES，测试Ciphertext(密文)这里的内容能被成功解密，然后替换成null，点击：添加为响应包解密方式，这样响应包的解密方式就变成了null，当然也可以像上面一样进行手动更改
- 需要对请求数据包进行正则匹配：一般很简单，只需要将内容变成： `(.*?)` 即可。如这里将CWppt9RAuoY=替换为了 `(.*?)`
- 记得保存配置  
	![image.png](https://i-blog.csdnimg.cn/img_convert/ce6c48c156121aeb7e291092ce7de399.png)

#### 选项模块设置

- 因为没有改变什么，只是多进行了一步正则匹配，所以和之前的一样即可
- 记得保存配置  
	![image.png](https://i-blog.csdnimg.cn/img_convert/4c65e7d7db74fd33ef083b3aea997c76.png)

#### 效果

- 我们使用123456进行发包  
	![image.png](https://i-blog.csdnimg.cn/img_convert/52d84bea01df3fe9879ddd6306a9b9c5.png)
- 来到logger（日志）这里查看bp发送的包，可以看到，123456在发出去的时候已经被替换了  
	![image.png](https://i-blog.csdnimg.cn/img_convert/7ef74a57e354eaa6817a593af4e7ecb2.png)

### multipart提交方式进行加密

#### 应用场景

- 有些时候遇到waf，或者特殊的登录口，需要使用 `multipart/form-data` 方式进行请求，但是如果有加密存在，没法通过很快捷的方式进行加密，研究了一下autoDecoder，其实是可以用的
- 其实就是指定文本加密，和上面是差不多的
```
Content-Type:multipart/form-data; boundary=----WebKitFormBoundaryrGKCBY7qhFd3TrwA
Content-Length: 214

------WebKitFormBoundaryrGKCBY7qhFd3TrwA
Content-Disposition: form-data; name="file"; filename="shell.png"
Content-Type: image/png

DeEdN5u8mv26L8IMu34JGJe7lxQETOBV7YmVl9oxSyzmQXYKPyrMmA==
------WebKitFormBoundaryrGKCBY7qhFd3TrwA--
```

这个时候，我们想要更改数据包的内容进行waf绕过测试就很不方便，需要一次一次手动上传文件，这个时候，我们就可以正则匹配进行加密  
![image.png](https://i-blog.csdnimg.cn/img_convert/5f0ba7a26f8cbfa2d178ebed558318f5.png)

#### 使用

- 其他的和之前一样，只需要更改正则表达式，正则模式数据包中的换行请用 `\r\n` 替换，然后需要替换的部分一般变成： `([^\n]*)` 即可，比如这里是：
```
Content-Type: image/png\r\n\r\n([^\n]*)\r\n
1
```

![image.png](https://i-blog.csdnimg.cn/img_convert/1e66090d6f73aef5e058d82c199aa224.png)  
来到日志这里查看，可以看到，已经被成功替换  
![image.png](https://i-blog.csdnimg.cn/img_convert/7f564c630925b86220b60fe66f2dfe01.png)

## 接口加解密算法使用

- 在实际应用中，自带的加解密算法是不能所有的使用的，这个时候我们可以自行配置加解密算法
- 已经给出了框架，我们照样去写算法就行了

### 使响应包不解密

比如我们，我们上面的请求包解密，响应包不解密，也可以使用接口的方法

#### 框架代码运行+解释

- 这里以flask为例，算法为DES/CBC/PKCS5Padding算法  
	首先记得安装flask框架
```
pip3 install flask
```

然后将下面的代码运行起来： `python3 ./app.py`

- 首先des\_encrypt函数是实现算法的部分，需要自己根据场景编写
- 然后encrypt函数，就是encode路由，这里对应的是请求包解密
- decrypt函数为decode路由，这里对应的是返回包，因为不需要加解密，所以获取post参数之后不做任何处理直接返回即可
- 其他的地方，就不能随意更改，很可能出现不能使用或者直接乱码
```
# -*- coding:utf-8 -*-  
# author:f0ngf0ng  
  
from flask import Flask,Response,request  
import base64,hashlib,json  
from pyDes import *  
  
def des_encrypt(s):  
    """  
    DES 加密    :param s: 原始字符串    :return: 加密后字符串，16进制  
    """    secret_key = "f0ngtest"  
    iv = "f0ngf0ng"  
    k = des(secret_key, CBC, iv, pad=None, padmode=PAD_PKCS5)  
    en = k.encrypt(s, padmode=PAD_PKCS5)  
    return base64.encodebytes(en).decode()  
  
app = Flask(__name__)  
  
@app.route('/encode',methods=["POST"])  
def encrypt():  
    param = request.form.get('dataBody')  # 获取  post 参数  
    data = json.loads(param)  
    encry_param = param.replace( data["id"],des_encrypt(data["id"]).strip())  
    return  encry_param  
  
@app.route('/decode',methods=["POST"]) # 不解密  
def decrypt():  
    param = request.form.get('dataBody')  # 获取  post 参数  
    return param  
  
if __name__ == '__main__':  
    app.debug = True # 设置调试模式，生产模式的时候要关掉debug  
    app.run(host="0.0.0.0",port="8888")
```

![image.png](https://i-blog.csdnimg.cn/img_convert/7164276112d7c0c5fded8984dfe446a4.png)

#### 接口加解密配置+测试

使用数据包进行测试

```
POST / HTTP/1.1
Host: www.baidu.com
Cookie: BAIDUID_BFESS=861F7F1760A2D89507466557B33F2A5E:FG=1; BIDUPSID=861F7F1760A2D89507466557B33F2A5E; PSTM=1733190281; BD_UPN=12314753; BA_HECTOR=21040l010l05200h04ah2kah8bik5g1jksokb1v; ZFY=p5vLR5uKoCNz4utc6N8s9znUF2jQLO9G6XQbOBW54w4:C; H_PS_PSSID=60276_61027_61096_61212_61209_61215_61187_61283_61298_60851; BD_HOME=1
Cache-Control: max-age=0
Sec-Ch-Ua: "Not?A_Brand";v="99", "Chromium";v="130"
Sec-Ch-Ua-Mobile: ?0
Sec-Ch-Ua-Platform: "Windows"
Accept-Language: zh-CN,zh;q=0.9
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/130.0.6723.70 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: none
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Accept-Encoding: gzip, deflate, br
Priority: u=0, i
Connection: keep-alive
Content-Type: application/x-www-form-urlencoded
Content-Length: 0

{"id":"1"}
```
- 我们只对请求包进行了加密，所以选择：请求数据包
- 将要加密的数据包放入加密框进行测试，点击加密，发现加密成功  
	![image.png](https://i-blog.csdnimg.cn/img_convert/b7cc966c9c5d7a41131340cd50c3dac9.png)

#### 选项模块设置

- 和之前一样即可，记得保存配置  
	![image.png](https://i-blog.csdnimg.cn/img_convert/a41089cb53de89771b264888d7a82f92.png)

#### 效果

进行发包，然后到日志中查看  
![image.png](https://i-blog.csdnimg.cn/img_convert/2adf004a90e0519e6c96e40d16e68415.png)  
发现已经成功加密  
![image.png](https://i-blog.csdnimg.cn/img_convert/48c37b92f83fd273f6f63be9a9d6222d.png)

## autoDecoder插件配合进行爆破

这里演示 `autoDecoder` 插件+ `captcha-killer-modified` 插件组合爆破

#### 应用场景

比如现在这个数据包，如果我们要进行爆破，就需要

#### 配置captcha-killer-modified插件

没有安装的可以看一下我以前的文章： [BurpSuite最新2024.10版安装captcha-killer-modified+使用](https://mp.weixin.qq.com/s/m_rS7j5QT9bk2x-tKTppTw)

- 首先找到验证码生成的接口  
	![image.png](https://i-blog.csdnimg.cn/img_convert/5b39db3ab2c305d7651b6b9d0d3ba378.png)
- 抓包，发送到captcha-killer-modified进行测试  
	![image.png](https://i-blog.csdnimg.cn/img_convert/c13982c821cd630f2ca5a4ba2385ae75.png)  
	点击获取，可以看到已经获取了图片，然后右击下面的框，使用dddocr模板  
	![image.png](https://i-blog.csdnimg.cn/img_convert/180bb454dff9b103f34a225e92a2f820.png)  
	运行
```
python3 codereg.py
```

![](https://i-blog.csdnimg.cn/img_convert/0e809964d0890d01d6db939f584bffba.png)

可以看到识别效果还不错  
![image.png](https://i-blog.csdnimg.cn/img_convert/220cd09036beee124156bf8549634e99.png)

- 最后，一定要记得点击使用该插件，不然验证码是不会自动更改的，就一直都是识别一个验证码  
	![image.png](https://i-blog.csdnimg.cn/img_convert/8b32875e3da0b89beff9a7c208bd6831.png)

#### 配置autoDecoder插件

##### 根据实际情况编写接口代码

可以发现这个页面也是对password进行了md5加密（我输入的是123456）  
![image.png](https://i-blog.csdnimg.cn/img_convert/52be3c76313b8d8e1a54c471012bc0d3.png)

这样我们也可以自己配置接口加解密

- 不要改模板，不然会出问题
- `request.form.get('dataBody')` 得到的是post的所有的值，进行正则匹配和替换即可
- 注意 `captcha-killer-modified` 插件用的是8888端口，记得更一下默认端口，我这里改成了8887
```
# -*- coding:utf-8 -*-  
# author:f0ngf0ng  
  
from flask import Flask, Response, request,jsonify  
import base64, hashlib, json,re  
from pyDes import *  
  
# md5加密函数  
def md5_encrypt(s):  
    md5 = hashlib.md5()  
    md5.update(s.encode('utf-8'))  
    return md5.hexdigest()  
  
  
app = Flask(__name__)  
  
  
@app.route('/encode', methods=["POST"])  
def encrypt():  
    param = request.form.get('dataBody')  
    # print(param)  
    pattern = r'p_md5=([^&]+)'  
    match = re.search(pattern, param)  
    md5=match.group(1)  
    encry_param = param.replace(md5, md5_encrypt(md5).strip())  
    # print(encry_param)  
    return encry_param  
  
  
@app.route('/decode', methods=["POST"])  # 不解密  
def decrypt():  
    param = request.form.get('dataBody')  
    return param  
  
  
if __name__ == '__main__':  
    app.debug = True  # 设置调试模式，生产模式的时候要关掉debug  
    app.run(host="0.0.0.0", port="8887")
```

##### 测试接口效果

来到接口加解密，测试一下效果

- 接口的端口记得和接口脚本端口保持一致，这里是8887
- 我们是对请求包进行加密，所以选择请求数据包  
	点击加密，可以发现，已经成功加密  
	![image.png](https://i-blog.csdnimg.cn/img_convert/f51b07b8949c49f709d1ff0c23116b9b.png)  
	来到选项这里
- 选择接口加解密
- 记得配置加解密域名
- **明文关键字一定要更改，因为这次明文里面没有 " ，所以如果不进行更改，是不会进行自动加解密，这里可以改成 &**
- 记得保存配置  
	![image.png](https://i-blog.csdnimg.cn/img_convert/a05869dd0ade309d069f14da3b38452e.png)

然后来到重放器这里进行测试，密码使用明文：123456  
![image.png](https://i-blog.csdnimg.cn/img_convert/9fb93e1d77788fd64d3b7d529bc234b4.png)  
来到日志这里，可以发现已经被替换成了md5加密之后的值  
![image.png](https://i-blog.csdnimg.cn/img_convert/bf8ac079d6237a672749121d63e88980.png)  
现在可以准备进行爆破了

### 配置爆破模块

- 选择Pitchfork模式
- payload1就是正常的添加爆破所需的密码即可，payload2需要选择Extension-generated类型，并且生成器选择 `captcha-killer-modified`  
	![image.png](https://i-blog.csdnimg.cn/img_convert/a02be783cfa1f5f50d91461d646353a7.png)

![image.png](https://i-blog.csdnimg.cn/img_convert/05220c33d0b9920e2eae7dda1536a726.png)

### 最终效果

- 进行爆破，可以发现 `captcha-killer-modified` 插件可以正常使用

![image.png](https://i-blog.csdnimg.cn/img_convert/1ea01b337d980fb379ea358f0c4a0e81.png)

- 我们可以来到日志这里，查看，密码是否被加密处理，可以发现已经成功替换了原来的密码  
	![](https://i-blog.csdnimg.cn/img_convert/10c9673cd52f8ad88c7209ec26f1f485.png)

## 总结

- 其实大家可以发现，autodecode的主要使用的地方其实就是他的接口加解密，平常遇到了就写一个接口文件，慢慢积累，之后就可以快速应对大部分情况了
- 通过自己编写加解密接口，已经可以实现所有可能遇到的场景了，官方有很多案例  
	由于篇幅原因，更多的可以参考官方  
	https://github.com/f0ng/autoDecoder-usages
