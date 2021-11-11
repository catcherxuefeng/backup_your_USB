# backup_your_USB
树莓派之类的设备实现插入U盘自动备份
# 查看要备份U盘的UUID
```bash
lsblk 
blkid /dev/sda1
```
# 配置插入U盘自动挂载
```
# 创建挂载点
mkdir /media/work_u
# 创建备份目录
mkdir /data/work_u_backup
# 编写挂载服务
vim /etc/systemd/system/media-work_u.mount
[Unit]
Description=Auto mount work_u
ConditionPathExists=/media/work_u

[Mount]
What=/dev/disk/by-uuid/9C5A-5C40
Where=/media/work_u
Type=auto
Options=defaults

[Install]
WantedBy=multi-user.target
# 编写自动挂载服务
vim /etc/systemd/system/media-work_u.automount
[Unit]
Description=Automount work_usb

[Automount]
Where=/media/work_u

[Install]
WantedBy=multi-user.target
# 启动自动挂载服务
systemctl daemon-reload
systemctl enable --now media-work_u.mount media-work_u.automount
```
# 创建路径监控服务
```
vim /etc/systemd/system/work_u_backup.path
[Unit]
Description = makesure usb are mounted

[Path]
PathExists = /dev/disk/by-uuid/9C5A-5C40
Unit = work_u_backup.service

[Install]
WantedBy = multi-user.target
```
# 创建备份服务
```
vim /etc/systemd/system/work_u_backup.service
[Unit]
Description = work_u_backup
ConditionPathIsDirectory = /data/work_u_backup
ConditionPathIsDirectory = /media/work_u

[Service]
Type = simple
# 主备份命令
ExecStartPre = rsync -aP /media/work_u/ /data/work_u_backup/
# 配合 虾推啥 进行微信通知
ExecStart = curl "https://wx.xtuis.cn/123123123123123123.send?text=arm_work_u_backup_was_ok&desp=now you can remove your usb"

[Install]
WantedBy = multi-user.target
# 启动服务是指生效
systemctl daemon-reload
# 无需自启动 work_u_backup.service 
# systemctl enable --now  work_u_backup.service
systemctl enable --now work_u_backup.path 
```
