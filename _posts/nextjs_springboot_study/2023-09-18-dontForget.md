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

저번 포스트에서 mapstruct를 사용하다 어려움이 있어서 단순 for문으로 처리를 했는데, 다시 mapstruct 적용을 시도 해보았다.
그리고 _No property named "chargerId" exists in source parameter(s). Did you mean "null"?_ 해결했다.
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
그리고 확인해보니 mapstruct 방식에는 Mapper에 Mapping 한 target과 source의 getter가 존재해야 하고, DTO에 생성자가 존재해야 
_No property named "chargerId" exists in source parameter(s). Did you mean "null"?_ 
이 오류가 발생하지 않는다. 

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
기본적인 동작은 생각안하고 무작정 가져다 붙이기만 해서 몇시간이나 씨름했던 문제였다. mapper가 빌드 됐을때 생성되는 MapperImpl이라는 클래스가 있다.
```java
public class ChargerMapperImpl implements ChargerMapper {
    public ChargerMapperImpl() {
    }

    public ChargerDTO chargerDTO(Charger charger) {
        if (charger == null) {
            return null;
        } else {
            ChargerDTO chargerDTO = new ChargerDTO();
            chargerDTO.setChargerId(charger.getChargerId());
            chargerDTO.setChargerCode(charger.getChargerCode());
            chargerDTO.setChargerName(charger.getChargerName());
            chargerDTO.setChargerPrice(charger.getChargerPrice());
            chargerDTO.setChargerBrand(charger.getChargerBrand());
            chargerDTO.setChargerStatus(charger.getChargerStatus());
            return chargerDTO;
        }
    }

        public List<ChargerDTO> entityToDtoList(List<Charger> charger) {
        if (charger == null) {
            return null;
        } else {
            List<ChargerDTO> list = new ArrayList(charger.size());
            Iterator var3 = charger.iterator();

            while(var3.hasNext()) {
                Charger charger1 = (Charger)var3.next();
                list.add(this.chargerDTO(charger1));
            }

            return list;
        }
    }
}
```
보면 entity를 받아서 반복문을 돌려 entity의 데이터를 get하고 그 데이터를 Dto에 Set하는 구조이다. 그러니 당연하게 DTO에는 Setter가 존재해야하고, entity에는 Getter가 존재해야 한다. 그리고 해당 파라미터를 받을 생성자도 당연히 존재해야 하는 것이고, 그리고 원인은 모르겠지만 entity에 table 맵핑을 해줘야한다.
해당 라이브러리의 동작 방식을 생각하지 않고 무작정 가져다쓰는 습관은 고쳐야 된다. 돌아간다고 다가 아니고 어떻게 돌아가는지를 알아야 이슈 트래킹이 가능하기 때문이다.
이 문제만해도 내가 이런 기본적인 구조만 생각을 해봤으면 금방 원인을 찾고 해결했을 것이다. 코딩을 하면서 생각을 하자.

계속 같은 문제가 발생한다. 결국에는 dto와 entity에 @Builder 어노테이션도 붙어있어야 오류없이 작동이 됐다.
