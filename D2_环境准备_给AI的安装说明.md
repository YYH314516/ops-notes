# 练习机环境准备说明（给另一个 AI 执行）

用途：在另一台电脑的空虚拟机上，装好做 D2~D8 运维练习所需的 Rocky Linux 环境。
**装完只需要一个干净可用的系统，不要预先完成任何练习题目**（见文末「不要做的事」）。

---

## 一、目标环境规格

| 项目 | 值 |
|---|---|
| 系统 | Rocky Linux 9.x x86_64（Minimal Install 即可） |
| 主机名 | `rocky9-01` |
| 时区 | `Asia/Shanghai` |
| IP | 静态 `192.168.158.101/24`，网关 `192.168.158.2` |
| DNS | `192.168.158.2`、`223.5.5.5` |
| root 密码 | `见本地密码本` |
| 内存 | 2 GB 起（4 GB 更稳） |
| CPU | 2 核 |
| 磁盘 | 40 GB 单盘（第二块盘留给 D4 现场加，不要预装） |
| SELinux | **Enforcing**（不要关，后面练习要用） |
| firewalld | 启用并开机自启 |
| 网络模式 | VMware NAT（VMnet8）；VirtualBox 用 NAT 也行，网段按实际调整 |

> 如果这台电脑没有 VMware/VirtualBox，用 Hyper-V 或云主机也可以，只要是一台能 SSH 的 Rocky 9 / RHEL 9 系机器，IP 那行按实际改。

---

## 二、安装阶段要点

1. 挂 Rocky-9-x86_64 DVD ISO，**Software Selection 选 Minimal Install**。
2. Installation Destination：整盘自动分区，40 GB。
3. Network：先开 DHCP 让安装器联网，装完再改静态。
4. Root Password：`见本地密码本`（练习机，允许弱密码；如果安装器报"too weak"，连点两次 Done 强行接受）。
5. 不需要创建普通用户，`root` 直接登录即可。

安装完重启，用 root 登录。

---

## 三、装完后的初始化（root 执行）

```bash
# 1. 基础更新
dnf -y update
dnf -y install dnf-plugins-core

# 2. 练习要用到的工具（不是全部必须，一次性装齐省事）
dnf -y install vim-enhanced bash-completion tree lsof tcpdump curl wget git \
               tar unzip acl policycoreutils-python-utils bind-utils tmux \
               sysstat man-pages man-db

# 3. 主机名、时区
hostnamectl set-hostname rocky9-01
timedatectl set-timezone Asia/Shanghai

# 4. 静态 IP（网卡名用 ip a 确认真实名字，常见 ens160 / ens33）
ip a
# 下面把 ens160 换成实际网卡名
nmcli con mod ens160 ipv4.method manual \
      ipv4.addresses 192.168.158.101/24 \
      ipv4.gateway 192.168.158.2 \
      ipv4.dns "192.168.158.2 223.5.5.5"
nmcli con up ens160

# 5. 服务状态确认
systemctl enable --now sshd firewalld chronyd
getenforce                       # 必须是 Enforcing
```

### 如果静态 IP 配不通

VMware 的 NAT 网段不一定是 158，先看宿主机的：
`VMware → 编辑 → 虚拟网络编辑器 → VMnet8 → NAT 设置`，把 `192.168.158.x` 换成实际的网段，网关一般是 `x.x.x.2`。
实在不通就先用 DHCP，把拿到的 IP 写下来，后面所有命令里替换掉即可。

---

## 四、验收清单（装完必须逐条跑一遍）

```bash
cat /etc/rocky-release        # Rocky Linux release 9.x
hostname                      # rocky9-01
timedatectl                   # Time zone: Asia/Shanghai (CST, +0800)
ip -4 a                       # 有 192.168.158.101/24
ip route                      # default via 192.168.158.2
ping -c2 223.5.5.5            # 通
getenforce                    # Enforcing
systemctl is-enabled sshd firewalld
rpm -q acl policycoreutils-python-utils vim-enhanced   # 都显示已装
df -h /                       # 根分区约 40G
```

从 Windows 宿主机验证：
- `ping 192.168.158.101` 通
- MobaXterm / 终端 `ssh root@192.168.158.101` 能登进去

**全部通过后打一个快照，命名 `base-D2`。**

---

## 五、不要做的事（很重要）

下面这些是练习内容本身，环境里**保持空白**，否则练习就白做了：

- 不要建 `dev` 组、`tom` / `bob` 用户
- 不要建 `/data` 目录
- 不要改 `/etc/sudoers` 或 `/etc/sudoers.d/`
- 不要预先调 umask、不要设 ACL
- 不要关 SELinux、不要关 firewalld
- 不要装 nginx / mysql / php（那是 D8 的事）
- 不要配 SSH 免密

只把"系统能跑、能 SSH、能上网"做到位就行。

---

## 六、顺手准备（可选，省后面的事）

```bash
# 把 Rocky ISO 放到机器里，D6 配本地源时直接挂载，不用再传文件
mkdir -p /opt/iso
# 把 Rocky-9-x86_64-dvd.iso 传到这里（MobaXterm 拖拽即可）

# 留一个习惯性备份目录
mkdir -p /root/work
```

---

## 七、直接粘给那边 AI 的提示词

```
在一台空的虚拟机上装 Rocky Linux 9 (Minimal Install)，规格如下：
- 主机名 rocky9-01，时区 Asia/Shanghai
- 静态 IP 192.168.158.101/24，网关 192.168.158.2，DNS 192.168.158.2 和 223.5.5.5
  （如果 VMware NAT 网段不同，按实际网段调整）
- root 密码见本地密码本，允许 root 密码 SSH 登录
- 2 核 2G，单盘 40G，SELinux 保持 Enforcing，firewalld 启用
- 装完后执行：dnf -y update；dnf -y install vim-enhanced bash-completion tree lsof
  tcpdump curl wget git tar unzip acl policycoreutils-python-utils bind-utils tmux sysstat
- 验收：getenforce 为 Enforcing、ip -4 a 有该地址、ping 223.5.5.5 通、
  ssh 能从宿主机登录、df -h / 显示约 40G
- 装完打快照，命名 base-D2

禁止预先创建 dev 组、tom/bob 用户、/data 目录，禁止修改 sudoers、umask、ACL，
禁止安装 nginx/mysql/php。这些是后面的练习内容，必须保持空白。

具体验收命令见：c规划/D2_环境准备_给AI的安装说明.md
```
