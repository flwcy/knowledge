### Java IO详解之File

`File`类是java io包下代表与平台无关的文件和目录，也就是说，如果希望在程序中操作文件和目录，都可以通过File类来完成。File不能访问文件内容本身。如果需要访问文件内容本身，则需要使用输入/输出流。

#### 构造函数

`File`类有四个构造函数：

`File(String pathname)`

`File(File parent, String child)`　　

`File(String parent, String child)`　　

`File(URI uri)`

利用构造方法，指定路径名、文件名等来构造`File`类的对象，之后调用该对象的`createNewFile()`方法就可以创建出相应的文件。

`parent`指定路径（父目录），可以是`File`类对象也可以是字符串，`child`中也可以加入路径层级，但要注意，**所用的路径必须存在，不存在的路径不会新建。**

```java
        // 初始化File对象
        File file = new File("D:/abc");
        // 在abc目录下创建对应的文件，abc目录必须存在，否则抛出java.io.IOException: 系统找不到指定的路径
        File fileChild = new File(file ,"test.text");
        //等同于如下代码
        // File newFile = new File("D:/abc","test.text");
        // 创建文件
        fileChild.createNewFile();
```

有两个方法用来判断`File`对象是文件还是目录：

+ `public boolean isDirectory()//是否为目录`　　
+ `public boolean isFile()//是否为文件`

**一个File类对象既可以表示目录也可以表示文件。**

#### 目录管理

操作目录的方法如下：

+ `public boolean mkdir()`　　
+ `public boolean mkdirs()`

示例：

```java
      File file = new File("D:/abc/test/hello");
      file.mkdir();
```

假设D盘下abc目录已经存在，但test目录不存在，则不能成功创建hello目录。**要用mkdir()创建目录，必须上一级目录存在，即最底层目录前面的目录全都存在。**要想把File对象包含的目录一次性全都创建好，可以使用`mkdirs()`方法。

`public String[] list()`返回抽象路径名表示目录下的文件名和目录名。　　

`public File[] listFiles()`返回抽象路径名下的所有文件和目录。

>    前面提到File类对象既可以是文件也可以是目录，创建文件是createNewFile()方法，相对应的，创建目录即调用mkdir()方法。

#### 文件管理

在进行文件操作时，常需要知道一个关于文件的信息。`Java`的`File`类提供了方法来操纵文件和获得一个文件的信息。如`getName()`返回文件名，`getParent()`返回父目录名，`exists()`测试文件或目录是否存在。

然而，File类是不对称的，说它不对称，意思是虽然存在允许验证一个简单文件对象属性的很多方法，但是没有相应的方法来修改这些属性，即有`get`无`set`。

另外，File类还可以对目录和文件进行删除（删除目录时，目录必须为空时才能删除）、属性修改等工作。方法名都比较直观，在此不多做介绍了。

#### 使用FilenameFilter

如果我们希望能够限制由`list()`方法返回的文件。为达到这样的目的，必须使用`list()`方法的带参数的重载形式：

​	`public String[] list(FilenameFilter filter)`

在该形式中，`filter`是一个实现了`FilenameFilter`接口的类的对象。该接口中有一个`accept()`方法，只有该方法判断为`true`的文件名会被`list`方法返回。所以可以在`accept`方法中设置筛选规则。（这里使用了策略模式，可以回想一下`Java`集合中的比较器）。

```java
        File file = new File("D:/abc");
        // 使用文件过滤器
        String[] fileNames = file.list(new FilenameFilter() {
            // 匿名内部类实现了FilenameFilter接口
            // 实现接口中的方法
            public boolean accept(File dir, String name) {
                if(name.endsWith(".text"))
                    return true;
                return false;
            }
        });
        // 只有accept方法中返回true的文件名才会被返回到fileNames数组中
        for (String fileName : fileNames){
            System.out.println(fileName);
        }
```

