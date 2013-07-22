---
layout: post
title: "【PHP】RabbitMQ for PHP on Linux"
description: '
RabbitMQ的安装以及PHP的RabbitMQ模块的安装。
'
tags: [Front-end]
published: false
---
第一步，先安装<a href="http://www.erlang.org/">Erlang</a>。

<pre class="brush:bash;">
wget http://www.erlang.org/download/otp_src_R16B01.tar.gz
tar zxvf otp_src_R16B01.tar.gz
cd otp_src_R16B01/
./configure --prefix=/usr/local/services/erlang
make
make install

ln -s /usr/local/services/erlang/bin/erl /usr/local/bin/erl
</pre>

然后安装<a href="http://www.rabbitmq.com/">RabbitMQ</a>，这里安装的是3.1.3版本。

<pre class="brush:bash;">
wget http://www.rabbitmq.com/releases/rabbitmq-server/v3.1.3/rabbitmq-server-generic-unix-3.1.3.tar.gz
tar zxvf rabbitmq-server-generic-unix-3.1.3.tar.gz
cp -r rabbitmq_server-3.1.3 /usr/local/services/rabbitmq
</pre>

如果没有安装Erlang，启动会报错：

<pre class="brush:bash;">
./rabbitmq-server
./rabbitmq-server: line 85: erl: command not found
</pre>

正常启动的话，会显示如下的信息：

<pre class="brush:bash;">
# ./rabbitmq-server 

              RabbitMQ 3.1.3. Copyright (C) 2007-2013 VMware, Inc.
  ##  ##      Licensed under the MPL.  See http://www.rabbitmq.com/
  ##  ##
  ##########  Logs: /var/log/rabbitmq/rabbit@linux.log
  ######  ##        /var/log/rabbitmq/rabbit@linux-sasl.log
  ##########
              Starting broker... completed with 0 plugins.
</pre>

OK。接下来安装RabbitMQ的客户端库，让Linux能访问RabbitMQ。我们这里用的是0.3.0版本的<a href="https://github.com/alanxz/rabbitmq-c/">rabbitmq-c</a>。

<pre class="brush:bash;">
wget https://github.com/alanxz/rabbitmq-c/archive/rabbitmq-c-v0.3.0.zip
unzip rabbitmq-c-v0.3.0
cd rabbitmq-c-rabbitmq-c-v0.3.0/
autoreconf -i &amp;&amp; ./configure &amp;&amp; make &amp;&amp; sudo make instal
</pre>

接下来我们安装PHP的<a href="http://pecl.php.net/package/amqp">AMQP</a>扩展，版本为1.2.0:

<pre class="brush:bash;">
wget http://pecl.php.net/get/amqp-1.2.0.tgz
tar xvf amqp-1.2.0.tgz
cd amqp-1.2.0/
/usr/local/services/php/bin/phpize
./configure --with-php-config=/usr/local/services/php/bin/php-config --with-amqp
</pre>

报错：

<pre class="brush:bash;">
......
checking for amqp files in default path... not found
configure: error: Please reinstall the librabbit-mq distribution
</pre>

把刚才解压的rabbitmq-c里的amqp_framing.h文件复制到/usr/local/include/里:
<pre class="brush:bash;">
cp path-to-rabbitmq-c/librabbitmq/amqp_framing.h /usr/local/include/
</pre>

顺利完成configure，但是make报错：

<pre class="brush:bash;">
make: *** [amqp.lo] Error 1
</pre>

把还是刚才解压的rabbitmq-c里的librabbitmq.la文件复制到/usr/local/include/里:

<pre class="brush:bash;">
cp path-to-rabbitmq-c/librabbitmq/librabbitmq.la /usr/local/include/
</pre>

继续make，继续报错：

<pre class="brush:bash;">
make: *** [amqp.la] Error 1
</pre>

还是继续copy文件，这次要复制的是librabbitmq.so，但是目标文件夹是/usr/local/lib：

<pre class="brush:bash;">
cp path-to-rabbitmq-c/librabbitmq/.libs/librabbitmq.so /usr/local/lib
</pre>

终于不报错了，顺便完成make和make install：

<pre class="brush:bash;">
Installing shared extensions:     /usr/local/services/php/lib/php/extensions/no-debug-non-zts-20100525/
</pre>

重启php-fpm，报错：

<pre class="brush:bash;">
<b>Warning</b>:  PHP Startup: Unable to load dynamic library '/usr/local/services/php/extensions/amqp.so' - librabbitmq.so.1: cannot open shared object file: No such file or directory in <b>Unknown</b> on line <b>0</b><br />
</pre>

还是继续复制大法：

<pre class="brush:bash;">
cp path-to-rabbitmq-c/librabbitmq/.libs/librabbitmq.so.1 /usr/local/lib
</pre>

搞定：

<img src="/images/2013/07/php_amqp.jpg" alt="" />