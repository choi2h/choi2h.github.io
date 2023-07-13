---
layout: post
title: "[코딩테스트] 프로그래머스 과제 진행하기"
author: hwa
categories: CodingTest
tags: Java, CodingTest
---

### 문제설명

과제를 받은 루는 다음과 같은 순서대로 과제를 하려고 계획을 세웠습니다.

- 과제는 시작하기로 한 시각이 되면 시작합니다.
- 새로운 과제를 시작할 시각이 되었을 때, 기존에 진행 중이던 과제가 있다면 진행 중이던 과제를 멈추고 새로운 과제를 시작합니다.
- 진행중이던 과제를 끝냈을 때, 잠시 멈춘 과제가 있다면, 멈춰둔 과제를 이어서 진행합니다.
    - 만약, 과제를 끝낸 시각에 새로 시작해야 되는 과제와 잠시 멈춰둔 과제가 모두 있다면, 새로 시작해야 하는 과제부터 진행합니다.
- 멈춰둔 과제가 여러 개일 경우, 가장 최근에 멈춘 과제부터 시작합니다.

과제 계획을 담은 이차원 문자열 배열 `plans`가 매개변수로 주어질 때, 과제를 끝낸 순서대로 이름을 배열에 담아 return 하는 solution 함수를 완성해주세요.


적당히 여러 조건에만 맞춰서 작성하면 되는 문제이다.  
거의 모든 조건을 지문에서 알려줬기 때문에 문제없이 풀거라 생각했다,,


### 조건

1. 과제는 _시작하기로 한 시간에 반드시 시작되어야 한다._
2. 새로운 과제를 시작해야할 시간이 되었을 때, _기존에 진행하기로 한 과제는 멈추고_ 새로운 과제를 시작해야 한다.
3. 진행중이던 과제가 끝나고, 다음 과제까지 시간이 남았을 때 _가장 최근에 하다가 멈춘 과제_ 를 마저 진행한다.

<br/>

지문과 거의 동일한 조건이다.  
처음에는 LocalTime을 사용해서 시간을 비교하면 간단할거라 생각하고 코드를 작성했다.  
밀린 과제는 가장 최근에 진행되었던 과제 먼저 진행해야 하므로 LIFO구조의 Stack을 사용하여 저장했다.  
아래 코드가 처음에 작성했던 코드이다.


### 문제 풀이 (실패)

```java
//-- 완료된 과제의 이름을 담을 리스트
List<String> finishList = new ArrayList<>();
//-- 중간에 밀린 과제를 담을 리스트
Stack<String[]> delayPlans = new Stack<>();

public String[] solution(String[][] plans) {
   //-- 시간순으로 오름차순 정렬
    Arrays.sort(plans, new Comparator<String[]>() {
        @Override
        public int compare(String[] o1, String[] o2) {
            LocalTime time1 = convertToLocalTime(o1[1]);
            LocalTime time2 = convertToLocalTime(o2[1]);
            return time1.isBefore(time2) ? -1 : time1.equals(time2) ? 0 : 1;
        }
    });

   //-- 진행중인 현재시간을 저장할 now 변수 선언
    LocalTime now;
    for(int i=0; i<plans.length-1; i++) {
        //-- 바로 진행할 과제
        String[] plan = plans[i];
        //-- 다음으로 진행될 과제
        String[] nextPlan = plans[i + 1];

        //-- 바로 진행할 과제의 시작시간을 지금으로 설정
        now = convertToLocalTime(plan[1]);
        //-- 다음으로 진행할 과제의 시작시간 할당
        LocalTime nextStartTime = convertToLocalTime(nextPlan[1]);

        //-- 현재 진행 plan 과제를 처리할 수 있을만큼 처리한 후의 시간 받아옴
        now = getFinishedPlanTime(plan, nextStartTime, now);
        System.out.println("@@@ now = " + now);

        //-- 다음 과제의 시작시간이 안됐을 때 밀린 과제가 있으면 밀린 과제 처리
        //-- 다음 과제의 시작시간이 안됐다면 밀린 과제를 계속 처리 할 수 있도록 while문 사용
        while(now.isBefore(nextStartTime) && delayPlans.size() > 0) {
            System.out.println("@@@ DelayPlan start.");
            //-- 밀린과제를 담아놓은 stack에서 가장 마지막에 넣은 과제 가져와서 진행 과제를 뜻하는 plan 변수에 할당
            plan = delayPlans.pop();
            //-- 지금 진행한 plan 과제를 처리할 수 있을만큼 처리한 후의 시간 받아옴
            now = getFinishedPlanTime(plan, nextStartTime, now);
            System.out.println("@@@ now = " + now);
            System.out.println("@@@ DelayPlan end.");
        }
    }

    //-- 마지막으로 시작하는 과제는 시간 제한이 없으므로 한번에 처리
    finishList.add(plans[plans.length-1][0]);
    //-- 마지막 과제를 처리하고도 남은 밀린 과제 순차적으로 처리
    while (delayPlans.size() > 0) {
        finishList.add(delayPlans.pop()[0]);
    }
    
    //-- 리스트에 담아놓은 과제 완료 순서를 배열에 할당
    String[] answer = new String[finishList.size()];
    int index = 0;
    for(String name : finishList) {
        answer[index++] = name;
    }

    return answer;
}

//-- 과제를 할 수 있는만큼 진행하고 진행 완료시간을 알려줌
private LocalTime getFinishedPlanTime(String[] plan, LocalTime nextStartTime, LocalTime now) {
    System.out.println("============= start ===============");

    //-- 현재 과제 처리 시간
    long playTime = Integer.parseInt(plan[2]);
    //-- 현재 과제의 예상 완료 시간
    LocalTime endTime = now.plusMinutes(playTime);

    System.out.println("NAME:" + plan[0] + " now=" + now + " nextStartTime=" + nextStartTime);
    System.out.println("NAME:" + plan[0] + " playTime=" + playTime + " endTime=" + endTime);

    //-- 예상 과제 완료 시간이 다음 과제 시작시간보다 이전이거나 같으면
    if (endTime.isBefore(nextStartTime) || endTime.equals(nextStartTime)) {
        //-- 현재 과제는 마무리 가능하므로 완료 리스트에 추가
        finishList.add(plan[0]);
        //-- 과제 마무리 시간으로 시간 이동
        now = endTime;
    } 
    //-- 예상 과제 완료시간이 다음 과제 시작시간보다 이후일 경우
    //-- 현재 과제를 마무리하지 못할것으로 판단
    else {
       //-- 다음 과제 시작시간과 예상 과제 완료시간의 시간차이를 구함
        long duration = Duration.between(nextStartTime, endTime).toMinutes();
        //-- 다음 과제 시작시간까지만 처리하고 남은 시간으로 과제 처리시간 수정
        plan[2] = String.valueOf(duration);
        System.out.println("NAME:" + plan[0] + " Need time : " + duration);
        //-- 밀린과제에 현재 과제 정보 추가
        delayPlans.push(plan);
        //-- 다음 과제 시작시간까지 시간을 다 썼으므로 다음 과제 시작시간으로 이동
        now = nextStartTime;
    }

    System.out.println("============= end ===============");
    //-- 현재 과제를 처리할 수 있을만큼 처리하고 난 시간 응답
    return now;
}

//-- String타입의 시간값을 LocalTime으로 변환
private LocalTime convertToLocalTime(String time){
    return LocalTime.of(Integer.parseInt(time.substring(0, 2)),
            Integer.parseInt(time.substring(3, 5)), 0);
}
```

  

기존 테스트케이스는 다 성공했으나,, 코드 제출을 하면 40점정도 나오고 해결이 안됐다,,  
여러 반례도 찾아보고 했는데 왜 안되나 했는데,,

해당 문제의 질문하기에 가면 **“하루가 지나도 과제를 계속 할 수 있다”**라는 말이 꽤 있다.  
처음에 뇌버깅으로는 그래도 코드에는 문제가 없지 않나 하고 계속 멈춰있었으나 나중에 생각해보고 반례를 만들고 로그를 찍으니 원인을 알았다...  

일단 반례는 아래와 같다.  
`{"A", "23:00", "50"}, {"B", "23:20", "20"}, {"C", "00:00", "40"}, {"D", "23:59", "30"}, {"E", "23:39", "18"}`  

정상 로직이라면 아래와 같이 \[C - E - B - D - A\] 결과가 나와야 한다.  
\- 과제명 - 과제를 한 시간\[진행 시간\] - 과제 완료까지 남은 시간  
1. C - 00:00-00:40\[40min\] - **finish**
2. A - 23:00-23:20 \[20min\] - 30 min
3. B - 23:20-23:39 \[19min\] - 1 min
4. E - 23:39-23:57 \[18min\] - **finish**
5. B - 23:58-23:58 \[1min\] - **finish**
6. A - 23:59-23:59 \[1min\] - 29min
7. D - 23:59-00:29\[30min\] - **finish**
8. A - 00:29-00:58\[29min\] - **finish**

그러나 위 로직은 6번에서 D과제를 시작해야 할 때, 아래 조건문에서 참이 나와 A를 마무리하고 D를 진행하게 된다.

```java
if (endTime.isBefore(nextStartTime) || endTime.equals(nextStartTime)) {}
```

5번이 끝나고 현재 시간은 23시58분, 현재부터 밀린 A과제 예상 마감 시간은 다음날 00시28분이 된다.  
다음으로 반드시 시작해야하는 D과제의 시작시간은 오늘 23시59분.  
A과제의 예상 마감 시간 00시28분과 D과제의 23시59분을 시간만 비교하면 A과제의 끝나는 시간이 더 이전으로 판단되어 참이 나온다.  
날짜를 제외하고 시간만 비교해서 나오는 문제이다.

그래서 위 실패 코드의 결과는  \[C - E - B - A - D\]가 나오는 것을 확인하게 되었다..  
혼자 유레카 하고 LocalTime으로 처리하던 로직을 분단위로 바로 바꿔서 누적하도록 변경하였다.


### 문제 풀이 (통과)

```java
//-- 완료된 과제의 이름을 담을 리스트
List<String> finishList = new ArrayList<>();
//-- 중간에 밀린 과제를 담을 리스트
Stack<String[]> delayPlans = new Stack<>();

public String[] solution(String[][] plans) {
   //-- 시간순으로 오름차순 정렬
    Arrays.sort(plans, new Comparator<String[]>() {
        @Override
        public int compare(String[] o1, String[] o2) {
            int time1 = convertToMinute(o1[1]);
            int time2 = convertToMinute(o2[1]);

            return Integer.compare(time1, time2);
        }
    });

    //-- 진행중인 현재시간을 저장할 now 변수 선언
    int now;
    for (int i = 0; i < plans.length - 1; i++) {
         //-- 바로 진행할 과제
        String[] plan = plans[i];
        //-- 다음으로 진행될 과제
        String[] nextPlan = plans[i + 1];

        //-- 바로 진행할 과제의 시작시간을 지금으로 설정
        now = convertToMinute(plan[1]);
        //-- 다음으로 진행할 과제의 시작시간 할당
        int nextStartTime = convertToMinute(nextPlan[1]);

        //-- 현재 진행 plan 과제를 처리할 수 있을만큼 처리한 후의 시간 받아옴
        now = getFinishedPlanTime(plan, nextStartTime, now);

        //-- 다음 과제의 시작시간이 안됐을 때 밀린 과제가 있으면 밀린 과제 처리
        //-- 다음 과제의 시작시간이 안됐다면 밀린 과제를 계속 처리 할 수 있도록 while문 사용
        while (now < nextStartTime && delayPlans.size() > 0) {
            //-- 밀린과제를 담아놓은 stack에서 가장 마지막에 넣은 과제 가져와서 진행 과제를 뜻하는 plan 변수에 할당
            plan = delayPlans.pop();
            //-- 지금 진행한 plan 과제를 처리할 수 있을만큼 처리한 후의 시간 받아옴
            now = getFinishedPlanTime(plan, nextStartTime, now);
        }
    }

    //-- 마지막으로 시작하는 과제는 시간 제한이 없으므로 한번에 처리
    finishList.add(plans[plans.length-1][0]);
    //-- 마지막 과제를 처리하고도 남은 밀린 과제 순차적으로 처리
    while (delayPlans.size() > 0) {
        finishList.add(delayPlans.pop()[0]);
    }
    
    //-- 리스트에 담아놓은 과제 완료 순서를 배열에 할당
    String[] answer = new String[finishList.size()];
    int index = 0;
    for(String name : finishList) {
        answer[index++] = name;
    }

    return answer;
}

//-- 과제를 할 수 있는만큼 진행하고 진행 완료시간을 알려줌
private int getFinishedPlanTime(String[] plan, int nextStartTime, int now) {
    //-- 현재 과제 처리 시간
    int playTime = Integer.parseInt(plan[2]);
    //-- 현재 과제의 예상 완료 시간
    int endTime = now + playTime;

    //-- 예상 과제 완료 시간이 다음 과제 시작시간보다 이전이거나 같으면
    if (endTime <= nextStartTime) {
        //-- 현재 과제는 마무리 가능하므로 완료 리스트에 추가
        finishList.add(plan[0]);
        //-- 과제 마무리 시간으로 시간 이동
        now = endTime;
    } 
    //-- 예상 과제 완료시간이 다음 과제 시작시간보다 이후일 경우
    //-- 현재 과제를 마무리하지 못할것으로 판단
    else {
        //-- 다음 과제 시작시간과 예상 과제 완료시간의 시간차이를 구함
        int duration = endTime - nextStartTime;
        //-- 다음 과제 시작시간까지만 처리하고 남은 시간으로 과제 처리시간 수정
        plan[2] = String.valueOf(duration);
        //-- 밀린과제에 현재 과제 정보 추가
        delayPlans.push(plan);
        //-- 다음 과제 시작시간까지 시간을 다 썼으므로 다음 과제 시작시간으로 이동
        now = nextStartTime;
    }

    //-- 현재 과제를 처리할 수 있을만큼 처리하고 난 시간 응답
    return now;
}

//-- String타입의 시간값을 분단위 시간 int로 변환하여 응답
private int convertToMinute(String time) {
    return Integer.parseInt(time.substring(0, 2))*60 + Integer.parseInt(time.substring(3, 5));
}
```

LocalTime으로 판단하던 시간을 분으로만 바꿔서 처리를 했다.  
실행해보니 위에서 문제됐던 케이스도  \[C - E - B - D - A\] 로 정상처리 되는 것을 확인할 수 있었다.


<details>
  <summary>채점 결과</summary>
<pre>
  채점을 시작합니다.
  정확성  테스트
  테스트 1 〉	통과 (0.66ms, 74.1MB)
  테스트 2 〉	통과 (0.58ms, 68.9MB)
  테스트 3 〉	통과 (0.51ms, 66.7MB)
  테스트 4 〉	통과 (0.70ms, 73MB)
  테스트 5 〉	통과 (0.71ms, 76.2MB)
  테스트 6 〉	통과 (0.96ms, 85.3MB)
  테스트 7 〉	통과 (1.23ms, 73.5MB)
  테스트 8 〉	통과 (1.90ms, 73.7MB)
  테스트 9 〉	통과 (2.58ms, 83.8MB)
  테스트 10 〉	통과 (3.07ms, 78.1MB)
  테스트 11 〉	통과 (3.91ms, 78.9MB)
  테스트 12 〉	통과 (5.70ms, 76.7MB)
  테스트 13 〉	통과 (5.14ms, 82.8MB)
  테스트 14 〉	통과 (9.07ms, 82.5MB)
  테스트 15 〉	통과 (9.33ms, 87.1MB)
  테스트 16 〉	통과 (0.42ms, 71.8MB)
  테스트 17 〉	통과 (0.78ms, 73.5MB)
  테스트 18 〉	통과 (0.44ms, 88.2MB)
  테스트 19 〉	통과 (0.54ms, 73.7MB)
  테스트 20 〉	통과 (2.43ms, 89MB)
  테스트 21 〉	통과 (1.99ms, 73.9MB)
  테스트 22 〉	통과 (9.61ms, 77.6MB)
  테스트 23 〉	통과 (8.78ms, 83MB)
  테스트 24 〉	통과 (8.95ms, 84.6MB)
  채점 결과
  정확성: 100.0
  합계: 100.0 / 100.0
</pre>
</details>
  

좀 더 넓게 많은 케이스를 생각해보기 위해 노력해야겠다.