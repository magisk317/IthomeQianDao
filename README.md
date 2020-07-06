# 更新 2020年7月6日17:57:26
## 签到破解

通过抓包发现，签到页面实际上是一个web页面，url为`https://my.ruanmei.com/app/user/signinwechat.html?signtype=wechatapp`，因为没有内嵌环境所以不能直接通过浏览器正常运行，关键js：`https://my.ruanmei.com/js/app/signinwechat.min.js?v=2020`由sojson.v5加密混淆，但还是能通过代码找出猫腻，通过搜索`endt`找到关键位置，然后就没有然后了，加密混淆后的代码顶不住。![UieGVO.png](https://s1.ax1x.com/2020/07/06/UieGVO.png)

## 理清思路

加密方式肯定没变，只不过是密钥变了，既然是web，那当然可以调试，所以想到了反编译小程序，然后通过小程序调试获取密钥。

## 通过小程序获取密钥

反编译小程序过程就不细讲了，拿到代码后发现了跳转签到页面的代码，这是个`web-view`，直接调试不了，百度：`小程序怎么调试web-view`，开发者工具中，在web-view出现的页面，点击鼠标右键，然后出现一个调试气泡提示信息，点击它就会跳出web-view的调试工具。

![UielKx.png](https://s1.ax1x.com/2020/07/06/UielKx.png)

调试和前端调试一样，控制台输入函数名`getEncryptKey()`输出`qs$^w`，那这个去解密发现不对，然后断点逐步调试，终于在一个地方发现正确的密钥。

![UieJaD.png](https://s1.ax1x.com/2020/07/06/UieJaD.png)

![UieoZT.png](https://s1.ax1x.com/2020/07/06/UieoZT.png)

##  在此之前的尝试

通过Fiddle断点替换js，js代码`'endt': _0x5e2efb['wMjTQ'](getEncryptStr)`改为`getEncryptStr`，然后通过url中的endt字段获取密钥，因为之前以为这个是待的加密文本所以endt没有变化，以为失败了就没再尝试，如果换成`getEncryptKey()`应该会有变化

# 简介
IT之家签到程序，~~可多次请求api获取上百金币，api链接中coinHistoryType字段为添加金币途径，设置为不同值就可以添加不同途径金币收入，具体收入明细可以去app中查看。(暂且失效)~~

安装python3，再在cmd里输入`pip install requests` `pip install Crypto` 然后改动代码里用户名和密码就可以了。

# it之家加密分析

除了签到加密方式，其它的加密方式都一样。

## 加密方法

DES加密，密钥为：`qs$^w<4!`，填充方式：`zeropadding`，输出方式： `hex`，加密代码见run.py中的getHash函数(长字符串会失效，因为没有填充到合适的位数，以后更新)

其它加密的密钥为：`(#i@x*l%`

## 在线解密

网站http://tool.chacuo.net/cryptdes

测试字符串为https://dyn.ithome.com/api/comment/getnewscomment?sn=1a84e1bfdc2ec68d 中的`1a84e1bfdc2ec68d`
[![NTvJTU.png](https://s1.ax1x.com/2020/07/01/NTvJTU.png)](https://imgchr.com/i/NTvJTU)

解密结果482628，上面的链接是评论api，482628是新闻id，所以原始新闻为https://www.ithome.com/0/482/628.htm

## 常见的加密字符串

userHash为getHash(用户名+md5(密码))
其它的一般直接加密

## 最后

加密算法破解方式：反编译app，~~其实之家一直都是这个加密方式，以前的app没有加壳，所以很容易得到。~~
