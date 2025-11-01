mysql重启报错如下
```shell
Most likely, you have hit a bug, but this error can also be caused by malfunctioning hardware.
Thread pointer: 0x7f2659b109e0
Attempting backtrace. You can use the following information to find out
where mysqld died. If you see no messages after this, something went
terribly wrong...
stack_bottom = 7f25ef68ec30 thread_stack 0x46000
/usr/sbin/mysqld(my_print_stacktrace(unsigned char const*, unsigned long)+0x3d) [0x20e9f6d]
/usr/sbin/mysqld(handle_fatal_signal+0x31b) [0xfc230b]
/lib64/libpthread.so.0(+0xf630) [0x7f3117c5a630]
/usr/sbin/mysqld(MDL_ticket::has_stronger_or_equal_type(enum_mdl_type) const+0x7) [0x1263e77]
/usr/sbin/mysqld(MDL_ticket_store::find_in_hash(MDL_request const&) const+0x65) [0x1265ab5]
/usr/sbin/mysqld(MDL_context::find_ticket(MDL_request*, enum_mdl_duration*)+0x15) [0x1265b55]
/usr/sbin/mysqld(MDL_context::try_acquire_lock_impl(MDL_request*, MDL_ticket**)+0x41) [0x1267741]
/usr/sbin/mysqld(MDL_context::acquire_lock(MDL_request*, unsigned long)+0x9f) [0x12681cf]
/usr/sbin/mysqld(open_table(THD*, TABLE_LIST*, Open_table_context*)+0x117c) [0xe1800c]
/usr/sbin/mysqld(open_tables(THD*, TABLE_LIST**, unsigned int*, unsigned int, Prelocking_strategy*)+0x473) [0xe1d673]
/usr/sbin/mysqld(dd::Open_dictionary_tables_ctx::open_tables()+0xb4) [0x20b5a04]
/usr/sbin/mysqld(bool dd::cache::Storage_adapter::get<dd::Item_name_key, dd::Abstract_table>(THD*, dd::Item_name_key const&, enum_tx_isolation, bool, dd::Abstract_table const**)+0xbc) [0x1ec72ec]
/usr/sbin/mysqld(bool dd::cache::Shared_dictionary_cache::get<dd::Item_name_key, dd::Abstract_table>(THD*, dd::Item_name_key const&, dd::cache::Cache_element<dd::Abstract_table>**)+0x7f) [0x1eba86f]
/usr/sbin/mysqld(bool dd::cache::Dictionary_client::acquire<dd::Item_name_key, dd::Abstract_table>(dd::Item_name_key const&, dd::Abstract_table const**, bool*, bool*)+0x27e) [0x1e748de]
/usr/sbin/mysqld(bool dd::cache::Dictionary_client::acquire<dd::Abstract_table>(std::basic_string<char, std::char_traits<char>, Stateless_allocator<char, dd::String_type_alloc, My_free_functor> > const&, std::basic_string<char, std::char_traits<char>, Stateless_allocator<char, dd::String_type_alloc, My_free_functor> > const&, dd::Abstract_table const**)+0x19e) [0x1e7628e]
/usr/sbin/mysqld(get_table_share(THD*, char const*, char const*, char const*, unsigned long, bool, bool)+0x7ff) [0xe165bf]
/usr/sbin/mysqld(open_table(THD*, TABLE_LIST*, Open_table_context*)+0xac4) [0xe17954]
/usr/sbin/mysqld(open_tables(THD*, TABLE_LIST**, unsigned int*, unsigned int, Prelocking_strategy*)+0x473) [0xe1d673]
/usr/sbin/mysqld(mysqld_show_create(THD*, TABLE_LIST*)+0x17f) [0xefebbf]
/usr/sbin/mysqld(mysql_execute_command(THD*, bool)+0x3550) [0xe95b20]
/usr/sbin/mysqld(mysql_parse(THD*, Parser_state*)+0x38b) [0xe97ccb]
/usr/sbin/mysqld(dispatch_command(THD*, COM_DATA const*, enum_server_command)+0x1e5a) [0xe99fca]
/usr/sbin/mysqld(do_command(THD*)+0x19c) [0xe9ac0c]
/usr/sbin/mysqld() [0xfb3730]
/usr/sbin/mysqld() [0x2665126]
/lib64/libpthread.so.0(+0x7ea5) [0x7f3117c52ea5]
/lib64/libc.so.6(clone+0x6d) [0x7f3116035b0d]

Trying to get some variables.
Some pointers may be invalid and cause the dump to abort.
Query (7f24b7186db8): show create table `blue`
Connection ID (thread ID): 2955931
Status: NOT_KILLED

The manual page at http://dev.mysql.com/doc/mysql/en/crashing.html contains
information that should help you find out what is causing the crash.
```
MySQL 8.0.20 是一个过渡版本，当时 bug 特别多，尤其是 数据字典 (DD) 和 event scheduler / metadata lock 相关的地方。
Oracle 官方自己都不推荐用 8.0.20（8.0.21 里就修了很多 crash bug）
建议至少升级到 MySQL 8.0.32+，目前社区推荐用 8.0.36 (GA 最新版)。
升级路径可以直接从 8.0.20 → 8.0.36，中间不用跳版本。
升级理由：
8.0.20 存在已知 crash bug（metadata lock、字典缓存、event scheduler）。
新版本修复了大量 InnoDB/DDL 崩溃。
向后兼容，不会破坏数据