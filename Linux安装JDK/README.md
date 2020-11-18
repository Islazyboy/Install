##### 1.首先要确保虚拟机上没有安装过jdk

```
java -version
```

##### 2.上传jdk并解压

```
tar -zxvf jdk*
```

##### 3.移动解压后的jdk文件到/usr/java/下并改名

```
cd /usr/	#移动到usr目录下
mkdir java	#在usr目录下创建java文件夹
cd ~	#进入到根目录
mv jdk1.8.0_271/ /usr/java/jdk18  #移动改名操作
```

###### 4.配置环境变量
<img src="https://images2017.cnblogs.com/blog/1169636/201709/1169636-20170912124501219-42493989.png">
<ol>首先输入命令
	

```
vim /etc/profile
```

<li>将下面代码粘到最底层</li>
<img src="https://images2017.cnblogs.com/blog/1169636/201709/1169636-20170912124844453-896188982.png">

```
export JAVA_HOME=/usr/java/jdk18  #jdk安装目录
 
export JRE_HOME=${JAVA_HOME}/jre
 
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib:$CLASSPATH
 
export JAVA_PATH=${JAVA_HOME}/bin:${JRE_HOME}/bin
 
export PATH=$PATH:${JAVA_PATH}
```
<img src="https://images2017.cnblogs.com/blog/1169636/201709/1169636-20170912124918110-734280848.png">
<li>输入命令保存</li></ol>

```
(ESC):wq!
```

##### 5.让环境变量生效
<img src="https://images2017.cnblogs.com/blog/1169636/201709/1169636-20170912124952125-844663745.png">

```
source /etc/profile
```

##### 6.测试

输入

```
java -version
```

测试安装是否成功