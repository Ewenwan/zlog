## 0. zlog是什么？

zlog是一个高可靠性、高性能、线程安全、灵活、概念清晰的纯C日志函数库。

    事实上，在C的世界里面没有特别好的日志函数库（就像JAVA里面的的log4j，或者C++的log4cxx）。C程序员都喜欢用自己的轮子。printf就是个挺好的轮子，但没办法通过配置改变日志的格式或者输出文件。syslog是个系统级别的轮子，不过速度慢，而且功能比较单调。
    所以我写了zlog。
    zlog在效率、功能、安全性上大大超过了log4c，并且是用c写成的，具有比较好的通用性。

## 1.安装

下载：https://github.com/HardySimpson/zlog/releases

	$ tar -zxvf zlog-latest-stable.tar.gz
	$ cd zlog-latest-stable/
	$ make 
	$ sudo make install
	or
	$ make PREFIX=/usr/local/
	$ sudo make PREFIX=/usr/local/ install

PREFIX指明了安装的路径，安转完之后为了让你的程序能找到zlog动态库

	$ sudo vi /etc/ld.so.conf
	/usr/local/lib
	$ sudo ldconfig

在你的程序运行之前，保证libzlog.so在系统的动态链接库加载器可以找到的目录下。上面的命令适用于linux，别的系统自己想办法。


## 2.介绍一下配置文件

zlog里面有三个重要的概念,category,format，rule

分类(Category)用于区分不同的输入，代码中的分类变量的名字是一个字符串，在一个程序里面可以通过获取不同的分类名的category用来后面输出不同分类的日志，用于不同的目的。

格式(Format)是用来描述输出日志的格式，比如是否有带有时间戳， 是否包含文件位置信息等，上面的例子里面的格式simple就配置成简单的用户输入的信息+换行符。

规则(Rule)则是把分类、级别、输出文件、格式组合起来，决定一条代码中的日志是否输出，输出到哪里，以什么格式输出。简单而言，规则里面的分类字符串和代码里面的分类变量的名字一样就匹配，当然还有更高级的纲目分类匹配。规则彻底解耦了各个元素之间的强绑定，例如log4j就必须为每个分类指定一个级别（或者从父分类那里继承），这在多层系统需要每一层都有自己的级别要求的时候非常不方便。

现在试着写配置文件，配置文件名无所谓，放在哪里也无所谓，反正在zlog_init()的时候可以指定

	$ cat /etc/zlog.conf

	[formats]
	simple = "%m%n"
	[rules]
	my_cat.DEBUG    >stdout; simple

在目前的配置文件的例子里面，可以看到my_cat分类，>=debug等级的日志会被输出到stdout(标准输出)，并且输出的格式是simple这个格式，也就是用户输入信息+换行符。如果要输出到文件并控制文件大小为1兆，规则的配置应该是
my_cat.DEBUG            "/var/log/aa.log", 1M; simple

##3.在代码中使用
$ vi test_hello.c
```c

#include <stdio.h> 

#include "zlog.h"

int main(int argc, char** argv)
{
        int rc;
        zlog_category_t *c;

        rc = zlog_init("/etc/zlog.conf");
        if (rc) {
                printf("init failed\n");
                return -1;
        }

        c = zlog_get_category("my_cat");
        if (!c) {
                printf("get cat fail\n");
                zlog_fini();
                return -2;
        }

        zlog_info(c, "hello, zlog");

        zlog_fini();

        return 0;
} 
```
## 4.编译、然后运行!

	$ cc -c -o test_hello.o test_hello.c -I/usr/local/include
	$ cc -o test_hello test_hello.o -L/usr/local/lib -lzlog -lpthread
	$ ./test_hello
	hello, zlog

## 5.高级使用

 *  syslog分类模型，比log4j模型更加直接了当
 *  日志格式定制，类似于log4j的pattern layout
 *  多种输出，包括动态文件、静态文件、stdout、stderr、syslog、用户自定义输出函数
 *  运行时手动、自动刷新配置文件（同时保证安全）
 *  高性能，在我的笔记本上达到25万条日志每秒, 大概是syslog(3)配合rsyslogd的1000倍速度 
 *  用户自定义等级
 *  多线程和多进程环境下保证安全转档
 *  精确到微秒
 *  简单调用包装dzlog（一个程序默认只用一个分类）
 *  MDC，线程键-值对的表，可以扩展用户自定义的字段
 *  自诊断，可以在运行时输出zlog自己的日志和配置状态
 *  不依赖其他库，只要是个POSIX系统就成(当然还要一个C99兼容的vsnprintf)

## 6.相关链接

主页：http://hardysimpson.github.com/zlog/

软件下载：https://github.com/HardySimpson/zlog/releases

作者邮箱：HardySimpson1984@gmail.com


auto tools版本: https://github.com/bmanojlovic/zlog

cmake版本: https://github.com/lisongmin/zlog

windows版本: https://github.com/lopsd07/WinZlog



0. What is zlog?
-------------

zlog is a reliable, high-performance, thread safe, flexible, clear-model, pure C logging library.

  Actually, in the C world there was NO good logging library for applications like logback in java or log4cxx in c++. Using printf can work, but can not be redirected or reformatted easily. syslog is slow and is designed for system use. 
  So I wrote zlog. 
  It is faster, safer and more powerful than log4c. So it can be widely used. 

1. Install
-------------

Downloads: https://github.com/HardySimpson/zlog/releases

    $ tar -zxvf zlog-latest-stable.tar.gz
    $ cd zlog-latest-stable/
    $ make 
    $ sudo make install
    
or

    $ make PREFIX=/usr/local/
    $ sudo make PREFIX=/usr/local/ install

PREFIX indicates the installation destination for zlog. After installation, refresh your dynamic linker to make sure your program can find zlog library.  

    $ sudo vi /etc/ld.so.conf
    /usr/local/lib
    $ sudo ldconfig

Before running a real program, make sure libzlog.so is in the directory where the system's dynamic lib loader can find it. The command metioned above are for linux. Other systems will need a similar set of actions.


2. Introduce configure file
-------------

There are 3 important concepts in zlog: categories, formats and rules.

Categories specify different kinds of log entries. In the zlog source code, category is a (zlog_cateogory_t *) variable. In your program, different categories for the log entries will distinguish them from each other.

Formats describe log patterns, such as: with or without time stamp, source file, source line.

Rules consist of category, level, output file (or other channel) and format. In brief, if the category string in a rule in the configuration file equals the name of a category variable in the source, then they match. Still there is complex match range of category. Rule decouples variable conditions. For example, log4j must specify a level for each logger(or inherit from father logger). That's not convenient when each grade of logger has its own level for output(child logger output at the level of debug, when father logger output at the level of error)

Now create a configuration file. The function zlog_init takes the files path as its only argument.
    $ cat /etc/zlog.conf

    [formats]
    simple = "%m%n"
    [rules]
    my_cat.DEBUG    >stdout; simple

In the configuration file log messages in the category "my_cat" and a level of DEBUG or higher are output to standard output, with the format of simple(%m - usermessage %n - newline). If you want to direct out to a file and limit the files maximum size, use this configuration

    my_cat.DEBUG            "/var/log/aa.log", 1M; simple

3. Using zlog API in C source file
-------------
	$ vi test_hello.c

    #include <stdio.h> 

    #include "zlog.h"

    int main(int argc, char** argv)
    {
    	int rc;
    	zlog_category_t *c;

    	rc = zlog_init("/etc/zlog.conf");
    	if (rc) {
    		printf("init failed\n");
    		return -1;
    	}

    	c = zlog_get_category("my_cat");
    	if (!c) {
    		printf("get cat fail\n");
    		zlog_fini();
    		return -2;
    	}

    	zlog_info(c, "hello, zlog");

    	zlog_fini();

    	return 0;
    } 

4. Compile, and run it!
-------------
    $ cc -c -o test_hello.o test_hello.c -I/usr/local/include
    $ cc -o test_hello test_hello.o -L/usr/local/lib -lzlog -lpthread
    $ ./test_hello
    hello, zlog

5. Advanced Usage
-------------
 *  syslog model, better than log4j model
 *  log format customization
 *  multiple output destinations including static file path, dynamic file path, stdout, stderr, syslog, user-defined ouput
 *  runtime manually or automatically refresh configure(safely)
 *  high-performance, 250'000 logs/second on my laptop, about 1000 times faster than syslog(3) with rsyslogd
 *  user-defined log level
 *  thread-safe and process-safe log file rotation
 *  microsecond accuracy
 *  dzlog, a default category log API for easy use
 *  MDC, a log4j style key-value map
 *  self debuggable, can output zlog's self debug&error log at runtime
 *  No external dependencies, just based on a POSIX system and a C99 compliant vsnprintf.

6.Links:
-------------
 * Homepage: http://hardysimpson.github.com/zlog
 * Downloads: https://github.com/HardySimpson/zlog/releases
 * Author's Email: HardySimpson1984@gmail.com
 * auto tools version: https://github.com/bmanojlovic/zlog
 * cmake verion: https://github.com/lisongmin/zlog
 * windows version: https://github.com/lopsd07/WinZlog


