使用说明
==============
[使用说明](./source_file/README.md)

安装
==============
```shell
git clone https://github.com/michael-liumh/binlog2sql.git && \
cd binlog2sql && \
pip3 install -r requirements.txt
```
git与pip的安装问题请自行搜索解决。
推荐使用 pypy3 运行，效率会比 python3 高

注意事项
==============
* 使用 binlogfile2sql 解析本地 binlog，也是需要连接数据库的，因为 binlog 里面存储的是 table_id，而不是 table_name，所以需要连接数据库将 table_id 转换成 table_name，database_name、column_name 也是同理。
* binlogfile2sql 连接的数据库实例，里面只需要有表结构即可，不需要有实际的数据，所以可以单独启动一个空实例，导入表结构即可使用。

修改
==============
* 添加日志模块
* 修改requirement.txt文件
  * 修改 mysql-replication 版本为 0.23
* 添加对 json 数据类型的支持
* 修改 flashback 模式临时文件的输出方式（测试发现从临时文件读取数据，根据指定块大小读取数据，部分二进制数据会被截断，从而导致输出异常，替代方案是直接读取整行数据）
* 新增一个对结果文件的排序脚本
  * sort_dml.sh
* 修改连接mysql实例使用的默认字符集为 utf8mb4
* 添加对 blob 数据类型的支持（包括 binary、varbinary 等二进制类型）
* 添加 binlog file 的解析支持
* 添加 binlog file dir，可指定解析某个目录下特定的几个 binlog file
  * --check 检查选择的文件是否正确
  * --file-dir 选择目录
  * --file-regex 正则匹配 binlog file
  * --start-file 选择目录中起始的 binlog file
  * --stop-file 选择目录中终止的 binlog file
* 添加忽略特定库、表、列的支持，方便解析后直接在另一个实例上执行
* 添加将 insert into 语句替换成 replace into 和 insert ignore into 语句的支持，方便解析后直接在另一个实例上执行
* 添加 update、delete 语句根据主键进行更新的支持，方便解析后直接在另一个实例上执行
* 添加将 binlogfile2sql 解析结果保存到特定文件、目录的支持，方便另一个应用直接读取这个目录下 SQL 文件来执行
* 添加 binlogfile2sql 只解析修改时间在指定分钟数前的支持，防止解析到未写入完成的 binlog，方便解析后直接在另一个实例上执行
* 添加对结果 SQL 中的库名进行重命名的支持，方便解析后直接在另一个实例上执行
* 添加去除注释的支持，方便解析后直接在另一个实例上执行
* 添加 binlogfile2sql 忽略虚拟列的支持
* 支持根据 GTID 过滤
* 支持直接设定条件过滤结果，参数：--where
* 支持直接将解析出来的 SQL 同步到另一个实例，但如果指定了 --rename-db 参数，DDL里的库名不会被更改，因此建议只同步 DML，不要同步 DDL

参数说明
==============
|  参数   | 说明  |
|  ----  | ----  |
| --help  | 输出帮助信息 |
| -h, --host  | 连接 MySQL 服务器的地址 |
| -P, --port  | 连接 MySQL 服务器的端口 |
| -u, --user  | 连接 MySQL 服务器的用户 |
| -p, --password  | 连接 MySQL 服务器的密码（不指定这个参数也可以，默认会要求通过终端交互的形式输入密码，空密码直接回车即可） |
| -d, --databases  | 选择输出指定库的 SQL |
| -t, --tables  | 选择输出指定表的 SQL |
| -id, --ignore-databases  | 排除输出指定库的 SQL |
| -it, --ignore-tables  | 排除数据指定表的 SQL |
| -ic, --ignore-columns  | 过滤掉 SQL 中的指定的列 |
| --ignore-virtual-columns  | 过滤掉 SQL 中的虚拟列 |
| --start-position, --start-pos  | 指定 binlog 的起始位点 |
| --stop-position, --stop-pos  | 指定 binlog 的结束位点 |
| --start-datetime  | 解析 binlog 中指定开始时间后的 SQL |
| --stop-datetime  | 解析 binlog 中指定结束时间前的 SQL |
| --include-gtids  | 只输出指定 gtid 的 SQL |
| --exclude-gtids  | 不输出被排除 gtid 的 SQL |
| --only-dml  | 只输出 DML 类型的 SQL（排除DDL） |
| --sql-type  | 只输出指定类型的 DML（但不排除DDL，要排除DDL的话需要加上 --only-dml 参数） |
| --stop-never  | 持续不间断解析从运行脚本这个时间点开始，往后新增的 binlog 内容 |
| -K, --no-primary-key  | 排除 SQL 中的主键字段 |
| -KK, --only-primary-key  | UPDATE 与 DELETE 类型的 SQL 的条件只保留主键字段 |
| -B, --flashback | 生成回滚 SQL 而不生成 binlog 记录中的 SQL |
| --replace  | 将 INSERT 类型的 SQL 中的 INSERT INTO 关键字改为 REPLACE INTO |
| --insert-ignore  | 将 INSERT 类型的 SQL 中的 INSERT INTO 关键字改为 INSERT IGNORE INTO |
| --result-file  | 将生成的结果保存到指定的文件中（请直接指定文件名，不要添加目录，添加了也会自动去除，请用 --result-dir 指定目录） |
| --record-file  | 当使用 --stop-never 参数时，自动记录解析过的 binlog 文件（仅对解析 binlog 文件生效） |
| --result-dir  | 指定生成结果文件的报存目录 |
| --table-per-file | 当使用 --stop-never 参数解析本地 binlog 时，输出的结果将按《库名.表名.日期.sql》的格式保存到对应的文件中 |
| --date-prefix | 当使用 --table-per-file 参数解析本地 binlog 时，输出的结果将按《日期.库名.表名.sql》的格式保存到对应的文件中 |
| -ma, --minutes-ago | 当解析本地 binlog 时，只解析最后修改时间在 n 分钟前的文件（可用这个参数排除还没记录完的 binlog，不想排除的话，直接参数值为 0 即可） |
| --need-comment | 选择输出的 SQL 是否需要保留注释，注释内容包括这条 SQL 在 binlog 中的起始位点、结束位点、gtid值，值为 1 表示保留（默认），0 表示不保留 |
| --rename-db | 选择将输出的 SQL 的库名进行重命名，格式：“旧库名 新库名” 或者 “新库名”，只提供新库名的话，会将未提供旧库名的其它所有库名全部重命名成指定库名，因此，无特殊需求的情况下，请不要只提供新库名 |
| --rename-tb | 选择将输出的 SQL 的表名进行重命名，格式：“旧表名 新表名” 或者 “新表名”，只提供新表名的话，会将未提供旧表名的其它所有表名全部重命名成指定表名，因此，无特殊需求的情况下，请不要只提供新表名 |
| --remove-not-update-col | 排除 UPDATE 语句中未被更新的字段（默认输出完整的更新前后的值） |
| --keep, --keep-not-update-col | 当使用--remove-not-update-col参数来排除 UPDATE 语句中未被更新的字段时，会保留一些没更新的，但你想保留的字段，多个字段用空格分隔。示例：--remove-not-update-col --keep id col1 col2 |
| --update-to-replace | 将 UPDATE 语句转化成 REPLACE INTO 语句 |
| -f, --file-path | 解析指定的本地 binlog 文件 |
| -fd, --file-dir | 解析指定目录下的所有本地 binlog 文件（可用下面的参数过滤） |
| -fr, --file-regex | 使用正则表达式指定选择的目录下的 binlog 文件 |
| --start-file | 通过字符串比较的方式，指定选择的目录下的起始的 binlog 文件 |
| --stop-file | 通过字符串比较的方式，指定选择的目录下的结束的 binlog 文件 |
| --check | 检查指定目录下被过滤的 binlog 文件是否符合预期 |
| --supervisor | 用 supervisor 管理后台解析进程 |
| --where | 根据指定条件过滤出需要的 SQL，支持同时传入多个条件，但不能将多个条件用一个括号包起来，多个条件直接传入多个参数即可。正确示例：--where 'c1=v1' 'c2=v2'；错误示例：--where 'c1=v1 and c2=v2'；单个条件支持使用 or，如：--where 'deleted_at = 0 or deleted_at is null' |
| --sync | 开启同步开关 |
| -sh, --sync-host | 指定要同步的目标实例地址 |
| -sP, --sync-port | 指定要同步的目标实例端口 |
| -su, --sync-user | 指定连接到同步目标实例的用户 |
| -sp, --sync-password | 指定连接到同步目标实例的密码 |
| -sd, --sync-database | 指定连接到同步目标实例的库名 |
| -sC, --sync-charset | 指定连接到同步目标实例的字符集 |

测试
==============
* mysql 测试版本：mysql 5.7.32、mysql 8.0.24
* python测试版本：python 3.9.9

TODO
==============
- [x] 添加对 json 数据类型的支持
- [x] 添加对 blob 数据类型的支持（包括 binary、varbinary 等二进制类型）
- [x] 直接解析指定目录下binlog文件（可指定binlog目录前缀，排除非binlog文件）
- [x] 修复 binlogfile2sql json 数据解析处理的SQL执行不了的 bug
- [x] 修复 binlogfile2sql start-position 参数不生效的 bug
- [x] 添加 GTID 支持
- [x] 添加条件过滤功能，参数：--where
- [x] 添加数据同步支持：参数：--sync（注意，最好只同步DML，因为 --rename-db 参数对 DDL 不生效）
- [ ] 添加数据转换支持，将 SQL 转换成 CSV
