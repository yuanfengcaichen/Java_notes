# Java笔记

## IO流

IO流包的继承关系（字节流）

java.lang.Object

​	java.io.InputStream/java.io.OutputStream

​		java.io.FilterInputStream/java.io.OutputStream

​			java.io.BufferedInputStream/java.io.BufferedOutputStream(不必为写入每个字节而导致底层系统的调用)

IO流包的继承关系（字符流）

- Reader
  - InputStreamReader -- FileReader
  - BufferedReader
- Writer
  - OutputStreanWriter -- FileWriter
  - BufferedWriter

IO流分类：

- 按数据的流向

  - 输入流：读数据
  - 输出流：写数据

- 按数据类型来分

  - 字节流
    - 字节输入流
    - 字节输出流
  - 字符流
    - 字符输入流
    - 字符输出流

  一般读得懂的也就是文本类型的使用字符流

  读不懂的，也就是图片、视频等使用字节流

### 字节流

InputStream\OutputStream(两个抽象基类)

子类名称特点：子类名称都是以其父类名作为子类名的后缀

字节流写入：

new FileOutputStream(文件地址)

FileOutputStream对象.write(String.getBytes());

字节流追加:

new FileOutputStream(文件地址,true);

FileOutputStream对象.write(String.getBytes());

### 字节缓冲流（真正读写数据还要依靠基本的字节流对象进行操作）

BufferedOutputStream与BufferedInputStream

创建方法：

BufferedOutputStream(OutputStream out)

BufferedInputStream(InputStream in)

#### 写文件

创建BufferOutputStream对象，通过该对象的write方法写文件

**注意写的内容需要是转为通过写入内容的getByte方法获取byte[]数组**

```java
BufferedOutputStream buf = new BufferedOutputStream(new FileOutputStream("writefile\\test.text"));
buf.write("hello\r\n".getBytes());
buf.write("world\r\n".getBytes());
buf.close();
```

#### 读文件

```java
BufferedInputStream bif = new BufferedInputStream(new FileInputStream("writefile\\test.text"));
//一次读取一个字节
int by;
while((by=bif.read())!=-1){
    System.out.println((char)by);
}
//一次读取一个字节数组
int len;
byte[] by1 = new byte[8192];//缓冲流默认接收的字节数组大小是8192
while((len=bif.read(by1))!=-1){
    System.out.println(new String(by1,0,len));
}
bif.close();
```

#### 复制视频

1. 创建字节输入流对象
2. 创建目的地输出流对象
3. 读写数据，释放资源

实现：四种方式复制视频，基本字节流与字节缓冲流相应的每次读取一个字节或一个字节数组，其中字节缓冲流读取字节数组为最优解（BufferoutputStream）

```java
 //基本字节流，每次读取一个字节
FileInputStream fis = new FileInputStream("E:\\writefile\\test01.text");
FileOutputStream fos = new FileOutputStream("E:\\writefile\\test02.text");
int by;
while ((by=fis.read())!=-1){
    fos.write(by);
}
fos.close();
fis.close();
//基本字节流，每次读取一个字节数组
FileInputStream fis1 = new FileInputStream("E:\\writefile\\test01.text");
FileOutputStream fos1 = new FileOutputStream("E:\\writefile\\test02.text");
byte[] bys = new byte[1024];
int len;
while ((len=fis1.read(bys))!=-1){
    fos1.write(bys,0,len);
}
fos1.close();
fis1.close();
//字节缓冲流，每次读取一个字节
BufferedInputStream bis = new BufferedInputStream(new FileInputStream("E:\\writefile\\test01.text"));
BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream("E:\\writefile\\test02.text"));
int by1;
while ((by1=bis.read())!=-1){
    bos.write(by1);
}
bos.close();
bis.close();
//字节缓冲流，每次读取一个字节数组
BufferedInputStream bis1 = new BufferedInputStream(new FileInputStream("E:\\writefile\\test01.text"));
BufferedOutputStream bos1 = new BufferedOutputStream(new FileOutputStream("E:\\writefile\\test02.text"));
byte[] bys1 = new byte[1024];
int len1;
while ((len1=bis1.read(bys1))!=-1){
    bos1.write(bys1,0,len1);
}
bos1.close();
bis1.close();
```

### 字符流

字符流 = 字节流+编码表

#### 字符流的编码解码问题

字符流抽象基类

- Reader:字符输入流的抽象类
- Writer：字符输出流的抽象类

字符流中和编码解码问题相关的两个类：

- InputStreamReader：字节流到字符流的桥梁
- OutputStreamWriter：字符流到字节流的桥梁

FileReader和FileWriter是InputStreamReader，OutputStreamWriter直接子类，用于读取和写入文件的便捷类

```java
FileOutputStream fileOutputStream = new FileOutputStream("writefile\\test01.text");
OutputStreamWriter osw = new OutputStreamWriter(fileOutputStream,"UTF-8");//new OutputStreamWriter(fileOutputStream)设置编码表和不设置编码表
osw.write("中国");
osw.close();
FileInputStream fileInputStream = new FileInputStream("writefile\\test01.text");
InputStreamReader isr = new InputStreamReader(fileInputStream,"UTF-8");//InputStreamReader(fileInputStream);
int ch;
while ((ch = isr.read())!=-1){
    System.out.println((char)ch);
}
/*未知正确性
char[] s = new char[1024];
int len;
while((len = isr.read(s))!=-1){
    System.out.println(s);
}*/
isr.close();
```

#### 字符流写数据5种方式

| 方法名                                  | 说明                 |
| --------------------------------------- | -------------------- |
| void write(int c)                       | 写一个字符           |
| void write(char[] cbuf)                 | 写入一个字符数组     |
| void write(char[] cbuf,int off,iny len) | 写入字符数组的一部分 |
| void write(String str)                  | 写一个字符串         |
| void write(String str,int off,int len)  | 写一个字符串的一部分 |

```java
OutputStreamWriter osw = new OutputStreamWriter(new FileOutputStream("writefile\\test01.text"));
osw.write(97);//写入一个字母a
//此时需要使用osw的flush方法才能真正写入，但是如果直接调用osw的close方法，也会写入，因为在osw的close方法中也存在flush方法
osw.flush();
char[] chs = {'a','b','c'};//写入一个字符数组
osw.write(chs);
osw.flush();
char[] chs1 = {'a','b','c','d','e','f'};//写入一个字符数组的一部分
osw.write(chs,0,3);
osw.flush();
osw.write("写入一个字符串");//写入一个字符串
osw.flush();
osw.write("写入一个字符串",0,2);//写入一个字符串的一部分
osw.flush();
osw.close();
```

#### 字符流读取数据

```java
InputStreamReader isr = new InputStreamReader(new FileInputStream("writefile\\test01.text"));
int ch;
while((ch=isr.read())!=-1){
    System.out.print((char)ch);
}
int len;
char[] cs = new char[1024];
while((len=isr.read(cs))!=-1){
    System.out.println(new String(cs,0,len));
}
isr.close();
```

#### 字符缓冲流

BufferedWriter与BufferedReader

将文本写入字符输出流，缓冲字符，以提供单个字符

将字符输入流读取文本，缓冲字符

构造方法：

BufferedWriter(Writer out)

BufferedReader(Reader in)

```java
BufferedWriter bw = new BufferedWriter(new FileWriter("testwrite.text"));
bw.write("hello\r\n");
bw.write("world\r\n");
bw.close();
BufferedReader br = new BufferedReader(new FileReader("testwrite.text"));
//一次读取一个字符数据
int ch;
while ((ch=br.read())!=-1){
    System.out.println((char)ch);
}
char[] chs = new char[1024];
int len;
while ((len=br.read(chs))!=-1){
    System.out.println(new String(chs,0,len));
}
br.close();
```



### 编码解码问题

```java
//编码
byte[] b1 = "这是一个字符串".getBytes();
byte[] b2 = "这是一个字符串".getBytes("UTF-8");
//解码
String s1 = new String(b1);
String s2 = new String(b2,"UTF-8");
```

### 标准输入输出流

InputStream in:标准输入流

PrintStream out:标准输出流

```java
InputStream in = System.in;
int by;
while((by=in.read())!=-1){
    System.out.println((char)by);
}
InputStreamReader isr = new InputStreamReader(in);//字节流转为字符流
//通过缓冲流传入字符流对象实现一次读取一行数据
BufferedReader br = new BufferedReader(isr);
String line = br.readLine();//读取String类型字符串
System.out.println(line);
Integer line = Integer.parseInt(br.readLine());//读取Integer类型数字

//自己实现键盘录入太麻烦了，Java提供了一个类供我们使用
Scanner sc = new Scanner(System.in);//Scanner对象有读取一行的方法
```

### 打印流

- 字节打印流PrintStream
- 字符打印流PrintWriter

打印流只负责输出数据，不负责读取数据，有自己的特有方法(同样也是写文件)

```java
PrintStream ps = new PrintStream("writefile\\test01.text");
ps.print(97);
ps.close();
```

#### 字符打印流PrintWriter

构造方法：

PrintWriter(String fileName)：使用指定文件名创建一个新的PrintWriter，而不需要自动执行刷新

PrintWriter(Write out,boolean autoFlush)：创建一个新的PrintWriter,如果布尔值为真，则println\printf\format方法将刷新输出缓冲区

```java
PrintWriter pw =new PrintWriter("writefile\\test01.text");
pw.write("helo");
pw.flush();
PrintWriter pw =new PrintWriter(new FileWriter("writefile\\test01.text"),true);
pw.println("hellojava");//此时不需要刷新 
```



### 文件打印流

FileReader与FileWrite 

```java
BufferedReader bfr = new BufferedReader(new FileReader("src\\student.java"));
BufferedWriter bfw = new BufferedWriter(new FileWriter("file02.text"));
String line;
while ((line=bfr.readLine())!=null){
    bfw.write(line);
    bfw.newLine();
    bfw.flush();
}
bfw.close();
bfr.close();

BufferedReader bfr1 = new BufferedReader(new FileReader("src\\student.java"));
PrintWriter pw = new PrintWriter(new FileWriter("file02.text"),true);
String line1;
while ((line1=bfr1.readLine())!=null){
    pw.println(line1);
}
pw.close();
bfr1.close();
```



#### FileReader与FileWrite

复制文件（视频中的复制文件中的异常处理）

```java
FileReader fr = new FileReader("test01.text");
FileWriter fw = new FileWriter("test02.text");
char[] chs = new char[1024];
int len;
while ((len=fr.read(chs))!=-1){
    fw.write(chs,0,len);
}
fw.close();
fr.close();
```

```java
//每次读取一个字节的方式
int ch;
while((ch=fr.read())!=-1){
    fw.write(ch);
}
```



### 写文件的多种方式



### Properties作为Map集合的使用

Properties继承自HashTable

```java
public static void s11(String[] args) {
    //创建集合对象
    Properties prop = new Properties();
    //存储元素
    prop.put("lisi",18);
    prop.put("xiaoming",20);
    prop.put("ss",26);
    Set set = prop.keySet();
    for(Object s : set){
        System.out.print(s+": ");
        System.out.println(prop.get(s));
    }
}
```

读取数据

| 方法名                                      | 说明                                                         |
| ------------------------------------------- | ------------------------------------------------------------ |
| Object setProperty(String ket,String value) | 设置集合的键和值，都是String类型，底层调用Hashtable方法put   |
| String getProperty(String key)              | 使用此属性列表中指定的键搜索属性                             |
| Set<String> stringPropertyNames()           | 从该属性列表中返回一个不可修改的键集，其中键及其对应的值是字符串（就是Hashmap对象的keySet对象） |

student.text文件中的内容是



```text
className=student
methodName=study
```

| 方法名（Properties对象）   | 说明                                         |
| -------------------------- | -------------------------------------------- |
| load(InputStream/Reader)   | 以字节流或字符流的方式读取属性列表（键值对） |
| store(OutputStream/Writer) | 将属性列表写入该Properties表中               |



```java
public static void main(String[] args) throws IOException {
    Properties prop = new Properties();
    FileReader fr = new FileReader("student.text");
    prop.load(fr);
    String className = prop.getProperty("className");
    String methodName = prop.getProperty("methodName");
    System.out.println(className);
    System.out.println(methodName);
}
```



## 对象序列化

对象序列化流：ObjectOutputStream

对象反序列化流：ObjectInputStream

分别继承自OutputStream和InputStream

### ObjectOutputStream

构造方法：

ObjectOutputStream（OutputStream out）:创建一个写入指定的OutputStream的ObjectOutputStream

序列化对象的方法：

void writeObject（Object obj）:将指定的对象写入ObjectOutputStream

```java
ObjectOutputStream oos=new ObjectOutputStream(new FileOutputStream("student-seralize.text"));
student s = new student("李四",18,"swim");
oos.writeObject(s);
oos.close();
```

### ObjectInputStream

构造方法：

ObjectInputStream（InputStream in）:创建一个读取指定的InputStream的ObjectInputStream

序列化对象的方法：

Object readObject（）:将指定的文件进行读取，返回Object类

```java
ObjectInputStream ois = new ObjectInputStream(new FileInputStream("student-seralize.text"));
Object o = ois.readObject();
student s =(student) o;
System.out.println(s);
ois.close();
```



## 网络编程

### UDP数据传输

#### 发送数据

1. 创建发送端Socket对象（DatagramSocket）
2. 创建数据，并将数据打包
3. 调用DatagramSocket对象的方法发送数据
4. 关闭发送端

```java
DatagramSocket ds = new DatagramSocket();
//DatagramPacket(byte buf[], int length,InetAddress address, int port)
byte[] by ="发送Udp数据包".getBytes();
int len = by.length;
InetAddress ia = InetAddress.getByName("192.168.1.1");
DatagramPacket dp = new DatagramPacket(by,len,ia,80);
ds.send(dp);
ds.close();
```

#### 接收数据

1. 创建接收端Socket对象
2. 创建数据包，用于接收数据
3. 调用DatagramSocket对象的方法接收数据
4. 解析数据包，并把数据在控制台显示
5. 关闭接收端

```java
DatagramSocket ds = new DatagramSocket(10085);
byte[] bs = new byte[1024];
DatagramPacket dp = new DatagramPacket(bs,bs.length);
ds.receive(dp);
byte[] data = dp.getData();
int len = dp.getLength();
String s = new String(data,0,len);
System.out.println("数据是："+s);
//简写方式
System.out.println("数据是："+new String(dp.getData(),0,dp.getLength()));
ds.close();
```

### Tcp数据传输

#### TCP发送数据

1. 创建客户端Socket对象
2. 获取输出流，写数据
3. 释放资源

```java
Socket socket = new Socket(InetAddress.getByName("192.168.1.1"),10085);
Socket socket1 = new Socket("192.168.1.1",10085);
OutputStream os = socket.getOutputStream();
os.write("hello tcp,I am coming".getBytes());
//os.close();
//接收服务器发送的反馈
InputStream is = socket.getInputStream();
byte[] bys = new byte[1024];
int len = is.read(bys);
String data = new String(bys,0,len);
System.out.println("服务端反馈"+data);
```

#### TCP接收数据

1. 创建服务器端的Socket对象（ServerSocket）
2. 获取输入流，读数据，并把数据显示在控制台
3. 释放资源

```java
ServerSocket ss = new ServerSocket(10085);
Socket s = ss.accept();
InputStream is = s.getInputStream();
byte[] bys = new byte[1024];
int len = is.read(bys);
String data = new String(bys,0,len);
System.out.println(data);
//给出反馈
OutputStream os=ss.getOutStream();
os.write("数据已经收到".getBytes());

s.close();
ss.close();
```

#### TCP数据传输练习

通过字符缓冲流在控制台输入字符串并发送数据

```java - send
public class send1 {
    public static void main(String[] args) throws IOException {
        Socket s = new Socket("127.0.0.1",10076);
        //数据来源于键盘录入，直到输入的数据是886，发送结束
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        //封装输出流对象
        BufferedWriter bw = new BufferedWriter(new OutputStreamWriter(s.getOutputStream()));
        String line;
        while ((line=br.readLine())!=null){
            if(line.equals("886")){
                break;
            }
            bw.write(line);
            bw.newLine();
            bw.flush();
        }
        s.close();
    }
}
```

```java - get
public class get1 {
    public static void main(String[] args) throws IOException {
        ServerSocket ss = new ServerSocket(10067);
        Socket s = ss.accept();
        BufferedReader br = new BufferedReader(new InputStreamReader(s.getInputStream()));
        String line;
        while ((line=br.readLine())!=null){
            System.out.println(line);
        }
        ss.close();
    }
}
```

#### Tcp练习1

- 客户端发送数据，接收服务器反馈
- 服务器接收数据，给出反馈

实现：

```java

public class get2 {
    public static void main(String[] args) throws IOException {
        ServerSocket ss = new ServerSocket(10085);
        Socket s = ss.accept();
        BufferedReader br = new BufferedReader(new InputStreamReader(s.getInputStream()));
        BufferedWriter bw = new BufferedWriter(new OutputStreamWriter(s.getOutputStream()));
        String line = br.readLine();
        System.out.println(line);
        s.getOutputStream().write("over".getBytes());
        s.close();

    }
}

public class send2 {
    public static void main(String[] args) throws IOException {
        Socket s = new Socket("192.168.9.101",10085);
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        BufferedWriter bw = new BufferedWriter(new OutputStreamWriter(s.getOutputStream()));
        String line;
        while ((line = br.readLine())!=null){
            bw.write(line);
            bw.newLine();
            bw.flush();

            BufferedReader br2 = new BufferedReader(new InputStreamReader(s.getInputStream()));
            System.out.println(br2.readLine());
        }
    }
}
```

##### 遗留问题

1. 如何实现持续通信，只是单纯的在服务端的输出语句上加While(true)无法实现
2. 通过封装服务端的socket.getOutPutStream()字节流为字符流对象BufferedWriter无法进行写操作

#### Tcp练习2

- 客户端：数据来自于键盘录入直到输入的数据是886，发送数据结束
- 服务器：接收的数据在控制台输出

实现：

```java
public class send1 {
    public static void main(String[] args) throws IOException {
        Socket s = new Socket("127.0.0.1",10076);
        //数据来源于键盘录入，直到输入的数据是886，发送结束
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        //封装输出流对象
        BufferedWriter bw = new BufferedWriter(new OutputStreamWriter(s.getOutputStream()));
        String line;
        while ((line=br.readLine())!=null){
            if(line.equals("886")){
                break;
            }
            bw.write(line);
            bw.newLine();
            bw.flush();
        }
        s.close();
    }
}

public class get1 {
    public static void main(String[] args) throws IOException {
        ServerSocket ss = new ServerSocket(10076);
        Socket s = ss.accept();
        BufferedReader br = new BufferedReader(new InputStreamReader(s.getInputStream()));
        String line;
        while ((line=br.readLine())!=null){
            System.out.println(line);
        }
        ss.close();
    }
}
```

#### Tcp练习3

- 客户端：数据来自于键盘录入，直到输入的数据是886，发送数据结束
- 服务器：接收到的数据写入文本文件

实现：

```java
public class send1 {
    public static void main(String[] args) throws IOException {
        Socket s = new Socket("127.0.0.1",10076);
        //数据来源于键盘录入，直到输入的数据是886，发送结束
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        //封装输出流对象
        BufferedWriter bw = new BufferedWriter(new OutputStreamWriter(s.getOutputStream()));
        String line;
        while ((line=br.readLine())!=null){
            if(line.equals("886")){
                break;
            }
            bw.write(line);
            bw.newLine();
            bw.flush();
        }
        s.close();
    }
}


```

读数据的方式是阻塞式的，如果没有读到就会一直等待

解决方法：

在客户端发送结束数据后自定义结束标示，或使用socket对象的shutdownOutput()方法

#### 服务器为每一个客户端开启一个线程

客户端：

```java
public class send2 {
    public static void main(String[] args) throws IOException {

        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        String line;
        while ((line = br.readLine())!=null){
            Socket s = new Socket("192.168.9.101",10085);
            BufferedWriter bw = new BufferedWriter(new OutputStreamWriter(s.getOutputStream()));
            bw.write(line);
            bw.newLine();
            bw.flush();
            s.shutdownOutput();
            BufferedReader br2 = new BufferedReader(new InputStreamReader(s.getInputStream()));
            System.out.println(br2.readLine());
        }
    }
}
```

服务端：为每一个连接创建一个线程

```java
package tcp;

import java.io.IOException;
import java.net.ServerSocket;
import java.net.Socket;

public class getBythread {
    public static void main(String[] args) throws IOException {
        ServerSocket ss = new ServerSocket(10085);
        while (true){
            Socket s = ss.accept();
            new Thread(new ServerThread(s)).start();
        }
    }
}
```



服务端线程：

```java
package tcp;

import java.io.*;
import java.net.Socket;

public class ServerThread implements Runnable {
    private Socket s;
    public ServerThread(Socket s) {
        this.s = s;
    }

    @Override
    public void run() {
        try {
            BufferedReader br = new BufferedReader(new InputStreamReader(s.getInputStream()));
            //BufferedWriter bw = new BufferedWriter(new FileWriter("tcp\\Copy.text"));

            String line;
            while ((line = br.readLine())!=null){
                System.out.println(line);
            }
            BufferedWriter bwServer = new BufferedWriter(new OutputStreamWriter(s.getOutputStream()));
            bwServer.write("数据传输成功");
            bwServer.newLine();
            bwServer.flush();
            s.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```



## 反射

### 获取Class类型的对象

~~~java
Class<Student> c1 = Student.class;//类的Class属性
System.out.println(c1);//输出c1的全路径，从包名开始
Student s = new Student();//对象的getClass()方法
System.out.println(s.getClass());//效果同上
Class<?> c2 = Class.forName("club.codeqi.Student");
System.out.println(c2);//输出c1的全路径，从包名开始
~~~

### 获取构造方法并构造对象

```java
Class c1 = Class.forName("club.codeqi.student");
Constructor con = c1.getConstructor(String.class,Integer.class);//根据传入String.class,Integer.class获取对应的需要(String,Integer)的构造器
Constructor[] con1 = c1.getConstructors();//获取该类的全部构造方法
Object lisi = con.newInstance("lisi", 18);//构造相应的对象
```

### 获取私有构造方法并构造对象

```java
Class c = Class.forName("student");
Constructor con = c.getDeclaredConstructor();
con.setAccessible(true);//取消访问检查
Object objects = con.newInstance();
System.out.println(objects);
```

### getConstructor与getDeclaredConstructor的区别

getConstructor与getConstructors只能获取public的构造方法

getDeclaredConstructor与getDeclaredConstructors可以获取全部构造方法

但仍不能创建private对象

可以通过setAccessible(True)取消访问检查



### 获取成员变量

class的方法

getFields():获取全部public的成员变量，返回Field对象数组

getDeclareFields():获取全部成员变量，返回Field对象数组

getFields(String name):根据传入名字获取该名字的成员变量，返回Field对象

Field对象的方法

set(obj,value)，根据获取指定名称的成员变量对象，设置某类对象的该属性值

### 获取成员方法

getMethods()//全部公共方法，包括父类的方法

getDeclaredMethods()//获取全部私有方法

getMethod(String name,Class<?>... parameterTypes)返回一个方法对象，该对象反应由该Class对象表示的类或接口的指定公共成员方法

method.invoke(Object,args);//method为通过反射获取的方法名，Object为该方法在哪个类中，args为调用该方法需要用到的参数。

 ```java
Class c = Class.forName("student");
Method m = c.getMethod("toString");
Constructor con = c.getConstructor();
Object o = con.newInstance();
String s = (String) m.invoke(o);
System.out.println(s);
 ```

### 反射获取方法，默认跳过泛型检测



## 文件操作

### File文件对象的创造

三种常用的构造方法

1. File(File parent,String child)//parent可以是文件夹路径，child可以是文件名
2. File(String pathname)//通过指定的路径名字符串转换为抽象路径
3. File(String parent,String child)//根据父路径字符串和子路径字符串创建File实例

 ### File文件在磁盘的创建

File对象的三种方法，返回值均为boolean

1. createNewFile()，该文件名不存在时，创建一个该路径名的新文件
2. mkdir()，创建抽象路径名的目录
3. mkdirs()，自动创建不存在的父目录

例子：

```java
//创建文件
File f1 = new File("E:\\writefile\\test01.text");
System.out.println(f1.createNewFile());
//创建文件夹
File f2 = new File("E:\\writefile\\test");
System.out.println(f2.mkdir());
//创建多级文件夹
File f3 = new File("E:\\writefile\\test01\\void");
System.out.println(f3.mkdirs());
```

### 复制文件夹（包含文件夹中的内容，单个文件夹）

步骤：

1. 创建文件夹
2. 获取数据源目录下的所以文件数组
3. 遍历数组获取每个File对象
4. 通过字节流复制文件

实现：

```java
File srcFile = new File("E:\\writefile");//源文件
String srcflodname = srcFile.getName();
File destFloder = new File("myCharStream",srcflodname);//目标文件夹
if(!destFloder.exists()){
    destFloder.mkdir();
}
File[] files = srcFile.listFiles();//源文件夹中的文件列表
for(File file:files){
    String name = file.getName();
    File destfile = new File(destFloder,name);
    BufferedInputStream bis = new BufferedInputStream(new FileInputStream(file));//读文件
    BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream(destfile));//写文件
    byte[] bys = new byte[1024];
    int len;
    while ((len=bis.read(bys))!=-1){
        bos.write(bys,0,len);
    }
    bos.close();
    bis.close();
}
```

### 复制文件夹(迭代)

```java
public void copyflods() throws IOException {
    File srcFile = new File("E:\\writefile");
    File destfile = new File("F:\\");
    copyfloder(srcFile,destfile);

}

private void copyfloder(File srcFile, File destfile) throws IOException {//递归方法，判断源文件是否为文件夹
    if(srcFile.isDirectory()){
        String srcfilename = srcFile.getName();
        File newFloder = new File(destfile,srcfilename);
        if(!newFloder.exists()){
            newFloder.mkdir();
        }
        File[] files = srcFile.listFiles();
        for(File file:files){
            copyfloder(file,newFloder);
        }
    }
    else{
        File newFile = new File(destfile,srcFile.getName());
        copyfile(srcFile, newFile);
    }
}

public void copyfile(File srcfile,File destfile ) throws IOException {//根据传入的源文件和目标文件进行复制文件
    BufferedInputStream bis = new BufferedInputStream(new FileInputStream(srcfile));//读文件
    BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream(destfile));//写文件
    byte[] bys = new byte[1024];
    int len;
    while ((len=bis.read(bys))!=-1){
        bos.write(bys,0,len);
    }
    bos.close();
    bis.close();
}
```

### 删除文件

File文件的方法：（可以删除文件或文件夹，如果文件夹中有内容就无法删除该文件夹，需要先删除该文件夹下的文件）

public boolean delete();//删除由此抽象路径所指向的文件

### 遍历目录

1. 获取给定的File目录下的所有的文件或者目录的File数组
2. 遍历该File数组，得到每一个File对象
3. 判断该File对象是否是目录
   - 是，递归调用
   - 否，获取绝对路径输出到控制台

```java
public void allfile(){
    File srcFile = new File("E:\\writefile");
    getallfilepath(srcFile);
}
public void getallfilepath(File srcfile){
    File[] files = srcfile.listFiles();
    if(files!=null){
        for(File file:files){//遍历File数组，得到每一个File对象
            if(file.isDirectory()){
                getallfilepath(file);
            }else{
                System.out.println(file.getAbsolutePath());
            }
        }
    }
}
```



### File类的判断和获取功能

1. boolean isDirectory()：file对象是否为目录
2. boolean isFile()：file对象是否为文件
3. boolean exists()：file对象是否为存在
4. String getAbsolutePath()：file对象的绝对路径
5. String getPath()：将此抽象路径名转换为路径名字符串
6. String getName()：文件或文佳佳的名称
7. String[] list()
8. File[] listFiles()

## 计算程序运行时间

``` java
long startTime = System.currentTimeMillis();
long endTime = System.currentTimeMillis();
System.out.println(endTime-startTime)
```

## 多线程

### 设置获取线程名称

获取：

```java
Thread.currentThread().getName();//获取当前正在执行线程的线程名称
MyThread my = new MyThread();//MyThread为继承Thread类的自定义类
my.getName();
```

设置：

```java
MyThread my = new MyThread();//MyThread为继承Thread类的自定义类
my.setName("飞机");
MyThread my = new MyThread("高铁");//MyThread为继承Thread类的自定义类，设置有参构造方法，在构造方法中设置super(name)
```

### 设置获取线程优先级

```java
Thread t = new Thread(()->System.out.println("线程启动了"));
t.getPriority();//获取线程优先级，默认是5
t.setPriority(10);//设置线程优先级，最小是1，最大是10
```

### 线程控制（sleep、join、setDaemon）

```java
//sleep方法在线程的run方法中使用
Thread.sleep(1000);//这里传入的是毫秒，也就是说线程停止1s

//join方法在线程start()后调用
Thread t1 = new Thread(()->System.out.println("线程启动了"));
Thread t2 = new Thread(()->System.out.println("线程启动了"));
Thread t3 = new Thread(()->System.out.println("线程启动了"));
t1.start();
try{
    t1.join();
}
catch(Exception e){ 
}
t2.start();
t3.start();
//join函数就是将程序剩余内容先不执行，将该线程中的内容加到正在运行的线程中，等到t1线程结束之后才会开始t2、t3线程

//setDaemon(true)将线程设置为守护线程，当运行的线程都是守护线程后，java虚拟机将退出
Thread t4 = new Thread(()->System.out.println("线程启动了"));
Thread t5 = new Thread(()->System.out.println("线程启动了"));

t4.setDaemon(true);
t5.setDaemon(true);
t4.start();
t5.start();
for(int i=0;i<10;i++){System.out.println(i)}
//当这个for循环执行结束后t4,t5会稍微挣扎一下就停止，因为主线程已经执行结束，只剩下守护线程了
```

### 卖票

三个窗口同时卖100张票

```java - mythread
public class mythread implements Runnable {
    private volatile int pickets = 100;
    @Override
    public void run() {
        while(true){
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            if(pickets>0){
                System.out.println(Thread.currentThread().getName()+"正在卖第"+pickets+"张票");
                pickets --;
            }
        }
    }
}

```

```java - mythreaddemo
public class mythreaddemo {
    public static void main(String[] args) {
        mythread r = new mythread();

        Thread t1 = new Thread(r);
        Thread t2 = new Thread(r);
        Thread t3 = new Thread(r);

        t1.start();
        t2.start();
        t3.start();


    }
}
```

### 线程同步

需要进行线程同步有以下三种条件

1. 是否是多线程环境
2. 是否有共享数据
3. 是否有多条语句操作共享数据

解决方法：让程序没有安全问题的环境

如何实现：

把多条语句操作共享数据的代码给锁起来，让任意时刻只能由一个线程执行即可

```java
public class mythread implements Runnable {
    private volatile int pickets = 100;
    private Object o = new Object();
    @Override
    public void run() {
        while(true){
            synchronized (o){
                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                if(pickets>0){
                    System.out.println(Thread.currentThread().getName()+"正在卖第"+pickets+"张票");
                    pickets --;
                }
            }
        }
    }
}
synchronized(o),也可以改写为synchronized(this),或synchronized(类名.class)
或者定义静态方法，加上synchronized关键字
```

### Lock锁

lock是个接口

方法：void lock():获得锁

​			void unlock():释放锁

通常使用lock的实现类ReentrantLock来实例化

```java
public class mythread implements Runnable {
    private int pickets = 100;
        private Lock lock = new ReentrantLock();
        @Override
        public void run() {
            while(true){
                lock.lock();
                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                if(pickets>0){
                    System.out.println(Thread.currentThread().getName()+"正在卖第"+pickets+"张票");
                    pickets --;
                }
                lock.unlock();
            }
        }
}
```

### 生产者消费者案例

- 奶箱类(Box): 定义一个成员变量，表示第x瓶奶，提供存储牛奶和获取牛奶的操作
- 生产者类(Producer):实现Runnable接口，重写run()方法，调用存储牛奶的操作
- 消费者类(Customer):实现Runnable接口，重写run()方法，调用获取牛奶的操作
- 测试类(BoxDemo):里面有main方法，main方法中的步骤如下：
  - 创建奶箱对象
  - 创建生产者对象，并将奶箱对象作为构造方法参数传递
  - 创建消费者对象，把奶箱对象作为构造方法参数传递
  - 创建两个线程，分别把生产者对象和消费者对象作为构造方法参数传递
  - 启动线程

Box.java

因为使用了wait和notifyAll方法，所以需要在box的put和get方法上添加synchronized关键字

原理：起初箱子是没有牛奶的，调用put方法将牛奶放入，并唤醒其他等待的线程，如果箱子有牛奶，则放入牛奶的进程就进行等待

起初，箱子里没有牛奶，取牛奶的操作就进行等待，待放入牛奶后唤醒全部线程后，进行取牛奶操作，结束取牛奶后再唤醒全部线程，进行放入牛奶操作。

```java
package milk;

public class Box {
    private int milk;
    private boolean state = false;

    public synchronized void put(int milk){
        if (state){//如果有牛奶
            try {
                wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        //如果没有牛奶，就生产牛奶
        this.milk = milk;
        System.out.println("送奶工将第"+this.milk+"瓶奶放入奶箱");
        //生产结束，修改奶箱状态
        this.state = true;
        notifyAll();
    }

    public synchronized void get(){

        if(!state){//如果没有牛奶
            try {
                wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        System.out.println("用户拿到第"+this.milk+"瓶奶");
        this.state = false;
        notifyAll();
    }
}
```

BoxDemo.java

```java
package milk;

public class BoxDemo {
    public static void main(String[] args) {
        Box b = new Box();
        Producer p = new Producer(b);
        Customer c = new Customer(b);
        Thread t1 = new Thread(p);
        Thread t2 = new Thread(c);
        t1.start();
        t2.start();
    }
}
```



Producer.java

```java
package milk;

public class Producer implements Runnable {
    private Box b;
    public Producer(Box b) {
        this.b = b;
    }

    @Override
    public void run() {
        for(int i=1;i<=30;i++){
            b.put(i);
        }
    }
}

```



Customer.java

```java
package milk;

public class Customer implements Runnable {
    private Box b;
    public Customer(Box b) {
        this.b = b;
    }

    @Override
    public void run() {
        while (true){
            b.get();
        }
    }
}

```



## 常见函数式接口

### Supplier<T>

包含一个无参方法get(),获取一个指定类型的数据

```java
Supplier<String> sup = ()->"小明";
String s = sup.get();
System.out.println(s);
```

### Consumer接口，也称消费型接口

其消费类型由泛型指定，方法名为accept(T t)，包含两个方法

accept:接口中待实现的方法

andThen：连续执行两个方法

例如下面程序，每传入一个字符串，在执行结束第一个Consumer函数后就紧接着执行第二个Consumer函数，达到格式化输出的效果

```java
public void testcusmor(){
    String[] strArrays = {"小明,16","小红,18","小李,17"};
    printinfo(strArrays,(String str)->{
        String name = str.split(",")[0];
        System.out.print("姓名："+name);
    },(String str)->{
        int age = Integer.parseInt(str.split(",")[0]);
        System.out.println(",年龄："+age);
    });
}

private void printinfo(String[] strArrays, Consumer<String> o, Consumer<String> o1) {
    for(String str:strArrays){
        o.andThen(o1).accept(str);
    }
}
```

### Predicate接口，进行判断，返回boolean

 

| 方法名                         | 意义             |
| ------------------------------ | ---------------- |
| test                           | 接口需实现的方法 |
| Predicate negate()             | 逻辑非           |
| Predicate and(Predicate other) | 逻辑与           |
| Predicate or(Predicate other)  | 逻辑或           |

```java
public void t1(){
    boolean h = testboo("helloworld", (s) -> s.length() > 8);
    System.out.println(h);
}
private boolean testboo(String s, Predicate<String> pre){
    return pre.test(s);//返回正常值
    return pre.negate().test(s);//返回正常值的逻辑非
}

public void test2(){
    boolean h = testboo1("helloworld", (s) -> s.length() > 8,(s)->s.length()<15);
    System.out.println(h);
}
private static boolean testboo1(String s, Predicate<String> pre,Predicate<String> pre2){
    boolean b1 = pre.test(s);
    boolean b2 = pre2.test(s);
    return b1&&b2;
    //等价于
    return pre.and(pre2).test(s);
}
```

#### Predicate接口的应用-Stream流

Stream流的filter方法需要传入一个Predicate的实现方法

例子:

需求：

将下列名字中以张开头，且长度为3的名字遍历输出

以前的方式

```java
public static void main(String[] args) {
    List<String> list = new ArrayList();
    list.add("张三丰");
    list.add("林黛玉");
    list.add("张无忌");
    list.add("柳岩");
    list.add("张敏");
    list.add("王祖贤");
    ArrayList<String> zlist = new ArrayList();//获取“张”开头的姓名
    for(String s : list){
        if (s.startsWith("张")){
            zlist.add(s);
        }
    }

    ArrayList<String> threelist = new ArrayList();//获取长度等于三的姓名
    for(String s : zlist){
        if(s.length()==3){
            threelist.add(s);
        }
    }

    for(String s : threelist){//遍历输出
        System.out.println(s);
    }
}
```

使用Stream流的方式

```java
public static void main(String[] args) {
    List<String> list = new ArrayList();
    list.add("张三丰");
    list.add("林黛玉");
    list.add("张无忌");
    list.add("柳岩");
    list.add("张敏");
    list.add("王祖贤");
    list.stream().filter(s->s.startsWith("张")).filter(s->s.length()==3).forEach(s -> System.out.println(s));
}
```

#### Stream流的函数

| 方法                                           | 意义                                                         |
| ---------------------------------------------- | ------------------------------------------------------------ |
| filter(Predicate)                              | 对流中的数据进行过滤                                         |
| limit(long maxSize)                            | 截取前n个数据                                                |
| skip(long n)                                   | 跳过n个数据                                                  |
| **static** Stream<T> concat(Stream a,Stream b) | 合并a和b两个流为一个流                                       |
| distinct()                                     | 返回由该流的不同元素（根据Object.equals(Object)）组成的流，即去重 |
| sorted()                                       | 自然排序                                                     |
| sorted(Comparator comparator)                  | 根据提供的Comparator进行排序                                 |
| map(T)                                         | 对数据进行操作                                               |
| forEach()                                      | 遍历                                                         |

这些操作之后的结果是Stream流，如何将Stream转为List,set,map集合中

List<String> names = listStream.collect(Collectors.toList());

Set<String> names = setStream.collect(Collectors.toSet());

String[] strArray = {"林青霞,30","张曼玉,35","王祖贤,33","柳岩,25"}

Stream<String> arrayStream = Stream.of(strArray).filter(s->Integer.parseInt(s.split(",")[1])>28);

Map<String,Integer> map = arrayStream.collect(Collectors.toMap(s->s.split(",")[0],s->Integer.parseInt(s.split(",")[1])));



### Function<T,R>

T是输入的类型，R是返回的类型

| 方法名                     | 描述                                                         |
| -------------------------- | ------------------------------------------------------------ |
| apply(T t)                 | 将此函数应用于给定的参数                                     |
| andThen(Function<,> after) | 返回一个组合函数，首先将该函数应用于其输入，然后将after函数应用于结果 |

```java
public static void main(String[] args) {
    convert("100",s -> Integer.parseInt(s));
    convert1(666,i->String.valueOf(i));
    convert2("100",s -> Integer.parseInt(s),i->String.valueOf(i+566));
}
//定义一个方法，将字符串转为int类型，在控制台输出
private static void convert(String s, Function<String,Integer> fun){
    int i = fun.apply(s);
    System.out.println(i);
}
//定义一个方法，把一个int类型的数据加上一个整数之后，装为字符串在控制台输出
private static void convert1(int i,Function<Integer,String> fun){
    String s = fun.apply(i);
    System.out.println(s+s.getClass().getName());
}
//定义一个方法，把一个字符串转换为int类型，把int类型的数据加上一个整数之后，转为字符串在控制台输出
private static void convert2(String s,Function<String,Integer> fun1,Function<Integer,String> fun2){
    Integer i = fun1.apply(s);
    String ss = fun2.apply(i);
    //等价于
    String ss1 = fun1.andThen(fun2).apply(s);
    System.out.println(ss);
}
```



## Date类

```java
System.out.println(new Date());//获取当前时间
long da = 15082596;
System.out.println(new Date(da));//根据传入的时间戳构造Date对象
```

### 获取时间戳

public long getTime()获取的日期对象是从1970年1月1日00:00:00到现在的毫秒数

public void setTime(long time) 设置时间，给的是毫秒值

### SimpleDateFormat类概述

日期格式化和解析

public SimpleDateFormat()：构造一个SimpleDateFormat，使用默认模式和日期格式

public SimpleDateFormat(String pattern)：构造一个SimpleDateFormat使用给定的模式和默认的日期格式

#### Date到String

public final String format(Date date):将日期格式化成日期/时间字符串

#### String到Date

public Date parse（String source）：从给定字符串的开始解析文本以生成日期

```java
 Date d = new Date();
SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
String s1 = sdf.format(d);
System.out.println(s1);

String s2 = "2048-11-10 22:00:00";
Date date = sdf.parse(s2);
System.out.println(date);
```

### Calendar类

claendar是抽象类，其直接子类是GregorianCalendar



Calendar提供了一个类方法getInstance用于获取Calendar对象，其日历字段已使用当前日期和时间初始化：Calendar now = Calendar.getInstance();//多态的形式

```java
Calendar c = Calendar.getInstance();
int year = c.get(Calendar.YEAR);
int month = c.get(Calendar.MONTH)+1;//月份是从0开始计算的
int date = c.get(Calendar.DATE);
System.out.println(year+"year"+month+"month"+date+"date");
```

 add与set方法

| 方法名                           | 意义                                                   |
| -------------------------------- | ------------------------------------------------------ |
| add(int field,int amount)        | 根据日历的规则，将指定的时间量添加或减去指定的日历字段 |
| set(int year,int month,int date) | 设置当前日历的年月日                                   |

```java
Calendar c = Calendar.getInstance();
c.add(Calendar.YEAR,-3);//三年前的今天
c.add(Calendar.YEAR,10);
c.add(Calendar.DATE,-5);//十年后的天前

c.set(2048,11,11);//设置日期为2048年11月11日
System.out.println(c);
```

#### 获取输入年份的二月有几天

思路：

1. 根据用户输入的年份，构造该年的三月1号的Calendar对象
2. Calendar对象日期减1就是该年的最后一天日期
3. 获取Calendar对象的Date值，就能知道该年二月有多少天

实现：

```java
public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        String s = sc.nextLine();
        Calendar c = Calendar.getInstance();
        c.set(Integer.parseInt(s),2,1);
        c.add(Calendar.DATE,-1);
        int i = c.get(Calendar.DATE);
        System.out.println(s+"年二月有"+i+"天");
    }
```



## TreeSet

TreeSet():根据其元素的自然排序进行排序

TreeSet(Comparator comparator):根据指定的比较器进行排序

```java
public void treesettest(){
    TreeSet<Integer> treeSet = new TreeSet<>();
    treeSet.add(1);
    treeSet.add(4);
    treeSet.add(3);
    treeSet.add(7);
    treeSet.add(2);
    for(Integer i : treeSet){
        System.out.println(i);
    }

}
```

如果调用了Treeset的无参构造器，且TreeSet中放入的是用户自定义的类，该类须实现自然排序Comparable接口

```java
public class student implements Comparable<student> {
     @Override
    public int compareTo(student o) {
        int num = o.age-this.age;
        int num2 = num==0?o.name.compareTo(this.name):num;
        int num3 = num2==0?o.favorite.compareTo(this.favorite):num2;
        return num3;
    }
}
```

如果调用Treeset的有参构造器，需传入Comparator对象

```java
TreeSet<student> t = new TreeSet<student>((s1,s2)->{
    int num = s1.getAge() - s2.getAge();
    int num2 = num==0?s1.getName().compareTo(s2.getName()):num;
    return num2;
});
student s1 = new student("li1",20,"游泳");
student s2 = new student("zhao2",18,"唱歌");
student s3 = new student("wang3",19,"跳舞");
t.add(s1);
t.add(s2);
t.add(s3);
System.out.println(t);
```

### 统计输入字符串中个字符的次数

```java
Scanner sc  = new Scanner(System.in);
String s = sc.nextLine();
TreeMap<Character,Integer> map = new TreeMap<Character, Integer>();
for(int i=0;i<s.length();i++){
    char c = s.charAt(i);
    Integer integer = map.get(c);
    if(integer == null){
        map.put(c,1);
    }
    else{
        //int value = integer++;
        map.put(c, ++integer);
    }
}
Set<Character> set = map.keySet();
for(Character ch:set){
    System.out.print(ch+":  ");
    System.out.println(map.get(ch));

}
```

### Set实现获取10个不重复的随机数

其中随机数大小为[1,20]，使用Random对象的nextInt(20)+1方法，获取0-20之间的随机数再加1

```java
public static void main(String[] args) {
        Set<Integer> set = new HashSet();
        Random r = new Random();
        while (set.size()<10){
            int i = r.nextInt(20) + 1;
            set.add(i);
        }
        System.out.println(set);
    }
```



## 常用Api

### Math

基本指数，对数，平方等

常用方法：

| 方法名                                      | 说明                                           |
| ------------------------------------------- | ---------------------------------------------- |
| public static int abs(int a)                | 返回参数的绝对值                               |
| public static double ceil(double a)         | 返回大于或等于参数的最小double值，等于一个整数 |
| public static double floor(double a)        | 返回小于或等于参数的最小double值，等于一个整数 |
| public static int round(float a)            | 按照四舍五入返回最接近参数的int                |
| public static int max(int a,int b)          | 返回两个int值中的较大值                        |
| public static int min(int a,int b)          | 返回两个int值中的较小值                        |
| public static double pow(double a,double b) | 返回a的b次幂的值                               |
| public static double random()               | 返回值为double的正值,取值范围：[0.0,1.0)       |

## Lambda语法

### 前提条件

接口中只含有一个未实现的方法

### 实现

#### 创建一个新的方法

方法中传入参数和接口名

```java
public void predicate(){
	boolean b1 =x("hello",(s)->s.length()>8);
	System.out.println(b1);
}
public boolean x(String s,Predicate<String> pre){
    return pre.test(s);
}
```

#### 创建该接口的对象

```java
Predicate<String> s2 = s->s.length()>8;
boolean b2 = s2.test("hello");
System.out.println(b2);
```

## 模块化开发

1. 在idea的左上角点击File-Prject structure
2. 选择Modules创建新的模块
3. 在src目录下创建一个名为module-info.java文件

导出语法：

```java
module 模块名(当前模块名){
    exports 包名;
}
```

导入语法：

```java
module 模块名(当前模块名){
    requires 模块名(待导入的模块名);
}
```

## 方法引用

Lambda表达式被类的实例方法替代的时候

第一个参数作为调用者

后面的参数全部传递给该方法作为参数

语法：A::B

例子1：在lambda语法中

​		(s)->System.out.print(s)

等价于：(s)->System.out::print

例子2：将String所有字符串转为大写

​		String s = "HelloWorld";s.toUpperCase();

等价于 "HelloWorld"::toUpperCase

例子3：截取字符串

​		String s = "HelloWorld";s.substring(2,10);

等价于 "HelloWorld"::substring(2,10)



test.java(测试类)

```java
public class test {
    public static void main(String[] args) {
        sub su = (s,x,y)->{return s.substring(x,y);};
        String h = su.mysub("helloworld", 2, 10);
        System.out.println(h);

        //引用方法
        sub sa = String::substring;
        String h1 = su.mysub("helloworld", 2, 10);
        System.out.println(h1);

        
    }

}
```

sub.java（接口类）

```java
package threadtest;

public interface sub {
    public String mysub(String s,int x,int y);
}

```



引用构造器

格式：类名::new

范例：Student::new

