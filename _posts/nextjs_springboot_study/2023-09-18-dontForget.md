---
layout: post
title: next.js and Spring-boot (2-1)
subtitle: Entity To DTO (SpringBoot) 보완
cover-img: ./assets/img/path.jpg
thumbnail-img: ./assets/img/springboot-logo.png
share-img: ./assets/img/path.jpg
tags: [project]
---

# JPA 사용을 위한 entity를 DTO 객체로 전달 2

[next.js and Spring-boot (2)](https://sytp93.github.io/2023-09-17-third/)

저번 포스트에서 mapstruct를 사용하다 어려움이 있어서 단순 for문으로 처리를 했는데, 복기를 해보니 매우 바보같은 방법이였다.
그리고 _No property named "chargerId" exists in source parameter(s). Did you mean "null"?_ 
해당 오류는 내가 매우 바보 같았다. 

```java
@Entity
@Getter
@Table(name = "Chargers")
public class Charger {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private long chargerId;
    private String chargerCode;
    private String chargerName;
    private String chargerPrice;
    private String chargerBrand;
    private String chargerStatus;
}

// ===== DTO ======
@Data
@Getter
@Setter
@NoArgsConstructor
public class ChargerDTO {
    private long chargerId;
    private String chargerCode;
    private String chargerName;
    private String chargerPrice;
    private String chargerBrand;
    private String chargerStatus;

    public ChargerDTO(long chargerId, String chargerCode, String chargerName, String chargerPrice, String chargerBrand, String chargerStatus) {
        this.chargerId = chargerId;
        this.chargerCode = chargerCode;
        this.chargerName = chargerName;
        this.chargerPrice = chargerPrice;
        this.chargerBrand = chargerBrand;
        this.chargerStatus = chargerStatus;
    }
}

```
내가 작성한 entity 코드이다. 오류가 발생할때 당시의 코드에는 @Table 테이블을 매핑해주는 어노테이션이 존재하지 않아서 해당 오류가 발생했다.
server log에 JPA 쿼리를 실행시키는 로그만 찍었더라면 어떤 문제인지를 금방 파악했을텐데... 그것을 찍지 않아서 발생한 매우 어리석은 실수였다. 
이걸 가지고 몇시간동안 씨름했다니 참... 내가 바보 같았다.
JPA Query log를 띄우는 것은 다른 분이 조언을 해주셨던 부분이였는데 내가 놓치고 있다가 이렇게 업보를 치렀다.
결국 그런 로그에 대한 중요성을 뼛속 깊숙이 깨달았다.

어쨌든 Mapstruct를 사용하여 entity to dto, dto to entity를 구현할 수 있게 되었다.

```java
@Mapper
public interface ChargerMapper {

    ChargerMapper mapper = Mappers.getMapper(ChargerMapper.class);

    @Mapping(target = "chargerId", source = "chargerId")
    @Named("v2")
    ChargerDTO entityToDto(Charger charger);

    @IterableMapping(qualifiedByName = "v2")
    List<ChargerDTO> entityToDtoList(List<Charger> charger);

    Charger dtoToEntity(ChargerDTO chargerDTO);

}
```

위와 같이 mapper interface를 구현하고

```java
@Service
public class ChargerService {

    @Autowired
    private ChargerRepository chargerRepository;
    
    public List<ChargerList> getAllChargers() {

        List<Charger> chargers = (List<Charger>) chargerRepository.findAll();
        List<ChargerDTO> chargerLists = ChargerMapper.mapper.entityToDtoList(chargers);
        return chargerLists;
    }
}
```

Service에서 데이터를 find하고 해당 리스트를 DTO 리스트에 담는 단순조회를 구현하였다.
다음부터는 로그 확인을 습관 시 하자, 로그에 답이 있다.

다시 확인해보니 내가 사용하고 있는 방식에는entity에 Table 매핑을 하고, Mapper에 Mapping 한 target과 source의 getter가 존재해야 하고, DTO에 생성자가 존재해야 
_No property named "chargerId" exists in source parameter(s). Did you mean "null"?_ 
이 오류가 발생하지 않는다. 

