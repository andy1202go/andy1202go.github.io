## logging_collector 日志收集器

默认值：off

建议：on

> 一个服务器参数，用于确定是否启用日志收集器。
>
> **`logging_collector`** 是一个 [服务器参数](https://pgpedia.postgresql.ac.cn/g/guc.html)，用于确定是否启用 `日志收集器`。日志收集器是一个后台进程，它捕获 `stderr` 输出并将其重定向到日志文件。
>
> 当它设置为 `on` 时
>
> PostgreSQL 会启动一个单独的后台进程（`logging_collector`），专门负责捕获来自服务器进程（如 `postgres` 主进程、工作进程等）发送到标准错误输出（`stderr`）的日志消息，并将它们写入到配置指定的日志文件（通常是 `data/log` 目录下的文件）。
>
> 当它设置为 `off` 时（默认）
>
> 日志消息会直接发送到标准错误输出。如果 PostgreSQL 是通过服务管理器（如 `systemd`、`supervisord`）启动的，这些日志通常会被服务管理器捕获并写入到其自己的日志系统（如 `journald`、系统日志等）

- 如果设置为off，stderr会输出到启动postgresql实例的服务管理器的日志系统中。

- 只能重启生效。

- 和其他几个配置搭配使用。

对于docker部署情况，如果使用k8s管理所有容器日志，可以不要打开，统一收集错误日志。如果只是单一实例，可以尝试修改为on。

## log_destination 日志目的地

默认值：stderr

建议：如果日志最终给人看，stderr即可；如果还需要收集使用，增加csvlog.

个人：stderr,csvlog

- 支持stderr,csvlog,jsonlog,syslog几种方式
- csvlog,jsonlog需要loggin_collector开启

### 一个典型的理解错误

log_destination如果设置是stderr，就只记录错误日志么？csvlog的话，是标准输出和标准错误都记录么？

> ## PostgreSQL日志目的地详解
>
> > 这就像选择记日记的方式：是用普通笔记本（stderr），还是用Excel表格（csvlog），或者是用公司公告栏（syslog）？
>
> ### 核心概念澄清
>
> 首先，**这是一个常见的误解**！让我用最直白的方式解释：
>
> **`log_destination = 'stderr'` 并不意味着"只记录错误日志"！**
> **`stderr` 在这里是"输出渠道"，不是"日志级别"！**
>
> 这就像说"用电子邮件发送报告"，而不是说"只发送错误报告"。电子邮件是发送方式，报告内容可以是任何东西。
>
> ### 详细解析
>
> #### 1. `stderr`到底是什么？
>
> **stderr（标准错误流）** 是Unix/Linux系统中的一个基本概念。程序有三个标准流：
> - `stdin`（标准输入）：程序读取数据的地方
> - `stdout`（标准输出）：程序正常输出的地方
> - `stderr`（标准错误）：程序输出错误和诊断信息的地方
>
> 但在PostgreSQL的上下文中，`log_destination = 'stderr'`意味着：
> - **所有配置要记录的日志内容**（不只是错误）都输出到stderr这个"通道"
> - 包括：普通查询日志、连接信息、慢查询、警告信息、错误信息等
> - 具体记录什么，由其他参数控制：`log_statement`、`log_connections`、`log_min_error_statement`等
>
> **生活化比喻**：
> 想象PostgreSQL是一个广播电台：
> - `stderr`是**FM 100.0频率**（输出渠道）
> - 电台可以在FM 100.0上播放：新闻（普通日志）、音乐（调试信息）、紧急广播（错误信息）
> - `log_destination = 'stderr'`只是说"我们在FM 100.0上广播"，没说"只播紧急广播"
>
> #### 2. `csvlog`的工作方式
>
> **`csvlog`是另一种输出格式，不是另一个输出流！**
>
> 当设置`log_destination = 'csvlog'`时：
> 1. PostgreSQL会生成**CSV格式**的日志
> 2. 这些CSV日志**仍然通过stderr输出**（如果`logging_collector=on`则写入文件）
> 3. 你可以同时指定多个目的地：`log_destination = 'stderr,csvlog'`
>
> **关键点**：
> - `csvlog`**不区分**stdout和stderr，它只是一种格式
> - 如果同时指定`stderr`和`csvlog`，你会得到两份日志：一份普通文本，一份CSV格式
> - CSV格式更容易用程序分析（比如用Excel、pandas、pgBadger）
>
> ### 对比表格
>
> | 特性         | `stderr`         | `csvlog`                         | `syslog`         | `eventlog` (Windows) |
> | ------------ | ---------------- | -------------------------------- | ---------------- | -------------------- |
> | **本质**     | 输出渠道         | 输出格式                         | 系统服务         | 系统服务             |
> | **内容**     | 所有配置的日志   | 所有配置的日志                   | 所有配置的日志   | 所有配置的日志       |
> | **格式**     | 纯文本，人类可读 | CSV格式，机器友好                | 系统日志格式     | Windows事件格式      |
> | **输出位置** | 标准错误流       | 文件（需`logging_collector=on`） | 系统日志守护进程 | Windows事件查看器    |
> | **典型用途** | Docker/简单部署  | 日志分析/监控                    | 企业统一日志     | Windows服务器        |
>
> ### 实际配置示例
>
> #### 示例1：只使用stderr（Docker环境典型配置）
> ```ini
> # postgresql.conf
> log_destination = 'stderr'
> logging_collector = off  # 让Docker捕获stderr
> 
> # 控制记录内容（这些才是决定"记什么"的参数！）
> log_statement = 'all'      # 记录所有SQL
> log_connections = on       # 记录连接
> log_disconnections = on    # 记录断开
> log_min_error_statement = error  # 错误语句才记录
> log_min_messages = warning # 警告及以上级别才记录
> ```
>
> 在Docker中查看：
> ```bash
> # 看到所有配置的日志，不只是错误
> docker logs postgres-container
> 
> # 输出示例：
> # 2024-01-15 10:00:00 UTC [1234] LOG:  database system is ready to accept connections
> # 2024-01-15 10:00:01 UTC [1235] LOG:  connection received: host=172.17.0.1 port=5432
> # 2024-01-15 10:00:02 UTC [1235] LOG:  statement: SELECT * FROM users;
> # 2024-01-15 10:00:03 UTC [1235] ERROR:  relation "users" does not exist
> ```
>
> #### 示例2：使用csvlog进行详细记录
> ```ini
> # postgresql.conf
> log_destination = 'csvlog'
> logging_collector = on
> log_directory = 'pg_log'
> log_filename = 'postgresql-%Y-%m-%d.csv'
> 
> # 详细记录配置
> log_statement = 'all'
> log_connections = on
> log_disconnections = on
> log_lock_waits = on
> log_temp_files = 0
> ```
>
> 生成的CSV日志文件内容：
> ```csv
> 2024-01-15 10:00:00 UTC,1234,,,LOG,00000,"database system is ready to accept connections",,,,,,,,,"","postgres"
> 2024-01-15 10:00:01 UTC,1235,172.17.0.1,5432,LOG,00000,"connection received: host=172.17.0.1 port=5432",,,,,,,,,"","postgres"
> 2024-01-15 10:00:02 UTC,1235,172.17.0.1,5432,LOG,00000,"statement: SELECT * FROM users",,,,,,,,,"","postgres"
> 2024-01-15 10:00:03 UTC,1235,172.17.0.1,5432,ERROR,42P01,"relation ""users"" does not exist",,,,,,,,,"","postgres"
> ```
>
> #### 示例3：同时使用两种格式
> ```ini
> # 同时获得人类可读和机器可读的日志
> log_destination = 'stderr,csvlog'
> logging_collector = on
> 
> # 这样会在pg_log目录下生成两种文件：
> # postgresql-2024-01-15.log    (文本格式)
> # postgresql-2024-01-15.csv    (CSV格式)
> ```
>
> ### 控制"记什么"的关键参数
>
> 真正决定日志内容的不是`log_destination`，而是这些参数：
>
> | 参数                      | 作用               | 示例值                                                       |
> | ------------------------- | ------------------ | ------------------------------------------------------------ |
> | `log_statement`           | 记录哪些SQL语句    | `none`(不记录), `ddl`(DDL语句), `mod`(修改语句), `all`(所有语句) |
> | `log_min_messages`        | 记录的最低消息级别 | `debug5`-`debug1`, `info`, `notice`, `warning`, `error`, `log`, `fatal`, `panic` |
> | `log_min_error_statement` | 错误语句记录级别   | `error`, `log`, `fatal`, `panic`                             |
> | `log_connections`         | 是否记录连接       | `on`/`off`                                                   |
> | `log_disconnections`      | 是否记录断开       | `on`/`off`                                                   |
> | `log_duration`            | 是否记录查询耗时   | `on`/`off`                                                   |
> | `log_lock_waits`          | 是否记录锁等待     | `on`/`off`                                                   |
> | `log_temp_files`          | 记录临时文件使用   | `0`(所有), `100`(>100KB)                                     |
>
> ### 实际问答场景
>
> **Q：我想要在Docker中只记录错误日志，怎么配置？**
> **A**：不是改`log_destination`，而是调整日志级别：
> ```ini
> log_destination = 'stderr'
> log_min_messages = error      # 只记录error及以上级别
> log_min_error_statement = error  # 错误语句才记录SQL
> log_statement = 'none'        # 不记录普通SQL
> log_connections = off         # 不记录连接
> log_disconnections = off      # 不记录断开
> ```
>
> **Q：csvlog和stderr同时使用时，日志会重复吗？**
> **A**：是的，内容会重复（两份），但格式不同。一份是给人读的文本，一份是给程序读的CSV。
>
> **Q：在csvlog中，错误日志有什么特殊标记吗？**
> **A**：有的！CSV格式中有一列专门表示日志级别。你可以这样过滤：
> ```bash
> # 在CSV日志中只查看错误
> grep ',ERROR,' postgresql-*.csv
> 
> # 或者用awk（第5列是日志级别）
> awk -F',' '$5 == "ERROR"' postgresql-*.csv
> ```
>
> ### 一句话总结
>
> **`log_destination`决定日志的"输出方式和格式"（怎么记），而其他`log_*`参数决定日志的"内容和级别"（记什么）。`stderr`是渠道，不是过滤器；`csvlog`是格式，不是目的地。**

### 重点总结

日志目的地，说明日志的输出方式（渠道）和格式。

### 引申问题：为什么pg要把日志默认往stderr输出，而不是stdout呢？

**一句话总结**：PostgreSQL选择stderr是**Unix哲学的传承**、**架构分离的智慧**、**历史兼容的考量**，以及**对未来（容器化）的意外但完美的准备**。这不是偶然，而是经过深思熟虑的优雅设计。

> 在Unix世界中，有一个不成文但被广泛遵守的约定：
>
> - **stdout用于"程序的主要输出"**：命令的结果、数据、正常输出
> - **stderr用于"诊断和状态信息"**：错误、警告、进度、日志
>
> **PostgreSQL遵循这个传统**：
>
> - 查询结果 → stdout（通过客户端如psql）
> - 日志和诊断信息 → stderr

## log_rotation_size 日志轮转大小

默认：10MB。

建议：10MB太小了，但是也不要设置到GB级别的日志，不好用。

官方：

- logging_collector启用时候，这个参数有效，指定日志大小
- 如果设置为0，则禁用以文件大小进行轮转
- 如果没有配置单位，默认单位是KB

## log_checkpoints 日志检查点

默认：on

建议：开启，这样才好知道检查点什么时候发生的。

## log_min_duration_statement  日志最小持续时间语句

默认：-1

建议：不要设置为0，根据自己系统流量情况和业务容忍情况，比如设置为1s

官方：

- 设置最小执行时间，超过该时间所有语句都将被记录。
- 如果没有设置单位，默认单位是ms
- 设置为0，打印所有语句
- 设置为-1，禁用语句记录









## 参考

[高效日志实践：logging_collector 故障排除与 CSV 日志输出详解](https://runebook.dev/zh/docs/postgresql/runtime-config-logging/GUC-LOGGING-COLLECTOR)