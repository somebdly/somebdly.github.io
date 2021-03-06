**1. 前言**

Ansible是自动化运维的工具，基于Python开发，实现了批量系统配置、批量程序部署、批量运行命令等功能。

Ansible是基于模块工作的，ansible提供一个框架，通过模块实现批量部署。

![img](http://note.youdao.com/yws/res/2988/8C838BCF6BB3483FAECED0685C835211)

**2. 安装，使用**

**2.1 安装Ansible**

使用epel的源安装，添加epel源此处不详述。

| 1    | # yum install ansible --enablerepo=epel |
| ---- | --------------------------------------- |
|      |                                         |

**2.2 设置密钥登录**

生成SSH公钥密钥对

| 1    | # ssh-keygen -t rsa -P '' |
| ---- | ------------------------- |
|      |                           |

拷贝公钥到被管理端的服务器

| 12   | # cat /root/.ssh/id_rsa.pub >> /root/.ssh/authorized_keys# chmod 600 /root/.ssh/authorized_keys |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

确认可以用密钥连接到管理端的服务器

**2.3 配置Ansible**

定义主机组，可以使用主机名或IP

| 1234 | # vi /etc/ansible/hosts[tests]test167test154 |
| ---- | -------------------------------------------- |
|      |                                              |

另外，Ansible的配置文件在 /etc/ansible/ansible.cfg，默认不需要修改。

**2.4 使用Ansible**

**2.4.1 Ping模块**

| 1    | # ansible tests -m ping |
| ---- | ----------------------- |
|      |                         |

![img](http://note.youdao.com/yws/res/2993/0A3C220E837F43DBB2166D4C59A0DB45)

**2.4.2 执行命令，command、shell模块**

| 1234 | # ansible tests -m command -a 'uptime'# ansible tests -m shell -a 'date' # ansible tests -m command -a 'cat  /etc/resolv.conf' |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

**2.4.3 查看配置，setup模块**

| 1    | # ansible tests -m setup |
| ---- | ------------------------ |
|      |                          |

**2.4.4 拷贝文件，copy模块**

| 1    | # ansible tests -m copy -a 'src=/home/ec2-user/test.txt dest=/tmp/test222.txt mode=0644' |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

**2.4.5 添加用户，user模块**

| 1    | # ansible tests -m user -a 'name=test comment="test user" uid=1000 password="crypted-password"' |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

密码生成方法：

| 1234 | # yum install python-pip# pip install passlib # python -c "from passlib.hash import sha512_crypt; import getpass; print sha512_crypt.encrypt(getpass.getpass())" |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

**2.4.6 安装软件，yum模块**

| 1    | # ansible tests -m yum -a 'name=vsftpd state=present' |
| ---- | ----------------------------------------------------- |
|      |                                                       |

**2.4.7 启动服务，设置开机自启动，service模块**

| 1    | # ansible tests -m service -a 'name=vsftpd state=started enabled=yes' |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

查看

| 12   | # ansible tests -m shell -a 'ps -ef\| grep ftp'# ansible tests -m shell -a 'ss -tln\| grep 21' |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

**2.4.8 支持管道，raw，shell模块**

| 1    | # ansible tests -m raw -a 'ss -tln\| grep 21' |
| ---- | --------------------------------------------- |
|      |                                               |

**2.5 其他命令**

**2.5.1 查看帮助**

列出所有已安装模块

| 1    | # ansible-doc -l |
| ---- | ---------------- |
|      |                  |

查看某模块的简介

| 1    | # ansible-doc -s ping |
| ---- | --------------------- |
|      |                       |

**2.5.2 ansible-pull**

使用pull模式，（默认是push模式）

**3. Playbook文件**

Playbook是由一个或多个“play”组成的列表，可以让它们联同起来按事先编排的机制执行

| 1    | # ansible-playbook test.yml |
| ---- | --------------------------- |
|      |                             |

Playbook文件的格式，YAML是一个可读性高的标记语言。

Role可以把playbook分成一个一个模块，使结构更清晰。

**3.1 Role的构造**

包括 tasks, defaults, vars, files, templates, mata, handlers 各目录，其中 tasks 是必需的。

![img](http://note.youdao.com/yws/res/2989/ADE1B4D4B4F64046B410ABE6B3968DBB)

**3.2 Role的例子**

本例子是最基本的构成，只包括tasks

**3.2.1 创建目录 roles/apache2/tasks**

| 1    | # mkdir -p roles/apache2/tasks |
| ---- | ------------------------------ |
|      |                                |

**3.2.2 创建 tasks/main.yml**

| 12345678 | ---- name: Install apache2 ([RedHat](http://www.linuxidc.com/topicnews.aspx?tid=10)).  yum: name=httpd  when: "ansible_os_family == 'RedHat'" - name: Install apache2 (Debian).  apt: name=apache2  when: "ansible_os_family == 'Debian'" |
| -------- | ------------------------------------------------------------ |
|          |                                                              |

**3.2.3 创建 Playbook (site.yml)**

| 1234567 | ---- name: Install Apache2  hosts: tests  remote_user: root   roles:    - apache2 |
| ------- | ------------------------------------------------------------ |
|         |                                                              |

**3.2.4 执行**

| 1    | # ansible-playbook site.yml |
| ---- | --------------------------- |
|      |                             |

![img](http://note.youdao.com/yws/res/2991/D10F13FA0DA4413786B478DC6AAA0064)

**3.3 官方的playbook例子**

https://github.com/ansible/ansible-examples

**3.4 playbook文件加密**

ansible-vault 对配置文件（比如playbooks），进行基于密码的加密，防止敏感信息泄露

**3.4.1 加密已存在文件**

| 1    | # ansible-vault encrypt ./site.yml |
| ---- | ---------------------------------- |
|      |                                    |

**3.4.2 加密并创建文件**

| 1    | # ansible-vault create filename |
| ---- | ------------------------------- |
|      |                                 |

**加密后的playbook**

![img](http://note.youdao.com/yws/res/2990/274CDF2616E340E2B15199A14247E27F)

**3.4.3 执行加密后的playbook**

| 1    | # ansible-playbook ./site.yml --ask-vault-pass |
| ---- | ---------------------------------------------- |
|      |                                                |

**3.4.4 解密**

| 1    | # ansible-vault decrypt ./site.yml |
| ---- | ---------------------------------- |
|      |                                    |

**4. 后记**

Ansible使用简单，不需要客户端，模块化程度高，定制灵活，当管理服务器的数量多的时候，能起到很大的帮助。是一个很好的自动化运维工具。

**下面关于****Ansible****的文章您也可能喜欢，不妨参考下：**

使用Ansible批量管理远程服务器  [http://www.linuxidc.com/Linux/2015-05/118080.htm](https://www.linuxidc.com/Linux/2015-05/118080.htm)

在 [CentOS](http://www.linuxidc.com/topicnews.aspx?tid=14) 7 中安装并使用自动化工具 Ansible  [http://www.linuxidc.com/Linux/2015-10/123801.htm](https://www.linuxidc.com/Linux/2015-10/123801.htm)

CentOS 7上搭建Jenkins+Ansible服务  [http://www.linuxidc.com/Linux/2016-12/138737.htm](https://www.linuxidc.com/Linux/2016-12/138737.htm)

Linux下源码编译安装Ansible及排错记录  [http://www.linuxidc.com/Linux/2017-03/141427.htm](https://www.linuxidc.com/Linux/2017-03/141427.htm)

Ansible基础—安装与常用模块  [http://www.linuxidc.com/Linux/2017-02/140216.htm](https://www.linuxidc.com/Linux/2017-02/140216.htm)

Ansible配置及使用  [http://www.linuxidc.com/Linux/2017-03/142121.htm](https://www.linuxidc.com/Linux/2017-03/142121.htm)

自动化运维工具之 Ansible 介绍及安装使用  [http://www.linuxidc.com/Linux/2016-12/138104.htm](https://www.linuxidc.com/Linux/2016-12/138104.htm)

自动化运维之Ansible详解  [http://www.linuxidc.com/Linux/2017-03/142191.htm](https://www.linuxidc.com/Linux/2017-03/142191.htm)

Ansible入门notify和handlers  [http://www.linuxidc.com/Linux/2017-02/140871.htm](https://www.linuxidc.com/Linux/2017-02/140871.htm)

CentOS 6.5安装自动化工具Ansible和图形化工具Tower  [http://www.linuxidc.com/Linux/2017-03/141422.htm](https://www.linuxidc.com/Linux/2017-03/141422.htm)

**Ansible 的详细介绍**：[请点这里](https://www.linuxidc.com/Linux/2014-11/109871.htm)

**Ansible 的下载地址**：[请点这里](https://www.linuxidc.com/down.aspx?id=1634)

**本文永久更新链接地址**：[http://www.linuxidc.com/Linux/2017-05/143593.htm](https://www.linuxidc.com/Linux/2017-05/143593.htm)

![img](http://note.youdao.com/yws/res/2992/D1268191E15D402680BE342D057ACDFA)