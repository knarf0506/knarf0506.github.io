# 02-Ansible 入門

[TOC]

## 2.1 安裝 Ansible

&emsp;&emsp;在本文中以 **CentOS 7** 為例，安裝<u>管理主機</u>及<u>遠端主機</u>。

### 2.1.1 管理主機

&emsp;&emsp;安裝 **Ansible**。

```bash
sudo yum install epel-release
sudo yum install ansible
```

&emsp;&emsp;設定 **Ansible** <u>管理主機</u>和<u>遠端主機</u>連接。

```bash
# 產生 SSH 金鑰
ssh-keygen

# 檢查 SSH 公鑰(id_rsa.pub) 及私鑰(id_rsa)
cd .ssh/
ll

# 複製管理主機的 SSH 金鑰到遠端主機，這樣管理主機之後透過 SSH 連線至遠端主機就不需要密碼
ssh-copy-id -i ./id_rsa.pub remoteserver
```

&emsp;&emsp;驗證 **SSH** 設定：在<u>管理主機</u>執行 **SSH** 指令遠端到<u>遠端主機</u>，若不需要密碼代表設定成功。

```bash
ssh remoteuser@remoteserver
```

### 2.1.2 遠端主機

&emsp;&emsp;**CentOS 7** 需要確保啟動 **SSH** 服務，並請安裝 **Python** 版本 2.4 以上。

## 2.2 管理主機設定

&emsp;&emsp;在上述中，<u>管理主機</u>透過 **SSH** 服務可以直接連線到<u>遠端主機</u>，<font color="red">但並不代表<u>管理主機</u>已經可以部署<u>遠端主機</u></font>。因此在<u>管理主機</u>上需要設定<u>主機目錄</u>設定檔。

### 2.2.1 預設設定檔

&emsp;&emsp;在 **Ansible** 中，預設的 **config file** 在：/etc/ansible/ansible.cfg，可以透過以下指令查看。

```bash
ansible --version

# 返回結果

ansible 2.9.18
  config file = /etc/ansible/ansible.cfg
  configured module search path = [u'/root/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python2.7/site-packages/ansible
  executable location = /usr/bin/ansible
  python version = 2.7.5 (default, Nov 16 2020, 22:23:17) [GCC 4.8.5 20150623 (Red Hat 4.8.5-44)]
```

&emsp;&emsp;打開：/etc/ansible/ansible.cfg，在最上方可以看到 **inventory**，代表<u>主機清單</u>，是用來告訴 **Ansible** 需要管理哪些<u>遠端主機</u>。其<u>主機清單</u>預設路徑在：/etc/ansible/hosts。

```bash
# config file for ansible -- https://ansible.com/
# ===============================================

# nearly all parameters can be overridden in ansible-playbook
# or with command line flags. ansible will read ANSIBLE_CONFIG,
# ansible.cfg in the current working directory, .ansible.cfg in
# the home directory or /etc/ansible/ansible.cfg, whichever it
# finds first

[defaults]

# some basic default values...

#inventory      = /etc/ansible/hosts          # 主機清單
#library        = /usr/share/my_modules/      # ansible 預設搜尋模組位置
#module_utils   = /usr/share/my_module_utils/
#remote_tmp     = ~/.ansible/tmp              # 複製一份 python script 放在 remote_tmp，被控制的節點都會執行這份腳本，執行完之後會刪除，並返回回傳結果
#local_tmp      = ~/.ansible/tmp              # 假設使用 ping 模組，會轉換成 python script 放在 local_tmp 的位置
#plugin_filters_cfg = /etc/ansible/plugin_filters.yml
#forks          = 5                           # 同時對 5 台遠端主機並行執行命令，直到都回傳後再傳下 5 台
#poll_interval  = 15
#sudo_user      = root
#ask_sudo_pass = True                         # playbook 預設為 False
#ask_pass      = True                         # 詢問是否提供密碼
#transport      = smart
#remote_port    = 22                          # 遠端 port
#module_lang    = C
#module_set_locale = False
#.
#.
#.
# uncomment this to disable SSH key host checking
#host_key_checking =:: False                    # 連線遠端主機時，檢查對方的公鑰有沒有在我們的 .ssh 的已知清單裡面，如果從來沒有連線過，則會跳出訊息，假設有 N 台則會有 N 條訊息
#.
#.
#.
# logging is off by default unless this path is defined
# if so defined, consider logrotate
#log_path = /var/log/ansible.log                # 在 ansible 裡面，預設 log 是不保存的，如果需要保存 log，加入 ansible.cfg
#.
#.
#.
# if set, always use this private key file for authentication, same as
# if passing --private-key to ansible or ansible-playbook
#private_key_file = /path/to/file               # 配置私鑰的路徑
```

&emsp;&emsp;打開：/etc/ansible/hosts，可以看到範例：

```bash
# This is the default ansible 'hosts' file.
#
# It should live in /etc/ansible/hosts
#
#   - Comments begin with the '#' character
#   - Blank lines are ignored
#   - Groups of hosts are delimited by [header] elements
#   - You can enter hostnames or ip addresses
#   - A hostname/ip can be a member of multiple groups

# Ex 1: Ungrouped hosts, specify before any group headers.

## green.example.com
## blue.example.com
## 192.168.100.1
## 192.168.100.10

# Ex 2: A collection of hosts belonging to the 'webservers' group

## [webservers]
## alpha.example.org
## beta.example.org
## 192.168.1.100
## 192.168.1.110

# If you have multiple hosts following a pattern you can specify
# them like this:

## www[001:006].example.com

# Ex 3: A collection of database servers in the 'dbservers' group

## [dbservers]
##
## db01.intranet.mydomain.net
## db02.intranet.mydomain.net
## 10.25.1.56
## 10.25.1.57

# Here's another example of host ranges, this time there are no
# leading 0s:

## db-[99:101]-node.example.com
```

### 2.2.2 撰寫設定檔

&emsp;&emsp;如果需要自己撰寫設定檔，則目錄下需要包含 <u>ansible.cfg</u> 及 <u>hosts</u>。
                
* Lesson1
  * ansible.cfg
  * hosts

&emsp;&emsp;打開：/Lesson1/ansible.cfg。

```bash
[defaults]

inventory = ./hosts # 預設路徑是 /etc/ansible/hosts
host_key_checking = False # 多次連線情況下可能會報錯
private_key_file = /root/.ssh/id_rsa # 設定私鑰
log_path = ./log/ansible.log # 執行 ansible 指令時儲存 log 至此檔案
```

&emsp;&emsp;打開：/Lesson1/hosts。

```bash
[node]

10.8.21.212
10.8.21.213
```

&emsp;&emsp;在 /Lesson1 執行以下指令：

```bash
ansible all -m ping # 對所有 hosts 的 group 執行 ping 模組

# 返回結果

10.8.21.212 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
10.8.21.213 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
```

&emsp;&emsp;在 /Lesson1 執行以下指令也可達到相同結果：

```bash
ansible node -m ping # 對 hosts 的 group node 執行 ping 模組

# 返回結果

10.8.21.212 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
10.8.21.213 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
```

&emsp;&emsp;hosts 也可以修改為：

```bash
[node]

10.8.21.21[2:3] # 類似正則表達式的寫法，程式會執行 10.8.21.212, 10.8.21.213
```

&emsp;&emsp;如果要修改連接埠，可以在 host 後面加上冒號及連接埠：

```bash
[node]

10.8.21.212:5000
10.8.21.213
```

&emsp;&emsp;如果對 2 個 host 做分組：

```bash
[node1]

10.8.21.212

[node2]

10.8.21.213
```

&emsp;&emsp;執行以下指令只會回傳 group 為 node1 的結果：

```bash
ansible node1 -m ping # 對 hosts 的 group node1 執行 ping 模組

# 返回結果

10.8.21.212 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
```

&emsp;&emsp;對 hosts 創造另一個 group，而這個 group 可以是另一個 group 的子成員。

```
[node1]

10.8.21.212

[node2]

10.8.21.213

[node:children]

node1
node2
```

&emsp;&emsp;執行以下指令：

```bash
ansible node -m ping

# 返回結果

10.8.21.212 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
10.8.21.213 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
```