---
layout: post
title: 토비의 스프링 3.1 (3)
subtitle: 1장 오브젝트와 의존관계
cover-img: ./assets/img/background/gray.png
thumbnail-img: ./assets/img/springboot-logo.png
share-img: ./assets/img/path.jpg
tags: [개발자로 성장하기,토비의 스프링3.1]
---

## 1.4 제어의 역전 (IoC)

### 팩토리 클래스의 생성
객체의 생성 방법을 결정하고 만들어진 오브젝트를 리턴해주는 것
추상 팩토리 패턴, 팩토리 메소드 패턴과는 다른 것

저번 포스트에서 아래와 같이 main 메소드에 connection을 만들고 UserDao에 connection을 인자 값으로 넘겨주는 방식으로 connection 과 UserDao를 분리시켰었다. 이번 포스트에서는 main메소드에 부과된 책임을 팩토리 클래스를 생성하여 그 책임을 분리시켜 주었다.

```java
public class UserMain {
    public static void main(String[] args) throws SQLException, ClassNotFoundException {
        ConnectionMaker connectionMaker = new DConnectionMaker();
        
        UserDao dao = new UserDao(connectionMaker);
    }
}
```

#### factory class
```java
public class DaoFactory {
    public UserDao userDao() {
        ConnectionMaker connectionMaker = new DConnectionMaker();
        UserDao userDao = new UserDao(connectionMaker);
        return userDao;
    }
}
```

#### main class
```java
public class UserMain {
    public static void main(String[] args) throws SQLException, ClassNotFoundException {
        UserDao dao = new DaoFactory().userDao();
    }
}
```
자, 위와 같이 커넥션과 dao를 생성하는 역할을 팩토리 클래스가 해주면서 main에서는 다시 dao만 생성해주면 되게 코드가 변화하였다.

제어의역전(IoC) 자신은 존재하고 있고 자신을 필요한 곳에서 호출해서 사용한다. 다만 자신은 언제 어떻게 사용될지를 모른다.

## 1.5 스프링의 IoC (Inversion of Control)

이번에는 스프링 frame-work를 사용하여 IoC를 구현해보겠다.

### Bean Factory 생성

```java
@Configuration
public class DaoFactory {

    @Bean
    public UserDao userDao() {
        return new UserDao(connectionMaker());
    }

    @Bean
    public ConnectionMaker connectionMaker() {
        return new DConnectionMaker();
    }
}
```
@Configuration - 오브젝트 설정을 담당하는 클래스 설정<br>
@Bean 오브젝트를 만들어주는 메소드, 스프링 IoC 방식으로 관리하는 오브젝트


```java
public class UserMain {
    public static void main(String[] args) throws SQLException, ClassNotFoundException {
        ApplicationContext context =
                new AnnotationConfigApplicationContext(DaoFactory.class);

        UserDao dao = context.getBean("userDao", UserDao.class);
    }
}
```

어플리케이션컨텍스트 생성 후 컨텍스트에서 userDao Bean을 호출해준다.


## 1.7 의존관계 주입 (DI, Dependency Injection)

### 의존관계 주입

두 개의 클래스를 연결해주는 제 3의 클래스를 사용하는 것

```java
public class UserDao {
    private ConnectionMaker connectionMaker;

    public UserDao(ConnectionMaker simpleConnectionMaker) {
        this.connectionMaker = simpleConnectionMaker;
    }
}
```

위의 코드가 UserDao와 ConnectionMaker와의 의존관계가 주입되는 코드이다.

DB connection 카운팅을 위한 코드를 DI 방식으로 구현

### CountingConnectionMaker

```java
public class CountingConnectionMaker implements ConnectionMaker{
    int counter = 0;
    private ConnectionMaker realConnectionMaker;

    public CountingConnectionMaker(ConnectionMaker realConnectionMaker){
        this.realConnectionMaker = realConnectionMaker;
    }

    @Override
    public Connection makeConnection() throws ClassNotFoundException, SQLException {
        this.counter++;
        return realConnectionMaker.makeConnection();
    }

    public int getCounter() {
        return this.counter;
    }
}
```

### CountingDaoFactory
```java
@Configuration
public class CountingDaoFactory {
    @Bean
    public UserDao userDao() {
        return new UserDao(connectionMaker());
    }

    @Bean
    private ConnectionMaker connectionMaker() {
        return new CountingConnectionMaker(realConnectionMaker());
    }

    @Bean
    private ConnectionMaker realConnectionMaker() {
        return new DConnectionMaker();
    }
}
```

### main
```java
public class UserMain {
    public static void main(String[] args) throws SQLException, ClassNotFoundException {

        ApplicationContext context =
                new AnnotationConfigApplicationContext(CountingDaoFactory.class);

        UserDao dao = context.getBean("userDao", UserDao.class);
            CountingConnectionMaker ccm = context.getBean("connectionMaker", CountingConnectionMaker.class);
    }
}
```

## 수정자 메소드를 이용한 DI 주입

DI를 하는 방법 중 한가지이다.

### DAO
```java
public class UserDao() {
    private ConnectionMaker connectionMaker;

    public void setConnectionMaker(ConnectionMaker connectionMaker) {
        this.connectionMaker = connectionMaker;
    }
}
```

### Factory
```java
@Configuration
public class CountingDaoFactory {
    @Bean
    public UserDao userDao() {
        UserDao userDao = new UserDao();
        userDao.setConnectionMaker(connectionMaker());
        return userDao;
    }
}
```