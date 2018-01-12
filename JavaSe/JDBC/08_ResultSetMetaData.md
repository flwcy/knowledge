### ResultSetMetaData

`ResultSetMetaData`有关`ResultSet`中列的名称和类型的信息。

```java
//从元数据中获得列数 
ResultSetMetaData rsmd; 
rsmd = results.getMetaData(); 
numCols = rsmd.getColumnCount(); 
```

#### 将结果集封装为Map

```java
package com.flwcy.test;

import com.flwcy.util.SelfDbUtils;

import java.sql.*;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

public class ResultSetMetaDataTest {

    public static void main(String[] args) {
        List<Map<String,Object>> result = getResult("select user_name as userName,`password`,email,birthday from db_user",null);
        for(Map<String,Object> data : result){
            for(Map.Entry<String,Object> entry : data.entrySet()){
                System.out.print(entry.getKey() + ":" + entry.getValue() + "\t");
            }
            System.out.println("");
        }
    }

    public static List<Map<String,Object>> getResult(String sql, Object params){
        Connection connection = null;
        PreparedStatement statement = null;
        ResultSet resultSet = null;

        List<Map<String,Object>> result = new ArrayList<>();

        try {
            connection = SelfDbUtils.getInstance().getConnection();

            statement = connection.prepareStatement(sql);

            resultSet = statement.executeQuery();

            ResultSetMetaData rsmd = resultSet.getMetaData();

            // 获取列数
            int count = rsmd.getColumnCount();
            String[] columnLabels = new String[count];
            // 列名数组
            for(int i=1;i<=count;i++){
                columnLabels[i - 1] = rsmd.getColumnLabel(i);

            }

            int index = 0;

            while(resultSet.next()){
                Map<String,Object> data = new HashMap<String,Object>();
                for(int i=1;i<=count;i++){
                    data.put(columnLabels[i - 1],resultSet.getObject(columnLabels[i - 1]));
                }
                result.add(data);
            }
        } catch (SQLException e) {
            e.printStackTrace();
        } finally {
            SelfDbUtils.getInstance().close(connection,statement,resultSet);
        }

        return result;
    }
}
```

#### 将结果集封装为对象

这个需求相对来说就显得有些复杂了，我们需要用到反射的技术

```java
package com.flwcy.test;

import com.flwcy.entity.User;
import com.flwcy.util.SelfDbUtils;
import org.junit.Test;

import java.lang.reflect.Constructor;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;
import java.sql.*;
import java.util.ArrayList;
import java.util.List;

public class ORMTest<T> {

    @Test
    public void test(){
        List<T> result = getUsers("select user_name as userName,password,email,birthday from db_user","com.flwcy.entity.User");
        for(T t : result){
            System.out.println(t);
        }
    }

    public List<T> getUsers(String sql,String className){
        List<T> result = new ArrayList<>();
        Connection connection = null;
        PreparedStatement statement = null;
        ResultSet resultSet = null;

        try {
            connection = SelfDbUtils.getInstance().getConnection();
            statement = connection.prepareStatement(sql);

            resultSet = statement.executeQuery();
            ResultSetMetaData rsmd = resultSet.getMetaData();

            int count = rsmd.getColumnCount();

            String[] columnLabels = new String[count];
            for(int i=1;i<=count;i++){
                columnLabels[i - 1] = rsmd.getColumnLabel(i);
            }
            Class clazz = getClass(className);
            Method[] methods = clazz.getDeclaredMethods();
            int index = 0;
            while (resultSet.next()){
                T instance = (T)getInstance(clazz);
                for(String columnLabel : columnLabels){
                    // 获取set方法
                    String setName = String.format("set%s%s" ,columnLabel.substring(0,1).toUpperCase(),columnLabel.substring(1));
                    for(Method method : methods){
                        if(method.getName().equals(setName)){
                            method.invoke(instance,resultSet.getObject(columnLabel));
                            break;
                        }
                    }
                }

                result.add(instance);
            }

        } catch (SQLException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (InvocationTargetException e) {
            e.printStackTrace();
        } finally {
            SelfDbUtils.getInstance().close(connection,statement,resultSet);
        }

        return result;
    }

    private Class getClass(String className){
        Class clazz = null;
        try {
            clazz = Class.forName(className);
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
        return clazz;
    }

    private Object getInstance(Class clazz){
        Object result = null;
        try {

            Constructor constructor = clazz.getConstructor(new Class[]{});
            result = constructor.newInstance(new Object[]{});
        } catch (NoSuchMethodException e) {
            e.printStackTrace();
        } catch (InstantiationException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (InvocationTargetException e) {
            e.printStackTrace();
        }

        return  result;
    }
}
```



