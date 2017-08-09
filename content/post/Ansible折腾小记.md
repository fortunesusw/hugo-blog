---
title: "Ansible折腾小记"
date: 2017-08-09T17:12:59+08:00
subtitle: ""
tags: ["ansible"]
---

比起人工手动ssh到远程，部署更新等等，耗时耗力，亟需自动化工具的使用

ansible是个什么东西呢？官方的title是“Ansible is Simple IT Automation”——简单的自动化IT工具， 对于自动化部署及管理很便捷， 基于ssh协议， python实现

# 安装
环境是mac， 直接使用homebrew

```
brew install ansible
```

# 配置
可以使用[官方的默认配置](https://raw.githubusercontent.com/ansible/ansible/devel/examples/ansible.cfg)做修改， 具体配置可以参考官方文档。

### Ansible环境

Ansible 配置文件是以.ini格式存储配置数据的，在Ansible中，几乎所有的配置项都可以通过Ansible的playbook或环境变量来重新赋值。在运行Ansible命令式，命令将会按照预先设定的顺序查找配置文件，如下所示：

1. ANSIBLE_CONFIG: 首先，Ansible命令会检查环境变量，及这个环境变量指向的配置文件。
2. /ansible.cfg: 其次，将会检查当前目录下的ansible.cfg配置文件
3. ~/ansible.cfg: 再次，将会检查当前用户home目录下的.ansible.cfg配置文件
4. /etc/ansible/ansible.cfg: 最后，将会检查再用软件包管理工具安装Ansible时自动产生的配置文件。

**注： 由于本人用homebrew安装， 在/etc/ansible/新建ansible.cfg，读不到配置， 警告信息如下**

```
 ~ ansible remote --list-hosts

 [WARNING]: Host file not found: /usr/local/etc/ansible/hosts

 [WARNING]: provided hosts list is empty, only localhost is available

 [WARNING]: No hosts matched, nothing to do

  hosts (0):
```

按提示的在`/usr/local/etc/ansible`目录下新建配置文件解决

# 运行
首先要保证你的`public SSH key`必须在远程机器的`authorized_keys`中， 可以参考无密钥登陆

### 运行 hello world
编辑`/usr/local/etc/ansible/hosts`， 新增

```
[remote]
45.77.x.x
```

运行

```
~ ansible remote -a  "echo hello world"

45.77.x.x | UNREACHABLE! => {
    "changed": false,
    "msg": "Failed to connect to the host via ssh: Permission denied (publickey,gssapi-keyex,gssapi-with-mic,password).\r\n",
    "unreachable": true
}
```

如上， 出错了， 原因是权限问题， 跟踪[官方issue](https://github.com/ansible/ansible/issues/19584)， 指定`ansible_user`解决

```
[remote]
45.77.x.x ansible_user=root
```

```
 ~ ansible remote -a  'echo hello world'
 
45.77.x.x | SUCCESS | rc=0 >>
hello world
```

# playbook

Playbooks 是 Ansible的配置，部署，编排语言。他们可以被描述为一个需要希望远程主机执行命令的方案，或者一组IT程序运行的命令集合。

playbook 文件格式为`YAML`语法，在编写playbook前需要对YAML有一定的了解，关于YAML语法可以通过 [yaml官网](http://yaml.org/spec/1.2/spec.html)进行学习。

playbook 由一个或多个 `plays` 组成，它的内容是一个以 `plays` 为元素的列表。在 play 之中，一组机器被映射为定义好的角色。在 ansible 中，play 的内容，被称为 tasks，即任务。在基本层次的应用中，一个任务是一个对 ansible 模块的调用

下面一个 playbook示例,其中仅包含一个 play:

```yaml
---
- hosts: remote
  vars:
    http_port: 80
    max_clients: 200
  remote_user: root
  tasks:
  - name: ensure apache is at the latest version
    yum: pkg=httpd state=latest
  - name: write the apache config file
    template: src=/srv/httpd.j2 dest=/etc/httpd.conf
    notify:
    - restart apache
  - name: ensure apache is running
    service: name=httpd state=started
  handlers:
    - name: restart apache
      service: name=httpd state=restarted
```

在 [ansible-examples](https://github.com/ansible/ansible-examples) 中有很多实例，如果你希望深入学习可以在单独的页面打开它。


### Task

每一个 play 包含了一个 task 列表（任务列表）。一个 task 在其所对应的所有主机上（通过 hosts 匹配的所有主机）执行完毕之后，下一个 task 才会执行。

### playbook变量与引用

1. 通过inventory文件定义主机以及主机变量
    在hosts定义
    
    ```
    [remote]
    45.77.x.x

    [remote:vars]
    key=test
    ```    

    playbook引用

    ```yaml
    ---
    - hosts: merge
      gather_facts: False
      tasks:
      - name: display Host Variable from hostfile
        debug: msg="The key value is {{ key }}"
    ```

2. 通过文件定义主机以及主机组变量

    我们可以通过host_vars和group_vars目录来针对主机和主机组定义变量。
    
3. 通过ansible-playbook命令行传入
    可以通过-e 命令传入变量
    
    ```
    ansbile-playbook test.yaml -e "key=test"
    ```
    
4. 可以在playbook文件内通过vars字段定义变量

5. 可以在playbook文件内通过vars_files字段引用变量，首先把所有的变量定义在某个文件内，然后再playbook文件内使用vars_files参数引用这个变量文件

6. 使用register内的变量
    Ansible playbook内task之间可以互相传递数据，比如第2个task需要获得第1个task的执行结果。
    
    我们可以通过下面的方式：

    ```yaml
    ---
    - hosts: merge
      gather_facts: False
      tasks:
      - name: register variable
        shell: hostname
        register: info
      - name: display variable
        debug: msg="The key value is {{ info }}"
    ```
    info的结果是一段python字典数据，里面存储着很多信息，包括执行时间状态变化输出等。
    
### 变量的优先级

如果同样名称的变量在多个地方都有定义,那么采纳是有个确定的顺序,如下:

1.xtra vars (在命令行中使用 -e)优先级最高，如:ansible-playbook -e var=value
2.然后是在inventory中定义的连接变量(比如ansible_ssh_user)
3.接着是大多数的其它变量(命令行转换,play中的变量,included的变量,role中的变量等)
4.然后是在inventory定义的其它变量
5.然后是由系统发现的facts
6.然后是 "role默认变量", 这个是最默认的值,很容易丧失优先权

### playbook 循环

1. 标准loops
    
    ```yaml
    ---
    - hosts: remote
      gather_facts: False
      tasks:
      - name: debug loops
        debug: msg="name ---------->  {{ item }}"
        with_items:
          - one
          - two
    ```

2. 嵌套loops

    ```yaml
    ---
    - hosts: remote
      gather_facts: False
      tasks:
      - name: debug loops
        debug: msg="name ---------->  {{ item[0] }} ---------> {{ item[1] }}"
        with_nested:
          - ['A']
          - ['a','b','c']
    

    ```
    
    3. 条件判断loops

        ```yaml
        ---
        - hosts: remote
          gather_facts: False
          tasks:
          - name: debug loops
            shell: cat /root/test.txt
            register: host
            until: host.stdout.startswith("test")
            retries: 5
            delay: 5
        ```

### playbook lookups
Ansible还支持从外部数据拉取信息，比如我们可以从数据库拉取信息，然后赋值给一个变量。

1. lookups file
    
    ```yaml
    ---
    - hosts: remote
      gather_facts: False
      vars:
        contents: "{{ lookup('file', '/root/openrc') }}"
      tasks:
      - name: debug lookups
        debug: msg="The contents is {% for i in contents.split("\n") %} {{ i }} {% endfor %}"
    ```

2. lookups password
    lookup('password', 'file_path')
    它会对传入的内容进行加密处理

### playbook conditionals

目前Ansible所有conditionals方式都是通过使用when进行判断，when的值是一个条件表达式，如果条件判断成立，这个task就执行某个操作，否则，该task不执行。

```yaml
tasks:
  - name: "shutdown Debian flavored systems"
    command: /sbin/shutdown -t now
    when: ansible_os_family == "Debian"
```

## 其他
playbook参照

http://ansible-tran.readthedocs.io/en/latest/docs/playbooks_intro.html

http://www.jianshu.com/p/41c4ed3ce779


