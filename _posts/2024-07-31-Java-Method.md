---
title: "Java Method"
author: hcbak
date: 2024-07-31 17:41:00 +0900
categories: [Language, Java]
---

> Java를 극도로 혐오하는 사람의 학습기...

# Method
메소드는 다른 언어의 함수와 매우 유사하다고 생각한다. 인터넷을 찾아보니 메소드와 함수의 차이는 종속적이냐 독립적이냐의 차이로 보는 것 같다.

그래서 함수와 큰 차이는 없는 것 같다. 사용하는 목적도 결과도 똑같... 함수라고 부르고 싶다.

## 1. 메소드 선언
메소드는 클래스 아래에 선언하여 사용한다.
```java
public void myMethod(int a) {
  ...
}
```

### * 명칭
메소드를 선언할 때 적히는 단어들의 위치에는 이름이 있다. 명확하게 하고 넘어가야 대화(?)가 통한다.

  
|접근제어자|반환형|메소드명|매개변수|
|-|-|-|-|
|public|void|myMethod|(int a)|
  

#### 접근제어자
해당 메소드의 접근을 제어하는 구문이다.


|||
|-|-|
|public|공개되어 어디서든 접근 가능|
|protected|상속 관계이거나 같은 패키지에서 접근 가능|
|default|생략이 가능, 같은 패키지에서 접근 가능|
|private|같은 클래스 내부에서만 접근 가능|


#### 반환형
return으로 출력하는 값의 데이터형을 말한다.

void는 return으로 반환하는 값이 없다는 의미이고, return을 생략할 수 있다.

추가로 우리가 함수에서 자주 쓰는 방식으로 if문 등에 return을 넣어서 조건이 맞으면 중간에 함수를 탈출하는 방법이 있는데, 같은 목적을 메소드에서 달성하기 위해 생략하지 않고 넣는 것은 가능하다.

#### 메소드명
그냥 이름이다. ㅎㅎ;

#### 매개변수
메소드를 호출할 때 넘겨주는 매개변수를 선언하는 곳이다.


## 2. 메소드 호출

참조 연산자(.)로 호출한다.

```java
...
public class Myclass
  public static void main(String[] args) {
    Myclass mc = new Myclass;

    mc.myMethod();
  }

  public void myMethod() {
    System.out.println("Hello, World!");
  }
```

