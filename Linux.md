
# 命令大全
## wc
参数 -l -c -b分别用于统计行数，字数，字节数

	$>ls -l |wc
    12     101     744

##watch
可以将命令的输出结果输出到标准输出设备，多用于周期执行  
	
	-n 指定多少秒执行一次
	-d 高亮差异
	-t 关闭title

### 1)查看网络攻击
	netstat -d -n 2 -an|grep :9200|wc -l|xargs|awk '{print "端口个数:"$1}'
 
## tr命令
tr命令用于替换字符串 

	# echo $username
	jay,li,may,kk
	# echo $username|tr "," '?'
	jay?li?may?kk




# 实用脚本

	watch -n 2 -d 'top -b -n 1|head -n +5|xargs echo `date "+%F %T [LOG]"` |tee -a useage.log'
