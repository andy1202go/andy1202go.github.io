## nohup命令的**&**

今天配置一个监控服务，在Linux上面使用docker部署后，怎么配置服务连接都是Connection Failure。

一直以为是网络问题，尝试切换过host网络，尝试在容器内安装ping工具测试，发现网络其实是通的。

然后以为是hertzbeat的设置问题。

唯独没有怀疑过期待被监控的Java服务的问题。

按照AI的说法，调整了下Java服务的actuator服务配置之后，按照惯例进行服务进程查找，发现...妈蛋，进程没在。

想了下，原来是上次启动的时候，少输入了一个&。

以前很少手动部署服务，现在懂了...

关键信息摘取：

> **nohup** 英文全称 no hang up（不挂起），用于在系统后台不挂断地运行命令，退出终端不会影响程序的运行。
>
> **nohup** 命令，在默认情况下（非重定向时），会输出一个名叫 nohup.out 的文件到当前目录下，如果当前目录的 nohup.out 文件不可写，输出重定向到 **$HOME/nohup.out** 文件中。
>
> ```bash
> nohup Command [ Arg … ] [　& ]
> nohup java -jar xxx.jar > system.log 2>&1 &
> ```
>
> **2>&1** 解释：
>
> 将标准错误 2 重定向到标准输出 &1 ，标准输出 &1 再被重定向输入到 runoob.log 文件中。
>
> - 0 – stdin (standard input，标准输入)
> - 1 – stdout (standard output，标准输出)
> - 2 – stderr (standard error，标准错误输出)

### 参考

[nohup](https://www.runoob.com/linux/linux-comm-nohup.html)