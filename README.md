# EK的 甜糖 使用经验

（我是Edna K，不过我乱起了个github名称，就这样把。哈哈。欢迎大家来我的电报群玩：https://t.me/shadowrocket_unofficial）

（看完成功用上甜糖的，别忘了 在甜糖 app里，填写我的邀请码 ：345520，甜糖给你10%的星愿加速，很合适）



这个是一家新的边缘计算公司，2020年6月才出现。

目前没有x86固件，只有路由器插件和安卓应用。docker的话只能用上面教程的办法。

据说30M上传带宽跑满一天有2元收入。

我先用我的nova手机试一下。

之前手机内存不够，其要求储存空间大于32G，我清理了一下。容量够了后，再次连上，就会显示“优质节点”。

后来在虚拟机上安装了docker的ttnode。根据说明，我在光猫开启了Upnp。然而开启后，似乎光猫的UPnP功能就失效了，甜糖显示网络不正常。没招。

	
esxi 甜糖教程
	https://www.right.com.cn/forum/thread-4057352-1-1.html
	
	这里我分区时，无法按回车执行默认分区，需要按p，回车，再输入1，然后再继续操作。
	
	我用的40G的esxi里的虚拟磁盘。
	
	chmod时 +777报错，777行。
		关于chmod
			https://www.runoob.com/linux/linux-comm-chmod.html
			
	
	运行
	./usr/node/ttnode -p /mnts 时，报
	node [8715fc923550c5f76ca3536905691b00,/mnts] exist,error = 11 错误
	
	但是还是产生了uid。不知道行不行。
	
	虚拟机名为 ttnode, ip 192.168.0.120, 用户名密码都是root
	
	看了，里面用的
		qemu-aarch64-static
		
	
	然而我发现配置了此ttnode后，手机上显示nova的网络变成未配置网络，然后网络类型为4，评级变为12.
	
		看网上信息，似乎它俩不冲突。不知为何会这样。
		
		
另外发现这个ttnode里的apk已经换好源了
	vi /etc/apk/repositories 即可查看源。
	
	我觉得可以用这个系统进行轻度作业。
	
	我安了 file, curl, vim（删除了原来指向busybox的vi，新创建vi指向vim），nodejs, npm， man-page，man-doc
	

甜糖定时获取星星
	https://github.com/744287383/AutomationTTnode
	
	好像必须每天登陆app收取星星。
	
	https://www.right.com.cn/FORUM/thread-4034075-1-1.html
	
	每天凌晨2点才计算生成了多少星星。
	Openwrt也有自动收集插件，luci-app-ttnode


		
提供的加速卡
	http://www.chinadsl.net/thread-168038-1-1.html
	
	
	
我是从第一天的凌晨试的，从凌晨到早8点基本不会产生缓存；从8点到12点产生了1G缓存；
	
	晚六点查看依然是1G，但是整机功率变为了60w（原来50w）。
	晚七点查看变为2Ｇ

	使用esxi的网页端查看，发现果然，ttnode的cpu占用率为24%
	
	网络使用情况，果然，上传带宽开始全部被占用，为40Mbps
	
	esxi的performance查看只能查看最近一个小时的情况
	
	所以我准备时机成熟时，关闭一下ttnode，然后多给他分配一个cpu核心，看看耗电情况会不会改善。

我个人还有个疑问。这个dmz是不是不一定非要启动？因为我有ipv6


我看了，它主要是一个32位的程序，叫ttnode。
	代码在 https://github.com/898811295/TTarm32
	
	它主要是通过 cron，每分钟运行一次 crash_monitor.sh
		然后如果观察到ttnode未运行，则运行ttnode
		
		TT272440.sh 和 272440.sh 都是用于安装的脚本，安装后会自动被删掉。
		
	它使用挂载到 /mnt/ttnode 的磁盘。里面存储其上传所需的缓存。
	
	它在第一次运行docker时，使用了 --restart=always，使得该容器可以在docker重启后自动启动。而docker的服务已经配置好了，运行service docker status就可以看到。
	
		所以开启后，docker先自动运行，然后内部的arm容器自动运行，然后cron自动运行，最后运行ttnode。
		
	这里有拉取docker的教程
		http://cdn.ssr0.cn:8000/archives/384.html
		
		就是先拉qemu的，后拉ttnode的。很明显，ttnode是建立在qemu层之上的。
		
	
		https://www.right.com.cn/forum/forum.php?mod=viewthread&tid=4052765
		
	至于甜糖的docker容器是如何制作的，没有教程。
	
	
	
	
提现规则
	每周三中午12点前提现的金额，周四统一到账，扣6%增值税
	
	每日签到获得2星愿
	
	星愿七天不采集，每天衰减10%
	
	App里有个活动，我领了个30天的“电费卡”，说是每天50星愿保底收益。
	
	100颗星星等于1RMB，满10RMB可提现
	
	
这个是用的docker+ qemu的方式。
我担心qemu效率太低，搜了一下，发现kqemu这个东西


关于qemu 和docker

	https://blog.csdn.net/ccgshigao/article/details/109631585
	
	
第二天两点后更新了，获得92星；缓存依然是2Ｇ. 第二天早八点半，缓存依然是2G，而且已经可以看出，流量有很明显的晚高峰特征。除了晚高峰以外，几乎不上传流量，cpu也不占用。

我重启了一下ttnode，发现uid更换了，iphone的app上会显示发现新设备，然后重新绑定。
	不过缓存和数据都还在。
	
	而旧的节点就会显示“获取信息中”，看来旧的id已经不可用，可以删掉。
	

第二天晚5点四十查看，发现缓存变为3G

给ttnode多分配一个核心后，总占用率变为12.5%，也就是说实际上效率没有变化。

第三天早七点四十查看，获得星愿119，缓存变为4Ｇ。
第三天（2020.12.16）下午三点，缓存变为5Ｇ，晚10点，缓存变8G，晚11点，缓存变9G，且节点评级变为1（原来节点评级一直为3）.


后来发现ttnode里的 crontab 出问题了，crontab -e 命令无法修改，就是说无论里面做了什么改动都不会生效。不过修改 /var/spool/cron/crontabs/root 文件是可以的，用crontab -l 能成功看到变化。

	我个人怀疑可能是我把vi 的软连接改成了 vim的缘故
	
	试了，果然，把vi重新指向busybox, crontab -e就可以正常保存了。
	
	总之alpine还是太奇葩。
	
后来猛然想起，开了DMZ后外网就可以接入我的ttnode虚拟机（密码太简单），很危险。所以就将ttnode的密码修改,改为较强的7位密码


第四天中午我断线1小时，用来测试宽带多拨，失败。再开机，发现每次ttnode上网被识别需要15分钟时间。且过两小时发现，显示的在线率变为92%，节点评级变为5。缓存变10Ｇ

第四天晚，我用我妈手机号注册了甜糖，绑定了我妈的手机号和微信号。同时填入了我主号的邀请码，应该能给我主号增加收益吧。

	后来才反应过来，它是获得所推广的好友的星愿收益的5%，所以自己还真没法刷，除非拿我手机登我妈的甜糖天天挂着。
	
这个甜糖之所以好玩，就在于它既可以用路由器，又可以用docker，还可以用手机。比一般的入门门槛低。

第五天早，收集188星愿。第五天，离上次关机24小时后，果然在线率变为100%，节点评级也回归1


