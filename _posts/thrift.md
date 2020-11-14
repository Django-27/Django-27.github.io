# 最简单的一个 Thrift-Python 例子
第一步：编写 Thrift 文件
```
/*
thrift IDL 接口定义文件, hello.thrift
*/
service HelloService {
    string say(1:string msg)
}
```
第二步： 生成 Thrift 文件
```
thrift -r -gen py hello.thrift
```
第三步：编写 Service 和 Client 并运行

python thrift_server.py  <->  python thrift_client.py
![20171102](https://github.com/Django-27/workspace/blob/master/pic/thrift-python.png)

注意：python thrift error ```TSocket read 0 bytes``` 或 No handlers could be found for logger "thrift.server.TServer"
通过添加日志，将异常打印；数据库中的时间在传递的时候需要转成字符串，否则有问题。

     
# 10 发一封邮件
实例一： 纯文本, email 负责构造邮件, smtplib 负责发送邮件
```
from email.mime.text import MIMEText
from email.header import Header
import smtplib

message =''' hello，world！ 来自我的电脑 '''
msg = MIMEText(message, 'plain', 'utf-8')
msg['Subject'] = Header('come from paython', 'utf-8')
msg['From'] = Header('nfs_fly@163.com')
msg['To'] = Header('qiyuan.wei@ds365.com')

from_addr = 'nfs_fly@163.com'
passwd='n**6'
to_addr = 'qiyuan.wei@ds365.com'
smtp_server = 'smtp.163.com'
server = smtplib.SMTP(smtp_server,25)
server.login(from_addr,passwd)
server.sendmail(from_addr,to_addr,msg.as_string())
server.quit()
```
实例二：发送 HTML 邮件, MTMEText 参数又 plain 变为 html 即可 (有可能：被对方服务器退回)
```
msg = MIMEText('<html><body><h1>Hello</h1>' +
    '<p>send by <a href="http://www.python.org">Python</a>...</p>' +
    '</body></html>', 'html', 'utf-8')
```
拍错：[企业退信的常见问题](http://help.163.com/09/1224/17/5RAJ4LMH00753VB8.html)

实例三：发送附件,带附件的邮件可以看做包含若干部分的邮件：文本和各个附件本身
MIMEMultipart对象代表邮件本身，然后往里面加上一个MIMEText作为邮件正文，再继续往里面加上表示附件的MIMEBase对象即可

```
msg = MIMEMultipart()
msg['Subject'] = Header(str(message), 'utf-8')
msg['From'] = Header('nfs_fly@163.com')
msg['To'] = Header('qiyuan.wei@ds365.com')
msg.attach(MIMEText('发送带附件的正文', 'plain', 'utf-8'))

# 结合之前的日志统计图片, 实现了讲数据图片发送给相关人员
IMAGE_NAME = "examples.png"
x, y = [1, 2, 3, 4, 5, 7], [1, 2, 1, 1, 3, 5]
plt.scatter(x, y) 
plt.title(u'统计原生支付-微信正扫，api发出请求到收到回调的时间( 单位/毫秒) ')
plt.xlabel(u'总共记录数 %s 选取统计的范围 [%s:%s]')
plt.ylabel(u'毫秒')
plt.grid()
plt.savefig(IMAGE_NAME)
# plt.show()

#添加附件就是加上一个MIMEBase，从本地读取一个图片
with open(IMAGE_NAME, 'rb') as f:
    mime = MIMEBase('image', 'png', filename='a.png')
    # 加上必要的头信息
    mime.add_header('Content-Disposition', 'attachment', filename=IMAGE_NAME)
    mime.add_header('Content-ID', '<0>')
    mime.add_header('X-Attachment-Id', '0')
    # 把附件的内容读进来
    mime.set_payload(f.read())
    # 用Base64编码
    encoders.encode_base64(mime)
    msg.attach(mime)
```