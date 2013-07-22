---
layout: post
title: "在Linux下让PHP支持MSSQL"
description: "PHP天然就对MySQL有良好的支持，但是想要用PHP对SQL Server进行操作，则需要花点时间了。今天刚好团队里的一个项目需要用PHP对SQL Server进行操作，遂帮忙配置好环境。"
tags: [Front-end]
published: true
---
PHP天然就对MySQL有良好的支持，但是想要用PHP对SQL Server进行操作，则需要花点时间了。今天刚好团队里的一个项目需要用PHP对SQL Server进行操作，遂帮忙配置好环境。

首先说明下，服务器的系统版本为SUSE Linux Enterprise Server 10 SP3。

<h2>1. 安装FreeTDS</h2>
<p><a href="http://www.freetds.org/" target="_blank">FreeTDS</a>是一个让Unix跟Liunx访问SQL Server跟Sybase数据库的库。目前最新版本为0.91，但是我安装的0.82版本。我们先进行它的安装：</p>

<pre class="brush:bash;gutter:false;first-line:1;">
wget http://ibiblio.org/pub/Linux/ALPHA/freetds/stable/freetds-stable.tgz
tar zxvf freetds-stable.tgz
cd freetds-0.82
./configure --prefix=/usr/local/freetds --with-tdsver=8.0 --enable-msdblib --enable-dbmfix
make &amp;&amp; make install
</pre>

成功安装完，最好更新下动态连接库缓存：
<pre class="brush: bash; gutter: false; first-line: 1; ">
echo &quot;/usr/local/freetds/lib&quot; &gt;&gt; /etc/ld.so.conf
ldconfig
</pre>

<h2>2. 配置FreeTDS及连接测试</h2>
FreeTDS的配置文件放在安装目录的etc里，根据第一步的configure参数，我们FreeTDS安装在/usr/local/freetds：
<pre class="brush: bash; gutter: false; first-line: 1; ">
vim /usr/local/freetds/etc/freetds.conf
</pre>
由于不太清楚FreeTDS的具体有哪些可配置项，这里就不深入了，但是提供个比较重要的配置，用来解决中文乱码的问题。在配置文件添加如下语句：
<pre class="brush: text; gutter: false; first-line: 1; ">client charset = utf8</pre>
然后，我们使用tsql命令测试下是否能正常连接上SQL Server数据库：
<pre class="brush: bash; gutter: false; first-line: 1; ">
cd /usr/local/freetds/bin
./tsql -H 192.168.192.71 -p 1433 -U sa -P aHieuW2012
</pre>
正常连接的话应该显示如下语句：
<pre class="brush: bash; gutter: false; first-line: 1; ">
locale is &quot;zh_CN.UTF-8&quot;
locale charset is &quot;UTF-8&quot;
1&gt; 
</pre>

<h2>3. 安装php的mssql扩展</h2>
服务器上的php版本为5.3.13，php已安装在/usr/local/services/php下，扩展的目录为/usr/local/services/php/extensions。下面是安装mssql扩展的方法：
<pre class="brush: bash; gutter: false; first-line: 1; ">
cd php-5.3.13/ext/mssql/
/usr/local/services/php/bin/phpize
./configure --with-php-config=/usr/local/services/php/bin/php-config --with-mssql=/usr/local/freetds
make #生成扩展文件，放在当前目录的module文件夹下
cp modules/mssql.so /usr/local/services/php/extensions/ #把扩展文件复制到PHP的扩展目录下
</pre>

<h2>4. 配置php.ini并验证安装结果</h2>
打开php.ini，添加如下扩展语句：
<pre class="brush: text; gutter: false; first-line: 1; ">extension=mssql.so</pre>
重启PHP服务后（服务器用的是php-fpm），打印phpinfo，出现如下配置则代表php能正常操作SQL Server了。

<img src="/images/2013/04/php_ext_mssql.png" alt="" title="php_ext_mssql" width="606" height="488" />

P.S. 此文章发表于2012年7月13日，由旧Blog转移过来。