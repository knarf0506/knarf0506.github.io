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

## 2.3 Ad-Hoc 指令

&emsp;&emsp;Ansible 透過 <u>Ad-Hoc Commands</u> 管理主機。假設我們現在要使用 <u>copy</u> 的模組，輸入以下指令，其中<font color=red>必填的參數</font>會以 <font color=red>= OPTIONS</font> 的形式呈現。在 copy 的例子中，必填的參數是 dest。

```bash
ansible-doc copy

# 返回結果

> ANSIBLE.BUILTIN.COPY    (/usr/local/lib/python3.6/site-packages/ansible/modules/copy.py)

        The `copy' module copies a file from the local or remote
        machine to a location on the remote machine. Use the
        [ansible.builtin.fetch] module to copy files from remote
        locations to the local box. If you need variable interpolation
        in copied files, use the [ansible.builtin.template] module.
        Using a variable in the `content' field will result in
        unpredictable output. For Windows targets, use the
        [ansible.windows.win_copy] module instead.

  * note: This module has a corresponding action plugin.

OPTIONS (= is mandatory):

- attributes
        The attributes the resulting file or directory should have.
        To get supported flags look at the man page for `chattr' on
        the target system.
        This string should contain the attributes in the same order as
        the one displayed by `lsattr'.
        The `=' operator is assumed as default, otherwise `+' or `-'
        operators need to be included in the string.
        (Aliases: attr)[Default: (null)]
        type: str
        version_added: 2.3
        version_added_collection: ansible.builtin

- backup
        Create a backup file including the timestamp information so
        you can get the original file back if you somehow clobbered it
        incorrectly.
        [Default: False]
        type: bool
        version_added: 0.7
        version_added_collection: ansible.builtin

- checksum
        SHA1 checksum of the file being transferred.
        Used to validate that the copy of the file was successful.
        If this is not provided, ansible will use the local calculated
        checksum of the src file.
        [Default: (null)]
        type: str
        version_added: 2.5
        version_added_collection: ansible.builtin

- content
        When used instead of `src', sets the contents of a file
        directly to the specified value.
        Works only when `dest' is a file. Creates the file if it does
        not exist.
        For advanced formatting or if `content' contains a variable,
        use the [ansible.builtin.template] module.
        [Default: (null)]
        type: str
        version_added: 1.1
        version_added_collection: ansible.builtin

- decrypt
        This option controls the autodecryption of source files using
        vault.
        [Default: True]
        type: bool
        version_added: 2.4
        version_added_collection: ansible.builtin

= dest
        Remote absolute path where the file should be copied to.
        If `src' is a directory, this must be a directory too.
        If `dest' is a non-existent path and if either `dest' ends
        with "/" or `src' is a directory, `dest' is created.
        If `dest' is a relative path, the starting directory is
        determined by the remote host.
        If `src' and `dest' are files, the parent directory of `dest'
        is not created and the task fails if it does not already
        exist.

        type: path

- directory_mode
        When doing a recursive copy set the mode for the directories.
        If this is not set we will use the system defaults.
        The mode is only set on directories which are newly created,
        and will not affect those that already existed.
        [Default: (null)]
        type: raw
        version_added: 1.5
        version_added_collection: ansible.builtin

- follow
        This flag indicates that filesystem links in the destination,
        if they exist, should be followed.
        [Default: False]
        type: bool
        version_added: 1.8
        version_added_collection: ansible.builtin

- force
        Influence whether the remote file must always be replaced.
        If `yes', the remote file will be replaced when contents are
        different than the source.
        If `no', the file will only be transferred if the destination
        does not exist.
        Alias `thirsty' has been deprecated and will be removed in
        2.13.
        (Aliases: thirsty)[Default: True]
        type: bool
        version_added: 1.1
        version_added_collection: ansible.builtin

- group
        Name of the group that should own the file/directory, as would
        be fed to `chown'.
        [Default: (null)]
        type: str

- local_follow
        This flag indicates that filesystem links in the source tree,
        if they exist, should be followed.
        [Default: True]
        type: bool
        version_added: 2.4
        version_added_collection: ansible.builtin

- mode
        The permissions of the destination file or directory.
        For those used to `/usr/bin/chmod' remember that modes are
        actually octal numbers. You must either add a leading zero so
        that Ansible's YAML parser knows it is an octal number (like
        `0644' or `01777')or quote it (like `'644'' or `'1777'') so
        Ansible receives a string and can do its own conversion from
        string into number. Giving Ansible a number without following
        one of these rules will end up with a decimal number which
        will have unexpected results.
        As of Ansible 1.8, the mode may be specified as a symbolic
        mode (for example, `u+rwx' or `u=rw,g=r,o=r').
        As of Ansible 2.3, the mode may also be the special string
        `preserve'.
        `preserve' means that the file will be given the same
        permissions as the source file.
        When doing a recursive copy, see also `directory_mode'.
        [Default: (null)]
        type: path

- owner
        Name of the user that should own the file/directory, as would
        be fed to `chown'.
        [Default: (null)]
        type: str

- remote_src
        Influence whether `src' needs to be transferred or already is
        present remotely.
        If `no', it will search for `src' at originating/master
        machine.
        If `yes' it will go to the remote/target machine for the
        `src'.
        `remote_src' supports recursive copying as of version 2.8.
        `remote_src' only works with `mode=preserve' as of version
        2.6.
        [Default: False]
        type: bool
        version_added: 2.0
        version_added_collection: ansible.builtin

- selevel
        The level part of the SELinux file context.
        This is the MLS/MCS attribute, sometimes known as the `range'.
        When set to `_default', it will use the `level' portion of the
        policy if available.
        [Default: (null)]
        type: str

- serole
        The role part of the SELinux file context.
        When set to `_default', it will use the `role' portion of the
        policy if available.
        [Default: (null)]
        type: str

- setype
        The type part of the SELinux file context.
        When set to `_default', it will use the `type' portion of the
        policy if available.
        [Default: (null)]
        type: str

- seuser
        The user part of the SELinux file context.
        By default it uses the `system' policy, where applicable.
        When set to `_default', it will use the `user' portion of the
        policy if available.
        [Default: (null)]
        type: str

- src
        Local path to a file to copy to the remote server.
        This can be absolute or relative.
        If path is a directory, it is copied recursively. In this
        case, if path ends with "/", only inside contents of that
        directory are copied to destination. Otherwise, if it does not
        end with "/", the directory itself with all contents is
        copied. This behavior is similar to the `rsync' command line
        tool.
        [Default: (null)]
        type: path

- unsafe_writes
        Influence when to use atomic operation to prevent data
        corruption or inconsistent reads from the target file.
        By default this module uses atomic operations to prevent data
        corruption or inconsistent reads from the target files, but
        sometimes systems are configured or just broken in ways that
        prevent this. One example is docker mounted files, which
        cannot be updated atomically from inside the container and can
        only be written in an unsafe manner.
        This option allows Ansible to fall back to unsafe methods of
        updating files when atomic operations fail (however, it
        doesn't force Ansible to perform unsafe writes).
        IMPORTANT! Unsafe writes are subject to race conditions and
        can lead to data corruption.
        [Default: False]
        type: bool
        version_added: 2.2
        version_added_collection: ansible.builtin

- validate
        The validation command to run before copying into place.
        The path to the file to validate is passed in via '%s' which
        must be present as in the examples below.
        The command is passed securely so shell features like
        expansion and pipes will not work.
        [Default: (null)]
        type: str


NOTES:
      * The [ansible.builtin.copy] module recursively copy
        facility does not scale to lots (>hundreds) of files.
      * Supports `check_mode'.


SEE ALSO:
      * Module ansible.builtin.assemble
           The official documentation on the
           ansible.builtin.assemble module.
           https://docs.ansible.com/ansible/2.10/modules/ansible
        .builtin.assemble_module.html
      * Module ansible.builtin.fetch
           The official documentation on the
           ansible.builtin.fetch module.
           https://docs.ansible.com/ansible/2.10/modules/ansible
        .builtin.fetch_module.html
      * Module ansible.builtin.file
           The official documentation on the
           ansible.builtin.file module.
           https://docs.ansible.com/ansible/2.10/modules/ansible
        .builtin.file_module.html
      * Module ansible.builtin.template
           The official documentation on the
           ansible.builtin.template module.
           https://docs.ansible.com/ansible/2.10/modules/ansible
        .builtin.template_module.html
      * Module ansible.posix.synchronize
           The official documentation on the
           ansible.posix.synchronize module.
           https://docs.ansible.com/ansible/2.10/modules/ansible
        .posix.synchronize_module.html
      * Module ansible.windows.win_copy
           The official documentation on the
           ansible.windows.win_copy module.
           https://docs.ansible.com/ansible/2.10/modules/ansible
        .windows.win_copy_module.html


AUTHOR: Ansible Core Team, Michael DeHaan

VERSION_ADDED_COLLECTION: ansible.builtin

EXAMPLES:

- name: Copy file with owner and permissions
  ansible.builtin.copy:
    src: /srv/myfiles/foo.conf
    dest: /etc/foo.conf
    owner: foo
    group: foo
    mode: '0644'

- name: Copy file with owner and permission, using symbolic representation
  ansible.builtin.copy:
    src: /srv/myfiles/foo.conf
    dest: /etc/foo.conf
    owner: foo
    group: foo
    mode: u=rw,g=r,o=r

- name: Another symbolic mode example, adding some permissions and removing others
  ansible.builtin.copy:
    src: /srv/myfiles/foo.conf
    dest: /etc/foo.conf
    owner: foo
    group: foo
    mode: u+rw,g-wx,o-rwx

- name: Copy a new "ntp.conf" file into place, backing up the original if it differs from the copied version
  ansible.builtin.copy:
    src: /mine/ntp.conf
    dest: /etc/ntp.conf
    owner: root
    group: root
    mode: '0644'
    backup: yes

- name: Copy a new "sudoers" file into place, after passing validation with visudo
  ansible.builtin.copy:
    src: /mine/sudoers
    dest: /etc/sudoers
    validate: /usr/sbin/visudo -csf %s

- name: Copy a "sudoers" file on the remote machine for editing
  ansible.builtin.copy:
    src: /etc/sudoers
    dest: /etc/sudoers.edit
    remote_src: yes
    validate: /usr/sbin/visudo -csf %s

- name: Copy using inline content
  ansible.builtin.copy:
    content: '# This file was moved to /etc/other.conf'
    dest: /etc/mine.conf

- name: If follow=yes, /path/to/file will be overwritten by contents of foo.conf
  ansible.builtin.copy:
    src: /etc/foo.conf
    dest: /path/to/link  # link to /path/to/file
    follow: yes

- name: If follow=no, /path/to/link will become a file and be overwritten by contents of foo.conf
  ansible.builtin.copy:
    src: /etc/foo.conf
    dest: /path/to/link  # link to /path/to/file
    follow: no


RETURN VALUES:
- backup_file
        Name of backup file created.

        returned: changed and if backup=yes
        sample: /path/to/file.txt.2015-02-12@22:09~
        type: str

- checksum
        SHA1 checksum of the file after running copy.

        returned: success
        sample: 6e642bb8dd5c2e027bf21dd923337cbb4214f827
        type: str

- dest
        Destination file/path.

        returned: success
        sample: /path/to/file.txt
        type: str

- gid
        Group id of the file, after execution.

        returned: success
        sample: 100
        
        type: int

- group
        Group of the file, after execution.

        returned: success
        sample: httpd
        type: str

- md5sum
        MD5 checksum of the file after running copy.

        returned: when supported
        sample: 2a5aeecc61dc98c4d780b14b330e3282
        type: str

- mode
        Permissions of the target, after execution.

        returned: success
        sample: 420
        
        type: str

- owner
        Owner of the file, after execution.

        returned: success
        sample: httpd
        type: str

- size
        Size of the target, after execution.

        returned: success
        sample: 1220
        
        type: int

- src
        Source file used for the copy on the target machine.

        returned: changed
        sample: /home/httpd/.ansible/tmp/ansible-
        tmp-1423796390.97-147729857856000/source
        type: str

- state
        State of the target, after execution.

        returned: success
        sample: file
        type: str

- uid
        Owner id of the file, after execution.

        returned: success
        sample: 100
        
        type: int
```

### 2.3.1 指令格式

```bash
ansible <host-pattern> [options]
```

### 2.3.2 指令功能

&emsp;&emsp;以下介紹基本的 Ansible 功能：

(1) 檢查 Ansible 環境：
&emsp;&emsp;檢查所有遠端主機，是否以 "root" 建立了 Ansible 管理主機可存取的環境。

```bash
ansible all -m ping -u root

# 返回結果

10.8.21.213 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
10.8.21.212 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
```

(2) 執行指令：
&emsp;&emsp;在所有遠端主機上執行 "echo hello"。

```bash
ansible all -a "echo hello"

# 返回結果

10.8.21.213 | CHANGED | rc=0 >>
hello
10.8.21.212 | CHANGED | rc=0 >>
hello
```

(3) 複製檔案：
&emsp;&emsp;複製檔案 /etc/hosts 到遠端主機 /tmp/hosts：

```bash
ansible all -m copy -a "src=/etc/hosts dest=/tmp/hosts"

# 返回結果

10.8.21.213 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": true,
    "checksum": "7335999eb54c15c67566186bdfc46f64e0d5a1aa",
    "dest": "/tmp/hosts",
    "gid": 0,
    "group": "root",
    "md5sum": "54fb6627dbaa37721048e4549db3224d",
    "mode": "0644",
    "owner": "root",
    "size": 158,
    "src": "/root/.ansible/tmp/ansible-tmp-1617674453.630371-7815-114563017107103/source",
    "state": "file",
    "uid": 0
}
10.8.21.212 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": true,
    "checksum": "7335999eb54c15c67566186bdfc46f64e0d5a1aa",
    "dest": "/tmp/hosts",
    "gid": 0,
    "group": "root",
    "md5sum": "54fb6627dbaa37721048e4549db3224d",
    "mode": "0644",
    "owner": "root",
    "size": 158,
    "src": "/root/.ansible/tmp/ansible-tmp-1617674453.6224563-7814-95691628361989/source",
    "state": "file",
    "uid": 0
}
```

(4) 安裝套件：
&emsp;&emsp;在所有遠端主機上透過 yum 安裝 tomcat 套件。

```bash
ansible all -m yum -a "name=tomcat state=present"

# 返回結果

10.8.21.213 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": true,
    "changes": {
        "installed": [
            "tomcat"
        ]
    },
    "msg": "",
    "rc": 0,
    "results": [
        "Loaded plugins: fastestmirror\nLoading mirror speeds from cached hostfile\n * epel: fedora.cs.nctu.edu.tw\nResolving Dependencies\n--> Running transaction check\n---> Package tomcat.noarch 0:7.0.76-16.el7_9 will be installed\n--> Finished Dependency Resolution\n\nDependencies Resolved\n\n================================================================================\n Package        Arch           Version                    Repository       Size\n================================================================================\nInstalling:\n tomcat         noarch         7.0.76-16.el7_9            updates          93 k\n\nTransaction Summary\n================================================================================\nInstall  1 Package\n\nTotal download size: 93 k\nInstalled size: 303 k\nDownloading packages:\nRunning transaction check\nRunning transaction test\nTransaction test succeeded\nRunning transaction\n  Installing : tomcat-7.0.76-16.el7_9.noarch                                1/1 \n  Verifying  : tomcat-7.0.76-16.el7_9.noarch                                1/1 \n\nInstalled:\n  tomcat.noarch 0:7.0.76-16.el7_9                                               \n\nComplete!\n"
    ]
}
10.8.21.212 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": true,
    "changes": {
        "installed": [
            "tomcat"
        ]
    },
    "msg": "",
    "rc": 0,
    "results": [
        "Loaded plugins: fastestmirror\nLoading mirror speeds from cached hostfile\n * epel: fedora.cs.nctu.edu.tw\nResolving Dependencies\n--> Running transaction check\n---> Package tomcat.noarch 0:7.0.76-16.el7_9 will be installed\n--> Finished Dependency Resolution\n\nDependencies Resolved\n\n================================================================================\n Package        Arch           Version                    Repository       Size\n================================================================================\nInstalling:\n tomcat         noarch         7.0.76-16.el7_9            updates          93 k\n\nTransaction Summary\n================================================================================\nInstall  1 Package\n\nTotal download size: 93 k\nInstalled size: 303 k\nDownloading packages:\nRunning transaction check\nRunning transaction test\nTransaction test succeeded\nRunning transaction\n  Installing : tomcat-7.0.76-16.el7_9.noarch                                1/1 \n  Verifying  : tomcat-7.0.76-16.el7_9.noarch                                1/1 \n\nInstalled:\n  tomcat.noarch 0:7.0.76-16.el7_9                                               \n\nComplete!\n"
    ]
}
```

(5) 啟用服務：
```bash
ansible all -m service -a "name=tomcat state=started"

# 返回結果

10.8.21.213 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": true,
    "name": "tomcat",
    "state": "started",
    "status": {
        "ActiveEnterTimestampMonotonic": "0",
        "ActiveExitTimestampMonotonic": "0",
        "ActiveState": "inactive",
        "After": "syslog.target systemd-journald.socket system.slice network.target basic.target",
        "AllowIsolate": "no",
        "AmbientCapabilities": "0",
        "AssertResult": "no",
        "AssertTimestampMonotonic": "0",
        "Before": "shutdown.target",
        "BlockIOAccounting": "no",
        "BlockIOWeight": "18446744073709551615",
        "CPUAccounting": "no",
        "CPUQuotaPerSecUSec": "infinity",
        "CPUSchedulingPolicy": "0",
        "CPUSchedulingPriority": "0",
        "CPUSchedulingResetOnFork": "no",
        "CPUShares": "18446744073709551615",
        "CanIsolate": "no",
        "CanReload": "no",
        "CanStart": "yes",
        "CanStop": "yes",
        "CapabilityBoundingSet": "18446744073709551615",
        "CollectMode": "inactive",
        "ConditionResult": "no",
        "ConditionTimestampMonotonic": "0",
        "Conflicts": "shutdown.target",
        "ControlPID": "0",
        "DefaultDependencies": "yes",
        "Delegate": "no",
        "Description": "Apache Tomcat Web Application Container",
        "DevicePolicy": "auto",
        "Environment": "NAME=",
        "EnvironmentFile": "/etc/sysconfig/tomcat (ignore_errors=yes)",
        "ExecMainCode": "0",
        "ExecMainExitTimestampMonotonic": "0",
        "ExecMainPID": "0",
        "ExecMainStartTimestampMonotonic": "0",
        "ExecMainStatus": "0",
        "ExecStart": "{ path=/usr/libexec/tomcat/server ; argv[]=/usr/libexec/tomcat/server start ; ignore_errors=no ; start_time=[n/a] ; stop_time=[n/a] ; pid=0 ; code=(null) ; status=0/0 }",
        "FailureAction": "none",
        "FileDescriptorStoreMax": "0",
        "FragmentPath": "/usr/lib/systemd/system/tomcat.service",
        "GuessMainPID": "yes",
        "IOScheduling": "0",
        "Id": "tomcat.service",
        "IgnoreOnIsolate": "no",
        "IgnoreOnSnapshot": "no",
        "IgnoreSIGPIPE": "yes",
        "InactiveEnterTimestampMonotonic": "0",
        "InactiveExitTimestampMonotonic": "0",
        "JobTimeoutAction": "none",
        "JobTimeoutUSec": "0",
        "KillMode": "control-group",
        "KillSignal": "15",
        "LimitAS": "18446744073709551615",
        "LimitCORE": "18446744073709551615",
        "LimitCPU": "18446744073709551615",
        "LimitDATA": "18446744073709551615",
        "LimitFSIZE": "18446744073709551615",
        "LimitLOCKS": "18446744073709551615",
        "LimitMEMLOCK": "65536",
        "LimitMSGQUEUE": "819200",
        "LimitNICE": "0",
        "LimitNOFILE": "4096",
        "LimitNPROC": "15064",
        "LimitRSS": "18446744073709551615",
        "LimitRTPRIO": "0",
        "LimitRTTIME": "18446744073709551615",
        "LimitSIGPENDING": "15064",
        "LimitSTACK": "18446744073709551615",
        "LoadState": "loaded",
        "MainPID": "0",
        "MemoryAccounting": "no",
        "MemoryCurrent": "18446744073709551615",
        "MemoryLimit": "18446744073709551615",
        "MountFlags": "0",
        "Names": "tomcat.service",
        "NeedDaemonReload": "no",
        "Nice": "0",
        "NoNewPrivileges": "no",
        "NonBlocking": "no",
        "NotifyAccess": "none",
        "OOMScoreAdjust": "0",
        "OnFailureJobMode": "replace",
        "PermissionsStartOnly": "no",
        "PrivateDevices": "no",
        "PrivateNetwork": "no",
        "PrivateTmp": "no",
        "ProtectHome": "no",
        "ProtectSystem": "no",
        "RefuseManualStart": "no",
        "RefuseManualStop": "no",
        "RemainAfterExit": "no",
        "Requires": "system.slice basic.target",
        "Restart": "no",
        "RestartUSec": "100ms",
        "Result": "success",
        "RootDirectoryStartOnly": "no",
        "RuntimeDirectoryMode": "0755",
        "SameProcessGroup": "no",
        "SecureBits": "0",
        "SendSIGHUP": "no",
        "SendSIGKILL": "yes",
        "Slice": "system.slice",
        "StandardError": "inherit",
        "StandardInput": "null",
        "StandardOutput": "journal",
        "StartLimitAction": "none",
        "StartLimitBurst": "5",
        "StartLimitInterval": "10000000",
        "StartupBlockIOWeight": "18446744073709551615",
        "StartupCPUShares": "18446744073709551615",
        "StatusErrno": "0",
        "StopWhenUnneeded": "no",
        "SubState": "dead",
        "SyslogLevelPrefix": "yes",
        "SyslogPriority": "30",
        "SystemCallErrorNumber": "0",
        "TTYReset": "no",
        "TTYVHangup": "no",
        "TTYVTDisallocate": "no",
        "TasksAccounting": "no",
        "TasksCurrent": "18446744073709551615",
        "TasksMax": "18446744073709551615",
        "TimeoutStartUSec": "1min 30s",
        "TimeoutStopUSec": "1min 30s",
        "TimerSlackNSec": "50000",
        "Transient": "no",
        "Type": "simple",
        "UMask": "0022",
        "UnitFilePreset": "disabled",
        "UnitFileState": "disabled",
        "User": "tomcat",
        "WatchdogTimestampMonotonic": "0",
        "WatchdogUSec": "0"
    }
}
10.8.21.212 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": true,
    "name": "tomcat",
    "state": "started",
    "status": {
        "ActiveEnterTimestampMonotonic": "0",
        "ActiveExitTimestampMonotonic": "0",
        "ActiveState": "inactive",
        "After": "syslog.target basic.target systemd-journald.socket network.target system.slice",
        "AllowIsolate": "no",
        "AmbientCapabilities": "0",
        "AssertResult": "no",
        "AssertTimestampMonotonic": "0",
        "Before": "shutdown.target",
        "BlockIOAccounting": "no",
        "BlockIOWeight": "18446744073709551615",
        "CPUAccounting": "no",
        "CPUQuotaPerSecUSec": "infinity",
        "CPUSchedulingPolicy": "0",
        "CPUSchedulingPriority": "0",
        "CPUSchedulingResetOnFork": "no",
        "CPUShares": "18446744073709551615",
        "CanIsolate": "no",
        "CanReload": "no",
        "CanStart": "yes",
        "CanStop": "yes",
        "CapabilityBoundingSet": "18446744073709551615",
        "CollectMode": "inactive",
        "ConditionResult": "no",
        "ConditionTimestampMonotonic": "0",
        "Conflicts": "shutdown.target",
        "ControlPID": "0",
        "DefaultDependencies": "yes",
        "Delegate": "no",
        "Description": "Apache Tomcat Web Application Container",
        "DevicePolicy": "auto",
        "Environment": "NAME=",
        "EnvironmentFile": "/etc/sysconfig/tomcat (ignore_errors=yes)",
        "ExecMainCode": "0",
        "ExecMainExitTimestampMonotonic": "0",
        "ExecMainPID": "0",
        "ExecMainStartTimestampMonotonic": "0",
        "ExecMainStatus": "0",
        "ExecStart": "{ path=/usr/libexec/tomcat/server ; argv[]=/usr/libexec/tomcat/server start ; ignore_errors=no ; start_time=[n/a] ; stop_time=[n/a] ; pid=0 ; code=(null) ; status=0/0 }",
        "FailureAction": "none",
        "FileDescriptorStoreMax": "0",
        "FragmentPath": "/usr/lib/systemd/system/tomcat.service",
        "GuessMainPID": "yes",
        "IOScheduling": "0",
        "Id": "tomcat.service",
        "IgnoreOnIsolate": "no",
        "IgnoreOnSnapshot": "no",
        "IgnoreSIGPIPE": "yes",
        "InactiveEnterTimestampMonotonic": "0",
        "InactiveExitTimestampMonotonic": "0",
        "JobTimeoutAction": "none",
        "JobTimeoutUSec": "0",
        "KillMode": "control-group",
        "KillSignal": "15",
        "LimitAS": "18446744073709551615",
        "LimitCORE": "18446744073709551615",
        "LimitCPU": "18446744073709551615",
        "LimitDATA": "18446744073709551615",
        "LimitFSIZE": "18446744073709551615",
        "LimitLOCKS": "18446744073709551615",
        "LimitMEMLOCK": "65536",
        "LimitMSGQUEUE": "819200",
        "LimitNICE": "0",
        "LimitNOFILE": "4096",
        "LimitNPROC": "15064",
        "LimitRSS": "18446744073709551615",
        "LimitRTPRIO": "0",
        "LimitRTTIME": "18446744073709551615",
        "LimitSIGPENDING": "15064",
        "LimitSTACK": "18446744073709551615",
        "LoadState": "loaded",
        "MainPID": "0",
        "MemoryAccounting": "no",
        "MemoryCurrent": "18446744073709551615",
        "MemoryLimit": "18446744073709551615",
        "MountFlags": "0",
        "Names": "tomcat.service",
        "NeedDaemonReload": "no",
        "Nice": "0",
        "NoNewPrivileges": "no",
        "NonBlocking": "no",
        "NotifyAccess": "none",
        "OOMScoreAdjust": "0",
        "OnFailureJobMode": "replace",
        "PermissionsStartOnly": "no",
        "PrivateDevices": "no",
        "PrivateNetwork": "no",
        "PrivateTmp": "no",
        "ProtectHome": "no",
        "ProtectSystem": "no",
        "RefuseManualStart": "no",
        "RefuseManualStop": "no",
        "RemainAfterExit": "no",
        "Requires": "basic.target system.slice",
        "Restart": "no",
        "RestartUSec": "100ms",
        "Result": "success",
        "RootDirectoryStartOnly": "no",
        "RuntimeDirectoryMode": "0755",
        "SameProcessGroup": "no",
        "SecureBits": "0",
        "SendSIGHUP": "no",
        "SendSIGKILL": "yes",
        "Slice": "system.slice",
        "StandardError": "inherit",
        "StandardInput": "null",
        "StandardOutput": "journal",
        "StartLimitAction": "none",
        "StartLimitBurst": "5",
        "StartLimitInterval": "10000000",
        "StartupBlockIOWeight": "18446744073709551615",
        "StartupCPUShares": "18446744073709551615",
        "StatusErrno": "0",
        "StopWhenUnneeded": "no",
        "SubState": "dead",
        "SyslogLevelPrefix": "yes",
        "SyslogPriority": "30",
        "SystemCallErrorNumber": "0",
        "TTYReset": "no",
        "TTYVHangup": "no",
        "TTYVTDisallocate": "no",
        "TasksAccounting": "no",
        "TasksCurrent": "18446744073709551615",
        "TasksMax": "18446744073709551615",
        "TimeoutStartUSec": "1min 30s",
        "TimeoutStopUSec": "1min 30s",
        "TimerSlackNSec": "50000",
        "Transient": "no",
        "Type": "simple",
        "UMask": "0022",
        "UnitFilePreset": "disabled",
        "UnitFileState": "disabled",
        "User": "tomcat",
        "WatchdogTimestampMonotonic": "0",
        "WatchdogUSec": "0"
    }
}
```

(6) 主機信息：

```bash
ansible all -m gather_facts

# 返回結果

10.8.21.212 | SUCCESS => {
    "ansible_facts": {
        "ansible_all_ipv4_addresses": [
            "10.8.21.212"
        ],
        "ansible_all_ipv6_addresses": [
            "fe80::cf92:3ff3:2f4e:69bb",
            "fe80::512a:c6f7:2f4:6997",
            "fe80::585:81cc:c9c3:2a60"
        ],
        "ansible_apparmor": {
            "status": "disabled"
        },
        "ansible_architecture": "x86_64",
        "ansible_bios_date": "12/12/2018",
        "ansible_bios_vendor": "Phoenix Technologies LTD",
        "ansible_bios_version": "6.00",
        "ansible_board_asset_tag": "NA",
        "ansible_board_name": "440BX Desktop Reference Platform",
        "ansible_board_serial": "None",
        "ansible_board_vendor": "Intel Corporation",
        "ansible_board_version": "None",
        "ansible_chassis_asset_tag": "No Asset Tag",
        "ansible_chassis_serial": "None",
        "ansible_chassis_vendor": "No Enclosure",
        "ansible_chassis_version": "N/A",
        "ansible_cmdline": {
            "BOOT_IMAGE": "/vmlinuz-3.10.0-1160.6.1.el7.x86_64",
            "LANG": "en_US.UTF-8",
            "crashkernel": "auto",
            "quiet": true,
            "rd.lvm.lv": "centos/swap",
            "rhgb": true,
            "ro": true,
            "root": "/dev/mapper/centos-root",
            "spectre_v2": "retpoline"
        },
        "ansible_date_time": {
            "date": "2021-04-06",
            "day": "06",
            "epoch": "1617675490",
            "hour": "10",
            "iso8601": "2021-04-06T02:18:10Z",
            "iso8601_basic": "20210406T101810320147",
            "iso8601_basic_short": "20210406T101810",
            "iso8601_micro": "2021-04-06T02:18:10.320147Z",
            "minute": "18",
            "month": "04",
            "second": "10",
            "time": "10:18:10",
            "tz": "CST",
            "tz_offset": "+0800",
            "weekday": "Tuesday",
            "weekday_number": "2",
            "weeknumber": "14",
            "year": "2021"
        },
        "ansible_default_ipv4": {
            "address": "10.8.21.212",
            "alias": "ens192",
            "broadcast": "10.8.21.255",
            "gateway": "10.8.21.253",
            "interface": "ens192",
            "macaddress": "00:50:56:95:fc:11",
            "mtu": 1500,
            "netmask": "255.255.255.0",
            "network": "10.8.21.0",
            "type": "ether"
        },
        "ansible_default_ipv6": {},
        "ansible_device_links": {
            "ids": {
                "dm-0": [
                    "dm-name-centos-root",
                    "dm-uuid-LVM-rR3AARPm531pFjQLdtiVmR5zFdPLztPSmxd2xIgwq5vIk9mZ2YiPB3e8tedv6TrN"
                ],
                "dm-1": [
                    "dm-name-centos-swap",
                    "dm-uuid-LVM-rR3AARPm531pFjQLdtiVmR5zFdPLztPSvV7InsduEjFcsNuIAGQOB4a4Rm2otU4R"
                ],
                "sda2": [
                    "lvm-pv-uuid-iqncfw-bGCP-AKN5-k8CP-ShoB-iHz7-mmUohd"
                ],
                "sr0": [
                    "ata-VMware_Virtual_SATA_CDRW_Drive_00000000000000000001"
                ]
            },
            "labels": {},
            "masters": {
                "sda2": [
                    "dm-0",
                    "dm-1"
                ]
            },
            "uuids": {
                "dm-0": [
                    "57996173-909b-4841-ad26-700841f5259e"
                ],
                "dm-1": [
                    "093b556a-2d87-4785-8ab0-0e68a099b4dc"
                ],
                "sda1": [
                    "289396f7-8dee-422e-845d-364c112ac85c"
                ]
            }
        },
        "ansible_devices": {
            "dm-0": {
                "holders": [],
                "host": "",
                "links": {
                    "ids": [
                        "dm-name-centos-root",
                        "dm-uuid-LVM-rR3AARPm531pFjQLdtiVmR5zFdPLztPSmxd2xIgwq5vIk9mZ2YiPB3e8tedv6TrN"
                    ],
                    "labels": [],
                    "masters": [],
                    "uuids": [
                        "57996173-909b-4841-ad26-700841f5259e"
                    ]
                },
                "model": null,
                "partitions": {},
                "removable": "0",
                "rotational": "1",
                "sas_address": null,
                "sas_device_handle": null,
                "scheduler_mode": "",
                "sectors": "63954944",
                "sectorsize": "512",
                "size": "30.50 GB",
                "support_discard": "0",
                "vendor": null,
                "virtual": 1
            },
            "dm-1": {
                "holders": [],
                "host": "",
                "links": {
                    "ids": [
                        "dm-name-centos-swap",
                        "dm-uuid-LVM-rR3AARPm531pFjQLdtiVmR5zFdPLztPSvV7InsduEjFcsNuIAGQOB4a4Rm2otU4R"
                    ],
                    "labels": [],
                    "masters": [],
                    "uuids": [
                        "093b556a-2d87-4785-8ab0-0e68a099b4dc"
                    ]
                },
                "model": null,
                "partitions": {},
                "removable": "0",
                "rotational": "1",
                "sas_address": null,
                "sas_device_handle": null,
                "scheduler_mode": "",
                "sectors": "7340032",
                "sectorsize": "512",
                "size": "3.50 GB",
                "support_discard": "0",
                "vendor": null,
                "virtual": 1
            },
            "sda": {
                "holders": [],
                "host": "Serial Attached SCSI controller: VMware PVSCSI SCSI Controller (rev 02)",
                "links": {
                    "ids": [],
                    "labels": [],
                    "masters": [],
                    "uuids": []
                },
                "model": "Virtual disk",
                "partitions": {
                    "sda1": {
                        "holders": [],
                        "links": {
                            "ids": [],
                            "labels": [],
                            "masters": [],
                            "uuids": [
                                "289396f7-8dee-422e-845d-364c112ac85c"
                            ]
                        },
                        "sectors": "2097152",
                        "sectorsize": 512,
                        "size": "1.00 GB",
                        "start": "2048",
                        "uuid": "289396f7-8dee-422e-845d-364c112ac85c"
                    },
                    "sda2": {
                        "holders": [
                            "centos-root",
                            "centos-swap"
                        ],
                        "links": {
                            "ids": [
                                "lvm-pv-uuid-iqncfw-bGCP-AKN5-k8CP-ShoB-iHz7-mmUohd"
                            ],
                            "labels": [],
                            "masters": [
                                "dm-0",
                                "dm-1"
                            ],
                            "uuids": []
                        },
                        "sectors": "71301120",
                        "sectorsize": 512,
                        "size": "34.00 GB",
                        "start": "2099200",
                        "uuid": null
                    }
                },
                "removable": "0",
                "rotational": "1",
                "sas_address": null,
                "sas_device_handle": null,
                "scheduler_mode": "deadline",
                "sectors": "73400320",
                "sectorsize": "512",
                "size": "35.00 GB",
                "support_discard": "0",
                "vendor": "VMware",
                "virtual": 1
            },
            "sr0": {
                "holders": [],
                "host": "SATA controller: VMware SATA AHCI controller",
                "links": {
                    "ids": [
                        "ata-VMware_Virtual_SATA_CDRW_Drive_00000000000000000001"
                    ],
                    "labels": [],
                    "masters": [],
                    "uuids": []
                },
                "model": "VMware SATA CD00",
                "partitions": {},
                "removable": "1",
                "rotational": "1",
                "sas_address": null,
                "sas_device_handle": null,
                "scheduler_mode": "deadline",
                "sectors": "2097151",
                "sectorsize": "512",
                "size": "1024.00 MB",
                "support_discard": "0",
                "vendor": "NECVMWar",
                "virtual": 1
            }
        },
        "ansible_distribution": "CentOS",
        "ansible_distribution_file_parsed": true,
        "ansible_distribution_file_path": "/etc/redhat-release",
        "ansible_distribution_file_variety": "RedHat",
        "ansible_distribution_major_version": "7",
        "ansible_distribution_release": "Core",
        "ansible_distribution_version": "7.9",
        "ansible_dns": {
            "nameservers": [
                "8.8.8.8"
            ]
        },
        "ansible_domain": "",
        "ansible_effective_group_id": 0,
        "ansible_effective_user_id": 0,
        "ansible_ens192": {
            "active": true,
            "device": "ens192",
            "features": {
                "busy_poll": "off [fixed]",
                "fcoe_mtu": "off [fixed]",
                "generic_receive_offload": "on",
                "generic_segmentation_offload": "on",
                "highdma": "on",
                "hw_tc_offload": "off [fixed]",
                "l2_fwd_offload": "off [fixed]",
                "large_receive_offload": "on",
                "loopback": "off [fixed]",
                "netns_local": "off [fixed]",
                "ntuple_filters": "off [fixed]",
                "receive_hashing": "on",
                "rx_all": "off [fixed]",
                "rx_checksumming": "on",
                "rx_fcs": "off [fixed]",
                "rx_gro_hw": "off [fixed]",
                "rx_udp_tunnel_port_offload": "off [fixed]",
                "rx_vlan_filter": "on [fixed]",
                "rx_vlan_offload": "on",
                "rx_vlan_stag_filter": "off [fixed]",
                "rx_vlan_stag_hw_parse": "off [fixed]",
                "scatter_gather": "on",
                "tcp_segmentation_offload": "on",
                "tx_checksum_fcoe_crc": "off [fixed]",
                "tx_checksum_ip_generic": "on",
                "tx_checksum_ipv4": "off [fixed]",
                "tx_checksum_ipv6": "off [fixed]",
                "tx_checksum_sctp": "off [fixed]",
                "tx_checksumming": "on",
                "tx_fcoe_segmentation": "off [fixed]",
                "tx_gre_csum_segmentation": "off [fixed]",
                "tx_gre_segmentation": "off [fixed]",
                "tx_gso_partial": "off [fixed]",
                "tx_gso_robust": "off [fixed]",
                "tx_ipip_segmentation": "off [fixed]",
                "tx_lockless": "off [fixed]",
                "tx_nocache_copy": "off",
                "tx_scatter_gather": "on",
                "tx_scatter_gather_fraglist": "off [fixed]",
                "tx_sctp_segmentation": "off [fixed]",
                "tx_sit_segmentation": "off [fixed]",
                "tx_tcp6_segmentation": "on",
                "tx_tcp_ecn_segmentation": "off [fixed]",
                "tx_tcp_mangleid_segmentation": "off",
                "tx_tcp_segmentation": "on",
                "tx_udp_tnl_csum_segmentation": "off [fixed]",
                "tx_udp_tnl_segmentation": "off [fixed]",
                "tx_vlan_offload": "on",
                "tx_vlan_stag_hw_insert": "off [fixed]",
                "udp_fragmentation_offload": "off [fixed]",
                "vlan_challenged": "off [fixed]"
            },
            "hw_timestamp_filters": [],
            "ipv4": {
                "address": "10.8.21.212",
                "broadcast": "10.8.21.255",
                "netmask": "255.255.255.0",
                "network": "10.8.21.0"
            },
            "ipv6": [
                {
                    "address": "fe80::cf92:3ff3:2f4e:69bb",
                    "prefix": "64",
                    "scope": "link"
                },
                {
                    "address": "fe80::512a:c6f7:2f4:6997",
                    "prefix": "64",
                    "scope": "link"
                },
                {
                    "address": "fe80::585:81cc:c9c3:2a60",
                    "prefix": "64",
                    "scope": "link"
                }
            ],
            "macaddress": "00:50:56:95:fc:11",
            "module": "vmxnet3",
            "mtu": 1500,
            "pciid": "0000:0b:00.0",
            "promisc": false,
            "speed": 10000,
            "timestamping": [
                "rx_software",
                "software"
            ],
            "type": "ether"
        },
        "ansible_env": {
            "HOME": "/root",
            "LANG": "C",
            "LC_ALL": "C",
            "LC_NUMERIC": "C",
            "LESSOPEN": "||/usr/bin/lesspipe.sh %s",
            "LOGNAME": "root",
            "LS_COLORS": "rs=0:di=01;34:ln=01;36:mh=00:pi=40;33:so=01;35:do=01;35:bd=40;33;01:cd=40;33;01:or=40;31;01:mi=01;05;37;41:su=37;41:sg=30;43:ca=30;41:tw=30;42:ow=34;42:st=37;44:ex=01;32:*.tar=01;31:*.tgz=01;31:*.arc=01;31:*.arj=01;31:*.taz=01;31:*.lha=01;31:*.lz4=01;31:*.lzh=01;31:*.lzma=01;31:*.tlz=01;31:*.txz=01;31:*.tzo=01;31:*.t7z=01;31:*.zip=01;31:*.z=01;31:*.Z=01;31:*.dz=01;31:*.gz=01;31:*.lrz=01;31:*.lz=01;31:*.lzo=01;31:*.xz=01;31:*.bz2=01;31:*.bz=01;31:*.tbz=01;31:*.tbz2=01;31:*.tz=01;31:*.deb=01;31:*.rpm=01;31:*.jar=01;31:*.war=01;31:*.ear=01;31:*.sar=01;31:*.rar=01;31:*.alz=01;31:*.ace=01;31:*.zoo=01;31:*.cpio=01;31:*.7z=01;31:*.rz=01;31:*.cab=01;31:*.jpg=01;35:*.jpeg=01;35:*.gif=01;35:*.bmp=01;35:*.pbm=01;35:*.pgm=01;35:*.ppm=01;35:*.tga=01;35:*.xbm=01;35:*.xpm=01;35:*.tif=01;35:*.tiff=01;35:*.png=01;35:*.svg=01;35:*.svgz=01;35:*.mng=01;35:*.pcx=01;35:*.mov=01;35:*.mpg=01;35:*.mpeg=01;35:*.m2v=01;35:*.mkv=01;35:*.webm=01;35:*.ogm=01;35:*.mp4=01;35:*.m4v=01;35:*.mp4v=01;35:*.vob=01;35:*.qt=01;35:*.nuv=01;35:*.wmv=01;35:*.asf=01;35:*.rm=01;35:*.rmvb=01;35:*.flc=01;35:*.avi=01;35:*.fli=01;35:*.flv=01;35:*.gl=01;35:*.dl=01;35:*.xcf=01;35:*.xwd=01;35:*.yuv=01;35:*.cgm=01;35:*.emf=01;35:*.axv=01;35:*.anx=01;35:*.ogv=01;35:*.ogx=01;35:*.aac=01;36:*.au=01;36:*.flac=01;36:*.mid=01;36:*.midi=01;36:*.mka=01;36:*.mp3=01;36:*.mpc=01;36:*.ogg=01;36:*.ra=01;36:*.wav=01;36:*.axa=01;36:*.oga=01;36:*.spx=01;36:*.xspf=01;36:",
            "MAIL": "/var/mail/root",
            "PATH": "/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin",
            "PWD": "/root",
            "SHELL": "/bin/bash",
            "SHLVL": "2",
            "SSH_CLIENT": "10.8.21.211 46810 22",
            "SSH_CONNECTION": "10.8.21.211 46810 10.8.21.212 22",
            "SSH_TTY": "/dev/pts/0",
            "TERM": "xterm",
            "USER": "root",
            "XDG_RUNTIME_DIR": "/run/user/0",
            "XDG_SESSION_ID": "322",
            "_": "/usr/bin/python"
        },
        "ansible_fibre_channel_wwn": [],
        "ansible_fips": false,
        "ansible_form_factor": "Other",
        "ansible_fqdn": "Ansible-Node1",
        "ansible_hostname": "Ansible-Node1",
        "ansible_hostnqn": "",
        "ansible_interfaces": [
            "lo",
            "ens192"
        ],
        "ansible_is_chroot": false,
        "ansible_iscsi_iqn": "",
        "ansible_kernel": "3.10.0-1160.6.1.el7.x86_64",
        "ansible_kernel_version": "#1 SMP Tue Nov 17 13:59:11 UTC 2020",
        "ansible_lo": {
            "active": true,
            "device": "lo",
            "features": {
                "busy_poll": "off [fixed]",
                "fcoe_mtu": "off [fixed]",
                "generic_receive_offload": "on",
                "generic_segmentation_offload": "on",
                "highdma": "on [fixed]",
                "hw_tc_offload": "off [fixed]",
                "l2_fwd_offload": "off [fixed]",
                "large_receive_offload": "off [fixed]",
                "loopback": "on [fixed]",
                "netns_local": "on [fixed]",
                "ntuple_filters": "off [fixed]",
                "receive_hashing": "off [fixed]",
                "rx_all": "off [fixed]",
                "rx_checksumming": "on [fixed]",
                "rx_fcs": "off [fixed]",
                "rx_gro_hw": "off [fixed]",
                "rx_udp_tunnel_port_offload": "off [fixed]",
                "rx_vlan_filter": "off [fixed]",
                "rx_vlan_offload": "off [fixed]",
                "rx_vlan_stag_filter": "off [fixed]",
                "rx_vlan_stag_hw_parse": "off [fixed]",
                "scatter_gather": "on",
                "tcp_segmentation_offload": "on",
                "tx_checksum_fcoe_crc": "off [fixed]",
                "tx_checksum_ip_generic": "on [fixed]",
                "tx_checksum_ipv4": "off [fixed]",
                "tx_checksum_ipv6": "off [fixed]",
                "tx_checksum_sctp": "on [fixed]",
                "tx_checksumming": "on",
                "tx_fcoe_segmentation": "off [fixed]",
                "tx_gre_csum_segmentation": "off [fixed]",
                "tx_gre_segmentation": "off [fixed]",
                "tx_gso_partial": "off [fixed]",
                "tx_gso_robust": "off [fixed]",
                "tx_ipip_segmentation": "off [fixed]",
                "tx_lockless": "on [fixed]",
                "tx_nocache_copy": "off [fixed]",
                "tx_scatter_gather": "on [fixed]",
                "tx_scatter_gather_fraglist": "on [fixed]",
                "tx_sctp_segmentation": "on",
                "tx_sit_segmentation": "off [fixed]",
                "tx_tcp6_segmentation": "on",
                "tx_tcp_ecn_segmentation": "on",
                "tx_tcp_mangleid_segmentation": "on",
                "tx_tcp_segmentation": "on",
                "tx_udp_tnl_csum_segmentation": "off [fixed]",
                "tx_udp_tnl_segmentation": "off [fixed]",
                "tx_vlan_offload": "off [fixed]",
                "tx_vlan_stag_hw_insert": "off [fixed]",
                "udp_fragmentation_offload": "on",
                "vlan_challenged": "on [fixed]"
            },
            "hw_timestamp_filters": [],
            "ipv4": {
                "address": "127.0.0.1",
                "broadcast": "",
                "netmask": "255.0.0.0",
                "network": "127.0.0.0"
            },
            "ipv6": [
                {
                    "address": "::1",
                    "prefix": "128",
                    "scope": "host"
                }
            ],
            "mtu": 65536,
            "promisc": false,
            "timestamping": [
                "rx_software",
                "software"
            ],
            "type": "loopback"
        },
        "ansible_local": {},
        "ansible_lsb": {},
        "ansible_lvm": {
            "lvs": {
                "root": {
                    "size_g": "30.50",
                    "vg": "centos"
                },
                "swap": {
                    "size_g": "3.50",
                    "vg": "centos"
                }
            },
            "pvs": {
                "/dev/sda2": {
                    "free_g": "0",
                    "size_g": "34.00",
                    "vg": "centos"
                }
            },
            "vgs": {
                "centos": {
                    "free_g": "0",
                    "num_lvs": "2",
                    "num_pvs": "1",
                    "size_g": "34.00"
                }
            }
        },
        "ansible_machine": "x86_64",
        "ansible_machine_id": "24afba1c0d404ea6ab152038697e7c65",
        "ansible_memfree_mb": 3068,
        "ansible_memory_mb": {
            "nocache": {
                "free": 3506,
                "used": 283
            },
            "real": {
                "free": 3068,
                "total": 3789,
                "used": 721
            },
            "swap": {
                "cached": 0,
                "free": 3583,
                "total": 3583,
                "used": 0
            }
        },
        "ansible_memtotal_mb": 3789,
        "ansible_mounts": [
            {
                "block_available": 199165,
                "block_size": 4096,
                "block_total": 259584,
                "block_used": 60419,
                "device": "/dev/sda1",
                "fstype": "xfs",
                "inode_available": 523947,
                "inode_total": 524288,
                "inode_used": 341,
                "mount": "/boot",
                "options": "rw,relatime,attr2,inode64,noquota",
                "size_available": 815779840,
                "size_total": 1063256064,
                "uuid": "289396f7-8dee-422e-845d-364c112ac85c"
            },
            {
                "block_available": 7308661,
                "block_size": 4096,
                "block_total": 7990465,
                "block_used": 681804,
                "device": "/dev/mapper/centos-root",
                "fstype": "xfs",
                "inode_available": 15901387,
                "inode_total": 15988736,
                "inode_used": 87349,
                "mount": "/",
                "options": "rw,relatime,attr2,inode64,noquota",
                "size_available": 29936275456,
                "size_total": 32728944640,
                "uuid": "57996173-909b-4841-ad26-700841f5259e"
            }
        ],
        "ansible_nodename": "Ansible-Node1",
        "ansible_os_family": "RedHat",
        "ansible_pkg_mgr": "yum",
        "ansible_proc_cmdline": {
            "BOOT_IMAGE": "/vmlinuz-3.10.0-1160.6.1.el7.x86_64",
            "LANG": "en_US.UTF-8",
            "crashkernel": "auto",
            "quiet": true,
            "rd.lvm.lv": [
                "centos/root",
                "centos/swap"
            ],
            "rhgb": true,
            "ro": true,
            "root": "/dev/mapper/centos-root",
            "spectre_v2": "retpoline"
        },
        "ansible_processor": [
            "0",
            "GenuineIntel",
            "Intel(R) Xeon(R) Silver 4110 CPU @ 2.10GHz",
            "1",
            "GenuineIntel",
            "Intel(R) Xeon(R) Silver 4110 CPU @ 2.10GHz",
            "2",
            "GenuineIntel",
            "Intel(R) Xeon(R) Silver 4110 CPU @ 2.10GHz",
            "3",
            "GenuineIntel",
            "Intel(R) Xeon(R) Silver 4110 CPU @ 2.10GHz"
        ],
        "ansible_processor_cores": 1,
        "ansible_processor_count": 4,
        "ansible_processor_nproc": 4,
        "ansible_processor_threads_per_core": 1,
        "ansible_processor_vcpus": 4,
        "ansible_product_name": "VMware Virtual Platform",
        "ansible_product_serial": "VMware-42 15 8a f2 9f c1 a1 95-61 be 72 71 22 8e cd d4",
        "ansible_product_uuid": "F28A1542-C19F-95A1-61BE-7271228ECDD4",
        "ansible_product_version": "None",
        "ansible_python": {
            "executable": "/usr/bin/python",
            "has_sslcontext": true,
            "type": "CPython",
            "version": {
                "major": 2,
                "micro": 5,
                "minor": 7,
                "releaselevel": "final",
                "serial": 0
            },
            "version_info": [
                2,
                7,
                5,
                "final",
                0
            ]
        },
        "ansible_python_version": "2.7.5",
        "ansible_real_group_id": 0,
        "ansible_real_user_id": 0,
        "ansible_selinux": {
            "status": "disabled"
        },
        "ansible_selinux_python_present": true,
        "ansible_service_mgr": "systemd",
        "ansible_ssh_host_key_ecdsa_public": "AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBPnLJJY8CIKPN4R+oL8xEE7vkAx8P2vIb3EfqZewxEef75mSdiXMMIHOWDpIcojd+r63tQ7P1snFuADRMJcxWbQ=",
        "ansible_ssh_host_key_ecdsa_public_keytype": "ecdsa-sha2-nistp256",
        "ansible_ssh_host_key_ed25519_public": "AAAAC3NzaC1lZDI1NTE5AAAAIA5jdSAH/icn4KaCPnfZrvvjxQsavBgOLDXLjBnvmAfK",
        "ansible_ssh_host_key_ed25519_public_keytype": "ssh-ed25519",
        "ansible_ssh_host_key_rsa_public": "AAAAB3NzaC1yc2EAAAADAQABAAABAQC8v3eR4uOFXmoAPsVa5uK9NGg1aeJKjR+Jt2jV//rxMUJsfldErG0vfQFtZiAJg3Qjcv1m0EWbLaXapQbXQ1HWdEoRhs7wXhZPQX+Nl8O3haFSaMrhDRYQDpUIkkBfhH8FtRE3YB2C09NFPRkub2hXXW1i8X0l+XynJ/2+aXa5qSpQXwsZXmSjByDgSqzPZ/y7gz0rreevW0FkoiJIN4vfQBb1Q4kjA4rPQE+XiQkLhDXx06GcDV3+1lkznv6oInBqK5JCd1Dx67Q3ZhAadV3O5WD1teYTQEgmpFS/wi50gYIPVYP0DaYVbSZ6VJ3uic5ywMS1qUf+NluvbDofHtUt",
        "ansible_ssh_host_key_rsa_public_keytype": "ssh-rsa",
        "ansible_swapfree_mb": 3583,
        "ansible_swaptotal_mb": 3583,
        "ansible_system": "Linux",
        "ansible_system_capabilities": [
            "cap_chown",
            "cap_dac_override",
            "cap_dac_read_search",
            "cap_fowner",
            "cap_fsetid",
            "cap_kill",
            "cap_setgid",
            "cap_setuid",
            "cap_setpcap",
            "cap_linux_immutable",
            "cap_net_bind_service",
            "cap_net_broadcast",
            "cap_net_admin",
            "cap_net_raw",
            "cap_ipc_lock",
            "cap_ipc_owner",
            "cap_sys_module",
            "cap_sys_rawio",
            "cap_sys_chroot",
            "cap_sys_ptrace",
            "cap_sys_pacct",
            "cap_sys_admin",
            "cap_sys_boot",
            "cap_sys_nice",
            "cap_sys_resource",
            "cap_sys_time",
            "cap_sys_tty_config",
            "cap_mknod",
            "cap_lease",
            "cap_audit_write",
            "cap_audit_control",
            "cap_setfcap",
            "cap_mac_override",
            "cap_mac_admin",
            "cap_syslog",
            "35",
            "36+ep"
        ],
        "ansible_system_capabilities_enforced": "True",
        "ansible_system_vendor": "VMware, Inc.",
        "ansible_uptime_seconds": 948249,
        "ansible_user_dir": "/root",
        "ansible_user_gecos": "root",
        "ansible_user_gid": 0,
        "ansible_user_id": "root",
        "ansible_user_shell": "/bin/bash",
        "ansible_user_uid": 0,
        "ansible_userspace_architecture": "x86_64",
        "ansible_userspace_bits": "64",
        "ansible_virtualization_role": "guest",
        "ansible_virtualization_type": "VMware",
        "discovered_interpreter_python": "/usr/bin/python",
        "gather_subset": [
            "all"
        ],
        "module_setup": true
    },
    "changed": false,
    "deprecations": [],
    "warnings": []
}
10.8.21.213 | SUCCESS => {
    "ansible_facts": {
        "ansible_all_ipv4_addresses": [
            "10.8.21.213"
        ],
        "ansible_all_ipv6_addresses": [
            "fe80::cf92:3ff3:2f4e:69bb",
            "fe80::512a:c6f7:2f4:6997",
            "fe80::585:81cc:c9c3:2a60"
        ],
        "ansible_apparmor": {
            "status": "disabled"
        },
        "ansible_architecture": "x86_64",
        "ansible_bios_date": "12/12/2018",
        "ansible_bios_vendor": "Phoenix Technologies LTD",
        "ansible_bios_version": "6.00",
        "ansible_board_asset_tag": "NA",
        "ansible_board_name": "440BX Desktop Reference Platform",
        "ansible_board_serial": "None",
        "ansible_board_vendor": "Intel Corporation",
        "ansible_board_version": "None",
        "ansible_chassis_asset_tag": "No Asset Tag",
        "ansible_chassis_serial": "None",
        "ansible_chassis_vendor": "No Enclosure",
        "ansible_chassis_version": "N/A",
        "ansible_cmdline": {
            "BOOT_IMAGE": "/vmlinuz-3.10.0-1160.6.1.el7.x86_64",
            "LANG": "en_US.UTF-8",
            "crashkernel": "auto",
            "quiet": true,
            "rd.lvm.lv": "centos/swap",
            "rhgb": true,
            "ro": true,
            "root": "/dev/mapper/centos-root",
            "spectre_v2": "retpoline"
        },
        "ansible_date_time": {
            "date": "2021-04-06",
            "day": "06",
            "epoch": "1617675223",
            "hour": "10",
            "iso8601": "2021-04-06T02:13:43Z",
            "iso8601_basic": "20210406T101343965457",
            "iso8601_basic_short": "20210406T101343",
            "iso8601_micro": "2021-04-06T02:13:43.965457Z",
            "minute": "13",
            "month": "04",
            "second": "43",
            "time": "10:13:43",
            "tz": "CST",
            "tz_offset": "+0800",
            "weekday": "Tuesday",
            "weekday_number": "2",
            "weeknumber": "14",
            "year": "2021"
        },
        "ansible_default_ipv4": {
            "address": "10.8.21.213",
            "alias": "ens192",
            "broadcast": "10.8.21.255",
            "gateway": "10.8.21.253",
            "interface": "ens192",
            "macaddress": "00:50:56:95:25:3a",
            "mtu": 1500,
            "netmask": "255.255.255.0",
            "network": "10.8.21.0",
            "type": "ether"
        },
        "ansible_default_ipv6": {},
        "ansible_device_links": {
            "ids": {
                "dm-0": [
                    "dm-name-centos-root",
                    "dm-uuid-LVM-rR3AARPm531pFjQLdtiVmR5zFdPLztPSmxd2xIgwq5vIk9mZ2YiPB3e8tedv6TrN"
                ],
                "dm-1": [
                    "dm-name-centos-swap",
                    "dm-uuid-LVM-rR3AARPm531pFjQLdtiVmR5zFdPLztPSvV7InsduEjFcsNuIAGQOB4a4Rm2otU4R"
                ],
                "sda2": [
                    "lvm-pv-uuid-iqncfw-bGCP-AKN5-k8CP-ShoB-iHz7-mmUohd"
                ],
                "sr0": [
                    "ata-VMware_Virtual_SATA_CDRW_Drive_00000000000000000001"
                ]
            },
            "labels": {},
            "masters": {
                "sda2": [
                    "dm-0",
                    "dm-1"
                ]
            },
            "uuids": {
                "dm-0": [
                    "57996173-909b-4841-ad26-700841f5259e"
                ],
                "dm-1": [
                    "093b556a-2d87-4785-8ab0-0e68a099b4dc"
                ],
                "sda1": [
                    "289396f7-8dee-422e-845d-364c112ac85c"
                ]
            }
        },
        "ansible_devices": {
            "dm-0": {
                "holders": [],
                "host": "",
                "links": {
                    "ids": [
                        "dm-name-centos-root",
                        "dm-uuid-LVM-rR3AARPm531pFjQLdtiVmR5zFdPLztPSmxd2xIgwq5vIk9mZ2YiPB3e8tedv6TrN"
                    ],
                    "labels": [],
                    "masters": [],
                    "uuids": [
                        "57996173-909b-4841-ad26-700841f5259e"
                    ]
                },
                "model": null,
                "partitions": {},
                "removable": "0",
                "rotational": "1",
                "sas_address": null,
                "sas_device_handle": null,
                "scheduler_mode": "",
                "sectors": "63954944",
                "sectorsize": "512",
                "size": "30.50 GB",
                "support_discard": "0",
                "vendor": null,
                "virtual": 1
            },
            "dm-1": {
                "holders": [],
                "host": "",
                "links": {
                    "ids": [
                        "dm-name-centos-swap",
                        "dm-uuid-LVM-rR3AARPm531pFjQLdtiVmR5zFdPLztPSvV7InsduEjFcsNuIAGQOB4a4Rm2otU4R"
                    ],
                    "labels": [],
                    "masters": [],
                    "uuids": [
                        "093b556a-2d87-4785-8ab0-0e68a099b4dc"
                    ]
                },
                "model": null,
                "partitions": {},
                "removable": "0",
                "rotational": "1",
                "sas_address": null,
                "sas_device_handle": null,
                "scheduler_mode": "",
                "sectors": "7340032",
                "sectorsize": "512",
                "size": "3.50 GB",
                "support_discard": "0",
                "vendor": null,
                "virtual": 1
            },
            "sda": {
                "holders": [],
                "host": "Serial Attached SCSI controller: VMware PVSCSI SCSI Controller (rev 02)",
                "links": {
                    "ids": [],
                    "labels": [],
                    "masters": [],
                    "uuids": []
                },
                "model": "Virtual disk",
                "partitions": {
                    "sda1": {
                        "holders": [],
                        "links": {
                            "ids": [],
                            "labels": [],
                            "masters": [],
                            "uuids": [
                                "289396f7-8dee-422e-845d-364c112ac85c"
                            ]
                        },
                        "sectors": "2097152",
                        "sectorsize": 512,
                        "size": "1.00 GB",
                        "start": "2048",
                        "uuid": "289396f7-8dee-422e-845d-364c112ac85c"
                    },
                    "sda2": {
                        "holders": [
                            "centos-root",
                            "centos-swap"
                        ],
                        "links": {
                            "ids": [
                                "lvm-pv-uuid-iqncfw-bGCP-AKN5-k8CP-ShoB-iHz7-mmUohd"
                            ],
                            "labels": [],
                            "masters": [
                                "dm-0",
                                "dm-1"
                            ],
                            "uuids": []
                        },
                        "sectors": "71301120",
                        "sectorsize": 512,
                        "size": "34.00 GB",
                        "start": "2099200",
                        "uuid": null
                    }
                },
                "removable": "0",
                "rotational": "1",
                "sas_address": null,
                "sas_device_handle": null,
                "scheduler_mode": "deadline",
                "sectors": "73400320",
                "sectorsize": "512",
                "size": "35.00 GB",
                "support_discard": "0",
                "vendor": "VMware",
                "virtual": 1
            },
            "sr0": {
                "holders": [],
                "host": "SATA controller: VMware SATA AHCI controller",
                "links": {
                    "ids": [
                        "ata-VMware_Virtual_SATA_CDRW_Drive_00000000000000000001"
                    ],
                    "labels": [],
                    "masters": [],
                    "uuids": []
                },
                "model": "VMware SATA CD00",
                "partitions": {},
                "removable": "1",
                "rotational": "1",
                "sas_address": null,
                "sas_device_handle": null,
                "scheduler_mode": "deadline",
                "sectors": "2097151",
                "sectorsize": "512",
                "size": "1024.00 MB",
                "support_discard": "0",
                "vendor": "NECVMWar",
                "virtual": 1
            }
        },
        "ansible_distribution": "CentOS",
        "ansible_distribution_file_parsed": true,
        "ansible_distribution_file_path": "/etc/redhat-release",
        "ansible_distribution_file_variety": "RedHat",
        "ansible_distribution_major_version": "7",
        "ansible_distribution_release": "Core",
        "ansible_distribution_version": "7.9",
        "ansible_dns": {
            "nameservers": [
                "8.8.8.8"
            ]
        },
        "ansible_domain": "",
        "ansible_effective_group_id": 0,
        "ansible_effective_user_id": 0,
        "ansible_ens192": {
            "active": true,
            "device": "ens192",
            "features": {
                "busy_poll": "off [fixed]",
                "fcoe_mtu": "off [fixed]",
                "generic_receive_offload": "on",
                "generic_segmentation_offload": "on",
                "highdma": "on",
                "hw_tc_offload": "off [fixed]",
                "l2_fwd_offload": "off [fixed]",
                "large_receive_offload": "on",
                "loopback": "off [fixed]",
                "netns_local": "off [fixed]",
                "ntuple_filters": "off [fixed]",
                "receive_hashing": "on",
                "rx_all": "off [fixed]",
                "rx_checksumming": "on",
                "rx_fcs": "off [fixed]",
                "rx_gro_hw": "off [fixed]",
                "rx_udp_tunnel_port_offload": "off [fixed]",
                "rx_vlan_filter": "on [fixed]",
                "rx_vlan_offload": "on",
                "rx_vlan_stag_filter": "off [fixed]",
                "rx_vlan_stag_hw_parse": "off [fixed]",
                "scatter_gather": "on",
                "tcp_segmentation_offload": "on",
                "tx_checksum_fcoe_crc": "off [fixed]",
                "tx_checksum_ip_generic": "on",
                "tx_checksum_ipv4": "off [fixed]",
                "tx_checksum_ipv6": "off [fixed]",
                "tx_checksum_sctp": "off [fixed]",
                "tx_checksumming": "on",
                "tx_fcoe_segmentation": "off [fixed]",
                "tx_gre_csum_segmentation": "off [fixed]",
                "tx_gre_segmentation": "off [fixed]",
                "tx_gso_partial": "off [fixed]",
                "tx_gso_robust": "off [fixed]",
                "tx_ipip_segmentation": "off [fixed]",
                "tx_lockless": "off [fixed]",
                "tx_nocache_copy": "off",
                "tx_scatter_gather": "on",
                "tx_scatter_gather_fraglist": "off [fixed]",
                "tx_sctp_segmentation": "off [fixed]",
                "tx_sit_segmentation": "off [fixed]",
                "tx_tcp6_segmentation": "on",
                "tx_tcp_ecn_segmentation": "off [fixed]",
                "tx_tcp_mangleid_segmentation": "off",
                "tx_tcp_segmentation": "on",
                "tx_udp_tnl_csum_segmentation": "off [fixed]",
                "tx_udp_tnl_segmentation": "off [fixed]",
                "tx_vlan_offload": "on",
                "tx_vlan_stag_hw_insert": "off [fixed]",
                "udp_fragmentation_offload": "off [fixed]",
                "vlan_challenged": "off [fixed]"
            },
            "hw_timestamp_filters": [],
            "ipv4": {
                "address": "10.8.21.213",
                "broadcast": "10.8.21.255",
                "netmask": "255.255.255.0",
                "network": "10.8.21.0"
            },
            "ipv6": [
                {
                    "address": "fe80::cf92:3ff3:2f4e:69bb",
                    "prefix": "64",
                    "scope": "link"
                },
                {
                    "address": "fe80::512a:c6f7:2f4:6997",
                    "prefix": "64",
                    "scope": "link"
                },
                {
                    "address": "fe80::585:81cc:c9c3:2a60",
                    "prefix": "64",
                    "scope": "link"
                }
            ],
            "macaddress": "00:50:56:95:25:3a",
            "module": "vmxnet3",
            "mtu": 1500,
            "pciid": "0000:0b:00.0",
            "promisc": false,
            "speed": 10000,
            "timestamping": [
                "rx_software",
                "software"
            ],
            "type": "ether"
        },
        "ansible_env": {
            "HOME": "/root",
            "LANG": "C",
            "LC_ALL": "C",
            "LC_NUMERIC": "C",
            "LESSOPEN": "||/usr/bin/lesspipe.sh %s",
            "LOGNAME": "root",
            "LS_COLORS": "rs=0:di=01;34:ln=01;36:mh=00:pi=40;33:so=01;35:do=01;35:bd=40;33;01:cd=40;33;01:or=40;31;01:mi=01;05;37;41:su=37;41:sg=30;43:ca=30;41:tw=30;42:ow=34;42:st=37;44:ex=01;32:*.tar=01;31:*.tgz=01;31:*.arc=01;31:*.arj=01;31:*.taz=01;31:*.lha=01;31:*.lz4=01;31:*.lzh=01;31:*.lzma=01;31:*.tlz=01;31:*.txz=01;31:*.tzo=01;31:*.t7z=01;31:*.zip=01;31:*.z=01;31:*.Z=01;31:*.dz=01;31:*.gz=01;31:*.lrz=01;31:*.lz=01;31:*.lzo=01;31:*.xz=01;31:*.bz2=01;31:*.bz=01;31:*.tbz=01;31:*.tbz2=01;31:*.tz=01;31:*.deb=01;31:*.rpm=01;31:*.jar=01;31:*.war=01;31:*.ear=01;31:*.sar=01;31:*.rar=01;31:*.alz=01;31:*.ace=01;31:*.zoo=01;31:*.cpio=01;31:*.7z=01;31:*.rz=01;31:*.cab=01;31:*.jpg=01;35:*.jpeg=01;35:*.gif=01;35:*.bmp=01;35:*.pbm=01;35:*.pgm=01;35:*.ppm=01;35:*.tga=01;35:*.xbm=01;35:*.xpm=01;35:*.tif=01;35:*.tiff=01;35:*.png=01;35:*.svg=01;35:*.svgz=01;35:*.mng=01;35:*.pcx=01;35:*.mov=01;35:*.mpg=01;35:*.mpeg=01;35:*.m2v=01;35:*.mkv=01;35:*.webm=01;35:*.ogm=01;35:*.mp4=01;35:*.m4v=01;35:*.mp4v=01;35:*.vob=01;35:*.qt=01;35:*.nuv=01;35:*.wmv=01;35:*.asf=01;35:*.rm=01;35:*.rmvb=01;35:*.flc=01;35:*.avi=01;35:*.fli=01;35:*.flv=01;35:*.gl=01;35:*.dl=01;35:*.xcf=01;35:*.xwd=01;35:*.yuv=01;35:*.cgm=01;35:*.emf=01;35:*.axv=01;35:*.anx=01;35:*.ogv=01;35:*.ogx=01;35:*.aac=01;36:*.au=01;36:*.flac=01;36:*.mid=01;36:*.midi=01;36:*.mka=01;36:*.mp3=01;36:*.mpc=01;36:*.ogg=01;36:*.ra=01;36:*.wav=01;36:*.axa=01;36:*.oga=01;36:*.spx=01;36:*.xspf=01;36:",
            "MAIL": "/var/mail/root",
            "PATH": "/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin",
            "PWD": "/root",
            "SHELL": "/bin/bash",
            "SHLVL": "2",
            "SSH_CLIENT": "10.8.21.211 34056 22",
            "SSH_CONNECTION": "10.8.21.211 34056 10.8.21.213 22",
            "SSH_TTY": "/dev/pts/0",
            "TERM": "xterm",
            "USER": "root",
            "XDG_RUNTIME_DIR": "/run/user/0",
            "XDG_SESSION_ID": "313",
            "_": "/usr/bin/python"
        },
        "ansible_fibre_channel_wwn": [],
        "ansible_fips": false,
        "ansible_form_factor": "Other",
        "ansible_fqdn": "Ansible-Node2",
        "ansible_hostname": "Ansible-Node2",
        "ansible_hostnqn": "",
        "ansible_interfaces": [
            "lo",
            "ens192"
        ],
        "ansible_is_chroot": false,
        "ansible_iscsi_iqn": "",
        "ansible_kernel": "3.10.0-1160.6.1.el7.x86_64",
        "ansible_kernel_version": "#1 SMP Tue Nov 17 13:59:11 UTC 2020",
        "ansible_lo": {
            "active": true,
            "device": "lo",
            "features": {
                "busy_poll": "off [fixed]",
                "fcoe_mtu": "off [fixed]",
                "generic_receive_offload": "on",
                "generic_segmentation_offload": "on",
                "highdma": "on [fixed]",
                "hw_tc_offload": "off [fixed]",
                "l2_fwd_offload": "off [fixed]",
                "large_receive_offload": "off [fixed]",
                "loopback": "on [fixed]",
                "netns_local": "on [fixed]",
                "ntuple_filters": "off [fixed]",
                "receive_hashing": "off [fixed]",
                "rx_all": "off [fixed]",
                "rx_checksumming": "on [fixed]",
                "rx_fcs": "off [fixed]",
                "rx_gro_hw": "off [fixed]",
                "rx_udp_tunnel_port_offload": "off [fixed]",
                "rx_vlan_filter": "off [fixed]",
                "rx_vlan_offload": "off [fixed]",
                "rx_vlan_stag_filter": "off [fixed]",
                "rx_vlan_stag_hw_parse": "off [fixed]",
                "scatter_gather": "on",
                "tcp_segmentation_offload": "on",
                "tx_checksum_fcoe_crc": "off [fixed]",
                "tx_checksum_ip_generic": "on [fixed]",
                "tx_checksum_ipv4": "off [fixed]",
                "tx_checksum_ipv6": "off [fixed]",
                "tx_checksum_sctp": "on [fixed]",
                "tx_checksumming": "on",
                "tx_fcoe_segmentation": "off [fixed]",
                "tx_gre_csum_segmentation": "off [fixed]",
                "tx_gre_segmentation": "off [fixed]",
                "tx_gso_partial": "off [fixed]",
                "tx_gso_robust": "off [fixed]",
                "tx_ipip_segmentation": "off [fixed]",
                "tx_lockless": "on [fixed]",
                "tx_nocache_copy": "off [fixed]",
                "tx_scatter_gather": "on [fixed]",
                "tx_scatter_gather_fraglist": "on [fixed]",
                "tx_sctp_segmentation": "on",
                "tx_sit_segmentation": "off [fixed]",
                "tx_tcp6_segmentation": "on",
                "tx_tcp_ecn_segmentation": "on",
                "tx_tcp_mangleid_segmentation": "on",
                "tx_tcp_segmentation": "on",
                "tx_udp_tnl_csum_segmentation": "off [fixed]",
                "tx_udp_tnl_segmentation": "off [fixed]",
                "tx_vlan_offload": "off [fixed]",
                "tx_vlan_stag_hw_insert": "off [fixed]",
                "udp_fragmentation_offload": "on",
                "vlan_challenged": "on [fixed]"
            },
            "hw_timestamp_filters": [],
            "ipv4": {
                "address": "127.0.0.1",
                "broadcast": "",
                "netmask": "255.0.0.0",
                "network": "127.0.0.0"
            },
            "ipv6": [
                {
                    "address": "::1",
                    "prefix": "128",
                    "scope": "host"
                }
            ],
            "mtu": 65536,
            "promisc": false,
            "timestamping": [
                "rx_software",
                "software"
            ],
            "type": "loopback"
        },
        "ansible_local": {},
        "ansible_lsb": {},
        "ansible_lvm": {
            "lvs": {
                "root": {
                    "size_g": "30.50",
                    "vg": "centos"
                },
                "swap": {
                    "size_g": "3.50",
                    "vg": "centos"
                }
            },
            "pvs": {
                "/dev/sda2": {
                    "free_g": "0",
                    "size_g": "34.00",
                    "vg": "centos"
                }
            },
            "vgs": {
                "centos": {
                    "free_g": "0",
                    "num_lvs": "2",
                    "num_pvs": "1",
                    "size_g": "34.00"
                }
            }
        },
        "ansible_machine": "x86_64",
        "ansible_machine_id": "24afba1c0d404ea6ab152038697e7c65",
        "ansible_memfree_mb": 3069,
        "ansible_memory_mb": {
            "nocache": {
                "free": 3504,
                "used": 285
            },
            "real": {
                "free": 3069,
                "total": 3789,
                "used": 720
            },
            "swap": {
                "cached": 0,
                "free": 3583,
                "total": 3583,
                "used": 0
            }
        },
        "ansible_memtotal_mb": 3789,
        "ansible_mounts": [
            {
                "block_available": 199165,
                "block_size": 4096,
                "block_total": 259584,
                "block_used": 60419,
                "device": "/dev/sda1",
                "fstype": "xfs",
                "inode_available": 523947,
                "inode_total": 524288,
                "inode_used": 341,
                "mount": "/boot",
                "options": "rw,relatime,attr2,inode64,noquota",
                "size_available": 815779840,
                "size_total": 1063256064,
                "uuid": "289396f7-8dee-422e-845d-364c112ac85c"
            },
            {
                "block_available": 7325131,
                "block_size": 4096,
                "block_total": 7990465,
                "block_used": 665334,
                "device": "/dev/mapper/centos-root",
                "fstype": "xfs",
                "inode_available": 15901544,
                "inode_total": 15988736,
                "inode_used": 87192,
                "mount": "/",
                "options": "rw,relatime,attr2,inode64,noquota",
                "size_available": 30003736576,
                "size_total": 32728944640,
                "uuid": "57996173-909b-4841-ad26-700841f5259e"
            }
        ],
        "ansible_nodename": "Ansible-Node2",
        "ansible_os_family": "RedHat",
        "ansible_pkg_mgr": "yum",
        "ansible_proc_cmdline": {
            "BOOT_IMAGE": "/vmlinuz-3.10.0-1160.6.1.el7.x86_64",
            "LANG": "en_US.UTF-8",
            "crashkernel": "auto",
            "quiet": true,
            "rd.lvm.lv": [
                "centos/root",
                "centos/swap"
            ],
            "rhgb": true,
            "ro": true,
            "root": "/dev/mapper/centos-root",
            "spectre_v2": "retpoline"
        },
        "ansible_processor": [
            "0",
            "GenuineIntel",
            "Intel(R) Xeon(R) Silver 4110 CPU @ 2.10GHz",
            "1",
            "GenuineIntel",
            "Intel(R) Xeon(R) Silver 4110 CPU @ 2.10GHz",
            "2",
            "GenuineIntel",
            "Intel(R) Xeon(R) Silver 4110 CPU @ 2.10GHz",
            "3",
            "GenuineIntel",
            "Intel(R) Xeon(R) Silver 4110 CPU @ 2.10GHz"
        ],
        "ansible_processor_cores": 1,
        "ansible_processor_count": 4,
        "ansible_processor_nproc": 4,
        "ansible_processor_threads_per_core": 1,
        "ansible_processor_vcpus": 4,
        "ansible_product_name": "VMware Virtual Platform",
        "ansible_product_serial": "VMware-42 15 36 c5 9b 99 48 81-1b a1 c6 32 8a 04 b4 66",
        "ansible_product_uuid": "C5361542-999B-8148-1BA1-C6328A04B466",
        "ansible_product_version": "None",
        "ansible_python": {
            "executable": "/usr/bin/python",
            "has_sslcontext": true,
            "type": "CPython",
            "version": {
                "major": 2,
                "micro": 5,
                "minor": 7,
                "releaselevel": "final",
                "serial": 0
            },
            "version_info": [
                2,
                7,
                5,
                "final",
                0
            ]
        },
        "ansible_python_version": "2.7.5",
        "ansible_real_group_id": 0,
        "ansible_real_user_id": 0,
        "ansible_selinux": {
            "status": "disabled"
        },
        "ansible_selinux_python_present": true,
        "ansible_service_mgr": "systemd",
        "ansible_ssh_host_key_ecdsa_public": "AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBPnLJJY8CIKPN4R+oL8xEE7vkAx8P2vIb3EfqZewxEef75mSdiXMMIHOWDpIcojd+r63tQ7P1snFuADRMJcxWbQ=",
        "ansible_ssh_host_key_ecdsa_public_keytype": "ecdsa-sha2-nistp256",
        "ansible_ssh_host_key_ed25519_public": "AAAAC3NzaC1lZDI1NTE5AAAAIA5jdSAH/icn4KaCPnfZrvvjxQsavBgOLDXLjBnvmAfK",
        "ansible_ssh_host_key_ed25519_public_keytype": "ssh-ed25519",
        "ansible_ssh_host_key_rsa_public": "AAAAB3NzaC1yc2EAAAADAQABAAABAQC8v3eR4uOFXmoAPsVa5uK9NGg1aeJKjR+Jt2jV//rxMUJsfldErG0vfQFtZiAJg3Qjcv1m0EWbLaXapQbXQ1HWdEoRhs7wXhZPQX+Nl8O3haFSaMrhDRYQDpUIkkBfhH8FtRE3YB2C09NFPRkub2hXXW1i8X0l+XynJ/2+aXa5qSpQXwsZXmSjByDgSqzPZ/y7gz0rreevW0FkoiJIN4vfQBb1Q4kjA4rPQE+XiQkLhDXx06GcDV3+1lkznv6oInBqK5JCd1Dx67Q3ZhAadV3O5WD1teYTQEgmpFS/wi50gYIPVYP0DaYVbSZ6VJ3uic5ywMS1qUf+NluvbDofHtUt",
        "ansible_ssh_host_key_rsa_public_keytype": "ssh-rsa",
        "ansible_swapfree_mb": 3583,
        "ansible_swaptotal_mb": 3583,
        "ansible_system": "Linux",
        "ansible_system_capabilities": [
            "cap_chown",
            "cap_dac_override",
            "cap_dac_read_search",
            "cap_fowner",
            "cap_fsetid",
            "cap_kill",
            "cap_setgid",
            "cap_setuid",
            "cap_setpcap",
            "cap_linux_immutable",
            "cap_net_bind_service",
            "cap_net_broadcast",
            "cap_net_admin",
            "cap_net_raw",
            "cap_ipc_lock",
            "cap_ipc_owner",
            "cap_sys_module",
            "cap_sys_rawio",
            "cap_sys_chroot",
            "cap_sys_ptrace",
            "cap_sys_pacct",
            "cap_sys_admin",
            "cap_sys_boot",
            "cap_sys_nice",
            "cap_sys_resource",
            "cap_sys_time",
            "cap_sys_tty_config",
            "cap_mknod",
            "cap_lease",
            "cap_audit_write",
            "cap_audit_control",
            "cap_setfcap",
            "cap_mac_override",
            "cap_mac_admin",
            "cap_syslog",
            "35",
            "36+ep"
        ],
        "ansible_system_capabilities_enforced": "True",
        "ansible_system_vendor": "VMware, Inc.",
        "ansible_uptime_seconds": 948250,
        "ansible_user_dir": "/root",
        "ansible_user_gecos": "root",
        "ansible_user_gid": 0,
        "ansible_user_id": "root",
        "ansible_user_shell": "/bin/bash",
        "ansible_user_uid": 0,
        "ansible_userspace_architecture": "x86_64",
        "ansible_userspace_bits": "64",
        "ansible_virtualization_role": "guest",
        "ansible_virtualization_type": "VMware",
        "discovered_interpreter_python": "/usr/bin/python",
        "gather_subset": [
            "all"
        ],
        "module_setup": true
    },
    "changed": false,
    "deprecations": [],
    "warnings": []
}
```

## 2.4 Ansible Varibles 配置

```yml
ansible folder/
    group_vars/
        node.yml
    host_vars/
        10.8.21.212.yml
        10.8.21.213.yml
    ansible.cfg
    hosts
```

&emsp;&emsp;其中 hosts 對應 group_vars/群組名稱.yml、host_vars/變數名稱.yml：

```yml
[node]

10.8.21.212
10.8.21.213
```

&emsp;&emsp;在 group_vars/node.yml 寫入以下文字：

```yml
A:
  B: 20
  C: 30

#等同於以下寫法
#
## A: {B: 20, C: 30}

D:
- E
- F
```

&emsp;&emsp;透過 debug 模組進行測試：

```bash
ansible all -m debug -a "msg={{ A }}"

# 返回結果

10.8.21.212 | SUCCESS => {
    "msg": {
        "B": 20,
        "C": 30
    }
}
10.8.21.213 | SUCCESS => {
    "msg": {
        "B": 20,
        "C": 30
    }
}
```

&emsp;&emsp;在 host_vars/10.8.21.212.yml 寫入以下文字：

```yml
demo: ansible-node1
```

&emsp;&emsp;在 host_vars/10.8.21.213.yml 寫入以下文字：

```yml
demo: ansible-node2
```

&emsp;&emsp;透過 debug 模組進行測試：

```bash
ansible all -m debug -a "msg={{ demo }}"

# 返回結果

10.8.21.212 | SUCCESS => {
    "msg": "ansible-node1"
}
10.8.21.213 | SUCCESS => {
    "msg": "ansible-node2"
}
```

### 2.4.1 加密信息

&emsp;&emsp;加密文件。

```bash
ansible-vault encrypt group_vars/node.yml

# 返回結果

New Vault password:
Confirm New Vault password:
Encryption successful
```

&emsp;&emsp;查看文件。

```bash
ansible-vault view group_vars/node.yml

#返回結果

Vault password:
A:
  B: 20
  C: 30

#等同於以下寫法
#
## A: {B: 20, C: 30}

D:
- E
- F
```

&emsp;&emsp;透過 debug 訪問文件，必須加入 --ask-vault-pass。

```bash
ansible all -m debug -a "msg={{ demo }}"

# 返回結果

ERROR! Attempting to decrypt but no vault secrets found

ansible all -m debug -a "msg={{ demo }}" --ask-vault-pass

# 返回結果

Vault password:
10.8.21.212 | SUCCESS => {
    "msg": "ansible-node1"
}
10.8.21.213 | SUCCESS => {
    "msg": "ansible-node2"
}

```

&emsp;&emsp;解密文件。

```bash
ansible-vault decrypt group_vars/node.yml

# 返回結果

Vault password:
Decryption successful
```

## 2.5 用 Playbook 管理主機

&emsp;&emsp;Ansible 提供指令稿功能，名為 Playbook，使用的是 YAML 格式，副檔名為 .yml or .yaml。以下介紹基本變數使用方式：

```yml
key: value # 基本表示方式，在冒號後需要有空白。

key:
 key1: value
 key2: value # 也可以表示成：key: {key1: value, key2: value}。

key:
 - value
 - value # 也可以表示成：key: [value, value]
```

### 2.5.1 Playbook 結構

**範例 (1)**

&emsp;&emsp;撰寫 site.yml 內容如下：

```yml
---
- hosts: node
  remote_user: root
  gather_facts: yes

  tasks:
   - name: debug demo
     debug:
       msg: "{{ demo }}"
```

&emsp;&emsp;執行 playbook：

```bash
ansible-playbook site.yml

# 返回結果

PLAY [node] ****************************************************************************************************

TASK [Gathering Facts] *****************************************************************************************
ok: [10.8.21.212]
ok: [10.8.21.213]

TASK [debug demo] **********************************************************************************************
ok: [10.8.21.212] => {
    "msg": "ansible-node1"
}
ok: [10.8.21.213] => {
    "msg": "ansible-node2"
}

PLAY RECAP *****************************************************************************************************
10.8.21.212                : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
10.8.21.213                : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

**範例 (2)**

&emsp;&emsp;提取 gather_facts 中，"ansible_distribution" 和 "ansible_distribution_major_version" 的資訊，site.yml 內容如下：

```yml
---
- hosts: node
  remote_user: root
  gather_facts: yes

  tasks:
   - name: debug demo
     debug:
       msg: "{{ ansible_distribution }} {{ ansible_distribution_major_version }}"
```

&emsp;&emsp;執行 playbook：

```bash
ansible-playbook site3.yml

# 返回結果

PLAY [node] ****************************************************************************************************

TASK [Gathering Facts] *****************************************************************************************
ok: [10.8.21.213]
ok: [10.8.21.212]

TASK [debug demo] **********************************************************************************************
ok: [10.8.21.212] => {
    "msg": "CentOS 7"
}
ok: [10.8.21.213] => {
    "msg": "CentOS 7"
}

PLAY RECAP *****************************************************************************************************
10.8.21.212                : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
10.8.21.213                : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

```

### 2.5.2 變數配置

&emsp;&emsp;在 2.4 章節提到 group_vars 和 host_vars 的變數配置，在這裡會提到 playbooks 中如何配置變數。

```yaml
ansible folder/
    group_vars/
        node.yml
    host_vars/
        10.8.21.212.yml
        10.8.21.213.yml
    ansible.cfg
    hosts
    vars/
        main.yml
```

**方法 (1)**

&emsp;&emsp;在 vars/main.yml 新增以下內容：

```yml
test: abc
```

&emsp;&emsp;site.yml 內容如下：

```yml
---
- hosts: node
  remote_user: root
  gather_facts: yes
  vars_files:
  - ./vars/main.yml

  tasks:
   - name: debug demo
     debug:
       msg: "{{ test }}"
```

&emsp;&emsp;執行 playbook：

```bash
ansible-playbook site.yml

# 返回結果

PLAY [node] ****************************************************************************************************

TASK [Gathering Facts] *****************************************************************************************
ok: [10.8.21.213]
ok: [10.8.21.212]

TASK [debug demo] **********************************************************************************************
ok: [10.8.21.212] => {
    "msg": "abc"
}
ok: [10.8.21.213] => {
    "msg": "abc"
}

PLAY RECAP *****************************************************************************************************
10.8.21.212                : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
10.8.21.213                : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

```

**方法 (2)**

&emsp;&emsp;site.yml 內容如下：

```yml
---
- hosts: node
  remote_user: root
  gather_facts: yes
  vars_files:
  - ./vars/main.yml

  vars:
   test2: cba

  tasks:
   - name: debug demo
     debug:
       msg: "{{ test2 }}"
```

```bash
ansible-playbook site.yml

# 返回結果

PLAY [node] ****************************************************************************************************

TASK [Gathering Facts] *****************************************************************************************
ok: [10.8.21.213]
ok: [10.8.21.212]

TASK [debug demo] **********************************************************************************************
ok: [10.8.21.212] => {
    "msg": "cba"
}
ok: [10.8.21.213] => {
    "msg": "cba"
}

PLAY RECAP *****************************************************************************************************
10.8.21.212                : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
10.8.21.213                : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

### 2.5.3 變數相同優先級別

&emsp;&emsp;如果變數在 group_vars、host_vars、vars_files 和 vars 都有相同變數情況下，優先順序如下：

```bash
# vars: 本地變數
# vars_files: 導入變數


# 後導入變數 > 先導入變數 > 本地變數 > 主機變數 > 群組變數
# main.yml  > main2.yml >   vars  > host_vars > group_vars
```

### 2.5.4 Ansible Lint

&emsp;&emsp;分析撰寫的 playbook 中，yml 檔是否符合 ansible 的風格規範。

&emsp;&emsp;site.yml 內容如下：

```yml
---
- hosts: node
  remote_user: root
  gather_facts: yes
  vars_files:
  - ./vars/main.yml

  vars:
   test2: cba

  tasks:
   - name: debug demo
     debug:
       msg: "{{test2}}"
```

&emsp;&emsp;執行指令：

```bash
ansible-lint site3.yml

# 返回結果

WARNING  Overriding detected file kind 'yaml' with 'playbook' for given positional argument: site3.yml
WARNING  Listing 1 violation(s) that are fatal
var-spacing: Variables should have spaces before and after: {{test2}}
site3.yml:12 Task/Handler: debug demo

You can skip specific rules or tags by adding them to your configuration file:
# .ansible-lint
warn_list:  # or 'skip_list' to silence them completely
  - var-spacing  # Variables should have spaces before and after:  {{ var_name }}
Finished with 1 failure(s), 0 warning(s) on 1 files.
```

&emsp;&emsp;上述不符合變數前後空白的風格，雖然執行會成功，但是 Ansible Lint 建議你更改風格：

```bash
{{ test2 }}
```

### 2.5.5 Playbook 最佳結構

&emsp;&emsp;在實際案例中，會有多個環境運行，可以參照 [Best Practices](https://docs.ansible.com/ansible/2.8/user_guide/playbooks_best_practices.html) 中的方式。

&emsp;&emsp;以 Alternative Directory Layout 來說，會遵照以下規範：

```yml
inventories/
   production/
      hosts               # inventory file for production servers
      group_vars/
         group1.yml       # here we assign variables to particular groups
         group2.yml
      host_vars/
         hostname1.yml    # here we assign variables to particular systems
         hostname2.yml

   staging/
      hosts               # inventory file for staging environment
      group_vars/
         group1.yml       # here we assign variables to particular groups
         group2.yml
      host_vars/
         stagehost1.yml   # here we assign variables to particular systems
         stagehost2.yml

library/
module_utils/
filter_plugins/

site.yml
webservers.yml
dbservers.yml

roles/
    common/
    webtier/
    monitoring/
    fooapp/
```

&emsp;&emsp;根據上述建立以下範例：
```yml
inventories/
    production/
        hosts
        group_vars/
            node.yml
        host_vars/
            10.8.21.212.yml
            10.8.21.213.yml
    test/
        hosts
        group_vars/
            node.yml
        host_vars/
            10.8.21.212.yml
            10.8.21.213.yml
```

&emsp;&emsp;在 production 及 test 中 group_vars/node.yml 內容分別如下：

```yml
# production/group_vars/node.yml
title: production

# test/group_vars/node.yml
title: test
```

&emsp;&emsp;編輯 ansible.cfg：

```yml
[defaults]

inventory = ./inventories/test
host_key_checking = False
private_key_file = /root/.ssh/id_rsa
```

&emsp;&emsp;執行 playbook：

```yml
ansible-playbook site.yml

# 返回結果

PLAY [node] ****************************************************************************************************

TASK [Gathering Facts] *****************************************************************************************
ok: [10.8.21.212]
ok: [10.8.21.213]

TASK [debug demo] **********************************************************************************************
ok: [10.8.21.212] => {
    "msg": "test"
}
ok: [10.8.21.213] => {
    "msg": "test"
}

PLAY RECAP *****************************************************************************************************
10.8.21.212                : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
10.8.21.213                : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0


```

### 2.5.6 Loops

**(1) with_items (單一迴圈)**

&emsp;&emsp;編輯 site.yml：

```yml
---
  - hosts: node1
    remote_user: root
    gather_facts: no

    vars:
      demo:
        - 1
        - 2
        - 3
    tasks:
    - name: debug test
      debug:
        var: item
      with_items: "{{ demo }}"
```

&emsp;&emsp;執行 playbook：

```yml
ansible-playbook site.yml

# 返回結果

PLAY [node1] ***************************************************************************************************

TASK [debug test] **********************************************************************************************
ok: [10.8.21.212] => (item=1) => {
    "ansible_loop_var": "item",
    "item": 1
}
ok: [10.8.21.212] => (item=2) => {
    "ansible_loop_var": "item",
    "item": 2
}
ok: [10.8.21.212] => (item=3) => {
    "ansible_loop_var": "item",
    "item": 3
}

PLAY RECAP *****************************************************************************************************
10.8.21.212                : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

**(2) with_nested (雙重迴圈)**

&emsp;&emsp;編輯 site.yml：

```yml
---
  - hosts: node1
    remote_user: root
    gather_facts: no

    vars:
      demo:
        - 1
        - 2
        - 3
      demo2:
        - a
        - b
        - c
    tasks:
    - name: debug test
      debug:
        msg: "{{ item[0] }} - {{ item[1] }}"
      with_nested:
       - "{{ demo }}"
       - "{{ demo2 }}"
```

&emsp;&emsp;執行 playbook：

```yml
ansible-playbook site.yml

# 返回結果

PLAY [node1] ***************************************************************************************************

TASK [debug test] **********************************************************************************************
ok: [10.8.21.212] => (item=[1, 'a']) => {
    "msg": "1 - a"
}
ok: [10.8.21.212] => (item=[1, 'b']) => {
    "msg": "1 - b"
}
ok: [10.8.21.212] => (item=[1, 'c']) => {
    "msg": "1 - c"
}
ok: [10.8.21.212] => (item=[2, 'a']) => {
    "msg": "2 - a"
}
ok: [10.8.21.212] => (item=[2, 'b']) => {
    "msg": "2 - b"
}
ok: [10.8.21.212] => (item=[2, 'c']) => {
    "msg": "2 - c"
}
ok: [10.8.21.212] => (item=[3, 'a']) => {
    "msg": "3 - a"
}
ok: [10.8.21.212] => (item=[3, 'b']) => {
    "msg": "3 - b"
}
ok: [10.8.21.212] => (item=[3, 'c']) => {
    "msg": "3 - c"
}

PLAY RECAP *****************************************************************************************************
10.8.21.212                : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

**(3) loops (單一迴圈)**

&emsp;&emsp;編輯 site.yml：

```yml
---
  - hosts: node1
    remote_user: root
    gather_facts: no

    vars:
      demo:
        - 1
        - 2
        - 3
    tasks:
    - name: debug test
      debug:
        var: item
      loop: "{{ demo }}"
```

&emsp;&emsp;執行 playbook：

```yml
ansible-playbook site.yml

# 返回結果

PLAY [node1] ***************************************************************************************************

TASK [debug test] **********************************************************************************************
ok: [10.8.21.212] => (item=1) => {
    "ansible_loop_var": "item",
    "item": 1
}
ok: [10.8.21.212] => (item=2) => {
    "ansible_loop_var": "item",
    "item": 2
}
ok: [10.8.21.212] => (item=3) => {
    "ansible_loop_var": "item",
    "item": 3
}

PLAY RECAP *****************************************************************************************************
10.8.21.212                : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

**(4) loops (雙重迴圈)**

&emsp;&emsp;編輯 site.yml：

```yml
---
  - hosts: node1
    remote_user: root
    gather_facts: no

    vars:
      demo:
        - 1
        - 2
        - 3
      demo2:
        - a
        - b
        - c
    tasks:
    - name: debug test
      debug:
        msg: "{{ item[0] }} - {{ item[1] }}"
      loop: "{{ demo | product(demo2) | list }}"

```

&emsp;&emsp;執行 playbook：

```yml
ansible-playbook site.yml

# 返回結果

PLAY [node1] ***************************************************************************************************

TASK [debug test] **********************************************************************************************
ok: [10.8.21.212] => (item=[1, 'a']) => {
    "msg": "1 - a"
}
ok: [10.8.21.212] => (item=[1, 'b']) => {
    "msg": "1 - b"
}
ok: [10.8.21.212] => (item=[1, 'c']) => {
    "msg": "1 - c"
}
ok: [10.8.21.212] => (item=[2, 'a']) => {
    "msg": "2 - a"
}
ok: [10.8.21.212] => (item=[2, 'b']) => {
    "msg": "2 - b"
}
ok: [10.8.21.212] => (item=[2, 'c']) => {
    "msg": "2 - c"
}
ok: [10.8.21.212] => (item=[3, 'a']) => {
    "msg": "3 - a"
}
ok: [10.8.21.212] => (item=[3, 'b']) => {
    "msg": "3 - b"
}
ok: [10.8.21.212] => (item=[3, 'c']) => {
    "msg": "3 - c"
}

PLAY RECAP *****************************************************************************************************
10.8.21.212                : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

**(5) flatten**

&emsp;&emsp;編輯 site.yml：

```yml
---
  - hosts: node1
    remote_user: root
    gather_facts: no

    vars:
      demo:
        - 1
        - [2, 3]
        - 4
    tasks:
    - name: debug test
      debug:
        msg: "{{ item }}"
      loop: "{{ demo | flatten }}"
```

&emsp;&emsp;執行 playbook：

```yml
ansible-playbook site.yml

# 返回結果

PLAY [node1] ***************************************************************************************************

TASK [debug test] **********************************************************************************************
ok: [10.8.21.212] => (item=1) => {
    "msg": 1
}
ok: [10.8.21.212] => (item=2) => {
    "msg": 2
}
ok: [10.8.21.212] => (item=3) => {
    "msg": 3
}
ok: [10.8.21.212] => (item=4) => {
    "msg": 4
}

PLAY RECAP *****************************************************************************************************
10.8.21.212                : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

**(6) loop index**

&emsp;&emsp;編輯 site.yml：

```yml
---
  - hosts: node1
    remote_user: root
    gather_facts: no

    vars:
      demo:
        - 1
        - [2, 3]
        - 4
    tasks:
    - name: debug test
      debug:
        msg: "{{ indexx }} - {{ item }}"
      loop: "{{ demo | flatten }}"
      loop_control:
        index_var: indexx
```

&emsp;&emsp;執行 playbook：

```yml
ansible-playbook site.yml

# 返回結果

PLAY [node1] ***************************************************************************************************

TASK [debug test] **********************************************************************************************
ok: [10.8.21.212] => (item=1) => {
    "msg": "0 - 1"
}
ok: [10.8.21.212] => (item=2) => {
    "msg": "1 - 2"
}
ok: [10.8.21.212] => (item=3) => {
    "msg": "2 - 3"
}
ok: [10.8.21.212] => (item=4) => {
    "msg": "3 - 4"
}

PLAY RECAP *****************************************************************************************************
10.8.21.212                : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

**(7) loops 參數應用**
&emsp;&emsp;在[官方網站](https://docs.ansible.com/ansible/latest/user_guide/playbooks_loops.html#id19)中，要使用 loops 的參數可以增加下列指令：

```yml
loop_control:
  extended: yes
```

&emsp;&emsp;編輯 site.yml：

```yml
---
  - hosts: node1
    remote_user: root
    gather_facts: no

    vars:
      demo:
        - 1
        - [2, 3]
        - 4
    tasks:
    - name: debug test
      debug:
        msg: "{{ ansible_loop.index }} - {{ item }}"
      loop: "{{ demo | flatten }}"
      loop_control:
        extended: yes
```

&emsp;&emsp;執行 playbook：

```yml
ansible-playbook site.yml

# 返回結果

PLAY [node1] ***************************************************************************************************

TASK [debug test] **********************************************************************************************
ok: [10.8.21.212] => (item=1) => {
    "msg": "0 - 1"
}
ok: [10.8.21.212] => (item=2) => {
    "msg": "1 - 2"
}
ok: [10.8.21.212] => (item=3) => {
    "msg": "2 - 3"
}
ok: [10.8.21.212] => (item=4) => {
    "msg": "3 - 4"
}

PLAY RECAP *****************************************************************************************************
10.8.21.212                : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

**(8) loops 顯示物件**

&emsp;&emsp;編輯 site.yml：

```yml
---
  - hosts: node1
    remote_user: root
    gather_facts: no

    vars:
      demo:
        - {name: testuser1, groups: 'wheel'}
        - {name: testuser2, groups: 'root'}
    tasks:
    - name: debug test
      debug:
        msg: "{{ item.name }} - {{ item.groups }}"
      loop: "{{ demo }}"
      loop_control:
        extended: yes
```

&emsp;&emsp;執行 playbook：

```yml
ansible-playbook site.yml

# 返回結果

PLAY [node1] ***************************************************************************************************

TASK [debug test] **********************************************************************************************
ok: [10.8.21.212] => (item={'name': 'testuser1', 'groups': 'wheel'}) => {
    "msg": "testuser1 - wheel"
}
ok: [10.8.21.212] => (item={'name': 'testuser2', 'groups': 'root'}) => {
    "msg": "testuser2 - root"
}

PLAY RECAP *****************************************************************************************************
10.8.21.212                : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

**(9) lookup**

&emsp;&emsp;編輯 site.yml：

```yml
---
  - hosts: node1
    remote_user: root
    gather_facts: no

    vars:
      users:
        alice: female
        bob: male
    tasks:
    - name: debug test
      debug:
        msg: "{{ item.key }} is {{ item.value }}"
      loop: "{{ lookup('dict', users) }}"
```

&emsp;&emsp;執行 playbook：

```yml
ansible-playbook site.yml

# 返回結果

PLAY [node1] ***************************************************************************************************

TASK [debug test] **********************************************************************************************
ok: [10.8.21.212] => (item={'key': 'alice', 'value': 'female'}) => {
    "msg": "alice is female"
}
ok: [10.8.21.212] => (item={'key': 'bob', 'value': 'male'}) => {
    "msg": "bob is male"
}

PLAY RECAP *****************************************************************************************************
10.8.21.212                : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

### 2.5.7 When

**(1) when**

&emsp;&emsp;編輯 site.yml：

```yml
---
  - hosts: node1
    remote_user: root
    gather_facts: no

    vars:
      demo: [1, 2, 3, 4]
    tasks:
    - name: debug test
      debug:
        msg: "{{ item }}"
      loop: "{{ demo }}"
      when: item < 3
```

&emsp;&emsp;執行 playbook：

```yml
ansible-playbook site.yml

# 返回結果

PLAY [node1] ***************************************************************************************************

TASK [debug test] **********************************************************************************************
ok: [10.8.21.212] => (item=1) => {
    "msg": 1
}
ok: [10.8.21.212] => (item=2) => {
    "msg": 2
}
skipping: [10.8.21.212] => (item=3)
skipping: [10.8.21.212] => (item=4)

PLAY RECAP *****************************************************************************************************
10.8.21.212                : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

**(2) register**

&emsp;&emsp;編輯 site.yml：

```yml
---
  - hosts: node1
    remote_user: root
    gather_facts: yes

    vars:
      demo: [1, 2, 3, 4]
    tasks:
    - name: command test
      command: echo "hello"
      register: result
      when:
       - ansible_facts['distribution'] == 'CentOS'
    - name: debug test
      debug:
        msg: "{{ result }}"
      when: result is succeeded
      # when: result is failed
      # when: result is skipped
```

&emsp;&emsp;執行 playbook：

```yml
ansible-playbook site.yml

# 返回結果

PLAY [node1] ***************************************************************************************************

TASK [Gathering Facts] *****************************************************************************************
ok: [10.8.21.212]

TASK [command test] ********************************************************************************************
changed: [10.8.21.212]

TASK [debug test] **********************************************************************************************
ok: [10.8.21.212] => {
    "msg": {
        "changed": true,
        "cmd": [
            "echo",
            "hello"
        ],
        "delta": "0:00:00.006723",
        "end": "2021-04-06 14:56:04.105735",
        "failed": false,
        "rc": 0,
        "start": "2021-04-06 14:56:04.099012",
        "stderr": "",
        "stderr_lines": [],
        "stdout": "hello",
        "stdout_lines": [
            "hello"
        ]
    }
}

PLAY RECAP *****************************************************************************************************
10.8.21.212                : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

### 2.5.8 Block

&emsp;&emsp;執行一個判斷 (例如 when) 只能執行一個任務，透過 block 來執行多個任務。

**(1) block**

&emsp;&emsp;編輯 site.yml：

```yml
---
  - hosts: node1
    remote_user: root
    gather_facts: yes

    vars:
      demo: [1, 2, 3, 4]
    tasks:
    - name: block test
      block:
      - name: command test
        command: echo "hello"
        register: result
      - name: debug test
        debug:
          msg: "{{ result }}"
      when:
        - ansible_facts['distribution'] == 'CentOS'
```

&emsp;&emsp;執行 playbook：

```yml
ansible-playbook site.yml

# 返回結果

PLAY [node1] ***************************************************************************************************

TASK [Gathering Facts] *****************************************************************************************
ok: [10.8.21.212]

TASK [command test] ********************************************************************************************
changed: [10.8.21.212]

TASK [debug test] **********************************************************************************************
ok: [10.8.21.212] => {
    "msg": {
        "changed": true,
        "cmd": [
            "echo",
            "hello"
        ],
        "delta": "0:00:00.008561",
        "end": "2021-04-06 15:03:08.263169",
        "failed": false,
        "rc": 0,
        "start": "2021-04-06 15:03:08.254608",
        "stderr": "",
        "stderr_lines": [],
        "stdout": "hello",
        "stdout_lines": [
            "hello"
        ]
    }
}

PLAY RECAP *****************************************************************************************************
10.8.21.212                : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

**(2) rescue**

&emsp;&emsp;編輯 site.yml：

```yml
---
  - hosts: node1
    remote_user: root
    gather_facts: yes

    vars:
      demo: [1, 2, 3, 4]
    tasks:
    - name: handle the error
      block:
        - command: /bin/false
          register: result
        - debug:
            var: result
      rescue:
        - debug:
            msg: 'do stuff here to fix it'
```

&emsp;&emsp;執行 playbook：

```yml
ansible-playbook site.yml

# 返回結果

PLAY [node1] ***************************************************************************************************

TASK [Gathering Facts] *****************************************************************************************
ok: [10.8.21.212]

TASK [command] *************************************************************************************************
fatal: [10.8.21.212]: FAILED! => {"changed": true, "cmd": ["/bin/false"], "delta": "0:00:00.004468", "end": "2021-04-06 15:09:28.395563", "msg": "non-zero return code", "rc": 1, "start": "2021-04-06 15:09:28.391095", "stderr": "", "stderr_lines": [], "stdout": "", "stdout_lines": []}

TASK [debug] ***************************************************************************************************
ok: [10.8.21.212] => {
    "msg": "do stuff here to fix it"
}

PLAY RECAP *****************************************************************************************************
10.8.21.212                : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=1    ignored=0
```

### 2.5.9 handlers

&emsp;&emsp;依照任務需要時才執行

&emsp;&emsp;編輯 site.yml：

```yml
---
  - hosts: node1
    remote_user: root
    gather_facts: yes

    vars:
      demo: [1, 2, 3, 4]
    tasks:
    - name: handle the error
      command: echo "hello"
      notify:
       - define the 1st handler
    handlers:
    - name: define the 1st handler
      debug:
        msg: "define the 1st handler"
```

&emsp;&emsp;執行 playbook：

```yml
ansible-playbook site.yml

# 返回結果

PLAY [node1] ***************************************************************************************************

TASK [Gathering Facts] *****************************************************************************************
ok: [10.8.21.212]

TASK [handle the error] ****************************************************************************************
changed: [10.8.21.212]

RUNNING HANDLER [define the 1st handler] ***********************************************************************
ok: [10.8.21.212] => {
    "msg": "define the 1st handler"
}

PLAY RECAP *****************************************************************************************************
10.8.21.212                : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

### 2.5.10 Playbook 示範

&emsp;&emsp;此次任務有五項，依序分別為：

1. 安裝 httpd
2. 本地 index.html 複製到遠端主機 /var/www/html
3. 開啟 httpd 服務
4. 停止 firewalld 服務
5. 連接到各節點 IP 查看訊息

&emsp;&emsp;建立 index.html：

```bash
echo "HelloWorld" > ./files/index.html
```

&emsp;&emsp;編輯 site.yml：

```yml
---
  - hosts: all
    remote_user: root
    gather_facts: no
    tasks:
      - name: Install httpd
        yum:
          name: httpd
      - name: copy index file
        copy:
          src: ./files/index.html
          dest: /var/www/html/
      - name: start httpd service
        service:
          name: httpd
          state: started
      - name: stop firewalld service
        service:
          name: firewalld
          state: stopped
```

&emsp;&emsp;執行 playbook：

```yml
ansible-playbook site.yml

# 返回結果

PLAY [all] *********************************************************************

TASK [Install httpd] ***********************************************************
ok: [10.8.21.212]
ok: [10.8.21.213]

TASK [copy index file] *********************************************************
ok: [10.8.21.213]
ok: [10.8.21.212]

TASK [start httpd service] *****************************************************
ok: [10.8.21.213]
ok: [10.8.21.212]

TASK [stop firewalld service] **************************************************
ok: [10.8.21.213]
ok: [10.8.21.212]

PLAY RECAP *********************************************************************
10.8.21.212                : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
10.8.21.213                : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

&emsp;&emsp;透過 curl 查看 IP 回傳訊息：

```bash
curl 10.8.21.212

# 返回結果

HelloWorld

curl 10.8.21.213

# 返回結果

HelloWorld
```

### 2.5.11 Tags

&emsp;&emsp;只針對有 tags 的任務去執行，編輯 site.yml：

```yml
---
  - hosts: all
    remote_user: root
    gather_facts: no
    tasks:
      - name: Install httpd
        yum:
          name: httpd
        tags: inshttpd
      - name: copy index file
        copy:
          src: ./files/index.html
          dest: /var/www/html/
      - name: start httpd service
        service:
          name: httpd
          state: started
      - name: stop firewalld service
        service:
          name: firewalld
          state: stopped
```

&emsp;&emsp;執行 playbook：

```yml
ansible-playbook site.yml --tags "inshttpd"

# 返回結果

PLAY [all] *********************************************************************

TASK [Install httpd] ***********************************************************
ok: [10.8.21.213]
ok: [10.8.21.212]

PLAY RECAP *********************************************************************
10.8.21.212                : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
10.8.21.213                : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

&emsp;&emsp;針對 tags 以外的任務去執行執行 playbook：

```yml
ansible-playbook site.yml --skip-tags "inshttpd"

# 返回結果

PLAY [all] *********************************************************************

TASK [Install httpd] ***********************************************************
ok: [10.8.21.213]
ok: [10.8.21.212]

PLAY RECAP *********************************************************************
10.8.21.212                : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
10.8.21.213                : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

&emsp;&emsp;列出任務名稱，執行 playbook：

```yml
ansible-playbook site.yml --list-tasks

# 返回結果

playbook: site.yml

  play #1 (all): all    TAGS: []
    tasks:
      Install httpd     TAGS: [inshttpd]
      copy index file   TAGS: []
      start httpd service       TAGS: []
      stop firewalld service    TAGS: []

```

#### 2.5.11.1 Always

&emsp;&emsp;如果 tags 設定為 always，不論是否使用 tags 都會執行此任務，編輯 site.yml：

```yml
---
  - hosts: all
    remote_user: root
    gather_facts: no
    tasks:
      - name: debug
        debug:
          msg: "Always runs"
        tags: always
      - name: Install httpd
        yum:
          name: httpd
        tags: inshttpd
      - name: copy index file
        copy:
          src: ./files/index.html
          dest: /var/www/html/
      - name: start httpd service
        service:
          name: httpd
          state: started
      - name: stop firewalld service
        service:
          name: firewalld
          state: stopped
```

&emsp;&emsp;執行 playbook：

```yml
ansible-playbook site.yml --tags "inshttpd"

# 返回結果
 
PLAY [all] *********************************************************************

TASK [debug] *******************************************************************
ok: [10.8.21.212] => {
    "msg": "Always runs"
}
ok: [10.8.21.213] => {
    "msg": "Always runs"
}

TASK [Install httpd] ***********************************************************
ok: [10.8.21.212]
ok: [10.8.21.213]

PLAY RECAP *********************************************************************
10.8.21.212                : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
10.8.21.213                : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

### 2.5.12 Template

&emsp;&emsp;把檔案傳輸到遠端主機，跟 cp 的差別在於 Template 使用 <font color=red>Jinja2</font> 的語法。首先新增一個檔案 ./templates/test.j2，內容如下：

```yml
{{ ansible_nodenmme }}
```

&emsp;&emsp;編輯 site.yml：

```yml
---
  - hosts: all
    remote_user: root
    gather_facts: yes
    tasks:
    - name: copy template
      template:
        src: test.j2
        dest: /tmp/test
```

&emsp;&emsp;執行 playbook：

```yml
ansible-playbook site.yml

# 返回結果

PLAY [all] ***********************************************************************

TASK [Gathering Facts] ***********************************************************
ok: [10.8.21.213]
ok: [10.8.21.212]

TASK [copy template] *************************************************************
changed: [10.8.21.213]
changed: [10.8.21.212]

PLAY RECAP ***********************************************************************
10.8.21.212                : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
10.8.21.213                : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

&emsp;&emsp;查看所有節點 /tmp/test 的內文：

```bash
ansible all -m command -a "cat /tmp/test"

# 返回結果

10.8.21.213 | CHANGED | rc=0 >>
Ansible-Node2
10.8.21.212 | CHANGED | rc=0 >>
Ansible-Node1
```

## 2.6 Nginx 配置

&emsp;&emsp;在管理主機上先安裝 nginx 套件並複製 config 到 templates 資料夾下：

```bash
yum install nginx

cp /etc/nginx/nginx.conf ./templates/nginx.conf.j2
```

