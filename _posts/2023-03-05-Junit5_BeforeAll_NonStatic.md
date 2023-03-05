---
publishedL: true
title: "@BeforeAll 테스트 메소드를 비정적으로 만들고 싶다!!"
date: 2023-03-05 18:32:02 +09:00
categories: [FrameWork / Spring]
tags: [java, BigDecimal]	
---


개인 프로젝트에서 테스트 클래스 진행 중 JUnitException이 발생했다.  
@BeforeAll 클래스를 정적클래스로 사용하지 않아서 발생한 예외였다.

<img width="1299" alt="image" src="https://user-images.githubusercontent.com/84762486/222952782-f9c6ee2a-a0b7-4e56-8aab-0b5026c2ce3b.png">

@BeforeAll과 @AfterAll 어노테이션은 왜 정적 메소드로 실행이 되어야 할까?

<br/>

### 메소드 생명주기

기존 Junit 1-4까지는 클래스 실행 시 인스턴스가 만들어졌다.  
그러나 Junit5는 각 테스트 메서드를 실행하기 전에 새 인스턴스를 만든다. 이를 **메소드 생명주기**라고 한다.  
메소드 생명주기로 작동하게 되면 각 메서드마다 각각의 인스턴스를 생성해서 사용하기 때문에 테스트별로 완전히 독립적인 환경에서 동작할 수 있게 된다.

그래서 Junit5의 @BeforeAll이나 @AfterAll은 static으로 만들어져야 한다는 조건이 있는데  
이는 한 테스트 클래스 내의 여러 테스트에서 사용할 수 있는 공통적인 테스트 인스턴스를 만들기 위함이다.

- @BeforeAll : 해당 테스트 클래스가 시작되기 전에 한번 실행된다.
- @AfterAll : 해당 테스트 클래스가 끝나기 전에 한번 실행된다.


<br/>

다른 테스트 메소드에서도 같은 값을 쓰기 위해 전역변수를 초기화해주고 싶은데,,  
@BeforeAll 메소드를 static으로 선언하게 되면 전역변수도 static변수로 선언해야 한다.

음,,정적변수로 선언하고싶지 않은데 정적메소드로 사용하고싶지 않다면 어떻게 해야할까?  
해당 클래스를 클래스단위로 인스턴스가 생성되도록 설정하면 된다.

<br/>

### @TestInstance

이 어노테이션은 인스턴스의 생명주기를 설정한다.  
기본적으로 Junit5 테스트 클래스는 `@TestInstance(LifeCycle.PER_METHOD)`가 적용되어있다고 보면 된다.


**적용 가능한 생명주기**
- LifeCycle.PER\_METHOD  
    메소드 단위로 인스턴스가 생성된다. Junit5에서는 이 설정을 기본으로 한다.
- LifeCycle.PER\_CLASS  
    클래스단위로 인스턴스가 생성되며 기존 Junit 버전 1에서 4까지의 동작과 유사하다.

우리는 메소드 단위가 아닌 클래스단위로 생명주기를 바꿔서 클래스가 실행될 때 인스턴스가 생성되도록 해주자.

<br/>

### 예제 코드  
``` java
@Transactional
@SpringBootTest
@TestInstance(TestInstance.Lifecycle.PER_CLASS)
class GetEmployeeInfoQueryTest {

    @Autowired
    private GetEmployeeInfoQuery getEmployeeInfoQuery;

    private Long employeeId;

    @BeforeAll
    public void insertEmployee() {
        ...
    }

```  

이처럼 테스트 클래스 위에 `@TestInstance(LifeCycle.PER_CLASS)` 를 붙여주면 static을 붙이지 않고도 사용할 수 있게된다.

테스트 결과  
<img width="502" alt="image" src="https://user-images.githubusercontent.com/84762486/222953061-65d1a623-e41b-4332-aaa3-37beecec6d16.png">
