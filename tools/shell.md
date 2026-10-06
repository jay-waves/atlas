
## Shell 

* bash
* ~~powershell~~ 弃用，太臃肿
* WSL 
* ~~Git Bash (MSYS2)~~ 弃用，兼容性不好
* [niubash](https://github.com/unixwin/niubash), Unix compatible shell in windows 

## utils

* rubash , CoreUtils + GNUTools
* ~~Microsoft.CoreUtils, POSIX 兼容工具集~~
* ~~[GNUwin32](https://gnuwin32.sourceforge.net/packages.html) 工具集原生移植~~

| GNU  | Rewrite-in-Rust | Powershell | Description |
| ----- | ------------- | ---------- | --------- |
| lsdisk  | duf         |    | 磁盘统计            |
| du    | dust          |    |  目录下文件体积统计（直方图） |
| grep  | ripgrep       | findstr   |     |
| find  | fd            |      |     |
| cat   | bat           | Get-Content |  |
| cloc  | tokei         |    |    |
| file  |               |    |    |
| cd    | zoxide        |    |    |
| man   | tldr          |    | 百科全书，简短版   |
| diff  | delta      |    |    |
| curl  |  xh           | Invoke-WebRequest   |    |
| ps, pgrep  |  procs           |  Get-Process  |    |
| ss, netstat |  rustnet           |  netstat  |    |

- iconv, uchardet [char-encoding](../hw/char-encoding.md)工具
- **fzf**, 模糊查找工具
- tldr
- rclone 云存储工具
- strings 读取二进制中的字符串片段


```
coreutils      ls cp mv rm cat cut sort uniq ...
findutils      find xargs
grep           grep
sed            sed
gawk           awk
procps-ng      ps top pgrep pkill free
psmisc         pstree killall fuser
binutils       strings nm objdump readelf
file           file
util-linux     lsblk dmesg mount kill taskset ...
```

### Powershell 特化工具

* psmux 
* pstop
* 

