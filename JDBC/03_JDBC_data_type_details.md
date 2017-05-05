# JDBC数据类型详解

标签（空格分隔）：Date Clob Blob

---



### 基本数据类型
JDBC中提供了我们所能见到的所有数据类型，其中像String、int等等，赋值使用的是PreparedStatement中的setter方法(类似setShot、setString等等)，取值使用的是ResultSet中的getter方法(类似getInt、getLong等等)可以查看Preparedstatement的API文档 ![API文档](..\img\jdbc\jdbc_data_01.jpg)

其中如果不知道数据表中数据类型或者不能确定是什么类型的情况下，可以直接使用setObject方法和getObject方法进行获取和设置.

### 日期类型
其中日期类型是比较特殊的一个类型，我们看一下ResultSet的getDate方法
![getDate](..\img\jdbc\jdbc_data_02.jpg)

>以Java编程语言中的java.sql.Date对象形式获取此ResultSet对象的当前行中指定列的值。
>参数:
>columnIndex  第一个列是1，第二个列是2，......
>返回：
>列值;如果值为SQL NULL，则返回值为null

其中返回值Date类型是java.sql.Date类型。
![java.sql.Date](..\img\jdbc\jdbc_data_03.jpg)

- java.sql.Date是java.util.Date的子类

基本上一般的数据库都支持三种形式的datetime字段(date/time/timestamp),每一种数据库的datetime字段在JDBC中都有与之对应的类，这些类都继承自java.util.Date.
- [java.sql.Date](http://docs.oracle.com/javase/8/docs/api/index.html?java/sql/Date.html)对应于SQL DATE，这意味着它存储了年、月和天，而小时、分钟、秒和毫秒均被忽略。此外sql.Date不依赖时区.
- [java.sql.Time](http://docs.oracle.com/javase/8/docs/api/index.html?java/sql/Time.html)对应于SQL TIME，应该是显而易见的仅包含了小时、分钟、秒和毫秒这些信息.
- [java.sql.Timestamp](http://docs.oracle.com/javase/8/docs/api/index.html?java/sql/Timestamp.html)对应于SQL TIMESTAMP，这是精确到纳秒的确切日期(请注意java.util.Date仅支持到毫秒),可以自定义精度.
```java
    @Test
    public void sqlDateTest() {
        Connection connection = null;
        PreparedStatement preparedStatement = null;

        try {
            connection = JdbcUtils.getConn();
            //新增记录 java.sql.date java.sql.Time java.sql.Timestamp
            String sql = "insert into db_date_test(sql_date,sql_time,sql_timestamp) VALUES(?,?,?)";
            preparedStatement = connection.prepareStatement(sql);

            java.sql.Date sqlDate = new java.sql.Date(System.currentTimeMillis());
            java.sql.Time sqlTime = new Time(System.currentTimeMillis());
            java.sql.Timestamp timestamp = new Timestamp(System.currentTimeMillis());

            preparedStatement.setDate(1,sqlDate);
            preparedStatement.setTime(2,sqlTime);
            preparedStatement.setTimestamp(3,timestamp);

            preparedStatement.executeUpdate();
        } catch (SQLException e) {
            e.printStackTrace();
        } finally {
            JdbcUtils.close(null, preparedStatement, connection);
        }
    }
```
查询数据库查看结果:
![结果](..\img\jdbc\jdbc_data_04.jpg)

至于java.sql.Date与java.util.Date两者之间的转换，有一个很简单的方法。 
因为两者都提供了一个long型的构造函数，两者通过getTime重新构造一下就行了。 
即： 

```java
java.util.Date date1 = ...; 
java.sql.Date date2 = ...; 

date1 = new java.sql.Date(date2.getTime()); 
date2 = new java.util.Date(date1.getTime()); 
```
在数据库时间建模和操作时习惯用java.sql.Timestamp，与数据库中DateTime对应，但业务流通层，还是习惯只用java.util.Date，因为这些时间直接的转换都是一样的方便，而且这样做逻辑理解上比较直观。
修改数据:
```java
        public int update(int id,java.util.Date date){
        Connection connection = null;
        PreparedStatement preparedStatement = null;
        int result = -1;
        try {
            connection = JdbcUtils.getConn();
            //新增记录 java.sql.date java.sql.Time java.sql.Timestamp
            String sql = "update db_date_test set sql_timestamp=? where id=?";
            preparedStatement = connection.prepareStatement(sql);

            preparedStatement.setTimestamp(1, new java.sql.Timestamp(date.getTime()));
            preparedStatement.setInt(2,id);
            result = preparedStatement.executeUpdate();
        } catch (SQLException e) {
            e.printStackTrace();
        } finally {
            JdbcUtils.close(null, preparedStatement, connection);
        }
        return result;
    }
```
查询数据:
```java
    public java.util.Date read(int id){
        Date date = null;
        Connection connection = null;
        PreparedStatement preparedStatement = null;
        ResultSet resultSet = null;
        try {
            connection = JdbcUtils.getConn();
            //新增记录 java.sql.date java.sql.Time java.sql.Timestamp
            String sql = "select sql_timestamp from db_date_test where id=?";
            preparedStatement = connection.prepareStatement(sql);

            preparedStatement.setInt(1,id);
            resultSet = preparedStatement.executeQuery();
            while(resultSet.next()){
                //Timestamp是java.util.Date的子类
                date = resultSet.getTimestamp(1);
            }
        } catch (SQLException e) {
            e.printStackTrace();
        } finally {
            JdbcUtils.close(null, preparedStatement, connection);
        }
        return date;
    }
```
### CLOB类型
当我们存储大量的文本信息时，数据库中的varchar或者varchar2肯定是不能满足的，varchar2最多只能有4000个长度，当我们需要存放一篇文章或者一个文本信息时，可以使用CLOB类型来满足我们的需求。在MySQL数据库中CLOB类型对应的是Text，DB2/Oracle中clob对应clob。
- 创建数据表
```SQL
DROP TABLE
IF EXISTS `db_clob_test`;

CREATE TABLE `db_clob_test` (
  `id` INT (11) NOT NULL AUTO_INCREMENT,
  `news` Text,
  PRIMARY KEY (`id`)
) ENGINE = INNODB AUTO_INCREMENT = 1 DEFAULT CHARSET = utf8 COLLATE = utf8_unicode_ci;
```
接下来写代码实现写入以及读取的操作
```java

    public int write() {
        int result = -1;
        Connection connection = null;
        PreparedStatement preparedStatement = null;
//        InputStream in = null;
        Reader reader = null;
        try {
            connection = JdbcUtils.getConn();
            String sql = "insert into db_clob_test(news) VALUES(?)";
            preparedStatement = connection.prepareStatement(sql);
//            in = new FileInputStream(new File("src/com/rooike/dao/impl/PreparedUserDaoImpl.java"));
//            preparedStatement.setAsciiStream(1,in);
            File file = new File("src/com/rooike/dao/impl/PreparedUserDaoImpl.java");
            reader = new BufferedReader(new FileReader(file));
            preparedStatement.setCharacterStream(1, reader,(int)file.length());
            result = preparedStatement.executeUpdate();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            JdbcUtils.close(null, preparedStatement, connection);
        }
        return result;
    }
```
- setAsciiStream(): 此方法用于提供大的ASCII值。
- setCharacterStream(): 此方法用于提供大的UNICODE值。
- setBinaryStream(): 使用此方法，以提供大的二进制值。
```java
    public void read(int id) {
        Connection connection = null;
        PreparedStatement preparedStatement = null;
        ResultSet resultSet = null;
        PrintWriter writer = null;
        BufferedReader reader = null;
        try {
            connection = JdbcUtils.getConn();
            String sql = "select news from db_clob_test where id=?";
            preparedStatement = connection.prepareStatement(sql);
            writer = new PrintWriter(new BufferedWriter(new FileWriter(new File("src/PreparedUserDaoImpl.java_tmp"))));
            preparedStatement.setInt(1, id);
            resultSet = preparedStatement.executeQuery();
            while (resultSet.next()) {
                reader = new BufferedReader(resultSet.getCharacterStream("news"));
                String result = null;
                while ((result = reader.readLine()) != null) {
                    writer.println(result);
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            JdbcUtils.close(resultSet, preparedStatement, connection);
            writer.close();
            try {
                reader.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
```
另外读取CLOB数据可以使用getClob方法,具体代码实现如下所示:
```java
public void readByClob(int id) {
        Connection connection = null;
        PreparedStatement preparedStatement = null;
        ResultSet resultSet = null;
        PrintWriter writer = null;
        try {
            connection = JdbcUtils.getConn();
            String sql = "select news from db_clob_test where id=?";
            preparedStatement = connection.prepareStatement(sql);
            writer = new PrintWriter(new BufferedWriter(new FileWriter(new File("src/PreparedUserDaoImpl.java_tmp"))));
            preparedStatement.setInt(1, id);
            resultSet = preparedStatement.executeQuery();
            while (resultSet.next()) {
                Clob clob = resultSet.getClob("news");
                if(clob != null){
                    writer.println(clob.getSubString((long)1,(int)clob.length()));
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            JdbcUtils.close(resultSet, preparedStatement, connection);
            writer.close();
        }
    }
```
### BLOB类型
BLOB和CLOB都是大字段类型,BLOB是按二进制存储的,而CLOB是可以直接存储文字的。通常像图片、文件、音乐等信息就使用BLOB字段来存储,先将文件转为二进制再存储进去.而像文章或者文本信息就使用CLOB存储。下面我们通过一个示例来进行演示
- 创建数据表
```SQL
DROP TABLE
IF EXISTS `db_blob_test`;

CREATE TABLE `db_blob_test` (
  `id` INT (11) NOT NULL AUTO_INCREMENT,
  `img` BLOB,
  PRIMARY KEY (`id`)
) ENGINE = INNODB AUTO_INCREMENT = 1 DEFAULT CHARSET = utf8 COLLATE = utf8_unicode_ci;
```
- 保存二进制数据至数据库中
```
    public int insert(){
        int result = -1;
        Connection connection = null;
        PreparedStatement preparedStatement = null;
        InputStream in = null;
        try {
            connection = JdbcUtils.getConn();
            String sql = "insert into db_blob_test(img) VALUES(?)";
            preparedStatement = connection.prepareStatement(sql);

            in = new FileInputStream(new File("src/paul.jpg"));
            preparedStatement.setBinaryStream(1,in);
            result = preparedStatement.executeUpdate();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            JdbcUtils.close(null,preparedStatement,connection);
            try {
                if(in != null)
                    in.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        return  result;
    }
```
程序运行正常，并且查看数据库，数据库中存放了图片的信息。由于通过SQL语句无法查看图片信息，所以我们通过读取的方式展示一下。
- 读取二进制文件
```java
    public int read(int id){
        int result = -1;
        Connection connection = null;
        PreparedStatement preparedStatement = null;
        ResultSet resultSet = null;
        OutputStream out = null;
        InputStream in = null;
        try {
            connection = JdbcUtils.getConn();
            String sql = "SELECT img FROM db_blob_test where id = ?";
            preparedStatement = connection.prepareStatement(sql);
            preparedStatement.setInt(1,id);

            File file = new File("src/paul_blob.jpg");
            resultSet = preparedStatement.executeQuery();
            while(resultSet.next()){
                //Blob blob = resultSet.getBlob("img");
                //in = blob.getBinaryStream();
                in = resultSet.getBinaryStream("img");
                out = new FileOutputStream(file);
                byte[] b = new byte[1024];
                int len = -1;
                while((len = in.read(b)) != -1){
                    out.write(b,0,len);
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            JdbcUtils.close(resultSet,preparedStatement,connection);
            try {
                if(out != null)
                    out.close();
                if (in != null)
                    in.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        return  result;
    }
```
程序运行结束之后，我们可以在src目录下看到该图片.
### 其他数据类型
其他数据类型可以查看[数据库](https://dev.mysql.com/doc/connector-j/5.1/en/connector-j-reference-type-conversions.html)和jdbc相关的文档,或者在java.sql.Types中查看所涉及的所有类型信息.
![types](..\img\jdbc\jdbc_data_05.jpg)






