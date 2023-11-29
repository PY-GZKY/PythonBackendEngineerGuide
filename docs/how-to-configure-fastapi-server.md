# 如何搭建一台后端服务器

## ssh连接服务器

```shell
ssh wutong@hostname
```

## 配置Python环境

```shell
# 安装 Python 环境
wget https://mirrors.bfsu.edu.cn/anaconda/miniconda/Miniconda3-py310_23.3.1-0-Linux-x86_64.sh
bash Miniconda3-py310_23.3.1-0-Linux-x86_64.sh -b
```

## 配置zsh

```shell
# 配置zsh
sh -c "$(wget https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh -O -)"

# 配置完成后一定要运行下面命令修改环境变量
~/miniconda3/bin/conda init bash
~/miniconda3/bin/conda init zsh
```

## 设置免密ssh登录服务器

首先进入本地电脑当前用户的.ssh文件夹中，再打开id_rsa.pub文件并复制其中所有内容

```shell
ssh-keygen
```

此然后运行下面命令获得自己的公钥:

```shell
cat ~/.ssh/id_rsa.pub
```

使用密码登录服务器，创建~/.ssh/authorized_keys文件，并设置相应权限。

```shell
touch ~/.ssh/authorized_keys
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
nano ~/.ssh/authorized_keys

# 添加你的公钥，如 ssh-rsa AAAA...
```

并设置相应权限。把刚刚复制的密钥粘贴到~/.ssh/authorized_keys文件中，如果文件已经有内容就添加到最后一行，这样每次使用ssh登录服务器就不用再输入密码了

## 重启



