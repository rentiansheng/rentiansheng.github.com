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
            '百度外卖' => array(
                        'start' => '2015',
                        'work' => 'PHP,Mysql,Redis,NMQ商户菜品、信息及审核'
                    ),
            '北京景行致界科技有限公司' => array(
                'start' => '2013,11',
                'end' => '2015,03',
                'project' => '小声APP(匿名社交)，疯拍(阅后即焚)',
                'work' => 'PHP,Mysql,Redis实现后台管理系统及APP接口'
            )

        );
    }

}

?>

{% endhighlight %}

