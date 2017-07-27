title: QQ邮箱操作模拟
tags:
  - spider
  - server
  - lib
categories:
  - server
date: 2015-08-08 23:51:12
---

在此输入正文
项目需要一用程序操作QQ邮箱执行一些操作，现用python实现一套QQ邮箱的操作接口。

## 登录
通过模拟mail.qq.com登录的http请求，来实现登录操作，主要是获取登录操作所需要的SID，也就是session id. 这个步骤是最关键的，过程也比较复杂, 下面一一道来。

<!--more-->

### 步骤一 加载邮箱的登录页，获取步骤一所要的xlogin地址
在登录的form前有一个iframe标签，它访问了一个地址获取cookie，第一步要做的就是拿到这个地址，访问后保存下返回的cookie，就是下面这货
![](http://res.astraylinux.com/spider/qqmail_1.png)

### 步骤二 访问check接口，检查是否需要验证码
有些QQ号如果换了登录地区或者多次输错可能就需要输入验证码。在登录页面上，当输入完QQ号后，js代码就会去访问下面这个地址, 检查是否需要验证码(XXXX是QQ号， YYYY可能是访问途径的编号，步骤一可以取到，ZZZZ也是步骤一的cookie)。
```bash
"https://ssl.ptlogin2.qq.com/check?regmaster=&pt_tea=1&pt_vcode=0&uin=XXXX&appid=YYYY&js_ver=10131&js_type=1&login_sig=ZZZZ&u1=https%3A%2F%2Fen.mail.qq.com%2Fcgi-bin%2Flogin%3Fvt%3Dpassport%26vm%3Dwpt%26ft%3Dloginpage%26target%3D&r=0.4220805915914432"
```

返回的结果如下， 括号内第一位如果是'1'的话，就需要验证码，如果是'0'，则第二位就是后面各个接口要用的验证码,第三位是QQ号的十六进制表示，第四位后面接口也要用到，第五位还不清楚。
```bash
ptui_checkVC('0','!TIH','\x00\x00\x00\x00\xa4\x3f\xcf\xb1','43bb5c434b0bfd3e7efc71a4deca8de60fc877e5688b5a6a088eeed858d79ff00d7e70e7e795f4eef98dbd8c4cd0110db7938b5833bc4595','0');
```

### 步骤三 访问https的登录接口
这是最麻烦的一步，这一步对应的是输入完账号密码后访问的第一个接口，腾讯和程序在这一步用JS对密码进行了多重加密，包括一个RSA加密，丧心病狂![](http://res.astraylinux.com/emoji/34.png)。所以，这一步必须实现这一整套的加密算法。本来想用python对着js实现一套的，写了一段放弃了，工作量太大。后面找了在python中执行js代码的方法，找到了[python-spidermonkey](https://code.google.com/p/python-spidermonkey/)(这个要到google code下载，github那个我下过来编译不过![](http://res.astraylinux.com/emoji/14.png)。

接着把`c_login_2.js`的代码取出来，找到`Encryption`和`RSA`模块，提取出来(先用一些工具格式化，站长之家有），并去掉里面需要DOM的部分（没找到在spidermonkey支持DOM的方法![](http://res.astraylinux.com/emoji/75.png)）。最后做一个像下面这样的代码
```js
//两个模块的代码，一千多行
...
en_password = $.Encryption.getEncryption("xxxx_password", "xxxx_salt", "xxxx_vcode",  xxxx_is_save);
en_password;
```
用将上面整块代码当成字符串，在python中调用。这里的salt就是上面用十六进表示的QQ号。
```python
#!/usr/bin/env python
# encoding: utf-8
import spidermonkey

def get_en_password(password, salt, vcode):
    sm_rt = spidermonkey.Runtime()
    sm_cx = sm_rt.new_context()
    script = lib.js_content.content.replace("xxxx_password", password)
    script = script.replace("xxxx_salt", salt).replace("xxxx_vcode", vcode)
    script = script.replace("xxxx_is_save", "false")
    return sm_cx.eval_script(script)
```
通过上面的函数获得加密后的密码，并与前面取得的cookie得信息合并起来，访问`https://ssl.ptlogin2.qq.com/login`接口，获取第四步要访问的地址和cookie。（这完全就是在玩破关游戏啊![](http://res.astraylinux.com/emoji/41.png))。得到类似下面的结果，进入第四步。
```bash
"ptuiCB('0','0','https://ssl.ptlogin2.mail.qq.com/check_sig?pttype=1&uin=2755645361&service=login&nodirect=0&ptsigx=0b2c4d8054a80b793c58fda08b89dfc6b6921442063ba599053f23ea9189c853827376c48101859e5031faddcd98b83a76d85ddb860ce5c67b82e0b9943fcf86&s_url=https%4A%2F%2Fen.mail.qq.com%2Fcgi-bin%2Flogin%3Fvt%3Dpassport%26vm%3Dwpt%26ft%3Dloginpage%26target%3D%26account%3D2755645361&f_url=&ptlang=1033&ptredirect=101&aid=522005705&daid=4&j_later=0&low_login_hour=0&regmaster=0&pt_login_type=1&pt_aid=0&pt_aaid=0&pt_light=0&pt_3rd_aid=0','1','Sign-in successful!', '⊥qz●‰');"
```

### 步骤四 验证登录信息（大概）
这一步就是访问第三步结果串里的地址。看似很简单，不过这里有个坑，参数中`nodirect=0`，也就是说，访问后会进行重定向，然后你就取不到想要的cookie和结果串了，花了我两个多小时才发现![](http://res.astraylinux.com/emoji/44.png)。

将`nodirect=0`改成`nodirect=1`后访问，就能取得所需要的cookie了。

### 步骤五 访问最终的登录页，取得SID
终于走的BOSS面前了，不容易啊![](http://res.astraylinux.com/emoji/18.png)。这一步其实也简单了，直接加上上一步取到的cookie访问`https://mail.qq.com/cgi-bin/login?vt=passport&vm=wpt&ft=loginpage&target=&account=XXXX`就可以了。

不过这里要注意cookie对应的地址，`mail.qq.com`与`qq.com`的cookie是有重复的，要防止qq.com的cookie覆盖mail的。最后拿到一大堆的cookie和sid，登录完成。
![](http://res.astraylinux.com/spider/qqmail_2.png)
