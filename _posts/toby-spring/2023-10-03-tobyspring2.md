---
layout: post
title: 토비의 스프링 3.1 (2)
subtitle: 1장 오브젝트와 의존관계
cover-img: ./assets/img/path.jpg
thumbnail-img: ./assets/img/springboot-logo.png
share-img: ./assets/img/path.jpg
tags: [개발자로 성장하기,토비의 스프링3.1]
---

## 1.3 DAO의 확장

저번 글에서 마지막에 같은 클래스 안에서 Connection 메소드를 만들어 connection에 대한 관심사를 분리를 해주었다.

```java
public class UserDao {
    public void add(User user) throws ClassNotFoundException, SQLException {
        Connection c = getConnection(); 

        PreparedStatement ps = c.prepareStatement(
                "insert into users(id, name, password) values(?,?,?)"); 
        ps.setString(1, user.getId());
        ps.setString(2, user.getName());
        ps.setString(3, user.getPassword()); 
        ps.executeUpdate();

        ps.close();
        c.close();
    }

    private Connection getConnection() throws ClassNotFoundException,
			SQLException {
		Class.forName("com.mysql.cj.jdbc.Driver");
		Connection c = DriverManager.getConnection("jdbc:mysql://localhost:3306/users?characterEncoding=UTF-8", "root",
                "1234");
		return c;
	}
}
```

이번에는 이 connection을 아예 클래스에서 분리시켜 UserDao 클래스는 connection에 관해서 전혀 관심을 가지지 않게 만드는 과정이다.

```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

public class SimpleConnectionMaker {
    public Connection makeNewConnection() throws ClassNotFoundException, SQLException {
        Class.forName("com.mysql.cj.jdbc.Driver");
        Connection c = DriverManager.getConnection("jdbc:mysql://localhost:3306/users?characterEncoding=UTF-8", "root",
                "1234");
        return c;
    }
}
```

기존에 UserDao에 만들어 놓았던 getConnection 메소드를 SimpleConnectionMaker라는 클래스로 분리를 시키고

```java
public class UserDao {

    private SimpleConnectionMaker simpleConnectionMaker;

    public UserDao() {
        simpleConnectionMaker = new SimpleConnectionMaker();
    }

    public void add(User user) throws ClassNotFoundException, SQLException {
        Connection c = simpleConnectionMaker.makeNewConnection();

        PreparedStatement ps = c.prepareStatement(
                "insert into users(id, name, password) values(?,?,?)");
        ps.setString(1, user.getId());
        ps.setString(2, user.getName());
        ps.setString(3, user.getPassword());

        ps.executeUpdate();

        ps.close();
        c.close();
    }
}
```
UserDao에는 SimpleConnectionMaker 객체를 생성하여 필요한 메소드에서 객체와 메소드를 호출하여 사용하게 된다.

다음은 connection에 대한 확장 변경을 위해 interface화 하여 생성하는 과정이다.

### ConnectionMaekr
```java
import java.sql.Connection;
import java.sql.SQLException;

public interface ConnectionMaker {
    public Connection makeConnection() throws ClassNotFoundException, SQLException;
}

```
ConnectionMaekr 라는 interface에 makeConnection 메소드를 생성해 놓은 다음,


### DConnectionMaker
```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

public class DConnectionMaker implements ConnectionMaker{
    @Override
    public Connection makeConnection() throws ClassNotFoundException, SQLException {
        Class.forName("com.mysql.cj.jdbc.Driver");
        Connection c = DriverManager.getConnection("jdbc:mysql://localhost:3306/users?characterEncoding=UTF-8", "root",
                   "1234");
        return c;
    }
}
```

자신이 원하는 connection class를 생성한 후 해당 클래스에 interface를 상속 받는다.

```java
public class UserDao {

    private ConnectionMaker connectionMaker;

    public UserDao() {
        connectionMaker = new DConnectionMaker();
    }

    public void add(User user) throws ClassNotFoundException, SQLException {
        Connection c = connectionMaker.makeNewConnection();

        PreparedStatement ps = c.prepareStatement(
                "insert into users(id, name, password) values(?,?,?)");
        ps.setString(1, user.getId());
        ps.setString(2, user.getName());
        ps.setString(3, user.getPassword());

        ps.executeUpdate();

        ps.close();
        c.close();
    }
}
```

그렇게 되면 UserDao에서 사용되는 클래스에 connection class만 변경해주면 내가 원하는 db 연결을 설정할 수 있다.


```java
public class UserDao {

    private ConnectionMaker connectionMaker;

    public UserDao(ConnectionMaker connectionMaker) {
        this.connectionMaker = connectionMaker;
    }

    public void add(User user) throws ClassNotFoundException, SQLException {
        Connection c = connectionMaker.makeNewConnection();

        PreparedStatement ps = c.prepareStatement(
                "insert into users(id, name, password) values(?,?,?)");
        ps.setString(1, user.getId());
        ps.setString(2, user.getName());
        ps.setString(3, user.getPassword());

        ps.executeUpdate();

        ps.close();
        c.close();
    }
}
```

```java
import java.sql.SQLException;

public class UserMain {
    public static void main(String[] args) throws SQLException, ClassNotFoundException {
        ConnectionMaker connectionMaker = new DConnectionMaker();
        
        UserDao dao = new UserDao(connectionMaker);
    }
}
```

이렇게 만들어주면 이제 ConnectionMaker에 대한 관심을 클라이언트에게로 분리시켰다.

Dao의 변경없이 connection만 바꾸어서 Dao를 사용이 가능하다.

>개방 폐쇄 원칙(OCP, Open-Closed Principle) 클래스나 모듈은 확장에는 열려 있어야 하고 변경에는 닫혀있어야 한다.

### 객체지향 설계 원칙(SOLID)
SRP(The Single Responsibility Principle): 단일 책임 원칙
OCP(The Open Closed Principle): 개방 폐쇄 원칙
LSP(The Liskov Subsititution Principle): 리스코프 치환 원칙
ISP(The interface Segregation Principle): 인터페이스 분리 원칙
DIP(The Dependency inversion Principle): 의존관계 역전 원칙
