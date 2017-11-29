### Java IO详解之File

`File`类是java io包下代表与平台无关的文件和目录，也就是说，如果希望在程序中操作文件和目录，都可以通过File类来完成。File不能访问文件内容本身。如果需要访问文件内容本身，则需要使用输入/输出流。

#### 构造函数

`File`类有四个构造函数：

`File(String pathname)`

`File(File parent, String child)`　　

`File(String parent, String child)`　　

`File(URI uri)`
