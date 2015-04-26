---
layout: post
title: php extensions(二) phpinfo输出内容
category: [php extension,c,开源]
tags: [php extension,c,开源]
---

### php extensions(二)  phpinfo输出内容

#### 引言

  我们在使用一个PHP扩展的时候，首先，将编译后的扩展放到PHP的扩展加载目录下，使用phpinfo函数来查看插件是否已经加载。


#### 代码实现

   在phpinfo函数添加内容特别简单的。
   1.首先修改module entry

   {% highlight php %}
         zend_module_entry reage_module_entry = {
             STANDARD_MODULE_HEADER,
             "reage",//扩展的名字
             NULL, // functions
             NULL, // minit
             NULL, //mshutdown
             NULL, //rinit
             NULL, //rshutdown
             PHP_MINFO(reage_info), //注册在phpinfo函数中输出内容的函数,reage_info
             "0.0.1", //版本号
             STANDARD_MODULE_PROPERTIES
         };
   {% endhighlight %}

   2.输出内容函数的实现

   {% highlight php %}
        PHP_MINFO_FUNCTION(reage_info) {
           php_info_print_table_start();
           php_info_print_table_header(2, "key", "value");
           php_info_print_table_row(2, "author", "Reage");
           php_info_print_table_end();
        }
   {% endhighlight %}

#### 使用到的函数
    php_info_print_table_start();


    php_info_print_table_start();
    php_info_print_table_header();
    php_info_print_table_row();
    php_info_print_table_end();

    除了这些函数，还有很多函数，需要的话请自行查阅

#### 特别说明

在搜索资料时，发现一个写的特别好的php扩展开发系列文章，希望对大家有用

[PHP扩展开发及内核应用](http://www.walu.cc/phpbook/index.md)
