---
title: 「Java教程」IO机制
date: 2017-05-05 17:31:22
tags: [Java, 后端开发]
categories: Java教程
---


输入输出（I/O）是指程序与外部设备或其他计算机进行交互的操作。几乎所有的程序都具有输入与输出操作，Java把这些输入与输出操作用流来实现，通过统一的接口来表示，从而使程序设计更为简单。
<!-- more -->


### 1. File类
- File与真实硬盘中的文件或文件夹  不是一个东西
    * File是在内存中的一个对象<---映射--->硬盘上的文件或文件夹
- java.io.File类用于文件或目录信息(名称、大小等)的抽象表示方式，不能对文件内容进行访问。
- File类中的常用的方法
    + canRead()，canWrite()，isHidden()，isFile()，isDirectory()
    + length()，获取文件中字节的个数
    + lastModified()，获取文件最后的修改时间--->毫秒值
    + *String path = getAbsolutePath()，获取文件的绝对路径   D://test//Test.txt
        * 绝对路径<---->相对路径
        * 绝对路径可以通过完整的字符串，定位盘符，文件夹，文件
        * 相对路径没有盘符的写法，当前工程(项目)所在的位置找寻
    + String name = getName()，获取文件的名字    Test.txt
    + *boolean = **createNewFile()**，创建新的文件
    + *boolean = **mkdir** ，创建新的文件夹  外层没有 不能创建
    + *boolean = **mkdirs**，创建新的文件夹  外层没有 可以自动创建
    + String pname = getParent()，获取当前file的父亲file名字
    + *File file = getParentFile()，获取当前file的父亲file对象
    + String[] names = list()，获取当前file的所有儿子名字
    + *File[] files = listFiles()，获取当前file的所有儿子对象
    + *boolean = delete()，删除文件或空的文件夹  不能删除带元素的文件夹
- 文件夹的路径(找父目录)

``` java
//查找当前file的所有父目录
File file = new File("D:\\test\\bbb\\inner\\InnerTest.txt");
File pfile = file.getParentFile();
while(pfile!=null){
    System.out.println(pfile.getAbsolutePath());
    pfile = pfile.getParentFile();//再找一遍
}
```

- 文件夹的遍历----需要一个递归

``` java
//设计一个方法  用来展示(遍历)文件夹,参数-->file(代表文件或文件夹)
public void showFile(File file){
    //获取file的子元素
    //files==null是个文件
    //files!=null是个文件夹
    //files.length!=0是一个带元素的文件夹
    File[] files = file.listFiles();//test文件夹所有子元素
    if(files!=null && files.length!=0){
        for(File f:files){
            this.showFile(f);
        }
    }
    //做自己的显示(file是文件或file是一个空的文件夹)
    System.out.println(file.getAbsolutePath());
}
```

- 文件夹的删除----需要一个递归

``` java
//设计一个方法 删除文件夹,参数 file
public void deleteFile(File file){
    //判断file不是空文件夹
    File[] files = file.listFiles();
    if(files!=null && files.length!=0){
        for(File f:files){
            this.deleteFile(f);
        }
    }
    //删除file (file是个文件或file是一个空文件夹)
    file.delete();
}
```


### 2. IO流
- 流的本质是数据传输，根据数据传输特性将流抽象为各种类，方便更直观的进行数据操作。 
- 流的分类:
    * 根据处理数据类型的不同分为：字符流和字节流
    * 根据数据流向不同分为：输入流in(读取)和输出流out(写入)
    * 操作的目标来区分:
        - 文件流，数组流，字符串流，数据流，对象流，网络流...
- IO流的框架结构

```
|——IO流
    |————字节流
        |————InputStream
            |————FileInputStream
            |————DataInputStream
            |————ObjectInputStream
        |————OutputStream
            |————FileOutputStream
            |————DataOutputStream
            |————ObjectOutputStream
            |————PrintStream
    |————字符流
        |————Reader
            |————BufferedReader
            |————InputStreamReader
        |————Writer
            |————BufferedWriter
            |————OutputStreamWriter
```


### 3. 文件流
读取文件中的信息in，将信息写入文件中out；文件流按照读取或写入的单位(字节数)大小来区分
- 字节型文件流(1字节)：FileInputStream/FileOutputStream
- 字符型文件流(2字节--1字符)：FileReader/FileWriter
- 字节流和字符流的区别：
    * 读写单位不同：字节流以字节（8bit）为单位，字符流以字符为单位，根据码表映射字符，一次可能读多个字节。
    * 处理对象不同：字节流能处理所有类型的数据（如图片、avi等），而字符流只能处理字符类型的数据。
- 结论：只要是处理**纯文本**数据，就优先考虑使用**字符流**。 除此之外都使用**字节流**。
- 输入流和输出流
    * 对输入流只能进行**读**操作，对输出流只能进行**写**操作。 



### 4. 字节型文件流
#### 4.1 字节型文件输入流FileInputStream(读)
- FileInputStream类在java.io包，继承自InputStream类(字节型输入流的父类)。
- 创建对象
    * 调用一个带File类型的构造方法
    * 调用一个带String类型的构造方法
- 常用方法
    + int code = read();    每次从流管道中读取一个字节，返回字节的code码
    + *int count = read(byte[] )  每次从流管道中读取若干个字节，存入数组内  返回有效元素个数
    + int count = available();   返回流管道中还有多少缓存的字节数
    + skip(long n);跳过几个字节  读取
        * 多线程--->利用几个线程同时读取文件
    + *close()    将流管道关闭---必须要做,最好放在finally里
        * 注意代码的健壮性，判断严谨（eg:非空判断）
    
#### 4.2 字节型文件输出流FileOutputStream(写)
- FileOutputStream类在java.io包，继承自OutputStream类(所有字节型输出流的父类)。
- 创建对象
    + 调用一个带File参数，还有File boolean重载
    + 调用一个带String参数，还有String boolean重载
    + eg: new FileOutputStream("D://test//bbb.txt", true)//第二个参控制每次写入追加还是重载
- 常用方法
    + write(int code);  将给定code对应的字符写入文件   '='
    + write(byte[]);  将数组中的全部字节写入文件   getByte()
    + write(byte[] b, int off, int len);
    + flush();    将管道内的字节推入(刷新)文件
    + close();    注意在finally中关闭

> - 创建的是文件输入流，若文件路径有问题，则抛出异常  FileNotFoundException
> - 创建的是文件输出流，若文件路径有问题，则直接帮我们创建一个新的文件

* 设计一个文件复制的方法

``` java
public void copyFile(File file, String path) {
    FileInputStream fis = null;
    FileOutputStream fos = null;
    try {
        //创建输入流读取信息
        fis = new FileInputStream(file);
        //创建一个新的File对象
        File newFile = new File(path +"\\"+ file.getName());//"E:\\test\\test.txt"
        //创建一个输出流
        fos = new FileOutputStream(newFile);
        byte[] b = new byte[1024];//通常1kb-8kb之间
        int count = fis.read(b);
        while(count != -1) {
            fos.write(b, 0, count);//将读取到的有效字节写入
            fos.flush();
            count = fis.read(b);
        }
        System.out.println("复制完毕！");
    } catch (FileNotFoundException e) {
        e.printStackTrace();
    } catch (IOException e) {
        e.printStackTrace();
    } finally {
        //关闭
        if(fis!=null) {
            try { fis.close(); }
            catch (IOException e) {e.printStackTrace();}
        }
        if(fos!=null) {
            try {fos.close();}
            catch (IOException e) {e.printStackTrace();}
        }
    }
}
```


### 5. 字符型文件流
FileReader/FileWriter：只能操作纯文本的文件 .txt / .properties

#### 5.1 字符型文件输入流FileReader(读)
- FileReader类在java.io包，继承自InputStreamReader，Reader
- 创建对象
    * 调用一个带File类型的构造方法
    * 调用一个带String类型的构造方法
- 常用方法
    * read()
    * read(char[])
    * close()

``` java
File file = new File("F://test//Test.txt");
try {
    FileReader fr = new FileReader(file);
    // int code = fr.read();
    // System.out.println(code);
    char[] c = new char[1024];
    int count = fr.read(c);
    while(count!=-1) {
        System.out.println(new String(c, 0, count));
        count = fr.read(c);
    }
} catch (FileNotFoundException e) {
    e.printStackTrace();
} catch (IOException e) {
    e.printStackTrace();
}
```

#### 5.2 字符型文件输出流FileWriter(写)
- FileWriter类在java.io包，继承自OutputStreamWriter，Writer
- 构造方法
    * 带file参数，带file,boolean参数
    * 带String参数，带String,boolean参数
- 常用方法
    * write(int)
    * write(char[])
    * write(string)
    * flush()，close()


### 6. *缓冲流
- 缓冲流,也叫高效流，是对4个基本的File...流的增强，所以也是4个流，按照数据类型分类：
    * 字节缓冲流：BufferedInputStream，BufferedOutputStream 
    * 字符缓冲流：BufferedReader，BufferedWriter
- 缓冲流的基本原理，是在创建流对象时，会创建一个内置的默认大小的缓冲区数组，通过缓冲区读写，减少系统IO次数，从而提高读写的效率。
- 缓冲流读写方法与基本的流是一致

#### 6.1 字节缓冲流
- BufferedInputStream，BufferedOutputStream
- 构造方法
    * public BufferedInputStream(InputStream in) ：创建一个 新的缓冲输入流。 
    * public BufferedOutputStream(OutputStream out)： 创建一个新的缓冲输出流。

``` java
// 创建字节缓冲输入流
BufferedInputStream bis = new BufferedInputStream(new FileInputStream("bis.txt"));
// 创建字节缓冲输出流
BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream("bos.txt"));
```


#### 6.2 字符缓冲流
- BufferedReader，BufferedWriter
- 构造方法
    * public BufferedReader(Reader in) ：创建一个 新的缓冲输入流。 
    * public BufferedWriter(Writer out)： 创建一个新的缓冲输出流。

``` java
// 创建字符缓冲输入流
BufferedReader br = new BufferedReader(new FileReader("br.txt"));
// 创建字符缓冲输出流
BufferedWriter bw = new BufferedWriter(new FileWriter("bw.txt"));
```

- 字符缓冲流的基本方法与普通字符流调用方式一致，不再阐述，我们来看它们具备的特有方法。
- 特有方法: 
    * BufferedReader：public String readLine(): 读一行文字。 
    * BufferedWriter：public void newLine(): 写一行行分隔符,由系统属性定义符号。

``` java
//设计一个方法，用来用户登录认证
public String login(String username, String password) {
    try {
        BufferedReader br = new BufferedReader(new FileReader("F://test//User.txt"));
        //User.txt每行存储格式：张三-123
        String user = br.readLine();//user表示一行记录，记录账号密码
        while(user!=null) {
            //将user信息拆分，分别与参数比较
            String[] value = user.split("-");//value[0]账号，value[1]密码
            System.out.println(value[0]);
            if(value[0].equals(username)) {
                if(value[1].equals(password)) {
                    return "登录成功";
                }
            }
            user = br.readLine();
        }
    } catch (IOException e) {
        e.printStackTrace();
    }
    return "账号或密码错误！";
}
```

- readLine方法演示:

```
try {
    BufferedWriter bw = new BufferedWriter(new FileWriter("F://test//User.txt", true));
    bw.newLine();
    bw.write("java-888");
    bw.flush();
} catch (IOException e) {
    e.printStackTrace();
}
```


### 7. 转换流
#### 7.1 字符编码
- 字符编码Character Encoding : 就是一套自然语言的字符与二进制数之间的对应规则。
- 字符集 Charset：也叫编码表。是一个系统支持的所有字符的集合，包括各国家文字、标点符号、图形符号、数字等。
- 常见字符集:
    * ASCII字符集 ：
        + ASCII（American Standard Code for Information Interchange，美国信息交换标准代码）
    * ISO-8859-1字符集：
        + 拉丁码表，别名Latin-1，用于显示欧洲使用的语言；ISO-8859-1使用单字节编码，兼容ASCII编码。
    * GBxxx字符集：
        + GB就是国标的意思，是为了显示中文而设计的一套字符集。
        + GB2312（简体中文码表），GBK（最常用的中文码表），GB18030（最新的中文码表）
    * Unicode字符集 ：
        + Unicode编码系统为表达任意语言的任意字符而设计，是业界的一种标准，也称为统一码、标准万国码。
        + UTF-8、UTF-16和UTF-32；最为常用的UTF-8编码。
- 编码引出的问题
    * 在IDEA中，使用FileReader 读取项目中的文本文件。由于IDEA的设置，都是默认的UTF-8编码，所以没有任何问题。但是，当读取Windows系统中创建的文本文件时，由于Windows系统的默认是GBK编码，就会出现乱码。


#### 7.2 InputStreamReader类 
转换流java.io.InputStreamReader，是Reader的子类，是从**字节流到字符流**的桥梁。它读取字节，并使用指定的字符集将其解码为字符。它的字符集可以由名称指定，也可以接受平台的默认字符集。
- 构造方法
    * InputStreamReader(InputStream in): 创建一个使用默认字符集的字符流。 
    * InputStreamReader(InputStream in, String charsetName): 创建一个指定字符集的字符流。

``` java
InputStreamReader isr = new InputStreamReader(new FileInputStream("in.txt"));
InputStreamReader isr2 = new InputStreamReader(new FileInputStream("in.txt") , "GBK");
```

- 指定编码读取:

``` java
public class ReaderDemo2 {
   public static void main(String[] args) throws IOException {
    // 定义文件路径,文件为gbk编码
       String FileName = "E:\\file_gbk.txt";
    // 创建流对象,默认UTF8编码
       InputStreamReader isr = new InputStreamReader(new FileInputStream(FileName));
    // 创建流对象,指定GBK编码
       InputStreamReader isr2 = new InputStreamReader(new FileInputStream(FileName) , "GBK");
// 定义变量,保存字符
       int read;
    // 使用默认编码字符流读取,乱码
       while ((read = isr.read()) != -1) {
           System.out.print((char)read); // ��Һ�
      }
       isr.close();
    // 使用指定编码字符流读取,正常解析
       while ((read = isr2.read()) != -1) {
           System.out.print((char)read);// 大家好
      }
       isr2.close();
  }
}
```


#### 7.3 OutputStreamWriter类 
转换流java.io.OutputStreamWriter ，是Writer的子类，是从**字符流到字节流**的桥梁。使用指定的字符集将字符编码为字节。它的字符集可以由名称指定，也可以接受平台的默认字符集。
- 构造方法
    * OutputStreamWriter(OutputStream in): 创建一个使用默认字符集的字符流。 
    * OutputStreamWriter(OutputStream in, String charsetName): 创建一个指定字符集的字符流。

``` java
OutputStreamWriter isr = new OutputStreamWriter(new FileOutputStream("out.txt"));
OutputStreamWriter isr2 = new OutputStreamWriter(new FileOutputStream("out.txt") , "GBK");
```

- 指定编码写出

``` java
public class OutputDemo {
   public static void main(String[] args) throws IOException {
    // 定义文件路径
       String FileName = "E:\\out.txt";
    // 创建流对象,默认UTF8编码
       OutputStreamWriter osw = new OutputStreamWriter(new FileOutputStream(FileName));
       // 写出数据
    osw.write("你好"); // 保存为6个字节
       osw.close();
// 定义文件路径
String FileName2 = "E:\\out2.txt";
    // 创建流对象,指定GBK编码
       OutputStreamWriter osw2 = new OutputStreamWriter(new FileOutputStream(FileName2),"GBK");
       // 写出数据
    osw2.write("你好");// 保存为4个字节
       osw2.close();
  }
}
```


### 8. 对象流
- 对象序列化和反序列化
    * Java 提供了一种对象**序列化**的机制。用一个字节序列可以表示一个对象，该字节序列包含该对象的数据、对象的类型和对象中存储的属性等信息。字节序列写出到文件之后，相当于文件中持久保存了一个对象的信息。 
    * 反之，该字节序列还可以从文件中读取回来，重构对象，对它进行**反序列化**。对象的数据、对象的类型和对象中存储的数据信息，都可以用来在内存中创建对象
    * 简单来讲
        + 对象的序列化指的是：将一个完整的对象 拆分成字节碎片 记录在文件中
        + 对象的反序列化指的是：将文件中记录的对象随便 反过来组合成一个完整的对象
        + 如果想要将对象序列化到文件中：需要让对象实现Serializable接口，是一个示意性接口； 
        如果想要将对象反序列化：需要给对象提供一个序列化的版本号，`private long serialVersionUID = 任意L`;


### 8.1 ObjectOutputStream类
- java.io.ObjectOutputStream 类，将Java对象的原始数据类型写出到文件,实现对象的持久存储。
- 构造方法
    * public ObjectOutputStream(OutputStream out)： 创建一个指定OutputStream的ObjectOutputStream。

``` java
FileOutputStream fileOut = new FileOutputStream("employee.txt");
ObjectOutputStream out = new ObjectOutputStream(fileOut);
```

- 序列化操作
    1. 一个对象要想序列化，必须满足两个条件:
        + 该类必须实现java.io.Serializable 接口，Serializable 是一个标记接口，不实现此接口的类将不会使任何状态序列化或反序列化，会抛出NotSerializableException 。
        + 该类的所有属性必须是可序列化的。如果有一个属性不需要可序列化的，则该属性必须注明是瞬态的，使用transient 关键字修饰。
    2. 写出对象方法
        + public final void writeObject (Object obj) : 将指定的对象写出。
``` java
//满足两个条件
public class Employee implements java.io.Serializable {
   public String name;
   public String address;
   public transient int age; // transient瞬态修饰成员,不会被序列化
   public void addressCheck() {
    System.out.println("Address check : " + name + " -- " + address);
  }
}
//写出对象方法
public class SerializeDemo{
  public static void main(String [] args)   {
  Employee e = new Employee();
  e.name = "zhangsan";
  e.address = "beiqinglu";
  e.age = 20; 
  try {
    // 创建序列化流对象
         ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream("employee.txt"));
      // 写出对象
      out.writeObject(e);
      // 释放资源
      out.close();
      fileOut.close();
      System.out.println("Serialized data is saved"); // 姓名，地址被序列化，年龄没有被序列化。
      } catch(IOException i)   {
           i.printStackTrace();
      }
  }
}
//输出结果：
//Serialized data is saved
```


#### 8.2 ObjectInputStream类
- ObjectInputStream反序列化流，将之前使用ObjectOutputStream序列化的原始数据恢复为对象。 
- 构造方法
    * public ObjectInputStream(InputStream in)： 创建一个指定InputStream的ObjectInputStream。
- 反序列化操作1
    * 如果能找到一个对象的class文件，我们可以进行反序列化操作，调用ObjectInputStream读取对象的方法。
    * 对于JVM可以反序列化对象，它必须是能够找到class文件的类。如果找不到该类的class文件，则抛出一个 ClassNotFoundException 异常。
    * public final Object readObject () : 读取一个对象。

``` java
public class DeserializeDemo {
  public static void main(String [] args)   {
       Employee e = null;
       try {
            // 创建反序列化流
            FileInputStream fileIn = new FileInputStream("employee.txt");
            ObjectInputStream in = new ObjectInputStream(fileIn);
            // 读取一个对象
            e = (Employee) in.readObject();
            // 释放资源
            in.close();
            fileIn.close();
      }catch(IOException i) {
            // 捕获其他异常
            i.printStackTrace();
            return;
      }catch(ClassNotFoundException c) {
      // 捕获类找不到异常
            System.out.println("Employee class not found");
            c.printStackTrace();
            return;
      }
       // 无异常,直接打印输出
       System.out.println("Name: " + e.name);// zhangsan
       System.out.println("Address: " + e.address); // beiqinglu
       System.out.println("age: " + e.age); // 0
  }
}
```


- 反序列化操作2
    * 另外，当JVM反序列化对象时，能找到class文件，但是class文件在序列化对象之后发生了修改，那么反序列化操作也会失败，抛出一个InvalidClassException异常。发生这个异常的原因如下：
        + 该类的序列版本号与从流中读取的类描述符的版本号不匹配 
        + 该类包含未知数据类型 
        + 该类没有可访问的无参数构造方法 
    * Serializable 接口给需要序列化的类，提供了一个序列版本号。serialVersionUID 该版本号的目的在于验证序列化的对象和对应类是否版本匹配。

``` java
public class Employee implements java.io.Serializable {
    // 加入序列版本号
    private static final long serialVersionUID = 1L;
    public String name;
    public String address;
    // 添加新的属性 ,重新编译, 可以反序列化,该属性赋为默认值.
    public int eid; 

    public void addressCheck() {
        System.out.println("Address check : " + name + " -- " + address);
    }
}
```


### 9. 打印流(PrintStream类)
- 平时我们在控制台打印输出，是调用print方法和println方法完成的，这两个方法都来自于java.io.PrintStream类，该类能够方便地打印各种数据类型的值，是一种便捷的输出方式。
- 构造方法
    * `public PrintStream(String fileName);`  使用指定的文件名创建一个新的打印流。

``` java
PrintStream ps = new PrintStream("ps.txt")；
```

- 改变打印流向
    * System.out就是PrintStream类型的，只不过它的流向是系统规定的，打印在控制台上。不过，我们可以改变它的流向。

``` java
public class PrintDemo {
    public static void main(String[] args) throws IOException {
        // 调用系统的打印流,控制台直接输出97
        System.out.println(97);
        // 创建打印流,指定文件的名称
        PrintStream ps = new PrintStream("ps.txt");
        // 设置系统的打印流流向,输出到ps.txt
        System.setOut(ps);
        // 调用系统的打印流,ps.txt中输出97
        System.out.println(97);
    }
}
```


### 10. Properties类的使用
- Java.util.Properties，主要用于读取Java的配置文件。
- Properties类继承自Hashtable
- 配置文件：在Java中，其配置文件常为.properties文件，格式为文本文件，文件的内容的格式是“键=值”的格式，文本注释信息可以用"#"来注释。
- Properties类的主要方法：
    1. getProperty ( String key)，用指定的键在此属性列表中搜索属性。也就是通过参数 key ，得到 key 所对应的 value。
    2. load ( InputStream inStream)，从输入流中读取属性列表（键和元素对）。通过对指定的文件（比如说上面的 test.properties 文件）进行装载来获取该文件中的所有键 - 值对。以供 getProperty ( String key) 来搜索。
    3. setProperty ( String key, String value) ，调用 Hashtable 的方法 put 。他通过调用基类的put方法来设置 键 - 值对。
    4. store ( OutputStream out, String comments)，以适合使用 load 方法加载到 Properties 表中的格式，将此 Properties 表中的属性列表（键和元素对）写入输出流。与 load 方法相反，该方法将键 - 值对写入到指定的文件中去。
    5. clear ()，清除所有装载的 键 - 值对。该方法在基类中提供。


