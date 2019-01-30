# Hadoop深入

## 1.core-site.xml文件

	<property>
	    <name>fs.defaultFS</name>
	    <value>hdfs://localhost:9000</value>
	</property>
	
	<!--可以指定额外的参数-->
	<property>
		<!--临时目录-->
		<name>hadoop.tmp.dir</name>
		<value>/D:/develop/hadoop/workplace/tmp</value>
	</property>
	<!--指定namenode的存放目录-->
	<property>
		<name>dfs.name.dir</name>
		<value>/D:/develop/hadoop/workplace/name</value>
	</property> 
	<property>
		<name>fs.default.name</name>
		<value>hdfs://localhost:9000</value>
	</property> 

一般情况下，指定完tmpdir之后，namedir就已经确定了

## dfs client

生成一个dfsClient，方法

	Configuration cfg = new Configuration();
	cfg.set("fs.defaultFS", "hdfs://localhost:9000");
	FileSystem fs = FileSystem.get(cfg);
	RemoteIterator<LocatedFileStatus> fi = fs.listFiles(new Path("/"), true);
	while(fi.hasNext()) {
		LocatedFileStatus next = fi.next();
		Path path = next.getPath();
		System.out.println(path.toString());
	}
## mapreduce 

### Split

关于splite的补充，split与block是不同的概念，block是物理上的切分
而split 是逻辑上的划分

	
mapTask 的个数取决于split的个数 ，split个取决于切片的大小以及文件的大小

默认情况下，切分的大小是按照32M进行切分文件的，如下图

	#最大按照32M进行切分
	file:/C:/hadoop/input/logback.log:0+33554432
	file:/C:/hadoop/input/logback.log:33554432+33554432
	file:/C:/hadoop/input/logback.log:67108864+33554432
	file:/C:/hadoop/input/logback.log:100663296+33554432
	file:/C:/hadoop/input/logback.log:134217728+32189152

但是我们也可以进行手动的修改。
如何修改

#### 方法1 
	
在代码中直接进行指定  

	FileInputFormat.setInputPaths(job,inpath);
	FileInputFormat.setMaxInputSplitSize(job, 1024*2);
	FileInputFormat.setMinInputSplitSize(job, 1024);

#### 方法2
	
	#通过对job中，设置后面两个参数接口，maxsize与miin
	mapreduce.input.fileinputformat.inputdir:file:/C:/hadoop/input
	mapreduce.input.fileinputformat.list-status.num-threads:1
	mapreduce.input.fileinputformat.split.maxsize:2048
	mapreduce.input.fileinputformat.split.minsize:1024

#### 结果
	
	#在后面的日志变回出现下面这种偏移量的移动
	file:/C:/hadoop/input/sweet story.txt:0+2048			#递增2048
	file:/C:/hadoop/input/sweet story.txt:2048+2048
	file:/C:/hadoop/input/sweet story.txt:4096+2048
	file:/C:/hadoop/input/sweet story.txt:6144+2048
	file:/C:/hadoop/input/sweet story.txt:8192+2048
	file:/C:/hadoop/input/sweet story.txt:10240+2048


### Block
默认情况下是按照128M进行文件的切分的。
。如果切分过程中，不够，将会存在一个小文件


### CombineTextInputFormat

#### 代码 
	CombineTextInputFormat.setInputPaths(job, inpath);
	CombineTextInputFormat.setMaxInputSplitSize(job, 1024*1024*20);
	//设置最小分片大小
	CombineTextInputFormat.setMinInputSplitSize(job, 1024*1024*10);

#### 配置
	mapreduce.input.fileinputformat.inputdir:file:/C:/hadoop/input
	mapreduce.input.fileinputformat.list-status.num-threads:1
	mapreduce.input.fileinputformat.split.maxsize:20971520
	mapreduce.input.fileinputformat.split.minsize:10485760


只有一个切片，因为文件远远未达到配置的目标值

	Processing split: 
	Paths:
	/C:/hadoop/input/sweet story (1).txt:0+2419,
	/C:/hadoop/input/sweet story (10).txt:0+2419,
	/C:/hadoop/input/sweet story (11).txt:0+2419,
	/C:/hadoop/input/sweet story (12).txt:0+2419,
	/C:/hadoop/input/sweet story (13).txt:0+2419,
	/C:/hadoop/input/sweet story (14).txt:0+2419,
	/C:/hadoop/input/sweet story (15).txt:0+2419,
	/C:/hadoop/input/sweet story (16).txt:0+2419,
	/C:/hadoop/input/sweet story (17).txt:0+2419,
	/C:/hadoop/input/sweet story (18).txt:0+2419,
	/C:/hadoop/input/sweet story (19).txt:0+2419,
	/C:/hadoop/input/sweet story (20).txt:0+2419,
	/C:/hadoop/input/sweet story (8).txt:0+2419,
	/C:/hadoop/input/sweet story (9).txt:0+2419

如果是将大小设置成如下

	//这个一定要在最前面,否则会报错
	job.setInputFormatClass(CombineTextInputFormat.class);

	CombineTextInputFormat.setInputPaths(job, inpath);
	CombineTextInputFormat.setMaxInputSplitSize(job, 2419*5);
	//设置最小分片大小
	CombineTextInputFormat.setMinInputSplitSize(job, 2419*5);
	//Combiner 与reducer 是一样的

那么合并的时候，将会是3个切片，每个切片大小是5*2419
	
	 Processing split: Paths:
		/C:/hadoop/input/sweet story (1).txt:0+2419,
		/C:/hadoop/input/sweet story (10).txt:0+2419,
		/C:/hadoop/input/sweet story (11).txt:0+2419,
		/C:/hadoop/input/sweet story (12).txt:0+2419,
		/C:/hadoop/input/sweet story (13).txt:0+2419
	 Processing split: Paths:
		/C:/hadoop/input/sweet story (14).txt:0+2419,
		/C:/hadoop/input/sweet story (15).txt:0+2419,
		/C:/hadoop/input/sweet story (16).txt:0+2419,
		/C:/hadoop/input/sweet story (17).txt:0+2419,
		/C:/hadoop/input/sweet story (18).txt:0+2419
	 Processing split: Paths:
		/C:/hadoop/input/sweet story (19).txt:0+2419,
		/C:/hadoop/input/sweet story (20).txt:0+2419,
		/C:/hadoop/input/sweet story (8).txt:0+2419,
		/C:/hadoop/input/sweet story (9).txt:0+2419
