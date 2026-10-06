
## 1. 兼容性

### 环境变量

windows 和 wsl 的环境变量是共享的. 意味着, wsl 环境可以直接使用 windows 上的程序, 但需要加 `.exe` 后缀. 如 `code.exe`.

windows 执行 wsl 环境, 需要设置 `PATH:WSLENV` 

windows 磁盘都挂在在 `/mnt` 下.

### 剪贴板

windows 剪贴板管理工具是 clip.exe, wsl 中某些情况下可能无法通过 UI 直接复制内容到 windows 剪贴板, 可以直接访问 clip.exe.

如, vim 中使用 `:'<'>w !iconv -f utf-8 -t utf-16LE | clip.exe`, 需注意 windows 平台使用的编码是 `utf-16le`.

但是 clip.exe 仅能将内容输出到剪贴板, 要获取剪贴板内容, 可以用:

```bash
powershell.exe -command get-clipboard
```

另一种方案见 [issue4440](https://github.com/microsoft/WSL/issues/4440), 使用 [win32yank](https://github.com/equalsraf/win32yank), 
操作步骤参考回答 [Andrey Kaipov](https://stackoverflow.com/questions/44480829/how-to-copy-to-clipboard-in-vim-of-bash-on-windows/61864749#61864749). 


## 2. 配置

### 2. 1 .wslconfig

~/.wslconfig文件:

```toml
[wsl2]
# Limits VM memory to use no more than 24 GB
memory=24GB 

# Sets the VM to use two virtual processors
processors=12

# Specify a custom Linux kernel to use with your installed distros. 
kernel=C:\\temp\\myCustomKernel
# Sets additional kernel parameters, in this case enabling older Linux base images such as Centos 6
kernelCommandLine = vsyscall=emulate

# Disable page reporting so WSL retains all allocated memory claimed from Windows and releases none back when free
pageReporting=false

# Disables nested virtualization
nestedVirtualization=false

# Turns on output console showing contents of dmesg when opening a WSL 2 distro for debugging
debugConsole=true

# Enable experimental features
[experimental]

# when true, any newly created VHD will be set to sparse automatically
sparseVhd=true

# Automatically releases cached memeory after detecting idle CPU usage. Set to `gradual` for slow release, `dropcache` for isntant release of cached memory.
autoMemoryReclaim=gradual
```

### 2.2 Swap 

```toml
[wsl2]
# Sets amount of swap storage space to 8GB, default is 25% of available RAM
swap=8GB

# Sets swapfile path location, default is %USERPROFILE%\AppData\Local\Temp\swap.vhdx
swapfile=C:\\temp\\wsl-swap.vhdx

```

### 2.3 配置网络

WSL2 可以自动将端口映射到 Host，对于远程调试和调用很好用

```toml
[wsl2]
# if value if `mirrored` then turns on mirrored networking mode.
networkingMode=NAT

# Turn on default connection to bind WSL 2 localhost to Windows localhost. 
# Setting is ignored when networkingMode=mirrored.  
localhostForwarding=true

# enforces WSL to use Windows' HTTP proxy information
autoProxy=false

# change how DNS requests are proxied from WSL to Windows
dnsTunneling=false
```

### 2.4 替换内核

内核来源：

* [Microsoft 微调的 WSL2 内核](https://github.com/microsoft/WSL2-Linux-Kernel/releases), 不过大部分补丁最终都会合并到上游
* [Linux 官方发行版](https://www.kernel.org/)

安装依赖：

```bash
sudo apt install build-essential flex bison libssl-dev libelf-dev bc
```

配置 KConfig，微软同样提供了一个针对 WSL 版本的配置：[config-wsl](https://github.com/microsoft/WSL2-Linux-Kernel/blob/linux-msft-wsl-5.15.y/Microsoft/config-wsl)

编译内核以及内核模块

```bash
sudo make -j 4
# sudo make modules
# sudo make modules_install
sudo make install
```

在用户文件夹的配置文件 `.wslconfig` 中, 写入

```toml
[wsl2]
kernel=...\\arch\\x86\\boot\\bzImage
```

单纯要更新内核，可以用 `wsl --update`

## 3. 管理虚拟机实例

### 3.1 创建多个实例 

直接复制镜像文件，即可创建一个新的实例：

```shell
wsl --export Ubuntu-22.04 D:\VM\backup.tar.gz
wsl --import Ubuntu-22.04-2 <path\to\install> D:\VM\backup.tar.gz

# or directly import vhdx
wsl --import-in-place <Distro> <FilePth>

# login the second instance  
wsl -d Ubuntu-22.04-2
```

## 3.2 Shrink VHDX 

VHDX 体积是动态增长的，并且空闲空间也不会自动归还给 Windows。

WSL 新版直接提供压缩命令。旧版，则需要 `diskpart` 命令来手动压缩体积。

```bash
wsl --manage <Distro> --compact
```

> 除了定期释放空间，还有一种 SparseVhd ，可以自动释放空闲空间，缩减镜像体积。
> sparse 是允许底层文件系统在空闲位置“打洞”，从而拿回空闲空间。但并不是没有风险。
> 
> `wsl --manage <distro> --set-sparse true`


## 参考

- https://github.com/microsoft/WSL/issues/4699
- https://devblogs.microsoft.com/commandline/windows-subsystem-for-linux-september-2023-update/#automatic-disk-space-clean-up-set-sparse-vhd
- [windows 10 - How do I get back unused disk space from Ubuntu on WSL2 - Super User](https://superuser.com/questions/1606213/how-do-i-get-back-unused-disk-space-from-ubuntu-on-wsl2)
- [WSL Install Manual](https://learn.microsoft.com/en-us/windows/wsl/install-manual)
> [How to build and use a kernel in WSL2 ⋅ Plume](https://bashell.com/~/Cwt/how-to-build-and-use-a-kernel-in-wsl2)
