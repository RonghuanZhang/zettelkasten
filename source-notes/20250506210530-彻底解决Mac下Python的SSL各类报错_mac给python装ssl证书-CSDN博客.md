---
type: "fleet-note"
title: "彻底解决Mac下Python的SSL各类报错_mac给python装ssl证书-CSDN博客"
id: 250506210530
created: 2025-05-06T21:08:30
source:
  - "web"
url: "https://blog.csdn.net/u011072037/article/details/102861658?spm=1001.2101.3001.6650.4&utm_medium=distribute.pc_relevant.none-task-blog-2~default~BlogCommendFromBaidu~Rate-4.pc_relevant_paycolumn_v3&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2~default~BlogCommendFromBaidu~Rate-4.pc_relevant_paycolumn_v3"
tags:
  - "fleet-note"
processed: false
archived: false
---
## 彻底解决Mac下Python的SSL各类报错

最新推荐文章于 2025-04-03 16:48:55 发布

[洛城-sola](https://blog.csdn.net/u011072037 "洛城-sola") 于 2019-11-01 17:33:14 发布

分类专栏：

版权声明：本文为博主原创文章，遵循 [CC 4.0 BY-SA](http://creativecommons.org/licenses/by-sa/4.0/) 版权协议，转载请附上原文出处链接和本声明。

本文链接： [https://blog.csdn.net/u011072037/article/details/102861658](https://blog.csdn.net/u011072037/article/details/102861658)



![](https://kunyu.csdn.net/1.png?p=58&adBlockFlag=0&adId=1068446&a=1068446&c=2732037&k=%E5%BD%BB%E5%BA%95%E8%A7%A3%E5%86%B3Mac%E4%B8%8BPython%E7%9A%84SSL%E5%90%84%E7%B1%BB%E6%8A%A5%E9%94%99&spm=1001.2101.3001.5002&articleId=102861658&d=1&t=3&u=9e9ddf161f054bca8895ddd228bd3170)[*Mac* 10.13安 *装* *python* 3.6和TensorFlow1.15，可解决 *ssl* 相关问题](https://guotong1988.blog.csdn.net/article/details/128980791)

[Hope^\_^](https://blog.csdn.net/guotong1988)

02-11 327[没有这个问题：https://blog.csdn.net/guotong1988/article/details/103498263。](https://guotong1988.blog.csdn.net/article/details/128980791)

4 条评论

[Beronhider](https://blog.csdn.net/Beronhider) 热评

很好 但是忘记密码了怎么办？好像没印象设过密码呀

写评论[关于生成 *SSL* 数字 *证书* 的总结\_ *mac* 支持 *python* requestshttps *证书* 资源...](https://download.csdn.net/download/firelightcz123/2377663)

4-26[一、生成 *SSL* 数字 *证书* 为了生成 *SSL* 数字 *证书*,我们需要使用keytool命令。keytool命令是一个Java 工具,用于生成数字 *证书* 。下面是一个生成 *证书* 的示例命令: keytool -genkey -alias jiacheng -keypass jiacheng -validity 3600 在上面的命令中,我们使用了-genkey选项来生成 *证书*,-alias选项指定了 *证书* 的别名,-keypass选项指定...](https://download.csdn.net/download/firelightcz123/2377663)[*Python* 提示 *SSL* 认证 「InsecureRequestWarning」\_insecurerequestwarni...](https://blog.csdn.net/ASUKA2020/article/details/84404907)

4-23[*Python* 提示 *SSL* 认证 「InsecureRequestWarning: Unverified HTTPSrequestis being made. Adding certificate verification is strongly advised」 解决方法: # 引入模块 from requests.packages.urllib3.exceptions import InsecureRequestWarning # 禁用安全请求警告...](https://blog.csdn.net/ASUKA2020/article/details/84404907)[*mac* 电脑通过pyenv 安 *装* *python* 失败，提示 *报错* ：Error The *Python* *ssl* extension was not compiled.](https://blog.csdn.net/mukvintt/article/details/135061974)

[mukvintt的博客](https://blog.csdn.net/mukvintt)

12-18 1301[I 选项后面跟着的是头文件的路径，这里使用 $(brew --prefix open *ssl*) 获取 Open *SSL* 的安 *装* 路径，然后添加 /include 子目录，表示 Open *SSL* 的头文件所在的路径。-L 选项后面跟着的是库文件的路径，这里同样使用 $(brew --prefix open *ssl*) 获取 Open *SSL* 的安 *装* 路径，然后添加 /lib 子目录，表示 Open *SSL* 的库文件所在的路径。而且对应的 *ssl* 的代码路径有些不一样，需要对齐调整，具体详情看 # solution.](https://blog.csdn.net/mukvintt/article/details/135061974)[*mac* 运行 *python* 报 *SSL*: CERTIFICATE\_VERIFY\_FAILED\] certificate verify failed: unable to get local issuer](https://blog.csdn.net/weixin_44342166/article/details/139067985)

[Binyee的博客](https://blog.csdn.net/weixin_44342166)

05-20 965[库下载模型也无法解决 *SSL* *证书* 验证的问题。这可能是由于 *mac* OS 系统本身对于 *SSL* *证书* 的配置和信任存在问题。，绕过了 *SSL* *证书* 验证,从而可以正常下载和处理图像。这个修改可以解决之前 *SSL* *证书* 验证失败的问题。](https://blog.csdn.net/weixin_44342166/article/details/139067985)[...16.04 添加永久免费https *SSL* *证书* (解决 *python* 2.7 - pip wheel faile...](https://blog.csdn.net/setoy/article/details/78442436)

4-29[*证书* 更新 先切换到 *Python* 2,切换 *python* 2/3,点这里。./certbot-auto certonly --apache --renew-by-default -d abc.com -d www.abc.com 1 转发一个自动更新的脚本: #!/bin/bash#===# Let's Encrypt renewal script for Apache on Ubuntu/Debian# @author Erika Heidi<erika@do.co># Usage:./le...](https://blog.csdn.net/setoy/article/details/78442436)[*python* -certifi:(*Python* 发行版)精心挑选的根 *证书* 集合,用于在验证...](https://download.csdn.net/download/weixin_42140710/16763535)

4-27[*证书*:*Python* *SSL* *证书* 提供了Mozilla精心策划的根 *证书* 集合,用于在验证TLS主机身份的同时验证 *SSL* *证书* 的可信赖性。 它已从“项目中提取。 安 *装* certifi可在PyPI上使用。 只需使用pip安 *装* 它: $ pip install certifi 用法 要引用已安 *装* 的 *证书* 颁发机构(CA)捆绑包,可以使用内置功能: >>> import certifi >>> certifi.whe...](https://download.csdn.net/download/weixin_42140710/16763535)[修复 *SSL* *证书* 链不完整问题certificate verify failed unable to get local issuer certificate](https://blog.csdn.net/qq_35578171/article/details/146980706)

[

最新发布

](https://blog.csdn.net/qq_35578171/article/details/146980706)

[Lorin 洛林的后端技术小站](https://blog.csdn.net/qq_35578171)

04-03 973[最近，我在服务器上更新了 *SSL* *证书* 后，虽然网站可以正常访问，浏览器显示 *证书* 有效，但在部分文章平台引用服务器上的图片时，图片无法被转存。排查过程中使用 *Python* 代码尝试下载图片时， *报错* 如下，怀疑是 *SSL* *证书* 链不完整：进一步使用进行验证，发现服务器的 *证书* 链确实不完整，导致部分客户端无法正确验证 *SSL* *证书* 。最终通过调整 Nginx 的 *SSL* 配置 解决了这个问题。本文将详细介绍 如何排查和解决 *SSL* *证书* 更新后图片无法转存的问题。](https://blog.csdn.net/qq_35578171/article/details/146980706)[*Mac* *python* 3 *报错* cannot import name ‘ *ssl* ‘](https://guotong1988.blog.csdn.net/article/details/103498263)

[Hope^\_^](https://blog.csdn.net/guotong1988)

12-11 1386[安 *装* 好brew后 brew install open *ssl* brew install open *ssl* -devel 打开 *Python* 3.6.8源码包里的Modules/Setup.dist文件 打开这五行的注释 \_socket socketmodule.c *SSL* =/usr/local/ *ssl* \_ *ssl* \_ *ssl*.c \\ -DUSE\_ *SSL* -I$(*SSL*)/include -I...](https://guotong1988.blog.csdn.net/article/details/103498263)[用 *mac* 系统后 *python* 遇到\[*SSL*: CERTIFICATE\_VERIFY\_FAILED\]](https://blog.csdn.net/qq_42412061/article/details/136459303)

[qq\_42412061的博客](https://blog.csdn.net/qq_42412061)

03-04 1171[*python* 是那个版本就把3.8写成那个版本的。解决办法就是在 *mac* 终端输入。](https://blog.csdn.net/qq_42412061/article/details/136459303)[*mac* 解决 *python* 连接 ws关于 *ssl* *证书* 问题](https://devpress.csdn.net/v1/article/detail/123532594)

[c1719561053的博客](https://blog.csdn.net/c1719561053)

03-16 4859[*报错* ： *ssl*.*SSL* CertVerificationError: \[*SSL*: CERTIFICATE\_VERIFY\_FAILED\] certificate verify failed: self signed certificate in certificate chain (\_ *ssl*.c:997) 解决： import websocket import json import *ssl* url = "wss://localhost:XXXXX" print("connect....") ws = web](https://devpress.csdn.net/v1/article/detail/123532594)[量化交易项目中 *Python* *Mac* 虚拟环境中 *SSL* 错误的问题和websockets库的示例](https://blog.csdn.net/secyson/article/details/135301266)

[secyson的博客](https://blog.csdn.net/secyson)

12-30 616[最后，贴上websockets库的使用示例，这个案例还挺好的，涉及到asycio的异步处理，虽然刚开始理解起来优点晦涩，但是搞懂了协程，就又进步了一个台阶。最近在做一个量化交易的小项目玩，在对接交易平台API接口的时候，正常通过requests库出去的请求，没什么问题，看了下源码，是通过https请求的。踩了半天坑，最后在stackoverflow上，看到一位友人的配置，解决了我的问题。以上，针对Apple Silicon， X86版本的 *Mac* OS *Python* 可能安 *装* 位置不同，自己找到相关目录。](https://blog.csdn.net/secyson/article/details/135301266)[*python* 3 环境问题-通过软链解决-发送https请求 *报错* (*ssl*)、解释器下左右键出现乱码](https://blog.csdn.net/jinli5621/article/details/116782604)

[金百万的测试博客](https://blog.csdn.net/jinli5621)

05-14 375[1、问题背景 使用的 *Python* 是 *Mac* 下的 *Python* 3.6.3，本来环境一切正常，安 *装* 一个工具后，出现的问题 （1）使用requests库，发送https请求时 *报错* （发送http正常） （2）在 *Python* 3解释器下，使用左右键跳转光标时，出现乱码 2、问题分析及解决方案 （1）针对问题1，通过搜索资料，发现是import *ssl* *报错* ，然后打开 *Python* 3解释器，直接import *ssl* ，查看 *报错* 信息，发现 *报错* 信息如下： *python* 3.6 import...](https://blog.csdn.net/jinli5621/article/details/116782604)[*mac* book pyenv 安 *装* *python* 3.6.8 *报错*](https://blog.csdn.net/qq_24137739/article/details/119875552)

[qq\_24137739的博客](https://blog.csdn.net/qq_24137739)

08-23 1347[else if(0 == \_NSGetExecutablePath(execpath, &nsexeclength) && execpath\[0\] == SEP) { ^~~~~~~~~~~~~ /Library/Developer/CommandLineTools/SDKs/ *Mac* OSX.sdk/usr/include/ *mac* h-o/dyld.h:98:54: note: pa...](https://blog.csdn.net/qq_24137739/article/details/119875552)[解决 *mac* os 安 *装* MySQL- *python* *报错*](https://blog.csdn.net/weixin_37830416/article/details/120885655)

[wavewavego](https://blog.csdn.net/weixin_37830416)

10-21 1311[解决 *mac* os 安 *装* MySQL- *python* *报错* 记录一次 *mac* os 安 *装* MySQL- *python* *报错* 1. my\_config.h not found解决方案2. library not found for -l *ssl* 解决方案 记录一次 *mac* os 安 *装* MySQL- *python* *报错* 同事说自己安 *装* MySQL- *python* *报错* 误，结果我自己试了试也是一样，本次安 *装* 总共遇到两个错误：\_mysql.c:44:10: fatal error: ‘my\_config.h’ file not found 和 ld: li](https://blog.csdn.net/weixin_37830416/article/details/120885655)[【 *Python* -因特网客户端编程-10】邮件系统用 *python* 发邮件 *报错* Failed to send email: Connection unexpectedly closed，什么原因怎么解决](https://blog.csdn.net/weixin_46453070/article/details/139344760)

[weixin\_46453070的博客](https://blog.csdn.net/weixin_46453070)

05-31 2275[Failed to send email: Connection unexpectedly closed” 错误通常是由于 SMTP 服务器连接中断或连接被拒绝引起的。如果问题仍然存在，可以尝试联系邮箱提供商的技术支持，获取更多帮助。确保本地防火墙或安全软件没有阻止 SMTP 端口（465 或 587）的连接。确保你的网络连接稳定，并且可以访问互联网。如果你的网易邮箱需要使用授权码而不是密码，请确保在代码中使用正确的授权码。尝试重新连接 SMTP 服务器，确保连接没有中途断开。](https://blog.csdn.net/weixin_46453070/article/details/139344760)[*Mac* *Python* 3 下载数据时候的 *SSL* 错误的解决](https://blog.csdn.net/Jemary_/article/details/88857096)

[Jemary\_的博客](https://blog.csdn.net/Jemary_)

03-27 2023[urllib.error.URLError: <urlopen error \[*SSL*: CERTIFICATE\_VERIFY\_FAILED\] certificate verify failed: unable to get local issuer certificate (\_ *ssl*.c:1051)> 先放错误，从tensorflow下载MNIST数据集的时候报了上述错误。 查了好...](https://blog.csdn.net/Jemary_/article/details/88857096)[解决 *python* 3.6 *mac* OS X *SSL* CERTIFICATE\_VERIFY\_FAILED](https://blog.csdn.net/fanjialiang2401/article/details/78979241)

[IceCola的博客](https://blog.csdn.net/fanjialiang2401)

01-05 1099[down vote StackOverflow 上有完美的解释 This isn't a solution to your specific problem, but I'm putting it here because this thread is the top Google result for " *SSL*: CERTIFICATE\_VERIFY\_FAILED",](https://blog.csdn.net/fanjialiang2401/article/details/78979241)[*mac* *python* request *ssl* 错误解决](https://blog.csdn.net/qq_35899407/article/details/109841066)

[长门有希的博客](https://blog.csdn.net/qq_35899407)

11-20 519[点击安 *装*](https://blog.csdn.net/qq_35899407/article/details/109841066)[*Mac* *报错* *ssl*.*SSL* Error：\[*SSL* ：CERTIFICATE\_VERIFY\_FAILED\]](https://blog.csdn.net/Xurui_Luo/article/details/109146645)

[Xurui\_Luo的博客](https://blog.csdn.net/Xurui_Luo)

10-18 1071[*mac* OS： *Mac* OSX *python* *ssl*.*SSL* Error：\[*SSL* ：CERTIFICATE\_VERIFY\_FAILED\] *证书* 验证失败 /Applications/ *Python* 3.7目录下，运行 Install Certificates.command](https://blog.csdn.net/Xurui_Luo/article/details/109146645)[*Mac* Book上pycharm使用urllib库出现 *SSL* *证书* 错误解决方法](https://blog.csdn.net/2301_80681778/article/details/144320431)

[2301\_80681778的博客](https://blog.csdn.net/2301_80681778)

12-08 525[这里的文件夹名有空格，直接cd *python* 3.12是不能识别的，所以要用\\符号，注意\\和3之间有空格，我觉得这个\\可以理解为让终端继续识别文件名。从文末参考的blog中我们可以知道 *Mac* Book中 *python* 的安 *装* 路径是/Applications;我的电脑查看后得到的就是以上图片，可以看到里面的 *python* 3.12就是我们要的 *python* 文件夹。进入 *python* 文件夹后，可以看到Install Certificates.command文件，输入。来运行这个文件，然后就完成了。（这里同样要用\\符号）](https://blog.csdn.net/2301_80681778/article/details/144320431)[*mac* OS中的certificate verify failed问题](https://blog.csdn.net/biubiu713/article/details/130736412)

[biubiu713的博客](https://blog.csdn.net/biubiu713)

05-17 848[【代码】 *mac* OS中的certificate verify failed问题。](https://blog.csdn.net/biubiu713/article/details/130736412)[*python* websocket *ssl* *报错* ，IndexError deque Index out of range](https://wenku.csdn.net/answer/3v855s2sga)

08-23[你好！关于你遇到的问题，出现 "IndexError: deque index out of range" 错误通常意味着你正在尝试访问一个不存在的列表或队列元素。这个错误在使用 *Python* 的 \`collections.deque\` 时经常出现。 要解决这个问题，你可以检查以下几点： 1. 确保你的队列（deque）不是空的：在访问队列元素之前，你应该先检查它是否为空。可以使用 \`if len(deque) > 0:\` 条件来检查队列中是否有元素。 2. 检查你正在访问的索引是否超出了队列的范围：确保你正在访问的索引值在队列的有效范围内。例如，如果你的队列长度为 \`n\`，那么有效的索引范围应该是从 0 到 \`n-1\`。 3. 确保你的索引值是合法的：如果你正在使用负数索引或超出队列长度的正数索引，都会导致 "IndexError" 错误。请确保你的索引值满足队列的长度限制。 如果你能提供更多的代码细节或示例代码，我可以给出更具体的帮助。](https://wenku.csdn.net/answer/3v855s2sga)

评论 4

被折叠的 0 条评论 [为什么被折叠?](https://blogdev.blog.csdn.net/article/details/122245662)[到【灌水乐园】发言](https://bbs.csdn.net/forums/FreeZone)

添加红包

实付 元

[使用余额支付](https://blog.csdn.net/u011072037/article/details/)

点击重新获取

扫码支付

钱包余额 0

抵扣说明：

1.余额是钱包充值的虚拟货币，按照1:1的比例进行支付金额的抵扣。  
2.余额无法直接购买下载，可以购买VIP、付费专栏及课程。

[余额充值](https://i.csdn.net/#/wallet/balance/recharge)

举报

 [![](https://csdnimg.cn/release/blogv2/dist/pc/img/toolbar/Group.png) 点击体验  
DeepSeekR1满血版](https://ai.csdn.net/?utm_source=cknow_pc_blogdetail&spm=1001.2101.3001.10583) ![](https://csdnimg.cn/release/blogv2/dist/pc/img/toolbar/full.png) 隐藏侧栏 ![程序员都在用的中文IT技术交流社区](https://g.csdnimg.cn/side-toolbar/3.6/images/qr_app.png)

程序员都在用的中文IT技术交流社区

![专业的中文 IT 技术社区，与千万技术人共成长](https://g.csdnimg.cn/side-toolbar/3.6/images/qr_wechat.png)

专业的中文 IT 技术社区，与千万技术人共成长

![关注【CSDN】视频号，行业资讯、技术分享精彩不断，直播好礼送不停！](https://g.csdnimg.cn/side-toolbar/3.6/images/qr_video.png)

关注【CSDN】视频号，行业资讯、技术分享精彩不断，直播好礼送不停！

客服 返回顶部