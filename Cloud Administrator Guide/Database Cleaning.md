# 数据清理
Barbican数据库中条目是软删除的，并且会随着时间的推移不断构建。这些条目可以通过执行清理命令进行清理。该命令应该与cron作业命令一起使用，以达到按时间间隔自动清理数据库。  

# 命令  
命令’**barbican-manage db clean**‘被用来清理数据库。通常它至少90天删除一次软删除。  

’**barbican-manage db clean --min-days 180**‘ （’**-m**‘）将遍历数据库并将删除自从上次删除依赖至少90天的软删除条目。默认90天。’**--min-days 0**‘将删除到今天为止的所有软删除条目。  

’**barbican-manage db clean --clean-unassociated-projects**‘ （’**-p**‘）将遍历数据库并杉树没有相关资源的项目。默认为false。  

’**`barbican-manage db clean --soft-delete-expired-secrets`**‘ （’**-e**‘）将遍历数据库并软删除过了有效期的数据。默认为false。如果’**-e**‘与’**--min-days 0**‘联用，将硬删除所有过期的机密。  

’**barbican-manage db clean --verbose**‘（’**-V**‘）将更多信息输出到终端。  

’**barbican-manage db clean --log-file**‘（’**-L**‘）设置日志文件位置。如果执行命令的用户没有访问日志位置或目标目录不存在，则日志的创建可能会失败。可在’/etc/barbican/barbican.conf‘日志中找到log_file的默认值。日志包含命令的详细输出。  

# Cron作业  
可以在Linux系统中创建一个cron作业，以给定的时间间隔自行清理barbican数据库。  

## Crontab  
1. 执行清理命令的用户使用‘**crontab -e**’打开crontab编辑器。  
2. 编辑crontab，以给定的时间间隔执行命令。‘**crontab -e**’ '**\<minute 0-59> \<hour 0-23,0=midnight> \<day 1-31> \<month 1-12> \<weekday 0-6, 0=Sunday> clean up command**'  

# Crontab示例  
* ‘**00 00 * * * barbican-manage db clean  -p -e**’每天午夜执行命令：删除自上次软删除起，至少90天以内的软删除条目；清理无关项目；软删除已过期机密。  
* ‘**00 03 01 * * barbican-manage db clean -m 30**’每月1号凌晨3点执行命令：删除自上次软删除起，至少30天以内的软删除条目。  
* ‘**05 01 07 * 6 barbican-manage db clean -m 180 -p -e -L /tmp/barbican-clean-command.log**’每月7号凌晨1点5分和每个周六都执行命令：删除自上次软删除起，至少180天以内的软删除条目；清理无关项目；软删除已过期机密；日志存入/tmp/barbican-clean-command.log文件中。  