
#java实现SSL 的必要素 


### 1.创建HTTPS的一种方式 
 

	//初始化keystore实例
	KeyStore ks = KeyStore.getInstance("JKS");
	//加载已经存在的实体ks
	ks.load(new FileInputStream("/ssl/cliKs"),"chenchen".toCharArray());
	//创建用于管理JKS密钥库的密钥管理器
	TrustManagerFactory tmf = TrustManagerFactory.getInstance("SunX509");
	tmf.init(ks);

	TrustManager[] trustManagers = tmf.getTrustManagers();

	//构造SSL环境，指定SSL版本为3.0，也可以使用TLSv1，但是SSLv3更加常用
	SSLContext ctx = SSLContext.getInstance("TLSV1");
	
	//初始化SSL环境。第二个参数是告诉JSSE使用的可信任证书的来源，
	//设置为null是从javax.net.ssl.trustStore中获得证书。
	//第三个参数是JSSE生成的随机数，这个参数将影响系统的安全性，
	//设置为null是个好选择，可以保证JSSE的安全性。
	ctx.init(null, trustManagers, null);

	



### 2.如下是第二种方式进行创建

	//第二种方式
	System.setProperty("javax.net.ssl.keyStore", "/ssl/testkeystore");
	System.setProperty("javax.net.ssl.keyStorePassword", "chenchen");
	System.setProperty("javax.net.ssl.trustStore", "/ssl/testtrust");
	System.setProperty("javax.net.ssl.trustStorePassword", "chenchen");




> 一旦经过了上面的步骤一，或者二，就可以直接通过getDefault进行获取**SSLContext**

	SSLServerSocketFactory socketFactory = (SSLServerSocketFactory) SSLServerSocketFactory.getDefault();
	socket = (SSLSocket) socketFactory.getSocketFactory(7070)
