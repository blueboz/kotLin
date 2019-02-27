# 关于Gradle 原理的深究

https://www.cnblogs.com/softidea/p/4368506.html
第17点

可以通过metaClass对属性以及方法进行动态的添加与删除等操作

tasks.withType(JavaCompile) {  
    options.encoding = "UTF-8"  
}

def tss=getProperty("tasks");
print(tss)

for(def p:this.properties) {
	println p.key;
}

this.metaClass.methods.forEach{
	println it.name
}
