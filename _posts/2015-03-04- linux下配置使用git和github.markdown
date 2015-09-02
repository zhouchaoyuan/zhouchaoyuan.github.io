---
layout: post
title:  java中的堆内存和栈内存小结
date:   2015-03-04 20:45:45
category: "学习"
---

折腾了一天大概有一点点明白了，先记录一下，总结总结！其实教程github上面已经非常详细。
First：Set Up Git

1、首先安装git，在终端输入：


    sudo apt-get install git  

2、接下来是认证用户名：


    git config --global user.name "YOUR NAME"  

其中引号里面要填写的就是自己的用户名

3、认证邮箱：


    git config --global user.email "your@mail.com"  


引号里面填写邮箱

4、认证github和git，我这里使用ssh（Authenticating with GitHub from Git）：

a、产生公钥：


    ssh-keygen -t rsa -C "your_email@example.com"  

其中   -C "your_email@example.com"   可以不要

输入命令后会有三个选项，github建议都采取默认，也就是按下三次enter        详细见：  Generating SSH keys

按下三次后产生的公钥会在~/.ssh/目录下的id_rsa.pub，采用如下命令获得密钥

    cat ~/.ssh/id_rsa.pub   

b、复制公钥内容，打开自己的github，设置 > SSH Keys 里面新建一个SSH Key，将复制的内容粘贴到内容，title可以乱写，也可以不写，保存就可以了！


到这里应该说就可以用命令同步各种数据了吧！目前这么理解
Second：Create A Repository

这一个步骤全部在github上面操作，github有详细的图文，我就不丢人了

传送门：Create A Repo


Third：Fork A Repo
Fork就是看到别人好的库fork到自己的帐号这边来，然后你可以研究里面的东西，甚至发现bug，报告给这个库原来的属主，属主觉得有道理便会修改

详细见：Fork A Repo

Fourth：Be Social
最后就是建立一个类似交际圈的“Be Social”，这里有介绍怎么使用github的按键，如follow  watch issue  star等等


以下是我操作的一个个人实例，基于我在上面第二个步骤里面已经建好的库：

1、创建一个目录，并进入目录初始化库：


    mkdir zhou  
    cd zhou  
    git init  

这样在zhou这个目录下就会有一个.git的隐藏目录，可以用如下命令查看：


    ls -a  

2、然后在当前目录下可以创建文件，可使用命令如下命令：

    touch README  
    > 1000.cpp  

便创建了两个文件，分别是README和1000.cpp，可以对文件进行编辑

3、编辑完成之后就可以将文件提交到git缓冲区（index），命令如下：


    git add 1000.cpp  

上面命令添加了一个，也可以添加全部，命令如下，注意有一个点：


    git add .  

4、然后将缓冲区（index）的修改提交到head：


    git commit -m "first commit"  


5、最后一步是提交到远程仓库：


    git push origin master  

如果上面执行出现错误，可以先执行git pull origin master，因为还不同步，将远程同步到本地，这样就可以执行了。
