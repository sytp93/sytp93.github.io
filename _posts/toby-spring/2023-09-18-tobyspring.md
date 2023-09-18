---
layout: post
title: 토비의 스프링 3.1 (1)
subtitle: 1장 오브젝트와 의존관계
cover-img: ./assets/img/path.jpg
thumbnail-img: ./assets/img/springboot-logo.png
share-img: ./assets/img/path.jpg
tags: [개발자로 성장하기,토비의 스프링3.1]
---

## 1.1 초난감 DAO

JDBC 작업의 일반적인 순서

1. DB 연결을 위한 Connection을 가져온다.
2. SQL을 담은 Statement를 만든다.
3. 만들어진 Statement를 실행한다.
4. 조회의 경우 SQL쿼리의 실행 결과를 ResultSet으로 받아서 정보를 저장할 오브젝트에 옮겨준다.
5. 작업 중에 생성된 Connection, Statement, ResultSet 같은 리소스는 작업을 마친 후 반드시 닫아준다.
6. JDBC API가 만들어내는 예외를 잡아서 직접 처리하거나, 메소드에 throws를 선언해서 예외가 발생하면 메소드 밖으로 던지게 한다.

사용자 정보를 DB에 넣고 관리할 수 있는 DAO 클래스 생성
```java
public class UserDao {
    public void add(User user) throws ClassNotFoundException, SQLException {
        Class.forName("com.mysql.cj.jdbc.Driver");
        Connection c = DriverManager.getConnection("jdbc:mysql://localhost:3306/users?characterEncoding=UTF-8", "root",
                "1234");

        PreparedStatement ps = c.prepareStatement(
                "insert into users(id, name, password) values(?,?,?)");
        ps.setString(1, user.getId());
        ps.setString(2, user.getName());
        ps.setString(3, user.getPassword());

        ps.executeUpdate();

        ps.close();
        c.close();
    }


    public User get(String id) throws ClassNotFoundException, SQLException {
        Class.forName("com.mysql.cj.jdbc.Driver");
        Connection c = DriverManager.getConnection("jdbc:mysql://localhost:3306/users?characterEncoding=UTF-8", "root",
                "1234");
        PreparedStatement ps = c
                .prepareStatement("select * from users where id = ?");
        ps.setString(1, id);

        ResultSet rs = ps.executeQuery();
        rs.next();
        User user = new User();
        user.setId(rs.getString("id"));
        user.setName(rs.getString("name"));
        user.setPassword(rs.getString("password"));

        rs.close();
        ps.close();
        c.close();

        return user;
    }

}
```

주의할 점은 현재 나는 mysql-connector-j 8.0.33을 사용하고 있기에
```java
Class.forName("com.mysql.jdbc.Driver");
Class.forName("com.mysql.cj.jdbc.Driver");
```
교재와의 버전이 다르기에 최신 버전의 mysql-connector-j를 사용하면 deprecated 오류가 발생할 것이다. 그래서 아래와 같이 바꿔주어야 한다.

그 다음으로는
```java
Connection c = DriverManager.getConnection("jdbc:mysql://localhost:3306/users?characterEncoding=UTF-8", "root", "1234");
```
위 부분은 자신의 mysql 서버와 name, pw로 변경해줘야 한다.


## 1.2 DAO의 분리

### 관심사의 분리
> 개발자가 객체를 설계할 때 가장 염두에 둬야 할 사항은 바로 미래의 변화를 어떻게 대비할 것인가이다.
> 프로그래밍의 기초 개념 중에 관심사의 분리(Separation of Concerns)라는 게 있다. 이를 객체지향에 적용해보면, 관심이 같은 것끼리는
하나의 객체 안으로 또는 친한 객체로 모이게 하고, 관심이 다른 것은 가능한 한 따로 떨어져서 서로 영향을 주지 않도록 분리하는 것이라고 생각할 수 있다.


