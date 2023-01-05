---
title: systemctl排查问题
date: 2023-01-05 12:30
updated: 2023-01-05 12:30
categories: linux
---


## 查看服务

### 查看所有
```bash
systemctl --type=service 
```

### 查看特定状态的服务
```
systemctl  --type=service --state=load 
systemctl  --type=service --state=loaded
systemctl  --type=service --state=failed
systemctl  --type=service --state=active
systemctl  --type=service --state=exited
systemctl  --type=service --state=running
```
### 查看 运行状态
```
nvidia@nvidia-desktop:~$ systemctl status fstrim.service
● fstrim.service - Discard unused blocks
   Loaded: loaded (/lib/systemd/system/fstrim.service; static; vendor preset: enabled)
   Active: failed (Result: exit-code) since Mon 2023-01-02 00:00:43 CST; 3 days ago
  Process: 1135 ExecStart=/sbin/fstrim -av (code=exited, status=64)
 Main PID: 1135 (code=exited, status=64)

Jan 02 00:00:32 nvidia-desktop systemd[1]: Starting Discard unused blocks...
Jan 02 00:00:32 nvidia-desktop fstrim[1135]: fstrim: /data: FITRIM ioctl failed: Input/output error
Jan 02 00:00:43 nvidia-desktop fstrim[1135]: /: 9.1 GiB (9720225792 bytes) trimmed
Jan 02 00:00:43 nvidia-desktop systemd[1]: fstrim.service: Main process exited, code=exited, status=64/n/a
Jan 02 00:00:43 nvidia-desktop systemd[1]: fstrim.service: Failed with result 'exit-code'.
Jan 02 00:00:43 nvidia-desktop systemd[1]: Failed to start Discard unused blocks.
```
## 查看失败错误信息
### 根据 进程id 查看
```
journalctl _PID=1135
```
### 根据 服务 查看
```
journalctl -u fstrim
```