# 1. IO流

IO流

- 文件
- IO流原理及流的分类
- 节点流和处理流
- 输入流
- 输出流
- Properties

## 1.1 文件

### 1）文件流

文件在程序中是以流的形式来操作的，输入输出是相对于Java程序来说的：

- 输入流：文件（磁盘、网络等）到Java程序
- 输出流：Java程序（内存）到文件（磁盘、网络等）

![image-20210419163739758](IO流.assets/image-20210419163739758.png)

### 2）File类

> **在Java中，File对应了文件和目录**

![image-20210419164409075](IO流.assets/image-20210419164409075.png)

#### 创建文件

- 创建文件对象相关构造器和方法

  - 通过构造器创建的对象只是在Java程序（内存）中创建了一个File对象
  - 调用该对象的createNewFile()方法才会在物理磁盘上创建文件

  ```java
  new File(String pathname)	//根据路径创建一个File对象
  new File(File parent, String child)	//根据父文件+ 子路径构建
  new File(String parent, String child)	//根据父目录 + 子路径构建
  //使用以上三种构造方法创建文件对象后，还需要对其调用这个方法完成创建
  createNewFile()	
  ```

  ```java
  // 方式1 new File(String pathname)
  public static void create1() {
      String filePath = "D:\\new1.txt";
      File file = new File(filePath);
      try {
          file.createNewFile();
          System.out.println("文件创建成功");
      } catch (IOException e) {
          e.printStackTrace();
      }
  }
  // 方式2 new File(File parent, String child)
  public static void create2() {
      File parentFile = new File("D:\\");
      String fileName = "new2.txt";
      //这一步只是在Java程序（内存）中创建一个了 File对象
      File file = new File(parentFile, fileName);
      try {
          //这一步才在物理磁盘上创建了文件
          file.createNewFile();
          System.out.println("创建成功");
      } catch (IOException e) {
          e.printStackTrace();
      }
  }
  
  // 方式3 new File(String parent, String child)
  public static void create3() {
      String parent = "D:\\";
      String fileName = "new3.txt";
      File file = new File(parent, fileName);
      try {
          file.createNewFile();
          System.out.println("创建成功");
      } catch (IOException e) {
          e.printStackTrace();
      }
  }
  ```

#### 获取文件信息

```java
public void info() {
    //先创建文件对象
    File file = new File("D:\\new1.txt");
    File dict = new File("D:\\");
    //调用相应的方法，得到对应的信息
    System.out.println("文件名字：" + file.getName());
    //获取绝对路径
    System.out.println("绝对路径：" + file.getAbsolutePath());
    //获取文件父目录
    System.out.println("父目录：" + file.getParent());
    //获取文件大小
    System.out.println("文件大小：" + file.length());
    //文件是否存在
    System.out.println("文件是否存在：" + file.exists());
    //文件是否是file
    System.out.println("文件是否存在：" + file.isFile());
    //文件是否是目录
    System.out.println("文件是否是目录：" + file.isDirectory());
}
```

#### 目录的操作和文件删除

##### 创建目录

- 创建一级目录：file.mkdir()

- 创建多级目录：file.mkdirs()

  ```java
  //判断目录d:\\demo02是否存在，如果存在则提示存在，不存在则创建该目录
  @Test
  public void m3() {
      String dire = "d:\\demo02";
      File file = new File(dire);
      if(file.exists())
          System.out.println(dire + "已存在");
      else
          if(file.mkdir())
              System.out.println("该目录创建成功");
      else
          System.out.println("创建失败");
  }
  ```

##### 删除文件和目录

```java
//判断目录d:\\demo01是否存在，如果存在则删除
//在Java中，目录也被当作File来处理
@Test
public void m2() {
    String filePath = "d:\\demo01";
    File file = new File(filePath);
    if(file.exists()) {
        if(file.delete())
            System.out.println("删除成功");
        else
            System.out.println("删除失败");
    } else
        System.out.println("该目录不存在");
}
```

## 1.2 IO流原理及流的分类

### 1）流的分类

Java中的IO流的**输入输出是相对Java程序**而言的：

- 输入流：读取外部数据到Java程序（内存）
- 输出流：将Java程序（内存）数据输出到外部（磁盘、网络等）

JavaIO流共涉及40多个类，都是由**四个抽象类基类**派生出来的。按照操作数据单位不同、数据流的流向不同、流的角色不同，对流进行分类：

- 操作数据单位：**字节流**（8bit，**适合处理二进制文件**）、**字符流**（按字符，**适合处理文本文件**）
  - 字节流：**InputStream、OutputStream**
  - 字符流：**Reader、Writer**
  
- 按数据流方向：输入流、输出流

- 按流的角色不同：**节点流**、**处理流/包装流**

  ![image-20210420082424005](IO流.assets/image-20210420082424005.png)

  - 节点流：对单个特定的数据源进行读写操纵
  - 处理流：对已存在的流（节点流或处理流）进行封装，为程序提供更强大的读写功能。处理流使用了**修饰者模式**。

#### 节点流和处理流

- 节点流是底层流，直接和数据源相接；

- **处理流**是包装节点流，即可以**消除不同节点流的实现差异**，也可以**提供更加方便的方法来完成输入输出**
- 处理流使用了**修饰器设计模式**，不会直接与数据源相连

处理流的功能：

- 性能的提高：**增加缓冲**的方式来提高输入输出效率
- 操作的便捷：处理流提供了一系列便捷的方法来完成**一次输入输出大批量的数据**

### 2）IO 流 vs File

**File相当于外部数据**，Java程序**通过IO流与File进行数据交换**。

IO流为资源，需要使用完毕在finally代码块中**释放IO流**。

### 3）常用的类

#### **节点流**

> 节点流构造方法需要传递String
>
> 处理流构造方法需要传递的是节点流或者处理流对象

当**文件二进制字节流**输入输出可以使用FileInputStream、FileOutputStream，**文件字符流**输入输出可以使用**FileReader**、**FileWriter**。

- 创建：通过**表明文件地址的字符串**作为构造方法的参数
- 输入、输出（read()、write()）
- 关闭（输出流需要调用close()或者flush()才会将数据输出）
- **字符输出流底部调用的是二进制字节流输出**

当创建输出流对象时，**构造方法传递第二个boolean参数**可以控制其输出到外部的方式是覆盖还是追加。

#### **处理流**

提高输入输出效率

- 转换流：用于将二进制字节流转换为某种编码格式的字符流
  - 构造方法需要：字节流、编码格式

- 对象流：用来将基本类型或者对象**序列化**和**反序列化**操作（**ObjectInputStream**、**ObjectOutputStream**）
  - 对象流是字节流，又是处理流，构造方法需要字节流
  - 对象输出流：writeInt()、writeChar()、writeString()、writeObject等
  - 对象输入流：readInt()、readChar()、readUTF()、readObject()





## 1.3 字节流

### 1）文件输入输出流

#### I. FileInputStream

##### ① 构造器

- 通过File对象来创建FileInputStream，即从该File对象中向Java程序读取数据；（File对象 --> Java程序）
- 通过路径指定的文件来创建FileInputStream，从该路径指定的文件来向Java程序读取数据；

```java
public FileInputStream(String name) throws FileNotFoundException {
}

public FileInputStream(File file) throws FileNotFoundException {
}

public FileInputStream(FileDescriptor fdObj) {
}
```

##### ② 常用方法

- read()：每次读取一个字节并返回读取的数据；如果读完了则返回-1
- read(byte[] b)：每次从输入流中最多读取b.length字节的数据到b中，并返回读取的数据的长度；如果读完了，则返回-1

#### II. FileOutputStream

##### ① 构造器

> 如果文件不存在时，会自动创建一个新的文件（**前提是存在目录**）

- 通过File对象来创建FileOutputStream，即从Java程序向对应的文件写入数据；（Java程序 --> File文件）

- 通过File对象来创建FileOutputStream，**并采用在文件末尾追加的方式写入**, 即从Java程序向对应的文件写入数据；（Java程序 --> File文件）

  ```java
  FileOutputStream fileOutputStream = new FileOutputStream(new File(filePath), true);
  ```

- 通过路径指定的文件来创建FileOutputStream，从Java程序向对应的文件写入数据；

  ```java
  FileOutputStream fileOutputStream = new FileOutputStream(filePath);
  ```

- 通过路径指定的文件来创建FileOutputStream，**并采用在文件末尾追加的方式写入**，从Java程序向对应的文件写入数据；

  ```java
  FileOutputStream fileOutputStream = new FileOutputStream(filePath, true);
  ```

##### ② 常用方法

> 写入时默认会覆盖掉原来的内容
>
> 构造方法传入true时，则会采用末尾追加的方式写入

- write(int b)

- write(byte[] b)

  ```java
  @Test
  public void writeFile() {
      String filePath = "D:\\a.txt";
      FileOutputStream fileOutputStream = null;
      try {
          fileOutputStream = new FileOutputStream(filePath);
          String str = "Hello world";
          fileOutputStream.write(str.getBytes());
      } catch (IOException e) {
          e.printStackTrace();
      } finally {
          try {
              fileOutputStream.close();
          } catch (IOException e) {
              e.printStackTrace();
          }
      }
  }
  ```

- write(byte[] b, int start, int offset)

##### ③ 实例 -- 文件拷贝

拷贝时需要主要要使用**write(bytes, start, offset)**方法将数据写出到外部，其中offset为从外部读取到的数据的长度。

```java
public static void main(String[] args) {
    //将文件从D:\a.txt拷贝到D:\test\a.txt
    /**
         * 1. 获取原来文件的输入流
         * 2. 创建文件输出流
         */
    String originalPath = "D:\\a.txt";
    String copyPath = "D:\\test\\a.txt";
    byte[] bytes = new byte[128];
    FileInputStream fileInputStream = null;
    FileOutputStream fileOutputStream = null;
    try {
        fileInputStream = new FileInputStream(originalPath);
        fileOutputStream = new FileOutputStream(copyPath);
        int readLen = 0;
        while((readLen = fileInputStream.read(bytes)) != -1) {
            fileOutputStream.write(bytes, 0, readLen);
        }
    } catch (IOException e) {
        e.printStackTrace();
    } finally {
        try {
            if(fileInputStream != null)
                fileInputStream.close();
            if(fileOutputStream != null)
                fileOutputStream.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

### 2）提高效率的包装流

#### I. BufferedInputStream

#### II. BufferedOutputStream

### 3）用于序列化的对象输入输出流

> ObjectOutputStream用于完成基本类型或者对象的序列化操作
>
> ObjectInputStream用于完成基本类型或者对象的反序列化操作
>
> 类中使用static、transient修饰的变量不会被序列化

#### I. ObjectOutputStream

> 内部维护了一个BlockDataOutputStream类，并创建一个该类对象，调用其writeInt()、writeBytes()等方法来完成实际的功能。

![image-20210420193601871](IO流.assets/image-20210420193601871.png)

##### ① 构造器

- 要创建ObjectOutputStream时，一般使用第一个构造方法，向其**传入一个字节输出流**，作为序列化内容会通过该字节输出流输出到指定的位置

```java
public ObjectOutputStream(OutputStream out)
protected ObjectOutputStream() 
```

##### ② 常用方法

> ObjectOutputStream仍然保留有OutputStream的一些常用方法，但一般都不用

- **序列化基本类型值**
  - writeBoolean(boolean)
  - writeByte(int)
  - writeChar(int v)
  - writeShort(int v)
  - writeInt(int v) 
  - writeLong(long v) 
  - writeFloat(float v)
  - writeDouble(double v) 
- **序列化字符串**
  - writeBytes(String)：将字符串序列化为字节序列
  - writeChars(String)：将字符串序列化为字符序列
  - writeUTF(String)：将字符串序列化为UTF-8字符串
- **序列化对象**
  - writeObject(Object)
- 刷新缓冲区

```java
public final void writeObject(Object obj) throws IOException {
}
```

#### II. ObjectInputStream

##### ① 构造器

- 要创建ObjectInputStream时，一般使用第一个构造方法，向其**传入一个字节输入流**，作为序列化内容会通过该字节输入流输入到Java程序，并通过其方法完成反序列化

```java
public ObjectInputStream(InputStream in) throws IOException {
}
protected ObjectInputStream() throws IOException, SecurityException {
}
```

##### ② 常用方法

> ObjectInputStream仍然保留有InputStream的一些常用方法，但一般都不用

- **序列化基本类型值**
  - boolean readBoolean()
  - byte readByte()
  - char readChar()
  - short readShort()
  - int readInt() 
  - long readLong() 
  - float readFloat()
  - double readDouble() 
- **序列化字符串**
  - String readUTF()
- **序列化对象**
  - Object readObject()
- 关闭流

## 1.4 字符流 

### 1）文件输入输出流

> 字符流的相关方法与字节流类似，只是在字符流中，需要用的是**char数组。**
>
> 其**底层其实是对字节流进行封装**

#### I. FileReader

##### ① 构造器

- new FileReader(File/String)

##### ② 常用方法

- read()：返回读取到的字符，返回类型为int；如果没有数据了，则返回-1
- read(char[])：将外部数据读取到char数组中，并返回读取的数据长度；如果没有数据了，则返回-1

#### II. FileWriter

> 使用FileWrite时，每次调用write方法，**必须要调用close()或者flush()来将内容刷新到外部**

##### ① 构造器

- new FileWriter(File/String)：覆盖模式
- new FileWriter(File/String, boolean )：追加true时，使用追加模式

##### ② 常用方法

- write(int)：写入单个字符
- write(char[])
- write(char[], start, len)
- write(String)
- write(String, start, len)

```java
@Test
public void write() {
    String filePath = "D:\\write.txt";
    FileWriter fileWriter = null;
    char[] chars = new char[128];
    try {
        fileWriter = new FileWriter(filePath);
        fileWriter.write('a');   //写单个字符
        fileWriter.write(new char[]{'b', 'c','张'}); //写入char数组
        fileWriter.write(new char[]{'b', 'c','张'}, 1, 2);   /
            fileWriter.write("你好");
        fileWriter.write("你好，陌生人！", 2, 5);
    } catch (IOException e) {
        e.printStackTrace();
    } finally {
        try {
            //如果不调用flush()或者writer()，则内容不会刷新到外部
            //fileWriter.flush();
            fileWriter.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

### 2）提高效率的输入输出流

#### I. BufferedReader

> BufferedReader是一个**处理流（包装流）**，用来包装其他节点流，以提供更强大的输入功能。
>
> 关闭处理流时，只需要关闭外层流（即**处理流会调用节点流的close()**）即可。

BufferedReader中成员变量：

- Reader in：用来接收其他**字符节点流**或者处理流
- char cb[]：用来接收输入的数据

```java
private Reader in;
private char cb[];
```

##### ① 构造器

- new BufferedReader(Reader, int size)：传入一个字符流（可以是节点流也可以是处理流），并且设置缓冲区大小为size
- new BufferedReader(Reader)：传入一个字符流，缓冲区大小设置为默认值（8192）

##### ② 常用方法

- readLine()：每次读取一行，当读到流的末尾时，返回null
- read()：返回读取到的字符，返回类型为int；如果没有数据了，则返回-1
- read(char[])：将外部数据读取到char数组中，并返回读取的数据长度；如果没有数据了，则返回-1

#### II. BufferedWriter

> 构造器与BufferedReader类似，常用方法与FileWriter类似

##### ① 构造器

- new BufferedWriter(Writer)
- new BufferedWriter(Writer, int)

##### ② 常用方法

- write(int)：写入单个字符
- write(char[])
- write(char[], start, len)
- write(String)
- write(String, start, len)
- **newLine()**：写入换行符

## 1.5 字节流转换为字符流

> 字符流默认按照UTF-8来读取，对于不是UTF-8编码的文本采用字符流输入输出时，就会出现乱码问题

### 1）InputStreamReader

<img src="IO流.assets/image-20210421115611233.png" alt="image-20210421115611233" style="zoom:50%;" />

#### ① 构造器

- 在通过构造方法创建InputStreamReader对象时，需要**传入一个字节输入流对象**。

- 可以传入一个**字符串或者Charset对象**，来指定输入二进制字节流的编码方式

```java
InputStreamReader(InputStream, Charset)
InputStreamReader(InputStream)
InputStreamReader(InputStream, String)
InputStreamReader(InputStream, CharsetDecoder)
```

#### ② 常用方法

- 通常通过一个二进制字节流和指定的编码来创建一个InputStreamReader
- 然后通过创建的InputStreamReader对象来创建BufferedReader对象

```java
public void test() {
    String filePath = "D:\\a.txt";
    try {
        // 1. 将FileInputStream流转换为编码为gbk的InputStreamReader
        InputStreamReader isr = new InputStreamReader(new FileInputStream(filePath), "gbk");
        // 2. 将InputStreamReader传入BufferedReader
        BufferedReader br = new BufferedReader(isr);
        // 3. 读取
        String s = br.readLine();
        System.out.println("读取内容：" + s);
        br.close();
    } catch (UnsupportedEncodingException e) {
        e.printStackTrace();
    } catch (IOException e) {
        e.printStackTrace();
    } 
}
```

### 2）OutputStreamWrite

![image-20210421115906229](IO流.assets/image-20210421115906229.png)

#### ① 构造器

- 在通过构造方法创建InputStreamReader对象时，需要**传入一个字节输出流对象**。

- 可以传入一个**String对象或者Charset对象**，来指定输入二进制字节流的编码方式

```java
OutputStreamReader(OutputStream, Charset)
OutputStreamReader(OutputStream)
OutputStreamReader(OutputStream, String)
OutputStreamReader(OutputStream, CharsetDecoder)
```

#### ② 常用方法

- 与InputStreamReader类似，通过一个二进制输出字节流和指定的编码格式来创建OutputStreamReader对象
- 然后通过OutputStreamReader对象来创建BufferedWrite对象来完成数据输出



## 1.6 标准输入输出流

### 1）System.in

- 声明为InputStream类型，实例化为BufferedInputStream；

- 标准输入，默认设备为键盘

- 另外，可以使用System.in来作为Scanner的构造参数，实例化一个Scanner对象，用来获取程序的输入

```java
@Test
public void test() {
    System.out.println(System.in.getClass());
    System.out.println(System.out.getClass());
    System.out.println(System.err.getClass());
    Scanner scanner = new Scanner(System.in);
    String next = scanner.next();

}
```



### 2）System.out

> 声明为PrintStream类型，实例化为PrintStream
>
> 标准输出，默认设备为显示器

- 可以通过System.setOut(PrintStream)修改System中的PrintStream对象，修改其输出位置

## 1.7 打印流

> 打印流分为字节流和字符流两种
>
> 打印流既可以输出到屏幕，也可以输出到文件中
>
> 构建方式：
>
> - 通过二进制/字符输出流来构建，构建时可以指定编码方式
>
> - 通过String指定文件路径构建
>
> - 通过File构建

### 1）PrintStream

#### ① 构造器

- 可以构建一个输出到文件的打印流

  ```java
  PrintStream(File)
  PrintStream(String)
  ```

- 可以指定打印流是否自动刷新

  ```java
  PrintStream(OutputStream)
  PrintStream(OutputStream, boolean)
  ```

- 指定打印流的编码格式

  ```java
  PrintStream(OutputStream, boolean, Charset)
  ```

##### ② 常用方法

> 底层调用write方法

- print()
- println()
- close()

### 2）PrintWriter

#### ① 构造器

> 既可以通过二进制字节流构建，也可以通过字符流构建

- 可以构建一个输出到文件的打印流

  ```java
  PrintWriter(File)
  PrintWriter(String)
  ```

- 通过字符流构建

  ```java
  PrintWriter(Writer)
  PrintWriter(Writer, boolean)
  ```

- 通过二进制字节流构建

  ```java
  PrintWriter(OutputStream)
  PrintWriter(OutputStream,boolean)
  ```

##### ② 常用方法

> 底层调用write方法

- print()
- println()
- close()

## 1.8 Properties

> 将程序中需要的一些配置信息，如数据库的用户名和密码等，从源代码中脱离出来，方便后期维护。

Properties类是HashTable的子类，专门用于读取配置文件的集合类

- 键值对"="两侧不想要有空格
- 值不需要用引号括起来，默认类型是String

### 1）构造器

### 2）常用方法

```java
//通过Reader/InputStream获取Properties
load(Reader)
load(InputStream)
//将Properties中的内容通过Writer/OutputStream输出
list(Writer)
list(OutputStream)
//根据键获取值
getProperty(key)
//设置键值对到Properties对象
setProperty(key, value)
```

### 3）根据配置文件创建Properties对象的几种方式

将配置文件加载Properties对象一般有如下几个步骤：

1. 创建Properties对象

   ```java
   Properties properties = new Properties();
   ```

2. 根据其配置文件创建输入流（可以是二进制字节流或字符流）

   - 字节流：一般通过**Class对象的getClassLoader().getResourceAsStream()**方法来获得该配置文件的输入流，其中**"/"表示从classpath的根路径**查找（配置文件一般放在resources目录中），不带"/"则表示从当前这个class所在的包中开始查找

   ```java
   InputStream is = this.getClass().getClassLoader().getResourceAsStream("/dp.properties");
   ```

3. 根据配置文件的输入流加载配置文件

   ```java
   properties.load(is);
   ```

4. 获取配置文件中的某些配置信息

   ```java
   properties.getPropert(String key);
   ```

### 4）乱码问题

**Windows默认中文编码格式为gbk**（Linux默认为UTF-8），所以如果创建的配置文件中有中文时，则配置文件会采用gbk的编码格式，如果直接使用字节加载配置文件则会出现乱码问题。

要解决这个中文乱码问题，可以使用**InputStreamReader**，将字节流转换为gbk编码格式的字符流，并创建一个**BufferedReader**对象传递个load方法。

```java
InputStream is = this.getClass().getResourceAsStream("/dog.properties");
Properties properties = new Properties();
BufferedReader br = null;
try {
    br = new BufferedReader(new InputStreamReader(is, "gbk"));
    properties.load(br);

} catch (IOException e) {
    e.printStackTrace();
} finally {
    if(br != null)
        try {
            br.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
}
```



## 1.9 本章作业

1）判断是否存在文件夹，没有创建

2）创建文件

如果已经存在，则提示