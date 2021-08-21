---
layout: mypost
title: 使用GCP进行BBRPlus和SSR的使用
categories: [科学上网]
---

# 阅读本文的前提  

1.你已经拥有一个Google账户；
2.能够正常访问google.com域名的上网工具；
3.已经注册GCP；
4.已绑定信用卡并获得新注册用户300刀的赠送余额。

## 一、创建实例

登录GCP平台的控制面板：[GCP](https://console.cloud.google.com/)    

打开侧边栏，找到**Compute Engine**，在列表里找到**VM实例**并单击选择；    

![1.jpg](1.jpg)   

GCP平台的实例，都是依附于对应的项目，所以我们需要先新建一个实例，项目名随你的喜好，当然了，首次创建项目建议区域选择香港、台湾或者新加坡，地区随你的选择，随后的机器类型我们选择小型或者微型都可以，足够我们进行使用了（如果你是土豪，当我没说，随你喜好），启动磁盘选择Debian9最好（个人认为），再其次的防火墙记得将允许HTTP流量和允许HTTPS流量勾选，这是很重要的一环。其他没有提及的一些配置不要碰！   

新创建的实例如下：   

![2.png](2.png)   

## 二、检查实例   

在我们创建实例之后需要对外部IP进行检测:[IPIP](ipip.net)   

![3.png](3.png)   

选择高亮的栏目，之后   

![4.png](4.png)   

在IPv4选择距离自己较近的地址，ICMP输入我们刚才得到的外部IP。在时间一列中显示都在100以下我们就可以进行下一步了，否则我们继续创建新的实例，以得到符合我们心意的实例；   

## 三、SSH配置（BBRPlus和SSR）

接着我们在VM实例里点击SSH,当出现命令行界面时，我们需要输入：   

`sudo -i`    


`wget "https://github.com/cx9208/Linux-NetSpeed/raw/master/tcp.sh" && chmod +x tcp.sh && ./tcp.sh`

等待一段时间之后：   

![5.png](5.png)   

我们输入相对应的数字，来安装我们的BBRPlus加速。如果提示证书错误的话： 

```    
apt-get -y install ca-certificates   
yum -y install ca-certificates   
```      

我们首先选择1-3切换内核(第一次显示为BBR内核我们也需要更换一遍)，之后会出现：   
![6.png](6.png)   

选择NO   

之后直接`./tcp.sh`,接着我们继续选择你要的加速   

开启之后再`./tcp.sh`,显示成功则成功开启。   

随后我们需要进行SSR的配置：   

   
~~wget –no-check-certificate -O shadowsocks-all.sh https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks-all.sh   
chmod +x shadowsocks-all.sh  
./shadowsocks-all.sh 2>&1 | tee shadowsocks-all.log~~ 此代码已失效，于2019.10.31更新为

`wget --no-check-certificate https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocksR.sh&&chmod +x shadowsocksR.sh`

  

等待一段时间之后输入

`./shadowsocksR.sh`

设置端口、密码、加密方式等等   
出现提示信息，选择要安装的版本，推荐ShadowsocksR，输入`2`,回车   

![7.png](7.png)   

设置密码   

![8.png](8.png)   

设置端口   

![9.png](9.png)   

选择加密方式，建议选择aes-256-cfb   

![10.png](10.png)   

选择混淆方式，建议选plain   

![11.png](11.png)   

然后按任意键开始安装，耐心等待一下，安装所需时间根据网速，机器配置等有所差别，出现类似下图窗口则安装完成，图中显示的就是你的SSR的IP, 端口, 密码, 协议, 混淆, 加密方式等连接信息, 用小本本记下来。   

## 四、设置防火墙   

1.左侧导航-计算-网络   

![12.png](12.png)   

2.外部IP地址 – 选择一个ip – 类型调整为静态   

![13.png](13.png)   

3.防火墙规则 – 创建防火墙规则（未提及的全部默认）：流量方向`入站`、来源ip地址`0.0.0.0/0`、协议和端口'全部允许'    

4.防火墙规则 – 创建防火墙规则（未提及的全部默认）：流量方向`出站`、来源ip地址`0.0.0.0/0`、协议和端口'全部允许'**注意要创建两次防火墙规则，一次出站，一次入站**

以上就是我们使用GCP的过程。
