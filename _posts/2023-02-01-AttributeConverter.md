---
publishedL: true
title: "[JPA] AttributeConverter / @Converter"
date: 2023-02-01 11:33:02 +09:00
categories: [Spring]
tags: [java, Spring, jpa, database]	
---

---

MySQL 타입에는 boolean 타입이 따로 존재하지 않는다.  
개발을 하다보면 보통 TinyInt를 사용해 0/1로 변환하거나 Char를 사용해 Y/N으로 변환하여 저장하게 되는데  
JPA를 이용하여 DB 연결을 하는 경우 Boolean형을 0 or 1 (0-false / 1-true) 로 자동으로 변환해준다.  
그런데 나는 0/1이 아닌 Y/N을 사용하고 싶다면 어떻게 해야할까?

바로 @Converter를 명시한 향변환기를 설정해주면 된다.  
형변환기는 새로운 클래스를 생성하여 AttributeConverter 인터페이스를 구현해줘야 한다.

<br/>

### AttributeConverter 인터페이스

이 인터페이스를 구현하는 클래스는 엔티티의 속성 타입을 데이터베이스의 필드에 저장되는 다른 타입으로 변환하는데 사용할 수 있다.

**Parameter type**
- X : 엔티티 속성의 타입
- Y :데이터베이스 필드에 저장될 타입

**Method**
- convertToDatabaseColumn : 엔티티 속성(X)의 타입을 데이터베이스 컬럼에 저장할 타입(Y)으로 변환
- convertToDatabaseColumn : 데이터베이스 컬럼(Y)에 저장된 타임의 데이터를 엔티티 속성의 타입(X)으로 변환

<img width="756" alt="image" src="https://user-images.githubusercontent.com/84762486/216098822-3578a62c-e823-43df-b7dd-85076ec215dd.png">  

<br/>

### @Converter

타입 변환을 위해 생성한 클래스에는 @Converter라는 어노테이션을 명시해줘야 하는데 이는 해당 클래스가 형변환 클래스임을 명시한다.

  

이 어노테이션은 autoApply라는 속성을 사용할 수 있는데 true로 설정할 경우 해당 타입에 관련된 모든 필드는 형변환을 사용하도록 설정된다.  
만약 아래 코드와 같이 Boolean 타입과 String타입을 변환해주는 클래스에 autoApply를 true로 설정한다면 모든 엔티티의 Boolean 타입은 자동으로 String 타입으로 형변환이 될것이다.

``` java
@Converter(autoApply = true)
public class BooleanToYNConverter implements AttributeConverter<Boolean, String> {..}
```

그렇다면 false를 사용하면 어떻게 될까?  
autoApply를 직접 명시해주지 않으면 자동으로 false와 같이 동작한다.  
false의 경우 `@Convert(converter = BooleanToYNConverter.class)` 를 사용하여 엔티티의 해당 속성이 이 클래스의 형변환 메소드를 사용하여 저장/조회 할 것임을 명시해주어야 한다.

이처럼 정말 쉽게 사용할 수 있지만 @Convert를 사용할 때 주의해야 할 점이 있다.  
아래와 같이 명시적인 어노테이션이 작성되어있는 필드의 경우 형변환이 적용되지 않는다.

- @Id
- @Version
- @OneToOne @OneToMany @ManyToOne @ManyToMany 과 같은 관계 속성
- @Enumerated
- @Temporal

확인하지 않고 무분별하게 사용하다가 원하는대로 작동되지 않을 수 있으니 기억해두자.

<br/>

### 형변환 클래스 예시 코드

``` java
import javax.persistence.AttributeConverter;
import javax.persistence.Converter;

@Converter
public class BooleanToYNConverter implements AttributeConverter<Boolean, String> {

    @Override
    public String convertToDatabaseColumn(Boolean attribute) {
        return (attribute != null && attribute) ? "Y" : "N";
    }

    @Override
    public Boolean convertToEntityAttribute(String yn) {
        return "Y".equalsIgnoreCase(yn);
    }
}
```

<br/>

### 형변환 클래스 사용 예시

형변환이 필요한 엔티티의 속성에 @Convert를 사용하여 형변환 클래스를 명시해주면 JPA 작업 시 해당 형변환 클래스에 구현된대로 타입이 변한되어 저장되고 조회되는 것을 확인할 수 있다.

``` java
@Column(name = "USE_YN")
@Convert(converter = BooleanToYNConverter.class)
private Boolean isUse;
```

AttributeConveter를 사용하면 기본형뿐만 아니라 Collection, enum, 클래스도 설정할 수 있으니 필요하다면 다양하게 실행해보자.