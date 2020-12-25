# EK的 甜糖 使用经验

（我是Edna K，不过我乱起了个github名称，就这样把。哈哈。

（看完成功用上甜糖的，别忘了 在甜糖 app里，填写我的邀请码 ：345520，甜糖给你10%的星愿加速，很合适。

这个填写也会给我加速的，所以希望大家觉得有用的，先填写我的邀请码，谢谢啦）

我会持续更新此博客，如果你希望观察一段时间，可以watch或star一下这篇文章。


## 简介

这个是一家新的边缘计算公司，2020年6月才出现。官网链接：http://m.tiantang.mogencloud.com/

目前没有x86固件，只有路由器插件和安卓应用。docker的话只能用下面教程的办法。

据说30M上传带宽跑满一天有2元收入。请接着往下看，了解我赚了多少。（从2020年12月13日开始）

## 手机安装（第0天，2020.12.13）

我先用我的nova手机试一下。进入上面链接，微信扫码，即可安装。手机有两个app，一个用于监视所有的节点，一个是运行节点的app，如果你要在手机上运行节点上传数据，则都要下。按官方教程操作即可。

之前手机内存不够，其要求储存空间大于32G，我清理了一下。容量够了后，再次连上，就会显示“优质节点”。

根据说明，用手机时，我在光猫开启了Upnp。后来在虚拟机上安装了docker的ttnode，又在光猫开启了DMZ，然而开启后，似乎光猫的UPnP功能就失效了，甜糖显示网络不正常。没招，只好直接用docker。

	
## 各种docker
https://hub.docker.com/search?q=ttnode&type=image

关于docker，我建议使用在hub.docker.com能查到的，更新频繁的，能显示dockerfile的。因为这样公开透明，更新好支持好。

我安装的那个esxi的虚拟机镜像，日后使用，感觉不是特别顺手，如果你不动电脑的话，可以。但是如果你懂，目前我认为这个docker比较好：

https://hub.docker.com/r/ericwang2006/ttnode


## esxi docker 甜糖教程 （直接在电脑上按docker也可以，不会的加群问我）

https://www.right.com.cn/forum/thread-4057352-1-1.html
	
	这里我分区时，无法按回车执行默认分区，需要按p，回车，再输入1，然后再继续操作。
	
	我用的40G的esxi里的虚拟磁盘。
	
	chmod时 +777报错，777行。
		关于chmod
			https://www.runoob.com/linux/linux-comm-chmod.html
			
	
	运行
	./usr/node/ttnode -p /mnts 时，报
	node [8715fc923550c5f76ca3536905691b00,/mnts] exist,error = 11 错误
	
	但是还是产生了uid。不知道行不行。经后来测试，没问题，能正常使用。
	
	产生这个错误的原因是因为 ttnode之前已经启动过一遍（自启动），然后因为这次为了获取uid又启动了一遍，所以它就提示了一下。
	
	虚拟机的用户名密码都是root
	
	看了，里面用的
		qemu-aarch64-static
		
	
	然而我发现配置了此ttnode后，手机上显示nova的网络变成未配置网络，然后网络类型为4，评级变为12.
	
		看网上信息，似乎它俩不冲突。不知为何会这样。
		
		
另外发现这个ttnode里的apk已经换好源了
	vi /etc/apk/repositories 即可查看源。
	
	我觉得可以用这个系统进行轻度作业。
	

### 甜糖定时获取星星（Python+server酱微信推送）
	https://github.com/744287383/AutomationTTnode
	
	好像必须每天登陆app收取星星。
	
	https://www.right.com.cn/FORUM/thread-4034075-1-1.html
	
	每天凌晨2点才计算生成了多少星星。
	
### Openwrt也有自动收集插件，luci-app-ttnode, 链接在： https://github.com/jerrykuku/luci-app-ttnode
	
	
	
我是从【第一天】（2020.12.14，周一）的凌晨试的，从凌晨到早8点基本不会产生缓存；从8点到12点产生了1G缓存；
我家的宽带是联通200Mbps，上传30Mbps，具有动态公网ipv6地址。
	
	晚六点查看依然是1G，但是整机功率变为了60w（原来50w）。
	晚七点查看变为2Ｇ

	使用esxi的网页端查看，发现果然，ttnode的cpu占用率为24%
	
	网络使用情况，果然，上传带宽开始全部被占用，为40Mbps
	
	esxi的performance查看只能查看最近一个小时的情况
	
	所以我准备时机成熟时，关闭一下ttnode，然后多给他分配一个cpu核心，看看耗电情况会不会改善。

我个人还有个疑问。这个dmz是不是不一定非要启动？因为我有ipv6。但是为了保险我还是启用了。


## 我看了，它主要是一个32位的程序，叫ttnode。
	代码在 https://github.com/898811295/TTarm32
	
	它主要是通过 cron，每分钟运行一次 crash_monitor.sh
		然后如果观察到ttnode未运行，则运行ttnode
		
		TT272440.sh 和 272440.sh 都是用于安装的脚本，安装后会自动被删掉。
		
	它使用挂载到 /mnt/ttnode 的磁盘。里面存储其上传所需的缓存。
	
	它在第一次运行docker时，使用了 --restart=always，使得该容器可以在docker重启后自动启动。
	
		而docker的服务已经配置好了，运行service docker status就可以看到。
	
		所以开启后，docker先自动运行，然后内部的arm容器自动运行，然后cron自动运行，最后运行ttnode。
		
	这里有拉取docker的教程
		http://cdn.ssr0.cn:8000/archives/384.html
		
		就是先拉qemu的，后拉ttnode的。很明显，ttnode是建立在qemu层之上的。
		
	
		https://www.right.com.cn/forum/forum.php?mod=viewthread&tid=4052765
		
	至于甜糖的docker容器是如何制作的，没有教程。
	
	
	
	
## 提现规则
	每周三中午12点前提现的金额，周四统一到账，扣6%增值税
	
	每日签到获得1-2星愿
	
	星愿七天不采集，每天衰减10%
	
	App里有个活动，我领了个30天的“电费卡”，说是每天50星愿保底收益。
	
	100颗星星等于1RMB，满10RMB可提现
	
	
### 这个是用的docker+ qemu的方式。
	我担心qemu效率太低，搜了一下，发现kqemu这个东西


### 关于qemu 和docker

	https://blog.csdn.net/ccgshigao/article/details/109631585
	
## 经验分享继续：
	
**【第二天】** （12.15，周二）两点后更新了，获得92星；缓存依然是2Ｇ. 第二天早八点半，缓存依然是2G，而且已经可以看出，流量有很明显的晚高峰特征。除了晚高峰以外，几乎不上传流量，cpu也不占用。

我重启了一下ttnode，发现uid更换了，iphone的app上会显示发现新设备，然后重新绑定。
	不过缓存和数据都还在。
	
	而旧的节点就会显示“获取信息中”，看来旧的id已经不可用，可以删掉。
	

**【第二天】** 晚5点四十查看，发现缓存变为3G

给ttnode多分配一个核心后，总占用率变为12.5%，也就是说实际上效率没有变化。

**【第三天】** 早七点四十查看，获得星愿119，缓存变为4Ｇ。
第三天（2020.12.16）下午三点，缓存变为5Ｇ，晚10点，缓存变8G，晚11点，缓存变9G，且节点评级变为1（原来节点评级一直为3）.


后来发现ttnode里的 crontab 出问题了，crontab -e 命令无法修改，就是说无论里面做了什么改动都不会生效。不过修改 /var/spool/cron/crontabs/root 文件是可以的，用crontab -l 能成功看到变化。

	我个人怀疑可能是我把vi 的软连接改成了 vim的缘故
	
	试了，果然，把vi重新指向busybox, crontab -e就可以正常保存了。
	
	总之alpine还是太奇葩。
	
后来猛然想起，开了DMZ后外网就可以接入我的ttnode虚拟机（密码太简单），很危险。所以就将ttnode的密码修改,改为较强的密码


**【第四天】** 中午我断线1小时，用来测试宽带多拨，失败。再开机，发现每次ttnode上网被识别需要15分钟时间。且过两小时发现，显示的在线率变为92%，节点评级变为5。缓存变10Ｇ

**【第四天】** 晚，我用我妈手机号注册了甜糖，绑定了我妈的手机号和微信号。同时填入了我主号的邀请码，应该能给我主号增加收益吧。

	后来才反应过来，它是获得所推广的好友的星愿收益的5%，所以自己还真没法刷，除非拿我手机登我妈的甜糖天天挂着。
	所以我希望通过写教程的方式，能让大家也用上甜糖。
	
这个甜糖之所以好玩，就在于它既可以用路由器，又可以用docker，还可以用手机。比一般的入门门槛低。

**【第五天】** 早，收集**188星愿（第四天131，第三天119，第二天92，第一天0）**。第五天，离上次关机24小时后，果然在线率变为100%，节点评级也回归1
第五天晚，缓存变为11G。

**【第六天】** （周六）早，收集**226**星愿，终于超过2元了。第六天上午十点，缓存变为12G。下午一点，缓存变13Ｇ。下午5点，变为14G。
可以看出，最快时，大概4小时能增加1G的缓存

# 星愿减少事件

**【第七天】** （周日）早四点，收集**176**星愿（不知为何，我一直开着的）, 缓存16Ｇ；晚上变为19Ｇ。

**【第八天】** (12.21，周一)早八点，收集星愿**129**，缓存25Ｇ，节点评级从1变成2。开始不香了。真是奇怪。我明明一直开机的。

我现在开始怀疑，是不是我添加了一个循环脚本导致的。正好在第六天，我开始添加了一个循环脚本，用于其他目的，也许它占用的cpu和磁盘使用率太高了。我现在把它关一下，再做观察

**【第九天】** 早（周二），收集星愿**64**，缓存27G，节点评级变为1，凉了。已证明不是其他问题，就是甜糖有鬼。交流群里也纷纷表示收益缩水。难道说甜糖杀熟？昨天前天还能跑满带宽的二分之一，今天只有三分之一。就算工作日缩水也说不通，因为我从周日早上开始就缩水。

另外也证明了，如果上传流量低，则下载缓存的速度也会跟着减慢。

另外，openwrt的自动采集星愿插件似乎有问题，无法保存设置，点击保存设置没反应，而刷新页面后之前的设置又被重置。今天我的星愿是自己采集的，它没能采集成功。

上午，我实在忍不住，决定做一些变化。将ttnode重启。重启后惊奇发现，uid依然没改，缓存竟然减少到24G，目前内存占用率才13%（内存2G，平时占用一般50%以上）。观察观察，会不会改善状况。目前我的排名也在下滑，最厉害时进入前1000，现在才2000多名，说明可能我这里也有问题，或者说更多的大鳄挤进了排名，抢走了我们的流量。

经观察，上述重启ttnode的操作没有改变任何现状，cpu使用率依然低迷，证明上传也很低迷。

**第九天晚八点半，无意中发现，自己家的电脑虽然具有ipv6地址，但是连不上ipv6网络！难道说这就是星愿减少的原因？**

**我立即重启了光猫，果然，重启后，光猫重新分配了新的ipv6地址，能连上ipv6了！这回再看一看后天得到的星愿吧。（明天是指不上了，因为算的是今天的，而今天已经到了晚上了我才反应过来。——）**

目前感觉确实是ipv6的问题。我未重启光猫前，ttnode的cpu使用率一直为0，但是重启光猫后，ttnode虽然获得了新的ipv6，cpu使用率还是0. 再重启ttnode后，cpu使用率恢复高上传时的使用率！

	这同时也证明，我家只有ipv6时有公网的，ipv4可能大内网都算不上，所以就算开了DMZ也没个屁用（或者只有“一个屁”的作用）。下回我有机会关掉DMZ试试，按我的逻辑应该没有任何区别。


**【第十天】** 凌晨2点，蹲点收**102**星愿。缓存26G，内存占用13%。个人认为，可能它所说的缓存是指已经存入硬盘的大小加上内存中的缓存的大小。所以才导致了上次重启后缓存大小反而减小的情况。而且之前内存占用一直50%多现在却保持在低位，确实有点意思。早9点，观察甜糖里的网速条形图，可以看到网速巨大地提升，可见ipv6有多重要。

不过仍然觉得不保险，正在学习一些方法，看看能不能抓包啥的看看是不是真的是在使用ipv6.

然而使用netstat -nape 发现，都是ipv4的连接，也不知道是netstat只能显示ipv4还是怎样。

*其实还有一种可能，就是光猫的DMZ之前也出现问题了，然后重启光猫后DMZ恢复了所以甜糖又好了，也不知道到底咋回事。光猫的DMZ是只针对ipv4的。*

*或者还有一种可能，就是光猫把ipv4的数据通过ipv6发送了？个人完全瞎猜*

# Netstat 探索

使用netstat -nape 或者 -np, 可以发现，甜糖使用ipv4的31121端口，通过tcp协议，向各个未知的ip地址的随机端口发送大量数据。

不用n参数时，还意外获取到了 adsl-pool.jlccptt.net.cn这个域名。

个人认为，netstat显示的不全. 毕竟我是用的这个esxi的虚拟机是个alpine的，用的busybox，而busybox所实现的netstat功能并不完整。

然而用apk add 了 iproute2后调用ss命令，还是只显示了这些ipv4点连接。

还有一个问题，会不会创建docker的人本身就只开启了ipv4？我不是很懂，但是只要不是掌控在自己手里，就可能是问题的来源。

另外我还不满的一点就是，上面教程提供的虚拟机为alpine, 但是里面docker又是一个ubuntu。（后来发现，是因为由于alpine的libc库有问题不能跑ttnode。但是那也可以用debian啊，ubuntu尺寸多大）

我在docker内部试验了，docker内部是能ping6通国际ipv6地址的。看来基本已经确认，ttnode使用的是ipv4. **那如此说来我光猫的问题出在DMZ故障，反正光猫垃圾就是**

其实到底是ipv6问题还是dmz问题非常好验证，只要关闭dmz，看看流量是不是还是那么大就行了。如果赚的星愿一样多，那肯定就是ipv6. 不过既然现在已经好了，不妨再等到下一次出问题再做尝试。

**不过最蹊跷的就是，如果真的是DMZ出问题了，那我的节点评级一定会变的很差，而当时我依然被判断为优质节点，节点评级在1～3之间徘徊。所以我依然保留【ttnode使用ipv6】的想法。**


# 日志继续

**【第11天】** 早八点，收集260星愿。我昨天一整天，除了凌晨1-3点没有量以外，都是满速运行，直到昨天晚上我关机出问题。但是还是出了个问题。昨晚我关闭了一下软路由，安装一下硬件啥的，但是发现自从重新开机后，流量迅速下滑，cpu使用率也一直是0。 现在似乎明了了一点，就是凡是直接在esxi中poweroff关机的，都会导致ttnode在下次开机时没流量。因此下一次我关机试一下先用shutdown命令。

此次没流量一直从昨晚大概我重新开机开始，到今晨8点收星愿止。因此可以说应该是关机导致的。

同样，此次，我先重启光猫，待光猫稳定后，再重新开机后，ttnode并没有自动恢复cpu使用率。我再次重启了ttnode虚拟机后依然不行。成功的做法是重启一下docker服务。不过这有待多次测试以验证。

这次与上次不同的是，这一次ipv6并未出现故障，也就是说，重启前，内网是可以ping6通国际ipv6地址的。因此这进一步证明很可能问题不是出在ipv6上。

**【第12天】** （12月25日）早八点，收集**190**星愿，在意料之中，毕竟昨天凌晨0点到早上八点一直没流量。缓存依然为26G，缓存都好几天没变了，比较奇怪。（我存储大小总共为40G，但是因为某种原因显示为39G，难道说甜糖喜欢只占用三分之二的空间？）。

另外今天甜糖出活动，双旦期间天天可以抽奖，双旦当天还可以在微信抽加速卡。

目前凌晨1-4点、下午2-4点，这两个时段流量会下降，其余时刻我都是在满速运行。目前的总结就是不要随便关机ttnode，如果关机了，再开机，有几率ttnode流量大减，结果就是像我之前的操作一番才能恢复。目前决定放置ttnode一周不动，观察观察。

另外，**最近这几次重启，uid都没有变化**。所以我认为，可能缓存达到很高（比如10G以上）后，uid就不会随意改变了。不过这只是目前的结论，有待进一步考察。

**第12天下午5点，缓存变27G，发现还是没流量，感觉不符合规律，忍不了，又重启了一遍，这次我没重启光猫。发现重启没反应。晚7点，还是没流量，缓存变28G。**我很诧异，为何一直没流量呢？**难道说必须要重启光猫吗？**因为前两次我确实都把光猫重启了一遍。不过还有一个现象值得注意，那就是这回缓存又开始增长了。**难道说ttnode时不时抽风一下，不让你走量，而光给你增加缓存？** 不过其实也不是完全没有流量，只是上传网速大概3Mbps每秒（甜糖客户端所显示的，没准还是下载用的，而不是上传的），根本不符合规律。

当然还有一种可能，那就是甜糖比较坏，看到我领了加速卡后，故意不给我量，以达到省钱的目的。

为了解决上述疑问，主要是要验证【是否甜糖具有 **"缓存期"** 和 **"上传期"** 这两种模式】，我还是决定**观察一段时间**。如果两天后，还是处于可疑的 **"缓存期"** ，那我还是要重启光猫。另外，此时甜糖依然显示我为优质节点，节点评级1.



### 关于流量变化

另外，从大概第四天起，我的甜糖从凌晨一直到早上，也都开始有流量上传。现在全天都有流量上传。

目前一般流量从晚22点开始下跌，到凌晨一点到凌晨四点为低谷，大概为最高带宽的三分之一，凌晨四点后开始上升，升到早7点有一个小高峰。中午12点还有一个小高峰。


# 其他

有时ttnode重启后uid会变化，但不一定。

获得uid方法：
	
	进入容器
	docker exec -it  ttnode /bin/bash
	运行
	./usr/node/ttnode -p /mnts
	
	
类似甜糖的产品：网心云，京东云无线宝

网心云也可以跑在各种平台
	https://post.m.smzdm.com/p/az5gp4zn/
	
	不过网心云要求的存储空间太高，推荐要480Ｇ。
	
	还是甜糖门槛低
	
最近甜糖还推出了电费卡，只有一个月时间，每天保底50星愿，具体在app中间“星愿”那个tab里，点那个聊天气泡的圆形按钮，里面有个通知，点进去，里面会告诉你怎么获取。就是要关注个甜糖的公众号，回复一个内容就行。然后再回到甜糖app，点那个礼物图标的按钮，就能找到电费卡，点击启用。

双旦期间天天可以抽奖，双旦当天还可以在微信抽加速卡。

