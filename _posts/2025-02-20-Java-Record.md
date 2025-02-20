---
title: "Java Record"
author: hcbak
date: 2025-02-20 16:36:30 +0900
categories: [Language, Java]
---

## Record

Record 클래스는 LTS 기준 Java 17 이상에서 DTO(Data Transfer Object) 등의 용도로 사용 가능한 클래스이다.

Record를 사용하면 방대한 보일러플레이트로 Lombok의 도움 없이는 대 혼란에 빠지는 데이터 전송 객체를 **아주 간단하게 작성**할 수 있다.

Record 클래스는 아래의 특징을 가지고 있다.

- 접근 제어
  - 클래스: public final
  - 필드: private final
- 메서드 자동 생성
  - 생성자
  - equals
  - hashCode
  - toString
  - 접근자(getter)

final이 클래스와 필드 단위에 붙어 있는 **불변 객체**이다!

### 예시

이번에 Java 21로 개인 홈페이지를 새로 만들면서 record를 한번 써 봤다.

아래는 Frontend에서 Header를 그릴 때 필요한 메뉴 목록을 불러오는 데 사용한 DTO이다.

```java
public record NavbarMenu(
    String name,
    String url,
    NavbarMenu[] sub
    ) {
}
```

아직 과거의 잔재가 남아 있어 줄을 바꿔 놓았지만, 공식 문서에서 자랑하는 대로 쓰면 이렇게 된다.

```java
record NavbarMenu(String name, String url, NavbarMenu[] sub) { }
```

어노테이션 하나도 없다. 이번에 어디까지 가능한지 궁금해서 Lombok은 프로젝트 단위에서 배제했다. 의존성조차 없다.

다른 것들은 대략 비슷하고 두 가지 정도 살펴보면 좋을 것 같다.

### 필드 접근 방법(기존 getter)

Lombok에서는 @Getter 어노테이션으로 각 필드에 대한 Getter를 만들어줄 수 있다. 그렇다면 Record도 "getName" 이런 방식으로 필드를 불러올까? 아니다!

```java
NavbarMenu navbarMenu = new NavbarMenu("holly", "molly", null);

System.out.println(navbarMenu.name());
System.out.println(navbarMenu.url());
```

이런 방식으로 필드 이름과 같은 이름의 메소드로 값을 불러올 수 있다.

final이 달려 있으니 private 없이 그냥 필드 자체에 접근하는 방법이 좋지 않나 싶기는 하지만... 왜 때문인지 일단 이렇게 되어 있다.  
(무의미한 getter 메소드 좀 버려줘 제발...)

### public 명시 문제

위에서 눈치를 챘을 수 있지만 내 코드에는 접근제어자인 public을 명시하고 있다. 분명 공식 문서에는 없어도 된다고 적혀있는 것으로 보이는데 진짜 없애면 IDE가 빨간 줄로 도배가 된다!

이 상태에서 컴파일을 하면 컴파일 오류가 발생하고, 오류를 무시하고 컴파일하면 실행은 또 잘 된다.  
(컴파일 후의 class 파일에 보면 public이 생성되어 있다.)

만약 명시를 하는 그 자체가 의미가 있다고 하면 필드에 private는 왜 명시 안해도 되는데 ㅂㄷㅂㄷ

아무튼 그래서 public은 붙여줘야 한다!

## 참고

- [Oracle - JDK 17 Documentation - Java Language Updates - 6 Record Classes](https://docs.oracle.com/en/java/javase/17/language/records.html){:target="_blank"}
- [OracleJDK - JEP 395: Records](https://openjdk.org/jeps/395){:target="_blank"}
