---
publishedL: true
title: "[스프링 분석-1] SpringContainer, BeanFactory, ApplicationContext"
date: 2023-03-05 18:32:02 +09:00
categories: [Framework / Spring]
tags: [Spring, Framework, SpringContainer, BeanFactory, ApplicationContext]	
---

다시 처음으로 돌아가,, 스프링 분석하기부터 시작해본다.  
기록을 마음 먹었으니, 평소에 헷갈렸던 부분 궁금했던 부분을 하나씩 공부해 나가고자 한다.  
이번에는 스프링 컨테이너에 대해 공부해보자.

<br/>  

## 스프링 컨테이너(SpringContainer)란?

그냥 컨테이너(Container) 혹은 IoC컨테이너 라고도 불리며 BeanFactory, ApplicationContext 를 통틀어 추상적으로 하는 말이다.  
스프링 컨테이너는 IoC(제어의 역전)방식으로 객체를 관리하고 이 컨테이너에 의해서 생성 및 관리되는 객체를 `Bean이라고 한다.  
스프링 컨테이너는 Bean의 구성정보를 바탕으로 의존관계를 주입해주며, 애플리케이션 구성 및 생애관리를 하는 것이다.  
스프링 컨테이너는 아래와 같은 구조로 보면 이해하기 쉽다.

![SpringContainer](https://github.com/choi2h/choi2h.github.io/assets/84762486/c3e9f4f6-30e5-4d0c-94a5-a21ff3519083){: .align-left width="50%" height="50%"}
   
그렇다면 BeanFactory는 뭐고 ApplicationContext는 뭔지 하나씩 알아보자.

<br/>

## 빈팩토리(BeanFactory)

BeanFactory는 객체의 생성, 의존관계 설정, 사용 등을 제어해주는 IoC 오브젝트이다.  
인터페이스 형태로 이를 구현한 클래스에서 `getBean()`메소드를 통해 오브젝트를 요청할 수 있다.

![BeanFactory](https://github.com/choi2h/choi2h.github.io/assets/84762486/25be78e8-0d1a-4598-900a-5980a1eb0aa2){: .align-left width="60%" height="60%"}

`getBean()`메소드의 사용방법에 대해서는 마지막에 알아보자.

<br/>

## 애플리케이션 컨텍스트(ApplicationContext)

애플리케이션 컨텍스트는 위 그림처럼 BeanFactory를 확장한 인터페이스다.  
실제 애플리케이션 컨텍스트 인터페이스에 들어가보면 많은 인터페이스를 상속받아 사용된다.

![image 3](https://github.com/choi2h/choi2h.github.io/assets/84762486/bc4e3c85-2bd1-46b5-a9ce-bf19ddb48668){: .align-left}

그 중 HierarchicalBeanFactory와 ListableBeanFactory는 아래와 같이 BeanFactory 인터페이스를 상속받은 인터페이스로,  애플리케이션컨텍스트 또한 빈팩토리를 구현했다고 볼 수 있다.  

![image 4](https://github.com/choi2h/choi2h.github.io/assets/84762486/8656c53a-52cc-4fb1-9201-1b9b8cb98115){: width="50%" height="50%"}

![image 5](https://github.com/choi2h/choi2h.github.io/assets/84762486/2e2694a7-5fd3-4f64-bfc3-61b3ff1dc35c){: width="50%" height="50%"}

  
애플리케이션 컨텍스트는 스프링에서 애플리케이션 전반에 걸쳐 모든 구성요소의 제어 작업을 담당하기 위해 생성된다.  
같이 알아본 빈팩토리 동작만 구현하는 것이 아니라 이외의 상속 인터페이스들의 기능도 한다.  
- EnvironmentCapable : 현재 프로그램이 실행중인 환경 정보(Environment)를 가지고 있는 인터페이스
- MessageSource : 국제화를 제공하는 인터페이스
- ApplicationEventPublisher : 이벤트 관련 기능을 제공하는 인터페이스
- ResourcePatternResolver : 리소스를 로드하기 위한 인터페이스

이처럼 애플리케이션컨텍스트는 빈의 생성, 의존성관리, 소멸 등을 포함하여 더 많은 작업을 실행하는 확장된 인터페이스이다.  

### ApplicationContext 동작방식

1. 설정정보를 가져와서 등록 (@Configuration 어노테이션이 작성된 클래스 가져옴)
2. 설정정보 등록 (@Bean 이 작성된 메소드들을 가져와 메소드명을 리스트로 관리)
3. getBean()을 통해 오브젝트를 요청하면 (2)에서 정리해놓은 리스트에 요청된 이름이 있는지 확인
4. 있을 시 빈을 생성하는 메소드를 호출
5. 생성된 오브젝트 응답


<br/>

### ApplicationContext의 getBean() 메소드를 사용해보자.
스프링의 빈 팩토리는 설정정보를 바탕으로 오브젝트를 생성하고, 사용할 관계를 맺어주는 등의 동작을 한다.  
설정정보를 생성하는 방식은 XML, Annotation 방식이 있으며, 우리는 Annotaion 방식으로 테스트 해볼 것이다.

#### @Configuration
설정정보를 작성하고자 하는 클래스에 @Configuration 어노테이션을 작성해주면 된다.  
해당 클래스가 빈 팩토리를 위해 오브젝트 설정을 담당하는 클래스라고 명시 하는 것이다.  

#### @Bean
@Configuration이 작성된 클래스의 내부 메소드를 작성한다.  
만들고자 하는 오브젝트를 반환하는 메소드를 작성하면 된다.

``` java
@Configuration
public class TestConfiguration {

    @Bean
    public TestBean testBean(
) {
        return new TestBean();
    }

}
```

이처럼 Annotation을 사용한 설정정보는 ApplicationContext를 구현한 AnnotationConfigApplicationContext클래스를 사용하여 가져올 수 있다.  
아래 처럼 AnnotationConfigApplicationContext를 생성한 후 getBean()메소드에 인자로 @Bean 어노테이션을 작성한 메소드명을 입력해주면 인스턴스화된 빈을 가져올 수 있다.

``` java
public class ConfigurationTest {

    public static void main(String[] args) {
        ApplicationContext applicationContext =
                new AnnotationConfigApplicationContext(TestConfiguration.class);

        TestBean testBean = (TestBean) applicationContext.getBean("testBean");
        System.out.println(testBean);
    }
}
```

상세로그를 찍어보면 아까 봤던 DefaultListableBeanFactory를 통해서 빈을 가져오는 것을 확인할 수 있다.
![image](https://github.com/choi2h/choi2h.github.io/assets/84762486/3cb9afc9-7e90-4690-91bb-1edde5f10fc2)
