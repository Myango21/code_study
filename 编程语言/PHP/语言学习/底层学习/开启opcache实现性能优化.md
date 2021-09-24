## 什么是opcache

OPcache 通过将 PHP 脚本预编译的字节码存储到共享内存中来提升 PHP 的性能， 存储预编译字节码的好处就是 省去了每次加载和解析 PHP 脚本的开销。

## opcache运行原理

### 不使用opcache
![Snipaste_2021-09-24_23-18-32](https://gitee.com/bruce_qiq/picture/raw/master/2021-9-24/1632496909208-Snipaste_2021-09-24_23-18-32.png)

### 使用opcache
![Snipaste_2021-09-24_23-18-16](https://gitee.com/bruce_qiq/picture/raw/master/2021-9-24/1632496899073-Snipaste_2021-09-24_23-18-16.png)

## opcache配置说明
```php
[opcache]
; 是否快开启opcache缓存。
;opcache.enable=1

; 是否在cli模式下开启opcache。
;opcache.enable_cli=1

; opcache共享内存的大小(单位是M)。
;opcache.memory_consumption=128

; 预留字符串的的内存大小(单位是M)。
;opcache.interned_strings_buffer=8

; 在hash表中存储的最大脚本文件数量，范围是200到1000000之间。实际的情况是在{ 223, 463, 983, 1979, 3907, 7963, 16229, 32531, 65407, 130987 }中找到第一个大于等于设置值的质数。最小范围是200。
;opcache.max_accelerated_files=10000

; 浪费内存的上线，如果超过这个上线，opcache将重新启动。
;opcache.max_wasted_percentage=5

; 如果启用，opcache将会在hash表的脚本键后面增加一个文件目录，避免吃同名的脚本产生冲突。禁用的话可以提高性能，但是也容易导致应用不可用。
;opcache.use_cwd=1

; 如果启用(1)，opcache会每隔设置的值时间来判断脚本是否更新。如果禁用(0)，则不会自动检测脚本更新，必须通过重启PHP服务，或者使用opcache_reset()、opcache_invalidate()函数来刷新缓存。
;opcache.validate_timestamps=1

; opcache检查脚本是否更新的时间周期(单位是秒)，如果设置为0则会针对每一个请求进行检查更新，如果validate_timestamps=0，该值不会生效。
;opcache.revalidate_freq=60

; 如果禁用，在统一include_path下面已经缓存的文件将被重用，因此无法找到该路径下的同名文件。
;opcache.revalidate_path=0

; 是否保存PHP脚本中的注释内容。禁用，则不会缓存PHP代码中的注释，可以减少文件中的体积，但是一些依赖注释或者注解将无法使用。
;opcache.save_comments=1

; 如果启用，则会使用快速停止续发事件。 所谓快速停止续发事件是指依赖 Zend 引擎的内存管理模块 一次释放全部请求变量的内存，而不是依次释放每一个已分配的内存块。
; 在php7.2.0开始，被移除，这类说的事件将会在PHP中自动处理。
;opcache.fast_shutdown=1

; 如果启用，在调用file_exists()、is_file()和is_readable()函数时，不管文件是否被缓存，都会检测操作码。如果禁用，可能读取的内容是一些旧数据。
;opcache.enable_file_override=0

; 控制优化级别，是一个二进制的位的掩码。
;opcache.optimization_level=0xffffffff

; 不进行编译优化的配置文件路径。该文件中配置具体哪些不被编译的文件。如果文中每行的开头是";"开头，则会被视为注释。黑名单中的文件名，可以是通配符，也可以使用前缀。
; 例如配置文件的路径是"/home/blacklist.txt"，则该配置的值就是该路径。
; 配置的内容可以是如下格式

; 这是一段注释，在解析的时候因为开头是;，则会被视为注释
;/var/www/a.php
;/var/www/a/b.php

;opcache.blacklist_filename=

; 以字节为单位的缓存的文件大小上限。设置为 0 表示缓存全部文件。
;opcache.max_file_size=0

; 每个N次请求会检查缓存校验和，0是不检查。该项对性能有较大影响，尽量在调试环境中使用。
;opcache.consistency_checks=0

; 如果缓存处于非激活状态，等待多少秒之后计划重启。 如果超出了设定时间，则 OPcache 模块将杀除持有缓存锁的进程， 并进行重启。
;opcache.force_restart_timeout=180

; 错误日志文件位置，不填写将默认输出到服务器的错误日志文件中。
;opcache.error_log=

; 错误日志文件等级。
; 默认情况下，仅有致命级别（0）及错误级别（1）的日志会被记录。 其他可用的级别有：警告（2），信息（3）和调试（4）。
; 如何设置的是1以上，在进行force_restart_timeout选项时，会将错误日志中插入一条警告信息。
;opcache.log_verbosity_level=1

; opcache首选的内存模块，不配置则自动选择。可以选择的值有mmap，shm, posix 以及 win32。
;opcache.preferred_memory_model=

; 保护共享内存，以避免执行脚本时发生非预期的写入。 仅用于内部调试。
;opcache.protect_memory=0

; 只允许指定字符串开头的PHP脚本调用opcache api函数，默认不做限制。
;opcache.restrict_api=

; 在 Windows 平台上共享内存段的基地址。 所有的 PHP 进程都将共享内存映射到同样的地址空间。 使用此配置指令避免“无法重新附加到基地址”的错误。
;opcache.mmap_base=

; 配置二级缓存目录并启用二级缓存。 启用二级缓存可以在 SHM 内存满了、服务器重启或者重置 SHM 的时候提高性能。 默认值为空字符串 ""，表示禁用基于文件的缓存。
;opcache.file_cache=

; 启用或禁用在共享内存中的 opcode 缓存。
;opcache.file_cache_only=0

; 当从文件缓存中加载脚本的时候，是否对文件的校验和进行验证。
;opcache.file_cache_consistency_checks=1

; 在 Windows 平台上，当一个进程无法附加到共享内存的时候， 使用基于文件的缓存。需要开启opcache.file_cache_only选项。建议开启此选项，否则可能导致进程无法启动。
;opcache.file_cache_fallback=1

; 启用或者禁用将 PHP 代码（文本段）拷贝到 HUGE PAGES 中。 此项配置指令可以提高性能，但是需要在 OS 层面进行对应的配置。
;opcache.huge_code_pages=1

; 针对当前用户，验证缓存文件的访问权限。
;opcache.validate_permission=0

; 在 chroot 的环境中避免命名冲突。 为了防止进程访问到 chroot 环境之外的文件，应该在 chroot 的情况下启用这个选项。
;opcache.validate_root=0
```