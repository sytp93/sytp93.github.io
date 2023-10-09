---
layout: post
title: 토비의 스프링 3.1 (2)
subtitle: 1장 오브젝트와 의존관계
cover-img: ./assets/img/path.jpg
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

## 1.5 스프링의 IoC

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
@Configuration - 오브젝트 설정을 담당하는 클래스 설정
@Bean 오브젝트를 만들어주는 메소드


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

