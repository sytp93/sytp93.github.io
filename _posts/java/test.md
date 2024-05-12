# 객체 생성과 파괴
## ITEM.1 생성자 대신 정적 팩터리 메서드를 고려하라
  1. 이름을 가질 수 있다.
  2. 호출될 때마다 인스턴스를 새로 생성하지는 않아도 된다.
  3. 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다. -> 잘 모르겠다. // 2024.05.12
  4. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다. -> 잘 모르겠다. // 2024.05.12
  5. 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.
     ```java
      @Getter
      @Setter
public class UserDto {
    private String id;
    private String name;
    
    public static UserDto createUser(String id, String name) {
        this.id = id;
        this.name = name;
    }
}
     ```
