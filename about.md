---
layout: page
title: 关于
---
个人简介!

{% highlight php %}



<?php

new tianshengren();

class tianshengren {

    function  __construct() {
        $me = array(
            'owner' => $this->owner(),
            'education' => $this->education(),
            'company' => $this->company()
        );
        print_r($me);
    }

    private function owner() {
        return array(
            'name' => 'tianshengren',
            'english name' => 'Reage',
            'email' => 'reage521@gmail.com'
        );
    }

    private function education() {
        return array(
            'bachelor degree' => array(
                'name' => '河南科技学院',
                'start' => '2009-09-13',
                'end' => '2014-07-01'
            )
        );
    }

    private function company() {
        return array(
            '美丽说网络科技有限公司' => array(
                        'start' => '2015-02-27',
                        'work' => 'PHP,Mysql商家活动'
                    ),
            '北京景行致界科技有限公司' => array(
                'start' => '2014-03-17',
                'end' => '2015-02-05',
                'project' => '小声APP(匿名社交)，疯拍(阅后即焚)',
                'work' => 'PHP,Mysql,Redis实现后台管理系统及APP接口'
            )

        );
    }

}

?>

{% endhighlight %}


---
#简历详情
---
#任天胜
---
####联系方式 手机:15083128577 [邮箱:reage521@gmail.com](reage521@gmail.com)
####技术相关  [www.ireage.com](www.ireage.com) [csdn博客](http://blog.csdn.net/reage11) [github](https://github.com/rentiansheng)

---

####工作经历

 * 2015-02-27 - 至今 **美丽说（北京）网络科技有限公司**

 	* **购物车价格展示**
 	  1. 实现价格模板创建及与报名活动商品关联的
 	  2. 为APP和网页版在活动预热和活动中提供购物车和单宝页模板
 	* **海景房素材**: 商家根据运营后台配置项目填写或上传素材图，最后生成效果图。
 	  1. 运营提供海景房配置及预览功能
 	  2. 商家海景房合成功能
 	  3. 审核功能
 	  4. 主站提供会馆商家分类



* 2014-03-17 - 2015-02-05 **北京景行至界科技有限公司 PHP工程师**

  * **现场直播**: 基于图片、视频的现场直播功能。
  	1. 页面静态化
  	2. 消息缓存处理
  	3. 视频及图片的轮播展示

  * **疯拍**: 一款基于图片的阅后即焚聊天软件。
  	1. 用户消息管理及后台消息发送
  	2. 网页版模仿表情功能
  	3. 用户通讯录管理功能

  * **小声**: 一款匿名社交类手机软件。
  	1. 为APP提供信息流、私信展示及消息发送。
  	2. 消息敏感词过滤及黑名单
  	3. 实现消息和私信审核、举报信息管理及后台图库管理

---
####教育经历

* 2009.9 - 2014.7 **河南科技学院 计算机科学与技术 本科**

---
####校内项目

* 2010.6 - 2011.10 **通用电子考试系统（C#）**：使用C#基于.NET2.0开发一套winform考试系统，支持科目包含计算机基础，英语。

* 2010.2 - 2010.10 **新乡市金鹊科技有限有公司OA管理系统**：为该公司开发一套管理进货、出货、库存管理、财务。

* 2011.10 – 2012.10 **Reage热键工具（c#）**：基于.NET2.0开发的类似MAC OS 的Docker的一个简单高效的工具栏，支持一键打开程序。[github地址](https://github.com/rentiansheng/reagehotkey)

*  2011.10 – 至今 **ReageHttpd(静态web服务器)** 使用C语言实现的一个简单的HTTP服务器，可以发送静态页面和执行CGI程序。[github地址](https://github.com/rentiansheng/rhttp)

---
####在校期间获得奖项
* 2012 **河南省第一届机器人大赛 微软轮式微型机器人5：5仿真比赛冠军**
* 2012 **ITAT技能大赛C语言  河南省三等奖**
* 2012 **“全国软件设计大赛(C语言本科组)”河南省二等奖**
* 2011 **“瑞萨杯 全国电子设计大赛”河南省三等奖**

---
####个人技能
* 熟悉LNMP环境下开发；
* 熟悉Redis、RabbitMQ开发；
* 熟悉PHP CI框架
* 熟悉.Net平台下winform项目开发，具备项目开发经验，了解ASP.NET开发过程。
* 了解使用gcc，gdb，vim
* 了解C语言网络编程


热爱学习，喜欢富有挑战性的工作。

---




