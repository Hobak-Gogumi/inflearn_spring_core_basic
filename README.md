# inflearn_spring_core_basic
인프런 김영한 강사님의 [스프링 핵심 원리 - 기본편](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%95%B5%EC%8B%AC-%EC%9B%90%EB%A6%AC-%EA%B8%B0%EB%B3%B8%ED%8E%B8#)을 들으면서 기록한 리포지토리 입니다.
***
+ 개발환경
    - Windows 10
    - 자바 11
    - IntelliJ Community
***
## 🍀 섹션 2. 스프링 핵심 원리 이해1 - 예제 만들기
순수 자바코드를 사용하여 프로젝트를 작성했다.
+ 회원 등급은 일반, VIP 두 가지로 나누어짐
+ 회원 등급에 따라 할인 정책이 달라짐
    - VIP는 무조건 1000원을 할인해 주는 **고정 금액 할인**`FixDiscountPolicy`을 적용
    - 단, 할인 정책은 변경 가능성이 높음
<br/>
<br/>

## 🍀 섹션 3. 스프링 핵심 원리 이해2 - 객체 지향 원리 적용
**정률 할인 정책**`RateDiscountPolicy`으로 할인정책이 바뀌는 상황이 발생했다.
+ 할인 정책 변경 시 클라이언트 코드를 고쳐야 하는 문제점 발생
    - 이는 DIP, OCP 위반
```java
public class OrderServiceImpl implements OrderService{
//    private final DiscountPolicy discountPolicy = new FixDiscountPolicy();
    private final DiscountPolicy discountPolicy = new RateDiscountPolicy();
}
```
+ 이를 해결하기 위해 인터페이스에만 의존하도록 코드 변경
    - 그러나 구현체가 없으므로 null pointer exception 발생
    - 이 문제를 해결하려면 **누군가가 OrderServiceImpl에 DiscountPolicy의 구현 객체를 대신 생성하고 주입**해주어야 함
```java
public class OrderServiceImpl implements OrderService{
//    private final DiscountPolicy discountPolicy = new RateDiscountPolicy();
    private DiscountPolicy discountPolicy;
}
```
+ AppConfig와 생성자 주입을 사용하여 이런 문제를 해결
    - AppConfig의 등장으로 애플리케이션이 크게 **사용영역과, 객체를 생성하고 구성(Configuration)하는 영역으로 분리**
+ 지금까지는 순수한 자바코드로 작성, 이제 스프링을 사용
    - 설정파일에는 `@Configuration`을 붙여주고, 각 메서드에 `@Bean`을 붙임
<br/>
<br/>

## 🍀 섹션 4. 스프링 컨테이너와 스프링 빈
+ ApplicationContext → 스프링 컨테이너
+ 빈 이름은 메서드 이름을 사용
+ `getBean()` 메서드를 사용해서 필요한 스프링 빈을 찾을 수 있음
<br/>
<br/>

## 🍀 섹션 5. 싱글톤 컨테이너
+ ApplicationContext → **스프링**에서 사용하는 것
    - 스프링 없이 작성했던 AppConfig는 요청할 때마다 객체를 새로 생성
+ 싱글톤 패턴
    - 클래스의 인스턴스가 딱 1개만 생성되는 것을 보장하는 디자인 패턴
+ **스프링 컨테이너는 객체 인스턴스를 싱글톤으로 관리**
+ 싱글톤 방식의 주의점
    - 무상태(stateless)로 설계해야 한다.
    - 특정 클라이언트에게 의존적인 필드가 있으면 안된다.
    - 가급적 읽기만 가능해야 한다.
+ `@Configuration`과 바이트코드 조작의 마법
    - 스프링이 CGLIB라는 바이트코드 조작 라이브러리를 사용해서 `@Configuration`을 적용한 설정 클래스를 상속받은 임의의 다른 클래스를 만들고, 그 다른 클래스를 스프링 빈으로 등록한다. → 임의의 다른 클래스가 싱글톤을 보장되도록 해준다.
    - `@Configuration`을 사용하지 않고, `@Bean`만 사용해도 스프링 빈으로 등록 되지만, 싱글톤을 보장하지 않는다.
    - 스프링 설정정보는 항상 `@Configuration`을 사용하자.
<br/>
<br/>

## 🍀 섹션 6. 컴포넌트 스캔
+ 지금까지는 `@Bean`을 사용하여 수동 등록했다.
    - 그러나 빈이 많아지면 귀찮고, 설정 파일이 커지고, 누락하는 문제가 발생
+ `@ComponentScan`을 사용하여 자동등록 시작
    - `@ComponentScan`을 설정 정보에 붙여준다.
    - 서비스, 리포지토리, 등등 등록할 빈(클래스)에 `@Component`를 붙여주고, 의존관계가 필요한 생성자에 `@Autowired`를 붙여준다. 이게 중요. **의존관계 자동 주입**
    - 이때 스프링 빈의 기본 이름은 클래스 명을 사용하되, 앞글자만 소문자를 사용한다.
<br/>
<br/>

## 🍀 섹션 7. 의존관계 자동 주입
+ 의존관계 주입 방법
    - 생성자 주입
    - 수정자 주입
    - 필드 주입
    - 일반 메서드 주입
+ 생성자 주입은 불변, 필수 의존관계에 사용. 무조건 1번만 사용됨
+ **생성자가 1개만 있으면 @Autowired를 생략해도 자동 주입**
+ 의존관계 자동 주입은 스프링 컨테이너가 관리하는 스프링 빈이어야 동작
+ 롬복의 `@RequiredArgsConstructor` 사용하면 final이 붙은 필드를 모아서 생성자를 자동으로 만들어 준다.
+ `@Autowired`는 타입으로 조회한다.
+ 만약 같은 타입이 2개라면
    - Autowired 필드명 매칭
    - `@Qualifier`
    - `@Primary`
<br/>
<br/>

## 🍀 섹션 8. 빈 생명주기 콜백
스프링에서 객체의 초기화와 종료작업 수행
+ 초기화 작업은 의존관계 주입이 모두 완료된 다음에 호출해야함
    - 스프링이 의존관계 주입 완료시점을 어떻게 아나?
+ 스프링 빈의 이벤트 라이프사이클
    - 스프링 컨테이너 생성 → 스프링 빈 생성 → 의존관계 주입 → 초기화 콜백 → 사용 → 소멸전 콜백 → 스프링 종료
    - 초기화 콜백: 빈이 생성되고, 빈의 의존관계 주입이 완료된 후 호출
    - 소멸전 콜백: 빈이 소멸되기 직전에 호출
+ 스프링은 크게 3가지 방법으로 빈 생명주기 콜백을 지원
    - 인터페이스 InitializingBean, DisposableBean
        - afterPropertiesSet(), destroy() 메서드 지원
    - 설정 정보에 초기화 메서드, 종료 메서드 지정
    - `@PostConstruct`, `@PreDestroy` 애노테이션 지원
<br/>
<br/>

## 🍀 섹션 9. 빈 스코프
빈 스코프: 빈이 존재할 수 있는 범위
+ 프로토타입 스코프
    - 스프링 컨테이너는 프로토타입 빈을 생성하고, 의존관계 주입, 초기화까지만 처리(종료 메서드 호출 X)
    - 항상 새로운 인스턴스를 생성해서 반환
```java
@Scope("prototype")
@Component
public class HelloBean{}
// 컴포넌트 스캔 자동 등록
```
+ 웹 스코프
    - 웹 스코프는 웹 환경에서만 동작
    - 스코프의 종료시점까지 관리
<br/>
<br/>