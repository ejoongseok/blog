# SQL의 복잡한 로직을 애플리케이션 로직으로 이관하기(CASE WHEN)

## 문제 상황
레거시 시스템에서 SQL에 비즈니스 로직이 집중되어 있는 상황을 자주 접해볼 수 있다.   
이런 경우 SQL 쿼리가 복잡해지고, 유지보수의 어려움이 발생할 수 있다.   

## 해결 방법
SQL에 포함된 비즈니스 로직을 애플리케이션 코드로 이관하여 유지보수성을 높이고 새로운 요구사항에 빠르게 대응할 수 있도록 개선한다.

## 레거시 SQL 
- 예시: 기존 레거시 SQL 코드에서는 다양한 조건에 따라 다른 테이블에서 데이터를 조회하는 복잡한 CASE 문을 사용하고 있다.
``` SQL
SELECT 
  CASE code
  WHEN 'default' THEN select type from default_table
  WHEN 'board' THEN select type from board_table
  WHEN 'card' THEN select type from card_table

```
이와 같은 SQL 쿼리는 코드가 길어지면 복잡도가 증가하고 유지보수의 어려움을 야기할 수 있다.

## 첫번째 개선
SQL 로직을 애플리케이션 코드로 옮기는 첫 번째 시도로, 각각의 조건에 맞는 DAO 메서드를 호출하도록 변경하고 통합 테스트를 작성한다.
``` java
class Service {
  public String getTypeCode(String code) {
    Type type = getType(code);
    return type.asString();
  }

  private Type getType(String code) {
    if ("board".equals(code)) return dao.getBoardType();
    if ("card".equals(code)) return dao.getCardType();
    return dao.getDefaultType();
  }
}


class Dao {
  @SQL("select type from default_table")
  Type getDefaultType();

  @SQL("select type from board_table")
  Type getBoardType();

  @SQL("select type from card_table")
  Type getCardType();
}

class Type {
  String type;

  String asString() {
    return type;
  }
}

---- Test Code
@SpringBootTest
class ServiceTest {
  @Autowired private Service sut;

  @Test
  void getDefaultTypeTest() {
    Assertions.assertThat(sut.getTypeCode("default")).isEquals("defaultType");
  }


  @Test
  void getBoardTypeTest() {
    Assertions.assertThat(sut.getTypeCode("board")).isEquals("boardType");
  }


  @Test
  void getCardTypeTest() {
    Assertions.assertThat(sut.getTypeCode("card")).isEquals("cardType");
  }
}

```

### 첫번째 개선의 문제점
이 방법은 SQL 로직을 분리하여 애플리케이션 코드로 옮겼지만 비즈니스 로직이 `Service` 클래스에 분산되어 도메인 모델의 응집도가 낮다.   
이 로직을 테스트하려면 `Service` 레이어를 통합 테스트로 수행하거나 Stub을 이용해야 하고 단위 테스트가 어렵다.  

## 두번째 개선
두 번째 개선에서는 비즈니스 로직을 도메인 객체에 집중시켜 응집도를 높이고, 단위 테스트가 용이하도록 변경한다.
``` java
class Service {
  public String getTypeCode(String code) {
    Type type = dao.getType(code);
    return type.asString();
  }
}

class Dao {
  @SQL("""
  SELECT
    (SELECT type FROM default_table) AS defaultType,
    (SELECT type FROM board_table) AS boardType,
    (SELECT type FROM card_table) AS cardType,
    #{code} AS code
  """)
  Type getType(String code);
}

class Type {
  String defaultType;
  String boardType;
  String cardType;
  String code;

  String asString() {
    if ("board".equals(code)) return boardType;
    if ("card".equals(code)) return cardType;
    return defaultType;
  }
}


---- Test Code

@SpringBootTest
class ServiceTest {
  @Autowired private Service sut;

  // 작성한 SQL이 런타임 에러 없이 의도한대로 동작하는지 확인하기 위한 통합 테스트
  @Test
  void getDefaultTypeTest() {
    Assertions.assertThat(sut.getTypeCode("default")).isEquals("defaultType");
  }
}

class TypeTest {
  // 비즈니스 로직에 대한 테스트는 Type의 단위 테스트로 검증한다.
  @Test
  void defaultTypeAsString() {
    Assertions.assertThat(createType(DEFAULT_CODE).asString()).isEquals("defaultType");
  }


  @Test
  void boardTypeAsString() {
    Assertions.assertThat(createType(BOARD_CODE).asString()).isEquals("boardType");
  }


  @Test
  void cardTypeAsString() {
    Assertions.assertThat(createType(CARD_CODE).asString()).isEquals("cardType");
  }
}

```

### 두 번째 개선의 장점
이 방법을 통해 비즈니스 로직을 도메인 모델 객체인 `Type` 클래스에 집중시켜 응집도를 높일 수 있게 되었다.  
또한, 비즈니스 로직을 단위 테스트로 검증 할 수 있어, 테스트가 훨씬 쉽고 가벼워졌다.

### 두 번째 개선의 단점
기존에는 CASE에 걸리는 SQL만 조회되었다면, 이제는 모든 CASE에 대한 SQL을 한 번에 가져오므로 쿼리가 무거워지고 이로 인해 성능 저하가 발생할 수 있다.  

## 결론
두 번째 개선을 통해 비즈니스 로직을 도메인 객체에 집중시킴으로써 응집도를 높이고, 유지보수성을 향상시킬 수 있었다.  
다만, 쿼리 성능에 대한 추가적인 고려가 필요하며, 상황에 따라 최적의 방식을 선택하는 것이 중요하다.  

실제 업무에서는 두 번째 개선 방법을 채택했다.  
성능 문제는 캐싱, 쿼리 튜닝, 또는 인덱싱과 같은 여러 방법을 통해 개선할 수 있다.   
그러나 복잡한 코드를 유지보수하는 비용은 성능 최적화 비용보다 훨씬 비싸기 때문에 유지보수성을 우선시하는 것이 더 적합하다고 판단했다.  
앞으로 성능 문제가 발생할 경우, 추가적인 최적화 작업을 단계적으로 진행할 계획이다.  
