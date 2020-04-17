---
title: "Python如何发送邮件"
date: 2020-04-16T11:02:04+08:00
draft: false
categories: ["Python"]
tags: ["邮件"]
featured_image: 
author: "Tifinity"
---

# Python如何发送邮件

>大部分人学习或者使用某样东西，喜欢在直观上看到动手后的结果，才会有继续下去的兴趣。

使用Python自带的email包和smtplib包就可以实现发送邮件，emil负责构造邮件，smtplib负责发送邮件。

提醒一下，千万不要把你的文件名设为 email.py 因为这个名字与email包名重合，当前目录的优先级比Python安装目录和环境变量优先级高，可以验证一下：

~~~python
import sys
print(sys.path)
~~~

所以Python会找不到模块，出现下面这样的错误：

```python
ModuleNotFoundError:
No module named 'email.mime'; 'email' is not a package
```

然后发送邮件可以使用本机上的邮件服务器来发送，也可以远程连接QQ邮箱或者网易邮箱等服务器来发送，这里我先使用QQ邮箱发一个最简单的纯文本邮件。

第一步，创建邮件并设置邮件信息：

~~~python
sender = 'XXX@qq.com' 			# 发送者
receivers = ['XXX@qq.com']  	# 接收者
subject = 'Python SMTP 邮件测试'
content = '你要发送的邮件内容' 
# 创建纯文本对象，参数：正文，类型plain表示纯文本，编码格式utf-8
message = MIMEText(content, 'plain', 'utf-8')
message['From'] = Header(sender, 'utf-8')		# 发送者
message['To'] =  Header(receivers[0], 'utf-8')	# 接收者
message['Subject'] = Header(subject, 'utf-8')	# 主题
~~~

第二步，设置第三方SMTP服务：

~~~python
# 第三方 SMTP 服务
mail_host="smtp.qq.com"  		# 服务器地址
mail_pass="XXXXX"   			# 授权码
~~~

有一点需要注意，授权码，不是账户密码，可查看QQ邮箱的帮助中心获取，可以先故意输错然后在报错信息中点开网址。

~~~shell
Error: 无法发送邮件:(535, b'Login Fail. Please enter your authorization code to login. More information in http://service.mail.qq.com/cgi-bin/help?subtype=1&&id=28&&no=1001256')
~~~

第三步，发送：

~~~python
# 连接邮件服务器
smtpObj = smtplib.SMTP_SSL(mail_host, 465) 
# 登录邮件服务器
smtpObj.login(sender, mail_pass)
# 发送邮件
smtpObj.sendmail(sender, receivers, message.as_string())
# 退出服务器
smtpObj.quit()
print("邮件发送成功")
~~~

这样一封基本的邮件就可以发出去了，然后更多关于邮件头的设置，以及HTML邮件，带附件的邮件等可以查看：[廖雪峰廖老师的教程](https://www.liaoxuefeng.com/wiki/1016959663602400/1017790702398272)



