# Ubuntu22.04 配置root远程登入
> 先确保root用户的密码已修改完成。
#sudo passwd root 设置root密码
普通用户登陆，打开终端使用su指令切换到root用户

### 修改配置
> vi打开文件/etc/pam.d/gdm-password和/etc/pam.d/gdm-autologin(两个都要改，不分顺序)。将以下语句注释掉:
auth required pam_succeed_if.so user ！= root quiet_success

修改/root/.profile文件
打开/root/.profile

注释mesg n 2 > /dev/null || true

加上tty -s && mesg m || true

前面完成后即可在登陆界面登陆root用户，如果要远程连接登陆root用户还需要以下修改
安装openssh，修改/etc/ssh/sshd_config

注释permitRootLogin prohibit-password

添加上PermitRootLongin yes

### 保存修改，重启ssh服务
service ssh restart # systemctl restart ssh
