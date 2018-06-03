---
layout: post
title: DUTCTF——Web
categories:
  - ctf
  - 安全
url: 1.html
id: 1
comments: false
date: 2015-04-22 07:06:18
tags:
---

首先声明一下，这不是writeup，如果想看writeup，请移步[DUTCTF writeup](http://bobao.360.cn/ctf/learning/129.html)，本文主要介绍作为一个CTF小白混迹DUTCTF比赛的经历。 DUTCTF是大连理工的第一届CTF，是大连理工的内部比赛，后来吸引了一批外来人口也参与到其中，我也是其中之一，虽然外校的没有奖品，但是涨涨姿势也是好的。 ==============以上纯属废话，下面进入正题===================== Web安全是我接触时间比较长的，因此首先我去看了一下Web方向的题目 Web 10pt DJ昨天才搭的网站，今天就被撸了，发现了一个一句话木马，密码是cmd，你能发现什么有用信息吗？ [http://dl.dutsec.cn/web/web10/index.php](http://dl.dutsec.cn/web/web10/index.php) 因为基本没有拿站的经验，所以这道题一上来有点蒙，后来出题人提示说需要拿站经验，咳咳，好歹也是接触过一点，php的一句话木马还是知道的，基本类似于

这种，密码应该就是post字段了，但是应该post什么值呢，思来想去，无果，后来去查了一下，看到了phpinfo()，突然想到应该就是它，试了一下，没用，于是考虑写成语句 phpinfo(); 果然奏效 [![2015-04-29 22:58:44的屏幕截图](http://skysider.com/wp-content/uploads/2015/04/2015-04-29-225844的屏幕截图.png)](http://skysider.com/wp-content/uploads/2015/04/2015-04-29-225844的屏幕截图.png) Web 50pt kow才开始学php，写了个小网站，但是有点安全问题，你能利用漏洞获取flag吗？[http://dl.dutsec.cn/web/05c035b1d7e82a97/index.php ](http://kow才开始学php，写了个小网站，但是有点安全问题，你能利用漏洞获取flag吗？ http://dl.dutsec.cn/web/05c035b1d7e82a97/index.php) 首先看了下源代码，发下有源代码，其中有一句：

$sql = "select user from php where (user='$user') and (pw='$pass')";

比较经典的sql注入，直接构造payload:  user=admin' or '1'='1')#&pass=123 [![2015-04-29 23:09:51的屏幕截图](http://skysider.com/wp-content/uploads/2015/04/2015-04-29-230951的屏幕截图.png)](http://skysider.com/wp-content/uploads/2015/04/2015-04-29-230951的屏幕截图.png) web 100pt DJ觉得kow写的网站太渣，他也来写了一个，但是说好的源码呢？（注意备份文件哦） [http://dl.dutsec.cn/web/c66ba13ab15ac925/index.php](http://dl.dutsec.cn/web/c66ba13ab15ac925/index.php) 看了一下网页源码，无任何额外信息，再看题目中的提示，备份文件，于是直接试了一下 index.php.bak，果然有东西，内容是index.php的源码，

if(eregi("hackerDJ",$_GET\[id\])) {
echo("<p>not allowed!</p>");
exit();
}

$\_GET\[id\] = urldecode($\_GET\[id\]);
if($_GET\[id\] == "hackerDJ")
{
echo "<p>Access granted!</p>";
echo "<p>flag: xxxxxxxxxxxxxxxxxxxxxxxxxxxxx </p>";
}

由于之前刚刚入过urldecode的坑，所以一眼就看到了urldecode的问题，对hackerDJ双重urlencode编码即可，浏览器会自动解码一次。 [![2015-04-29 23:19:12的屏幕截图](http://skysider.com/wp-content/uploads/2015/04/2015-04-29-231912的屏幕截图.png)](http://skysider.com/wp-content/uploads/2015/04/2015-04-29-231912的屏幕截图.png)   web 100pt kow一气之下又写了一个网站，但是他好像又疏忽了[http://dl.dutsec.cn/web/f73b6ed70786ef40/index.php](http://dl.dutsec.cn/web/f73b6ed70786ef40/index.php) 这道题目还是代码审计的题目（看样子大连理工比较偏爱PHP审计），看源代码，发现index.php.bak，下载打开看了一下

$user = $_POST\[user\];

$pass = md5($_POST\[pass\]);

$sql = "select pw from php where user='$user'";
$query = mysqli_query($conn,$sql);
if (!$query) {
printf("Error: %s\\n", mysqli_error($conn));
exit();
}
$row = mysqli\_fetch\_array($query);
//echo $row\["pw"\];
if (($row\[pw\]) && (!strcasecmp($pass, $row\[pw\]))) {
echo "<p>Logged in! Key: ################################ </p>";
}
else {
echo("<p>Log in failure!</p>")；
}
}

一开始考虑strcasecmp函数那有问题，众所周知，php的比较是个坑，因为php是弱类型，所以比较容易出问题，考虑是否其中一个参数为数组，$pass为md5之后的结果，只能是字符串，$row本身是查表返回跟用户名匹配的密码，所以它要么是空，要么是一个array("pw"=>"xxxx")的结构，也没有成为数组的可能，同时if中的第一个判断条件也杜绝了$row为NULL的情况，后来想到应该是sql查询出了问题，但是始终不知道应该怎么利用，赛完经过提示，此处是利用union select（之前对union的认识有误区），payload: user=' union select '202cb962ac59075b964b07152d234b70'#&pass=123 （前面是对123的md5） [![2015-04-29 23:33:04的屏幕截图](http://skysider.com/wp-content/uploads/2015/04/2015-04-29-233304的屏幕截图.png)](http://skysider.com/wp-content/uploads/2015/04/2015-04-29-233304的屏幕截图.png)