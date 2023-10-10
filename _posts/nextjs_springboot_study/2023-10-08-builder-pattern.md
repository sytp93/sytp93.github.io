---
layout: post
title: next.js and Spring-boot (4)
subtitle: dto to entity를 위한 Builder Pattern 적용
cover-img: ./assets/img/path.jpg
thumbnail-img: ./assets/img/thumb.png
share-img: ./assets/img/path.jpg
tags: [project]
---

## Builder Pattern
빌더 패턴이란 무엇인가? 아래는 위키피디아에서 설명하는 빌더패턴의 정의이다.<br>
The builder pattern is a design pattern designed to provide a flexible solution to various object creation problems in object-oriented programming. The intent of the builder design pattern is to separate the construction of a complex object from its representation. It is one of the Gang of Four design patterns. <br>
**출처 : [wikipedia](https://en.wikipedia.org/wiki/Builder_pattern)**

번역하면 __'빌더 패턴은 객체 지향 프로그래밍의 다양한 객체 생성 문제에 대한 유연한 솔루션을 제공하기 위해 설계된 디자인 패턴입니다. 빌더 디자인 패턴의 목적은 복잡한 객체의 구성과 표현을 분리하는 것입니다. Gang of Four 디자인 패턴 중 하나입니다.'__ 이라고 한다.

스프링부트에서는 Builder Annotation을 지원해주고 있어서 매우 간단하게 생성할 수 있지만 직접 구현과 어노테이션 구현의 차이점을 알기위해 먼저 직접 구현해보았다.

아래는 코드에 Builder Pattern을 직접 구현하여 코드에 적용한 내용이다. 먼저 기존 entity에 inner class로 Builder class를 생성해 준다.
```java
@Entity
@Getter
@Table(name = "chargers")
@NoArgsConstructor
@AllArgsConstructor
public class Charger {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private long chargerId;
    private String chargerCode;
    private String chargerName;
    private String chargerPrice;
    private String chargerBrand;
    private String chargerStatus;

    public static class Builder{
        private long chargerId;
        private String chargerCode;
        private String chargerName;
        private String chargerPrice;
        private String chargerBrand;
        private String chargerStatus;

        public Builder chargerId(long chargerId) {
            this.chargerId = chargerId;
            return this;
        }

        public Builder chargerCode(String chargerCode) {
            this.chargerCode = chargerCode;
            return this;
        }

        public Builder chargerName(String chargerName) {
            this.chargerName = chargerName;
            return this;
        }

        public Builder chargerPrice(String chargerPrice) {
            this.chargerPrice = chargerPrice;
            return this;
        }

        public Builder chargerBrand(String chargerBrand) {
            this.chargerBrand = chargerBrand;
            return this;
        }

        public Builder chargerStatus(String chargerStatus) {
            this.chargerStatus = chargerStatus;
            return this;
        }

        public Charger build() {
            Charger charger = new Charger();
            charger.chargerId = chargerId;
            charger.chargerCode = chargerCode;
            charger.chargerName = chargerName;
            charger.chargerPrice = chargerPrice;
            charger.chargerBrand = chargerBrand;
            charger.chargerStatus = chargerStatus;
            return charger;
        }
    }

}

```
그 다음 사용할 메소드에서 Charger를 생성할때 Builder class를 호출해서 값을 입력해주면 된다.
```java
@Service
public class ChargerService {
	@Transactional
    public void createCharger(ChargerDto chargerDto) {
        Charger charger = new Charger.Builder()
                .chargerCode(chargerDto.getChargerCode())
                .chargerBrand(chargerDto.getChargerBrand())
                .chargerName(chargerDto.getChargerName())
                .chargerPrice(chargerDto.getChargerPrice())
                .chargerStatus(chargerDto.getChargerStatus())
                .build();
        chargerRepository.save(charger);
    }
}
```

직접 구현해보았을때, 간단하게 dto로 client로 부터 받아온 데이터를 entity로 변환할 수 있었다. 다음으론 Builder 어노테이션을 사용하여 구현해 보겠다.
Builder 어노테이션을 사용할 경우 코드가 매우 간단해진다. 먼저 enitity에 내가 원하는 생성자 메소드를 생성해준다.

```java
@Entity
@Getter
@Table(name = "chargers")
@NoArgsConstructor
@AllArgsConstructor
public class Charger {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private long chargerId;
    private String chargerCode;
    private String chargerName;
    private String chargerPrice;
    private String chargerBrand;
    private String chargerStatus;

    @Builder
    public Charger(String chargerCode, String chargerName, String chargerPrice, String chargerBrand, String chargerStatus) {
        this.chargerCode = chargerCode;
        this.chargerName = chargerName;
        this.chargerPrice = chargerPrice;
        this.chargerBrand = chargerBrand;
        this.chargerStatus = chargerStatus;
    }
}
```

그리고 해당 생성자 메소드에 빌더 어노테이션만 붙혀주면 코드가 끝이난다. 그 다음 사용할 메소드에서 호출만 해주면 실행이 된다.

```java
@Service
public class ChargerService {
	@Transactional
    public void createCharger(ChargerDto chargerDto) {
        Charger charger = new Charger(chargerDto.getChargerCode()
                , chargerDto.getChargerName()
                , chargerDto.getChargerPrice()
                , chargerDto.getChargerBrand()
                , chargerDto.getChargerStatus());
        chargerRepository.save(charger);
    }
}
```

눈에 띄게 코드가 간단해진것을 볼 수 있다. 롬복(lombok) 어노테이션을 사용한다는게 매우 편하고 생산성이 올라가는 일이기는 하지만 필요한 지식을 얻지 못하게 하는데 안좋은쪽으로 도움을 주는 기능인 것 같다.

빌더 패턴을 다른 방식으로 적용할 수 있는것을 알게 되어서 기록을 남긴다.

```java
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
public class ChargerDto {
    private long chargerId;
    private String chargerCode;
    private String chargerName;
    private String chargerPrice;
    private String chargerBrand;
    private String chargerStatus;


    @Builder
    public Charger toUpdateEntity () {
        return Charger.builder()
                .chargerId(chargerId)
                .chargerName(chargerName)
                .chargerCode(chargerCode)
                .chargerPrice(chargerPrice)
                .chargerBrand(chargerBrand)
                .chargerStatus(chargerStatus)
                .build();
    }
}
```

entity에 생성자 메소드를 생성하고, 그 다음 DTO에 entity를 리턴해주는 메소드 하나를 생성한 뒤 그 안에 entity.builder를 리턴해준다.
이렇게 만들어 놓게되면 아래와 같이 가독성도 좋아지고 필요할때 번거롭게 생성해주지 않아도 되고 해당 메소드를 호출해주면 된다.
```java
@Service
public class ChargerService {
	@Transactional
    public void createCharger(ChargerDto chargerDto) {
        Charger charger = chargerDto.toCreateEntity();
        chargerRepository.save(charger);
    }
}
```

