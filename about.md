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
                'start' => '2014-09-13',
                'end' => '2014-07-01'
            )
        );
    }

    private function company() {
        return array(
            '北京景行至界科技有限公司' => array(
                'start' => '2014-03-17',
                'end' => '2015-02-05',
                'project' => '小声APP(匿名社交)，疯拍(阅后即焚)',
                'work' => 'PHP,Mysql,Redis实现后台管理系统及APP接口'
            ),
            '美丽说网络科技有限公司' => array(
                'start' => '2015-02-27',
                'work' => 'PHP,Mysql商家活动'
            )
        );
    }

}

?>

{% endhighlight %}