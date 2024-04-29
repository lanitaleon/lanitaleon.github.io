记录一下Windows10安装Docker踩的坑，一是装不上，二是打不开。

#### 一、WSL升级为WSL2

##### 1.1问题描述

官网下载安装Docker，启动提示对应的WSL不是最新的。

执行 wsl --update 无响应或失败。

##### 1.2解决方案

第一步用管理员权限打开PowerShell，执行如下指令：

```
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
```

第二步执行如下指令：

```
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```

第三步：重启电脑

第四步：下载并安装Linux内核更新包，链接如下：

<https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi>

第五步：管理员身份打开PowerShell执行如下指令：

```
wsl --set-default-version 2
```

该指令是将wsl2设置为默认版本。

第六步：检查下wsl2安装是否成功：

```
wsl --list --verbose
```

安装成功的输出可参照如下信息：

```
PS C:\Users\dell> wsl --list --verbose
  NAME                  STATE            VERSION
* Ubuntu-18.04          Running          2
  docker-desktop        Running          2
  docker-desktop-data   Running          2
PS C:\Users\dell>
```

#### 二、Docker Desktop starting forever

##### 2.1问题描述

Docker Desktop启动失败，一直显示starting，wsl指令也没响应。

##### 2.2解决方案

最后定位是LxssManager服务出问题，需要重启。

> WSL子系统是基于LxssManager服务运行的

管理员权限打开CMD，查询PID：

```
sc queryex LxssManager
```

杀死该进程，注意直接kill无效，需要进行如下操作：

```
wmic process where ProcessID=xxxxx delete
```

重启LxssManager

```
net start LxssManager
```

以上重启过程一次不行可以多试几次，我就搞了两次才成功。
