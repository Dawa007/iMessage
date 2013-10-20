iMessage安全性如何

iMessage安全性如何，最近这些天挺火的，那么我也来凑凑热闹。

基于对iOS 6.1.3的分析。

iMessage加密主要有两块。

（1）设备和Apple服务器的加密通讯；

（2）iMessage消息内容的加密；

1、通讯流程

系统中有一个apsd（全称apple push serviced）的进程，该进程负责与苹果服务器进行通讯，采用TLS加密传输。

既然是加密传输，那么需要一个证书吧。

苹果设备在首次使用时，都需要激活，该激活过程就采用https方式。

albert.apple.com/WebObjects/ALUnbrick.woa/wa/deviceActivation

好奇的话，赶紧去截获一下协议吧，当然里面有几个数据加密的，用肉眼是看不出来，但是gdb可以帮你。

激活成功后，苹果就会返回一个证书。

下面是一个比较简单的通讯监控流程：

apsd 会通过DNS去查询 "push.apple.com" 对应的ip地址。 [ nslookup -query=txt push.apple.com] 

通过修改DNS，把"push.apple.com"指向我们的电脑，

然后再我们电脑上写个简单的中转程序，来转发数据包给真正的苹果服务器，这样就可以截获到通讯数据。（记得从iOS上面把证书拷贝出来）

然后你就能看到：当huluwateam@gmail.com向mailto:zyyevo@126.com消息时，你就能看到。


<dict>

        <key>D</key>
        <true/>
        <key>E</key>
        <string>pair</string>
        <key>P</key>
        <data>
        H4sIAAAAAAAAAwFoAZf+AgEdOqZhIJfiVPZgBG6sepOBwAfmyRJ6hI+fIKe2WF5pu8OF
        yODtUNNcH7HsotE27wopJoXqj+erUoGs0S3iUb2U0Qsj+2j0DUzxklWq20ZMPYFXgUhU
        s+VKU74C901o77g6Z9j6CYuECYBWubOZQah90aFtbSOgfZSHnrgDnFqZ8Vkh89zFi+o7
        ibGJAcyrbMM7TpaEVWPCSWiwUJgs9upS2tsDcLPUx/YOPXAPCSL7UDQWBGqfAm1enlbS
        0QnKZFJcWaqV0SUGTI+yRt8FdkzCRltrECy4sVAtHjPtK1SjgC8FF4fDXWsNMNjc6Uhg
        pgneij3oRUa/aasqLfbBd14TwD0VSEpMTYfHAAmML0VMFvSgRHifsPEPxFJD73TPRzBF
        AiAAjJ51M0pflzS6sLzD5wnQwecp+tBjZiuLebpNCDI0cQIhAKlLdRvKxWZaFxXJgUup
        UYJI6Q/phbB9dg8DzfSeHNYE7Nyt3mgBAAA=
        </data>
        <key>U</key>
        <data>
        cenrWmCST6q2mSVvaM4PRA==
        </data>
        <key>c</key>
        <integer>100</integer>
        <key>e</key>
        <integer>1379911703359339674</integer>
        <key>pri</key>
        <integer>0</integer>
        <key>sP</key>
        <string>mailto:huluwateam@gmail.com</string>
        <key>t</key>
        <data>
        OxAsXdTK3pu5U3WVVlhxvLG11U/k0XdKGSx8UGkfRh8=
        </data>
        <key>tP</key>
        <string>mailto:zyyevo@126.com</string>
        <key>v</key>
        <integer>1</integer>
</dict>


即使用姐的24K铝合钛金狗眼，自然无法看出huluwateam是发送了什么消息。

但时间能看出来1379911703359339674

但是P字段肯定就是消息的内容，当然必须是加密的。

2、消息加密

iOS keychain中存储着两个私钥 RSA "iMessage encryption key" 和 ECDSA "iMessage signing key"。

跟踪一下，首先用AES256对明文进行加密，然后用"iMessage encryption key"加密一下AES密钥，

最后用"iMessage signing key"签名一下密钥和密文。

解密出来就是这样子的。


<dict>

        <key>p</key>
        <array>
                <string>mailto:huluwateam@126.com</string>
                <string>mailto:zyyevo@126.com</string>
        </array>
        <key>t</key>
        <string>Do you have a boyfriend?</string>
        <key>v</key>
        <string>1</string>
</dict>

先举个例子，

如果张三想给李四发消息，

先会用李四的RSA公钥对消息进行加密，

然后再用自己的ECDSA私钥对数据包进行签名。

李四接受到消息了，先用张三的ECDSA公钥校验一下，确定该消息是来自三张。

然后再用自己的RSA私钥解密。

所以，如果想通过截获通讯数据包，来破解或者篡改消息，还是省省。

换个方式，考虑一下密钥生成和分发的安全性。

上面两个私钥都是在设备上生成的,然后在iMessage注册的时候，会把两个私钥对应的公钥发送给苹果服务器。

Apple spokeswoman Trudy Muller, in a statement to AllThingsD: 

iMessage is not architected to allow Apple to read messages. 

The research discussed theoretical vulnerabilities that would require Apple to re-engineer the iMessage system to exploit it, and Apple has no plans or intentions to do so.

但苹果想读的话，你的设备就有可能在你不经意的情况下，偷偷把"iMessage encryption key"上传到服务器中。

有没有这样的后门，我就不知道了。但目前是没人发现。

iMessage垃圾短信特别多，幸好iOS7上面，有手动添加黑名单的功能。

其实苹果挺为难，没办法在服务端进行拦截，一旦拦截，网络上的各大新闻媒体头条“苹果正在看你聊天”。

相比于短信，国内的各种即使通讯，iMessage是安全的。

DaWa

huluwateam@gmail.com

