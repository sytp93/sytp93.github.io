---
layout: post
title: next.js and Spring-boot (3)
subtitle: 스프링부트 테스트코드 작성
cover-img: ./assets/img/path.jpg
thumbnail-img: ./assets/img/thumb.png
share-img: ./assets/img/path.jpg
tags: [project]
---

## 환경설정

먼저 test코드 작성을 위해 필요한 dependency를 설정해주어야 한다.
기존에 spring initializr로 프로젝트를 생성할 때 spring web dependency를 추가했으면 아래와 같은 dependency가 추가되어있다. 
```
testImplementation 'org.springframework.boot:spring-boot-starter-test'
```
하지만 나는 다른 dependency를 사용하기위해 아래와 같은 dependency를 추가해 주었다.
```
testImplementation("org.assertj:assertj-core:3.24.2")
```
위의 dependency를 사용하면 asserThat이라는 메소드를 사용가능하게 되는데 이 메소드가 테스트를 편하게 도와준다.

이제 내가 구현해 놓은 CRUD 기능에 대해 테스트코드를 작성해보겠다.

### 조회 테스트
```java
@Test
@DisplayName("전체 조회 테스트")
void readTest() {
	List<Charger> chargers = (List<Charger>) chargerRepository.findAll();
	assertThat(chargers).isNotNull();
}
```
JPA findAll 메소드를 통해 데이터를 select하고 해당 데이터가 null인지 아닌지를 판단하게 코드를 작성하였다.

### 생성 테스트
```java
@Test
@DisplayName("저장 테스트")
@Transactional
void saveTest() {
	ChargerDto dto = new ChargerDto();
	dto.setChargerCode("AAAA");
	dto.setChargerBrand("TEST");
	dto.setChargerName("TESTCG");
	dto.setChargerPrice("30000");
	dto.setChargerStatus("USE");
	Charger charger = ChargerMapper.mapper.dtoToEntity(dto);
	charger = chargerRepository.save(charger);

	assertThat(chargerRepository.findById((int) charger.getChargerId())).isNotNull();
}
```
DTO에 데이터를 set하고 해당 데이터를 entity로 변환 후 save하고 생성된 데이터의 id를 가져와 findById 메소드를 통해 해당 데이터가 존재하는지 파악하는 방식으로 코드를 작성하였다.

### 업데이트 테스트
```java
@Test
@DisplayName("수정 테스트")
@Transactional
void updateTest() {
	Optional<Charger> chargerOptional =  chargerRepository.findById(1);
	Charger charger = chargerOptional.get();
	ChargerDto dto = ChargerMapper.mapper.entityToDto(charger);
	dto.setChargerName("Update");
	charger = ChargerMapper.mapper.dtoToEntity(dto);
	chargerRepository.save(charger);
	chargerOptional = chargerRepository.findById((int) dto.getChargerId());
	charger = chargerOptional.get();
	assertThat(charger.getChargerName()).isEqualTo(dto.getChargerName());
}
```
findById 메소드를 통해 해당 아이디에 해당하는 데이터를 가져와서 entity에 값을 할당하고 entity를 dto로 변환 후 Name 값을 set을 통해 변환 시켜주고 다시 save를 하고 update되는 데이터와 dto에 set한 데이터 값이 같은지에 대해 비교하는 방식으로 코드로 작성하였다.


