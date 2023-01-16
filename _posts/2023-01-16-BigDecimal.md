---
title: BigDecimal을 사용해보자.
date: 2023-01-17 00:33:02 +/- TTTT
categories: [Language / Java]
tags: [java, BigDecimal]	
---

---

### BigDecimal을 사용하는 이유

Float과 double 타입은 넓은 범위의 수를 빠르게 정밀한 ‘**근사치’**로 계산하도록 세심하게 설계되었다. 
구하고자 하는 정확한 결과가 나오지 않을 수 있으며 정확한 결과를 요구하는 **금융 관련 계산**에는 사용하면 안된다.

이때 사용할 수 있는 것이 BigDecimal이다.  
BigDecimal은 JAVA 내에서 정밀하게 값을 계산 / 표현할 수 있는 타입이다. 특히 금융 관련 계산 시 BigDecimal 혹은 int, long을 사용하는게 정확한 결과값을 구하는데 적합하다고 할 수 있다.

#### Effective Java에서는 아래처럼 사용하기를 권장하고 있다.
- 9자리 십진수로 표현이 가능한 경우 int 사용  
- 18자리 십진수 표현 long 사용  
- 18자리 초과하는 수 BigDecimal 사용

<br/>

### BigDecimal의 단점

그러나 BigDecimal의 경우 크게 두가지 단점을 가지고 있다.  

1. 기존 숫자 타입들처럼 사칙연산이 불가능하며 내부의 메소드를 사용하여 계산해야 하기때문에 사용이 불편하다.  
2. 정밀한 계산을 위해 속도가 느리다는 단점이 있어 성능이 중요한 개발에는 잘 고려해서 사용해야 한다.

<br/>

### BigDecimal 사용 방법
실제 사용방법을 확인해보자.
``` java
public static void main(String[] args) {
        BigDecimal bd1 = new BigDecimal("10.5");
        BigDecimal bd2 = new BigDecimal("20.5");

        // 더하기
        // 10.5 + 20.5
        BigDecimal addBd = bd1.add(bd2);
        System.out.println(addBd); //31.0

        // 빼기
        // 10.5 - 20.5
        BigDecimal minBd = bd1.subtract(bd2);
        System.out.println(minBd); // -10.0

        // 곱하기
        // 10.5 * 20.5
        BigDecimal mulBd = bd1.multiply(bd2);
        System.out.println(mulBd); //215.25

        // 나누기(0의 자리 / 반올림)
        // 10.5 / 20.5
        BigDecimal divBd = bd1.divide(bd2, 0, RoundingMode.HALF_UP);
        System.out.println(divBd); //1

        // 나누기(1의 자리 /반올림)
        // 10.5 / 20.5
        BigDecimal div2Bd = bd1.divide(bd2, 1, RoundingMode.HALF_UP);
        System.out.println(div2Bd); //0.5

        BigDecimal bd3 = new BigDecimal("10.5");
        BigDecimal bd4 = new BigDecimal("5.3");

        // 두 수의 비교
        // 10.5 < 20.5 [결과 : 작음(-1)]
        int compare1 = bd1.compareTo(bd2);
        System.out.println(compare1); // -1

        // 10.5 == 10.5 [결과 : 같음(0)]
        int compare2 = bd1.compareTo(bd3);
        System.out.println(compare2); // 0

        // 10.5 > 5.3 [결과 : 큼(1)]
        int compare3 = bd1.compareTo(bd4);
        System.out.println(compare3); // 1
}
```

<br/>

### 주의사항

#### 주의 1. BigDecimal 생성 시 파라미터는 문자열로 넘겨주자.

BigDecimal 선언 시 파라미터에 실수형 타입의 값을 넣으면 아래와 같은 경고가 뜬다.  
> Unpredictable 'new BigDecimal()' call  // 예측할 수 없는 'new BigDecimal()' 호출

실제로 실행해보면 아래와 같은 결과가 나온다.

``` java
double d = 0.1;
BigDecimal b1 = new BigDecimal(d);
BigDecimal b2 = new BigDecimal(0.1);
BigDecimal b3 = new BigDecimal(String.valueOf(d));
BigDecimal b4 = new BigDecimal("0.1");

System.out.println(b1); //0.1000000000000000055511151231257827021181583404541015625
System.out.println(b2); //0.1000000000000000055511151231257827021181583404541015625
System.out.println(b3); // 0.1
System.out.println(b4); // 0.1
```

double이나 Float형의 경우 부동 소수점 방식을 기반으로 정수보다 더 넓은 범위의 값을 표현할 수 있으나 정밀도에 문제가 있어 원하던 결과와 오차가 있을 것을 유의해야한다. 때문에, BigDecimal 파라미터값으로 실수형 타입의 값을 넘겨주는 경우 원하는 결과가 나오지 않을 수 있다. 

정확한 값을 도출해내기 위해서 BigDecimal 생성자 선언 시 꼭 문자열로 넘겨주자.  
만약 사용하던 실수형 타입을 넘겨줘야 한다면 `String.valueOf()` 를 사용하여 넘겨주자.

더 자세한 내용은 아래 글을 참고 바란다.  
[자바 BigDecimal: 정확한 실수의 표현과 부동 소수점](https://madplay.github.io/post/the-need-for-bigdecimal-in-java)



#### 주의 2. 두 BigDecimal을 나누기 할 시 RoundingMode를 지정해주자.

``` java
BigDecimal div2Bd = bd1.divide(bd2, 1, RoundingMode.HALF_UP);
BigDecimal div3Bd = bd1.divide(bd2);
```

BigDecimal에서 나누기 실행 시 RundingMode를 지정해주지 않으면 아래와 같은 경고가 뜬다.  
> 'BigDecimal.divide()' called without a rounding mode argument  

BigDecimal에서 나누기 실행 시 0.3333333333.....과 같이 무한소수 계산이 나올 수 있으므로 정확한 몫을 나타낼 수 있도록 RoundingMode를 꼭 지정해주어야한다. 만약 지정해주지 않아 정확한 몫이 계산되지 않을 시 ArithmeticException이 발생할 수 있다고 한다.

ArithmeticException - 산술, 캐스팅 또는 변환 작업에서 오류가 발생한 경우 throw되는 예외

참고 : [ArithmeticException](https://learn.microsoft.com/ko-kr/dotnet/api/system.arithmeticexception?view=net-7.0)  

나누기가 아니더라도 특정 자리수를 설정하기 위해서 `.setScale()` 이라는 함수를 사용할 수 있다.

``` java
BigDecimal bd1 = new BigDecimal("10.53");
BigDecimal ssBd2 = bd1.setScale(1, RoundingMode.UP); //소수점 1의 자리까지 올림하도록 지정
System.out.println(ssBd2); //10.6
```

<br/>
