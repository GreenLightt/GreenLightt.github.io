---
title: Python 模块之 os
date: 2017-11-1 11:04:41
description: 整理 Python os 模块
tags:
categories:
- Python
copyright: false
---

`os` 模块包含普遍的操作系统功能，与具体的平台无关，比如操作文件、目录等；本篇笔记基于 `Python 2.7.1` 进行描述；

# 运行参数
| 命令 | 描述 |
|: --- :|: --- :|
| os.environ | 一个 `mapping` 对象表示环境 |
| os.chdir(path) | 改变当前工作目录到指定的路径 |
| os.fchdir(fd) | 通过文件描述符改变当前工作目录 |
| os.getcwd() | 返回当前工作目录 |
| os.ctermid() | 返回进程控制终端的文件名 |
| os.getegid() | 返回当前进程有效的 `group` 的 `id`；对应于当前进程的可执行文件的"`set id` "的bit位。 |
| os.geteuid() | 返回当前进程有效的 `user` 的 `id` |
| os.getgid() | 返回当前进程当前 `group` 的 `id` |
| os.getgroups() | 返回当前进程支持的 `groups` 的 `id` 列表 |
| os.initgroups(username, gid) | |
| os.getlogin() | 返回进程控制终端登陆用户的名字 |
| os.getpgid(pid) | 返回 `pid` 进程的 `group id`。如果 `pid` 为 0，返回当前进程的 `group id `|
| os.getpgrp() | 返回当前进程组的 `id` |
| os.getpid() | 返回当前进程的 `id` |
| os.getppid() | 返回当前父进程的 `id` |
| os.getresuid() | |
| os.getresgid() | |
| os.getuid() | 返回当前当前进程用户的 `id` |
| os.getenv(varname[, value]) | 返回 `environment` 变量 `varname` 的值，如果 `value` 不存在，默认为 `None` |
| os.putenv(varname, value) | 设置 `varname` 环境变量为 `value` 值。此改变影响以`os.system()`， `popen()` 或 `fork()`和`execv()`启动的子进程 |
| os.setegid(egid) | 设置当前进程有效组的 `id` |
| os.seteuid(euid) | 设置当前进程有效用户的 `id` |
| os.setgid(gid) | 设置当前进程组的 `id` |
| os.setgroups(groups) | 设置当前进程支持的 `groups id` 列表 |
| os.setpgrp() | 调用 `system` 的 `setpgrp()` 或 `setpgrp(0, 0)()` ，依赖于使用的是哪个版本的 `system` |
| os.setpgid(pid, pgrp) | 调用 `system` 的 `setpgid()` 设置 `pid` 进程 `group` 的 `id` 为 `pgrp`。 |
| os.setregid(rgid, egid) |  |
| os.setresgid(rgid, egid, sgid) | |
| os.setresuid(ruid, euid, suid) | |
| os.setreuid(ruid, euid) | |
| os.getsid(pid) | |
| os.setsid() | |
| os.setuid(uid) | 设置当前 `user id` |
| os.strerror(code) | 返回程序中错误 `code` 的错误信息 |
| os.umask(mask) | 设置当前权限掩码，同时返回先前的权限掩码 |
| os.uname() | |
| os.unsetenv(varname) | 删除 `varname` 环境变量|

# 文件对象创建
| 命令 | 描述 |
|: --- :|: --- :|
| os.fdopen(fd[, mode[, bufsize]]) | 返回一个文件描述符号为 `fd` 的打开的文件对象。`mode` 和 `bufsize` 参数，和内建的 `open()` 函数是同一个意思 |
| os.popen(command[, mode[, bufsize]]) | 给或从一个 `command` 打开一个管理。返回一个打开的连接到管道文件对象，文件对象可以读或写 |
| os.tmpfile() | 返回一个打开的模式为( `w+b` )的文件对象 .这文件对象没有文件夹入口，没有文件描述符，将会自动删除. |
| os.popen2(cmd[, mode[, bufsize]]) | |
| os.popen3(cmd[, mode[, bufsize]]) | |
| os.popen4(cmd[, mode[, bufsize]]) | |

# 文件描述符操作
| 命令 | 描述 |
|: --- :|: --- :|
| os.close(fd) | 关闭文件描述符 `fd`  |
| os.closerange(fd_low, fd_high) |  关闭从`fd_low`（包含）到 `fd_high`（不包含）所有的文件描述符 |
| os.dup(fd) | 返回文件描述符 `fd` 的 `cope` |
| os.dup2(fd, fd2) | 复制文件描述符 `fd` 到 `fd2`， 如果有需要首先关闭 `fd2` |
| os.fchmod(fd, mode) | 改变文件描述符为 `fd` 的文件 "`mode`" 为 `mode` |
| os.fchown(fd, uid, gid) | 改变文件描述符为 `fd` 的文件的所有者和 `group` 的 `id` 为  `uid` 和 `gid` |
| os.fdatasync(fd) | 强制将文件描述符为 `fd` 的文件写入硬盘. 不强制更新 `metadata` |
| os.fpathconf(fd, name) | 返回一个打开的文件的系统配置信息。 `name` 为检索的系统配置的值，它也许是一个定义系统值的字符串 |
| os.fstat(fd) | 返回文件描述符 `fd` 的状态 |
| os.fstatvfs(fd) | 返回包含文件描述符 `fd` 的文件的文件系统的信息，像 `statvfs()` |
| os.fsync(fd) | 强制将文件描述符为 `fd` 的文件写入硬盘 |
| os.ftruncate(fd, length) | 裁剪文件描述符 `fd` 对应的文件, 所以它最大不能超过文件大小 |
| os.isatty(fd) | 如果文件描述符 `fd` 是打开的，同时与 `tty(-like)`设备相连，则返回 `true`, 否则 `False` |
| os.lseek(fd, pos, how) | 设置文件描述符 `fd` 当前位置为 `pos`, `how`方式修改: <br/> `SEEK_SET` 或者 0 设置从文件开始的计算的 `pos`; <br/> `SEEK_CUR` 或者 1 则从当前位置计算; <br/> `os.SEEK_END` 或者2则从文件尾部开始 |
| os.SEEK_SET | |
| os.SEEK_CUR | |
| os.SEEK_END | |
| os.open(file, flags[, mode]) | 打开 `file` 同时根据 `flags` 设置变量 `flags` ，如果有 `mode`，则设置它的`mode`。 默认的 `mode` 是 0777 (八进制) |
| os.openpty() | |
| os.pipe() | 创建一个管道。 返回一对文件描述符(`r`, `w`) 分别为读和写 |
| os.read(fd, n) | 从文件描述符 `fd` 中读取最多 `n` 个字节。返回包含读取字节的 `string`。 文件描述符 `fd` 对应文件已达到结尾， 返回一个空 `string` |
| os.tcgetpgrp(fd) | |
| os.tcsetpgrp(fd, pg) | |
| os.ttyname(fd) | |
| os.write(fd, str) | 写入字符串到文件描述符 `fd` 中。 返回实际写入的字符串长度 |

# 文件和目录
| 命令 | 描述 |
|: --- :|: --- :|
| os.access(path, mode) | 使用现在的 `uid`/`gid` 尝试访问 `path` |
| os.F_OK | 作为 `access()` 的mode参数，测试 `path` 是否存在 |
| os.R_OK | 包含在 `access()` 的 `mode` 参数中 ， 测试 `path` 是否可读 |
| os.W_OK | 包含在 `access()` 的 `mode` 参数中 ，测试 `path` 是否可写 |
| os.X_OK | 包含在 `access()` 的 `mode` 参数中 ，测试 `path` 是否可执行|
| os.chdir(path) | 改变当前工作目录 |
| os.fchdir(fd) | |
| os.getcwd() | 返回当前工作目录的字符串 |
| os.getcwdu() | 返回一个当前工作目录的 `Unicode` 对象 |
| os.chflags(path, flags) | |
| os.chroot(path) | |
| os.chmod(path, mode) | |
| os.chown(path, uid, gid) | |
| os.lchflags(path, flags) | |
| os.lchmod(path, mode) | |
| os.lchown(path, uid, gid) | |
| os.link(source, link_name) | |
| os.listdir(path) | 返回 `path` 指定的文件夹包含的文件或文件夹的名字的列表 |
| os.lstat(path) | 像 `stat()`，但是没有符号链接。这是 `stat()` 的别名 |
| os.mkfifo(path[, mode]) | |
| os.mknod(filename[, mode=0600[, device=0]]) | 创建一个名为 `filename` 文件系统节点（文件，设备特别文件或者命名 `pipe`）。 `mode` 指定创建或使用节点的权限 |
| os.major(device) | 从原始的设备号中提取设备 `major` 号码 |
| os.minor(device) | 从原始的设备号中提取设备 `minor` 号码 |
| os.makedev(major, minor) | 以 `major` 和 `minor` 设备号组成一个原始设备号 |
| os.mkdir(path[, mode]) | 以数字 `mode` 的 `mode` 创建一个名为 `path` 的文件夹.默认的 `mode` 是 0777 (八进制) |
| os.makedirs(path[, mode])  | 递归文件夹创建函数 |
| os.pathconf(path, name) | |
| os.pathconf_names | |
| os.readlink(path) | |
| os.remove(path) | 删除路径为 `path` 的文件。如果 `path`  是一个文件夹，将抛出`OSError`；  |
| os.removedirs(path) | 递归删除 `directorie`。 像 `rmdir()`, 如果子文件夹成功删除， `removedirs()` 才尝试它们的父文件夹，直到抛出一个 `error` |
| os.rename(src, dst) | 重命名 `file` 或者 `directory src` 到 `dst`。如果 `dst` 是一个存在的 `directory`， 将抛出 `OSError` |
| os.renames(old, new) | 递归重命名文件夹或者文件 |
| os.rmdir(path) | 删除 `path` 文件夹. 仅当这文件夹是空的才可以, 否则, 抛出 `OSError` |
| os.stat(path) | 执行一个 `stat()` 系统调用在给定的 `path` 上. 返回值是一个对象，属性与 `stat` 结构成员有关 |
| os.stat_float_times([newvalue]) | 决定 `stat_result` 是否以 `float` 对象显示时间戳 |
| os.statvfs(path) | |
| os.symlink(source, link_name) | |
| os.tempnam([dir[, prefix]]) | 为创建一个临时文件返回一个唯一的 `path`；<br/>**警告**: 使用 `tempnam()` 对于 `symlink` 攻击是一个漏洞; 考虑使用 `tmpfile()` 代替 |
| os.tmpnam() | 为创建一个临时文件返回一个唯一的 `path` |
| os.TMP_MAX | 将产生唯一名字的最大数值 |
| os.unlink(path) | 删除 `file` 路径. 与 `remove()` 相同 |
| os.utime(path, times) | 返回指定的 `path` 文件的访问和修改的时间。如果时间是 `None`, 则文件的访问和修改设为当前时间  |
| os.walk(top, topdown=True, onerror=None, followlinks=False) | 输出在文件夹中的文件名通过在树中游走，向上或者向下.在根目录下的每一个文件夹(包含它自己), 产生 `3-tuple` (`dirpath`, `dirnames`, `filenames`)【文件夹路径, 文件夹名字, 文件名】 |

# 进程管理
| 命令 | 描述 |
|: --- :|: --- :|
| os.abort() | 产生一个 `SIGABRT` 标识到当前的进程 |
| os.execl(path, arg0, arg1, ...) | |
| os.execle(path, arg0, arg1, ..., env) | |
| os.execlp(file, arg0, arg1, ...) | |
| os.execlpe(file, arg0, arg1, ..., env) | |
| os.execv(path, args) | |
| os.execve(path, args, env) | |
| os.execvp(file, args) | |
| os.execvpe(file, args, env) | 这些函数将执行一个新程序，替换当前进程； 他们没有返回。在 `Unix`，新的执行体载入到当前的进程， 同时将和当前的调用者有相同的`id`。 将报告`Errors` 当抛出 `OSError` 时。  |
| os._exit(n) | 使用状态 `n` 退出系统，没有调用清理函数，刷新缓冲区。 |
| os.EX_OK | |
| os.EX_USAGE | |
| os.EX_DATAERR | |
| os.EX_NOINPUT | |
| os.EX_NOUSER | |
| os.EX_NOHOST | |
| os.EX_UNAVAILABLE | |
| os.EX_SOFTWARE | |
| os.EX_OSERR | |
| os.EX_OSFILE | |
| os.EX_CANTCREAT | |
| os.EX_IOERR | |
| os.EX_TEMPFAIL | |
| os.EX_PROTOCOL | |
| os.EX_NOPERM | |
| os.EX_CONFIG | |
| os.EX_NOTFOUND | |
| os.fork() | |
| os.forkpty() | |
| os.kill(pid, sig) | |
| os.killpg(pgid, sig) | |
| os.nice(increment) | |
| os.plock(op) | |
| os.popen(...) | |
| os.popen2(...) | |
| os.popen3(...) | |
| os.popen4(...) | |
| os.spawnl(mode, path, ...) | |
| os.spawnle(mode, path, ..., env) | |
| os.spawnlp(mode, file, ...) | |
| os.spawnlpe(mode, file, ..., env) | |
| os.spawnv(mode, path, args) | |
| os.spawnve(mode, path, args, env) | |
| os.spawnvp(mode, file, args) | |
| os.spawnvpe(mode, file, args, env) | |
| os.P_NOWAIT | |
| os.P_NOWAITO | |
| os.P_WAIT | |
| os.P_DETACH | |
| os.P_OVERLAY | |
| os.startfile(path[, operation]) | |
| os.system(command) | 在 `shell` 中执行 `string` 命令. |
| os.times() | 返回一个 `5-tuple` 的浮点数字， 表示(处理器或者其它)累积时间， 以秒为单位。 `items`为：用户时间， 系统`time`， 子用户`time`， 子系统`time`， 和从过去一个固定的点真实流逝的时间 |
| os.wait() | |
| os.waitpid(pid, options) | `Unix`：等待一个指定的 `pid` 的子进程完成，返回一个 `tuple` 返回它的进程`id`和退出状态 。 一般情况下`option`设为 0。<br/>`Windows`: 等待一个指定的 `pid` 的进程完成, 返回一个 `tuple` 返回它的进程 `id` 和退出状态向左移动了 8 位 。 如果 `pid` 小于或等于 0 没有特别的意思,将抛出`exception`。 `integer options` 没有任何影响。 `pid` 可以指向任何进程的 `id`，不一定是子进程的`id`。   |
| os.wait3(options) | |
| os.wait4(pid, options) | |
| os.WNOHANG | |
| os.WCONTINUED | |
| os.WUNTRACED | |
| os.WCOREDUMP(status) | |
| os.WIFCONTINUED(status) | |
| os.WIFSTOPPED(status)  | |
| os.WIFSIGNALED(status) | |
| os.WIFEXITED(status) | |
| os.WEXITSTATUS(status) | |
| os.WSTOPSIG(status) | |
| os.WTERMSIG(status) | |

# 各种各样的系统信息
| 命令 | 描述 |
|: --- :|: --- :|
| os.curdir | 操作系统用此常数字符串作为当前文件夹的引用。|
| os.pardir | 操作系统用此常数字符串作为父文件夹的引用。  |
| os.sep | 系统使用此字符来分割路径。 |
| os.altsep | 系统使用另外一个字符来分割路径，如果只有一个分割字符存在，则是 `None`。|
| os.extsep | 分割基本文件名和扩展名的字符。 |
| os.pathsep | 系统使用此字符来分割搜索路径。如 `POSIX` 上 '`:`'，`Windows`上的'`;`' |
| os.defpath | 默认的搜索路径 |
| os.linesep | 当前平台上的换行符字符串。 在 `POSIX` 上是 '`\n`' ,或者 在 `Windows` 上是 '`\r\n`'。  |
| os.devnull | 空设备的文件路径。例如: `POSIX`上 '`/dev/null`'。|
| os.confstr(name) | |
| os.confstr_names | |
| os.getloadavg() | |
| os.sysconf(name) | |
| os.sysconf_names | |


# 其他函数
| 命令 | 描述 |
|: --- :|: --- :|
| os.urandom(n) | 随机产生 `n` 个字节的字符串，可以作为随机加密 `key` 使用 |









