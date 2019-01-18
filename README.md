# 一、启动

## 1.命令行

-f，指定配置文件的路径
logstash -f ../conf/logs.conf

-e 后面跟字符串
启动，可以使用带e的参数进行启动，不过仅仅使用于命令行
bin/logstash -e 'input { stdin { } } output { stdout {} }'

-l 日志输出地址，默认是stdout

-t 测试配置是否正确

PIPELINE是关于input->filter->output的流水线称呼，有关于他的配置成为piplineconfig

	--pipeline.id ID
	-M, --modules.variable
	-w, --pipeline.workers COUNT
	-b, --pipeline.batch.size SIZE
	-u, --pipeline.batch.delay DELAY_IN_MS
	--log.format FORMAT			//注释'--log.format': Invalid value "JSON". Options are: ["json", "plain"]
	--node.name NAME
	-f, --path.config CONFIG_PATH
	-e, --config.string CONFIG_STRING
	--config.debug
	--log.level LEVEL
	-t, --config.test_and_exit
	-r, --config.reload.automatic
	-V, --version

# 二、Yaml 配置
**严重提醒，配置冒号后面必须必须必须有一个空格
config.reload.interval: 3**

支持两种配置方式
##1.yml

	pipeline:
	  batch:
	    size: 125
	    delay: 50


## 2.序列
	pipeline.batch.size: 125
	pipeline.batch.delay: 50

支持环境变量引用

	pipeline:
	  batch:
	    size: ${BATCH_SIZE}
	    delay: ${BATCH_DELAY:50}

> 注意到  
> ${VAR_NAME:default_value}  
> 分别是 环境变量名:默认值

模块配置方式

	modules:
	  - name: MODULE_NAME1
	    var.PLUGIN_TYPE1.PLUGIN_NAME1.KEY1: VALUE
	  - name: MODULE_NAME2
	    var.PLUGIN_TYPE1.PLUGIN_NAME1.KEY1: VALUE
	    var.PLUGIN_TYPE1.PLUGIN_NAME1.KEY2: VALUE

常用的配置说明

	queue.max_bytes				队列最大容量
	queue.max_events			最大事件数
	path.queue					队列缓存目录
	queue.type					队列类型 memory/persisted
	config.reload.interval 		配置重载时间间隔
	path.config					默认情况下，是
	http.host					绑定主机
	http.port					rest 端口
	log.level					指定日志级别
	config.reload.automatic		自动重新加载



## 3.目录与映射

	home			{extract.path}- Directory created by unpacking the archive
	bin				{extract.path}/bin
	settings		{extract.path}/config		path.settings
	logs			{extract.path}/logs			path.logs
	plugins			{extract.path}/plugins		path.plugins
	data			{extract.path}/data			path.data


## 4.安全配置(keystore)
在配置logstash时，可能需要指定敏感设置或者配置，可以使用logstash 密钥库来存储   
引用键的语法和环境变量相同   
${KEY}

例如在配置文件.conf

	output { elasticsearch {...password => "${ES_PWD}" } } }
也可以在logstash.yml文件中使用

	xpack.management.elasticsearch.password: ${ES_PWD}

怎么设置秘钥库的密码
通过设置环境变量

> LOGSTASH_KEYSTORE_PASS 


然后再执行下面脚本

	bin / logstash-keystore create
logstash 用户运行的时候，需要有这个环境变量的定义，否则logstash 将无法
访问密钥库的内容



# 5.管道配置

## 5.1管道的基础配置如下

	# This is a comment. You should use comments to describe
	# parts of your configuration.
	input {
	  ...
	}
	
	filter {
	  ...
	}
	
	output {
	  ...
	}

## 5.2例子
	input {
	  file {
	    path => "/var/log/messages"
	    type => "syslog"
	  }
	
	  file {
	    path => "/var/log/apache/access.log"
	    type => "apache"
	  }
	}


## 5.3值类型
### a)数组
	users => [ {id => 1, name => bob}, {id => 2, name => jane} ]

### b)布尔
	ssl_enable => true

## c)字节
	my_bytes => "1113"   # 1113 bytes
	my_bytes => "10MiB"  # 10485760 bytes
	my_bytes => "100kib" # 102400 bytes
	my_bytes => "180 mb" # 180000000 bytes

### d)CodeC
	codec => "json"
  
### 哈希
	match => {
	  "field1" => "value1"
	  "field2" => "value2"
	  ...
	}


### 数字
	port => 33

### 注释
	# this is a comment


### CodeC插件
关于codec插件的使用
[https://www.elastic.co/guide/en/logstash/current/codec-plugins.html](https://www.elastic.co/guide/en/logstash/current/codec-plugins.html "https://www.elastic.co/guide/en/logstash/current/codec-plugins.html")   

	CodeC插件修改了事件数据的展示方式
	codec=>protobuf 		读取谷歌序列化框架的数据
	codec=>line				读取一行文本
	codec=>json
	codec=>json_lines
	codec=>dots				从控制台输出一个.以便对消息进行调试
	codec=>collectd			通过udp协议读取数据
	codec=>gzip_lines		读取gzip编码的内容
	codec=>metadata
	一共有一个属性metadata，用于设置是否需要展示元数据

# 读取多行
	codec => multiline {
	      pattern => "pattern, a regexp"		//正则匹配为一行
	      negate => "true" or "false"			//要求正则不能匹配到，
	      what => "previous" or "next"			//如果匹配，这个消息属于后一个还是前一个事件
	    }

例子1
下面这个这么理解，如果不是以时间开头的，那么久属于前一个消息的

	input {
	  file {
	    path => "/var/log/someapp.log"
	    codec => multiline {
	      # Grok pattern names are valid! :)
	      pattern => "^%{TIMESTAMP_ISO8601} "
	      negate => true
	      what => "previous"
	    }
	  }
	}

日志如下

	1	2012-01-01 this is a short line
	2	2012-01-02 this is a long line
	3		which exceed more than one single line
	4	2012-01-03 this is a short line

其他

	input {
	  stdin {
	    codec => multiline {
	      pattern => "^\s"
	      what => "previous"
	    }
	  }
	}
以空格开始的一行属于前一个事件，也就是杀了这一行时间，把他丢到前面去


对于第三行，将属于第二个消息的，而这个由what 决定到底是属于前还是属于后


# 事件
如何在配置文件中接触到Event数据（事件数据）   
*首先我们得知道什么是事件数据，事件数据是在pipeline之间传递的数据*   

https://www.elastic.co/guide/en/logstash/current/first-event.html

注意由于，时间是在input生成，filter过滤，output处理
所以，input时候是不嫩使用event，因为他还没有存在

> Field references, sprintf format and conditionals, described below, will not work in an input block.


# 字段的引用
对于事件中，一共有三类数据可以引用

	{
	  "agent": "Mozilla/5.0 (compatible; MSIE 9.0)",
	  "ip": "192.168.24.44",
	  "request": "/index.html"
	  "response": {
	    "status": 200,
	    "bytes": 52353
	  },
	  "ua": {
	    "os": "Windows 7"
	  },
	  "tags":{
		[0] "msg1"
		[1] "msg2"
	  },
	  "@timestamp" => 2019-01-17T07:40:43.597Z,
	  "@version" => "1",
	  "@metadata" => {
			"test" => "Hello first metadata",
			"no_show" => "this data will not be in the output"
	   }
	}
假设我们的Event如上面，那么
1.field，就是这个json对象的key
如果我们需要引用这个%{}格式即可
如%{[ip]}就可以得到ip地址、

对于多级可以使用,[ua][os]方式进行调用
对于列表，可以使用[tags][0]等下标 的方式进行获取


## sprintf 格式化
sprintf format 允许对这些属性进行引用
如下
	
	format => "%{host},\n%{[@metadata][test]} \n,%{@metadata} \n%{[@metadata]}\n%{tags}\n%{[tags][0]}"
同样的，也可以对@timestamp进行格式化成为一个字符串

## 时间类型格式化
使用+FORMAT 进行格式化   
如   

- %{+yyyy.MM.dd.hh} 2012.12.12.12   
- %{+yyyy-MM-dd HH:mm:ss}
   
参见http://joda-time.sourceforge.net/apidocs/org/joda/time/format/DateTimeFormat.html

## 表达式引用
判断表达式
	
	if EXPRESSION {
	  ...
	} else if EXPRESSION {
	  ...
	} else {
	  ...
	}

运算符

1. 普通：== != < > <= >=
1. 正则：=~ ，!~,				校验右边正则是否匹配左边
1. 包含：in, not in
1. 布尔：and,or,nand,xor
1. 其他：!

### 例子1

	filter {
	  if [action] == "login" {
	    mutate { remove_field => "secret" }
	  }
	}

### 例子2
