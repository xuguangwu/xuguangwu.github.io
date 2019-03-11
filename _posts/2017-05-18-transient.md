---
title: transient关键字
categories:
 - Java
tags: Java
---

我们最常用的transient场景就是当对象中某些字段不需要序列化，则可以在该字段上添加transient修饰。
1）一旦变量被transient修饰，变量将不再是对象持久化的一部分，该变量内容在序列化后无法获得访问。
(换个通俗点的说法：将不需要序列化的属性前添加关键字transient，序列化对象的时候，这个属性就不会序列化到指定的目的地中。)
2）transient关键字只能修饰变量，而不能修饰方法和类。注意，本地变量是不能被transient关键字修饰的。
变量如果是用户自定义类变量，则该类需要实现Serializable接口。
3）被transient关键字修饰的变量不再能被序列化，一个静态变量不管是否被transient修饰，均不能被序列化。
(这个概念通俗一点理解就是序列化的是对象，而类变量不是对象的，所以不管是否被虚化，静态变量都是不能被序列化的)

```
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import java.io.Serializable;

/**
 * @description 使用transient关键字不序列化某个变量
 *        注意读取的时候，读取数据的顺序一定要和存放数据的顺序保持一致
 *        
 */
public class TransientTest {
    
    public static void main(String[] args) {
        
        User user = new User();
        user.setUsername("xxx");
        user.setPasswd("123456");
        
        System.out.println("read before Serializable: ");
        System.out.println("username: " + user.getUsername());
        System.err.println("password: " + user.getPasswd());
        
        try {
            ObjectOutputStream os = new ObjectOutputStream(
                    new FileOutputStream("/Users/clear/temp.txt"));
            os.writeObject(user); // 将User对象写进文件
            os.flush();
            os.close();
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
        try {
            // 在反序列化之前改变username的值
            User.username = "bbb";
            
            ObjectInputStream is = new ObjectInputStream(new FileInputStream(
                    "/Users/clear/temp.txt"));
            user = (User) is.readObject(); // 从流中读取User的数据
            is.close();
            
            System.out.println("\nread after Serializable: ");
            System.out.println("username: " + user.getUsername());
            System.err.println("password: " + user.getPasswd());
            
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
}

class User implements Serializable {
    private static final long serialVersionUID = 1L;  
    
    public static String username;
    private transient String passwd;
    
    public String getUsername() {
        return username;
    }
    
    public void setUsername(String username) {
        this.username = username;
    }
    
    public String getPasswd() {
        return passwd;
    }
    
    public void setPasswd(String passwd) {
        this.passwd = passwd;
    }

}
```
输出结果
```
read before Serializable: 
username: xxx
password: 123456

read after Serializable: 
username: bbb
password: null
```