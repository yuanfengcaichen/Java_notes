https://blog.csdn.net/weixin_40326608/article/details/100123178

# 1 配置Procrun

验证jar包好用之后，就是本篇的重点了，配置Procrun。
Procrun是Apache推出的一套能让Java应用程序在Windows平台以服务的方式运行的插件。它主要包括两个程序：

- 服务应用程序（名为prunsrv.exe），用于转换任一应用程序作为Win服务运行。
- 监视器应用程序（名为prunmgr.exe），用于监视和配置procrun服务。

## 1.1 下载Procrun

当时我找这个下载地址还很费了一点劲儿哈哈。
地址为： http://www.apache.org/dist/commons/daemon/binaries/windows/
下载名为：commons-daemon-1.2.0-bin-windows.zip的压缩包，如下图：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190829152349710.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDMyNjYwOA==,size_16,color_FFFFFF,t_70)
下载并解压后，长这个样子：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190829171905326.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDMyNjYwOA==,size_16,color_FFFFFF,t_70)

## 1.2 组织目录设置

首先我们创建一个目录，比如我这里叫做SpringForWinServiceDemo_Procrun，然后在其下创建三个文件夹，分别是JAR、Logs和Service，如下图：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190829172631567.png)
JAR文件夹，用来放Spring打出来的jar包，如下图：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190829172724235.png)
Logs文件夹用来保存Win Service的运行日志，目前是空的。
Service文件夹用来存放Procrun的exe。首先，将解压的commons-daemon-1.2.0-bin-windows\amd64文件夹下的prunsrv.exe，拷贝到Service文件夹中，如下图：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190829173217303.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDMyNjYwOA==,size_16,color_FFFFFF,t_70)
其次，将commons-daemon-1.2.0-bin-windows文件夹下的prunmgr.exe，拷贝到Service文件夹中，并重命名，我这里命名为SpringForWinServiceDemo.exe，如下图：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190829173353956.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDMyNjYwOA==,size_16,color_FFFFFF,t_70)
这样组织目录就设置好了。

## 1.3 编写安装服务的脚本

在编写安装服务的脚本install.bat之前，有个**非常重要，但是非常容易被忽视的问题**（别的博客也没有提到过），这个问题会直接影响你的服务是否能够运行起来，那就是：
“**确认你的jar包的Main-Class**”。
对于我这个工程，我最开始想都不想，就觉得jar包里的Main-Class就是com.demo.springforwinservice.SpringforwinserviceApplication，然后配到脚本里，服务倒是装上了，但是怎么都启动不起来。。。蛋疼，最后搞了半天才得以解决，方法如下：
找到jar包，使用压缩软件打开，如下图：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190829174856658.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDMyNjYwOA==,size_16,color_FFFFFF,t_70)
进入这个META-INF文件夹，使用记事本打开MANIFEST.MF文件，如下图
![在这里插入图片描述](https://img-blog.csdnimg.cn/2019082917494533.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDMyNjYwOA==,size_16,color_FFFFFF,t_70)
看见了么，Main-Class是org.springframework.boot.loader.JarLauncher，而不是com.demo.springforwinservice.SpringforwinserviceApplication。
确定了这个后，开始写install.bat，如下：

```shell
@echo off
 
rem 设置程序名称
set SERVICE_EN_NAME=SpringForWinServiceDemo
set SERVICE_CH_NAME=Spring演示服务
 
rem 设置java路径
set JAVA_HOME=%JAVA_HOME%
 
rem 设置程序依赖及程序入口类
cd..
set BASEDIR=%CD%
set CLASSPATH=%BASEDIR%\JAR\springforwinservice-0.0.1-SNAPSHOT.jar
set MAIN_CLASS=org.springframework.boot.loader.JarLauncher
 
rem 设置prunsrv路径
set SRV=%BASEDIR%\Service\prunsrv.exe
 
rem 设置日志路径及日志文件前缀
set LOGPATH=%BASEDIR%\Logs
 
rem 输出信息
echo SERVICE_NAME: %SERVICE_EN_NAME%
echo JAVA_HOME: %JAVA_HOME%
echo MAIN_CLASS: %MAIN_CLASS%
echo prunsrv path: %SRV%
 
rem 设置jvm
if "%JVM%" == "" goto findJvm
if exist "%JVM%" goto foundJvm
:findJvm
set "JVM=%JAVA_HOME%\jre\bin\server\jvm.dll"
if exist "%JVM%" goto foundJvm
echo can not find jvm.dll automatically,
echo please use COMMAND to localation it
echo then install service
goto end
:foundJvm
echo 正在安装服务...
rem 安装
"%SRV%" //IS//%SERVICE_EN_NAME% --DisplayName="%SERVICE_CH_NAME%" "--Classpath=%CLASSPATH%" "--Install=%SRV%" "--JavaHome=%JAVA_HOME%" "--Jvm=%JVM%" --JvmMs=256 --JvmMx=1024 --Startup=auto --JvmOptions=-Djcifs.smb.client.dfs.disabled=false ++JvmOptions=-Djcifs.resolveOrder=DNS --StartMode=jvm --StartClass=%MAIN_CLASS% --StartMethod=main --StopMode=jvm --StopClass=%MAIN_CLASS% --StopMethod=main --StopParams=  --LogPath=%LOGPATH% --StdOutput=auto --StdError=auto
echo 安装服务完成。
pause
12345678910111213141516171819202122232425262728293031323334353637383940414243
```

详细解释如图：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190829175659803.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDMyNjYwOA==,size_16,color_FFFFFF,t_70)

## 1.4 编写卸载服务的脚本

卸载服务的uninstall.bat脚本比较简单，编写完成后同样放到Service文件夹下即可，如下：

```shell
@echo off
 
cd..
set basedir=%CD%
set SERVICE_NAME=SpringForWinServiceDemo
set SRV=%BASEDIR%\Service\prunsrv.exe
echo 正在卸载服务...
"%SRV%" //DS//%SERVICE_NAME%
echo 服务卸载完毕。
pause
12345678910
```

# 2. 部署Windows服务

## 2.1 安装服务

执行install.bat，如下图：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190830112022719.png)
提示“安装服务完成”后，在Windows任务栏的搜索框中直接输入“服务”或者“Service”，就可以打开Windows服务，如下图：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190830100300362.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDMyNjYwOA==,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190830112057104.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDMyNjYwOA==,size_16,color_FFFFFF,t_70)
看，服务已经装好了。

## 2.2 启动服务

装好的服务还没有启动，我们需要手动将其启动，回到Service文件夹，双击打开SpringForWinServiceDemo.exe，然后点击“开始”，如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/2019083011305830.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDMyNjYwOA==,size_16,color_FFFFFF,t_70)
如果一切正常的话，服务就可以成功启动了，如下图：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190830113252504.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDMyNjYwOA==,size_16,color_FFFFFF,t_70)
再次回到Windows服务列表页，可以看到服务的状态为“正在运行”，如下图：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190830113427250.png)
注意，如果服务起不来的话，一定要去检查Procrun的log，在这里：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190830113526894.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDMyNjYwOA==,size_16,color_FFFFFF,t_70)

## 2.3 调用服务

仍然使用Postman，来调用下这个Windows服务的接口，如下图：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190830113807871.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDMyNjYwOA==,size_16,color_FFFFFF,t_70)
Good, every good!
本地服务部署成功咯~~

## 2.4 卸载服务

如果想要卸载服务，运行uninstall.bat即可，如下图：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190830114010826.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDMyNjYwOA==,size_16,color_FFFFFF,t_70)
然后重启电脑即可看见对应的服务已经没有了~~~~