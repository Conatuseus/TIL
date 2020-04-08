

# AOP란?

AOP는 **애플리케이션 전체에 걸쳐 사용되는 기능을 재사용**하도록 지원하는 것입니다.

AOP(Aspect-Oriented Programming)란 단어를 번역하면 **관점(관심) 지향 프로그래밍**으로 됩니다.



![image](https://user-images.githubusercontent.com/22893111/78748855-fd09d200-79a7-11ea-8fe2-9e313a7cc4ce.png)

(핵심기능에서 바라본 관점)

각각의 Service는 핵심기능 관점에서 봤을 때는 Board, User, XXX 등 **공통된 요소가 없습니다.**

이런 괌점에서 각각의 Service는 각자 코드를 구현하고 있습니다. 하지만, 이 관점을 돌려서 **부가기능** 이란 관점에서 바라보면 상황이 달라집니다.

![image](https://user-images.githubusercontent.com/22893111/78748870-0430e000-79a8-11ea-86e9-6353b6af26c7.png)

부가기능의 관점에서 바라보면 각각의 Service는 수행시간 측정을 나타내는 before, after라는 메서드가 공통되는 것을 알 수 있습니다.

**AOP는 여기서부터 시작합니다**

기존에 OOP에서 바라보던 관점을 다르게 하여 부가기능적인 측면에서 보았을 때 공통된 요소를 추출하자는 것입니다.

이때 가로(횡단) 영역의 공통된 부분을 잘라냈다고 하여, AOP를 **크로스 컷팅(Cross-Cutting)** 이라고 불리기도 합니다.

AOP라고 해서 전에 없던 새로운 개념이 등장한 것이 아니라, 결국은 **공통된 기능을 재사용하는 기법**입니다.

OOP에선 공통된 기능을 재사용하는 방법으로 상속이나 위임을 사용합니다. 하지만 전체 어플리케이션에서 여기저기 사용되는 **부가기능**들을 상속이나 위임으로 처리하기에는 깔끔하게 모듈화가 어렵습니다. 그리고 Java는 다중상속을 지원하지 않으므로 문제가 발생합니다.

그래서 이 문제를 해결하기 위해 AOP가 등장하게 됩니다.

**AOP의 장점**은 2가지입니다.

- 어플리케이션 전체에 흩어진 공통 기능이 하나의 장소에서 관리된다는 점
- 다른 서비스 모듈들이 본인의 목적에만 충실하고 그외 사항들은 신경쓰지 않아도 된다는 점



# AOP 용어



**타겟(Target)**

부가기능을 부여할 대상을 얘기합니다.

여기선 핵심기능을 담당하는 getBoards, getUsers를 하는 Service 들을 얘기합니다.



**애스펙트 (Aspect)**

객체지향 모듈을 오브젝트라 부르는 것과 비슷하게 부가기능 모듈을 애스펙트라고 부르며, **핵심기능에 부가되어 의미를 갖는** 특별한 모듈이라 생각하시면 됩니다.

애스펙트는 부가될 기능을 정의한 **어드바이스**와 어드바이스를 어디에 적용할지를 결정하는 **포인트컷**을 함께 갖고 있습니다.



**어드바이스(Advice)**

실질적으로 부가기능을 담은 구현체를 얘기합니다.

어드바이스의 경우 타겟 오브젝트에 종속되지 않기 때문에 순순하게 **부가기능에만 집중**할 수 있습니다.

어드바이스는 애스팩트가 '무엇'을 '언제' 할지를 정의하고 있습니다.



**포인트컷(PointCut)**

부가기능이 적용될 대상(메서드)를 선정하는 방법을 얘기합니다.

즉, 어드바이스를 적용할 조인포인트를 선별하는 기능을 정의한 모듈을 얘기합니다.



**조인포인트(JoinPoint)**

어드바이스가 적용될 수 있는 위치를 얘기합니다.

다른 AOP 프레임워크와 달리 Spring에서는 **메서드 조인포인트만 제공**하고 있습니다. 따라서 Spring 프레임워크 내에서 조인포인트라 하면 **메서드**를 가리킨다고 생각하면 됩니다.



**프록시(Proxy)**

타겟을 감싸서 타겟의 요청을 대신 받아주는 랩핑(Wrapping) 오브젝트입니다. 호출자(클라이언트)에서 타겟을 호출하게 되면 타겟이 아닌 타겟을 감싸고 있는 프록시가 호출되어, 타겟 메서드 실행전에 선처리, 타겟 메서드 실행 후, 후처리를 실행시키도록 구성되어 있습니다.

![image](https://user-images.githubusercontent.com/22893111/78749901-21ff4480-79aa-11ea-8041-d9443e3ac8a6.png)



**인트로덕션(Introduction)**

타겟 클래스에 코드 변경없이 신규 메서드나 멤버변수를 추가하는 기능을 얘기합니다.



**위빙(Weaving)**

지정된 객체에 애스펙트를 적용해서 새로운 프록시 객체를 생성하는 과정을 얘기합니다. 

예를 들면 A라는 객체에 트랜잭션 애스팩트가 지정되어 있다면, A라는 객체가 실행되기전 커넥션을 오픈하고 실행이 끝나면 커넥션을 종료하는 기능이 추가된 프록시 객체가 생성되고, 이 프록시 객체가 앞으로 A 객체가 호출되는 시점에서 사용됩니다. 이때의 프록시객체가 생성되는 과정을 **위빙**이라 생각하시면 됩니다



# 사용법

~~~ java
@Aspect
public class Performance {

    @Around("execution(* com.blogcode.board.BoardService.getBoards(..))")
    public Object calculatePerformanceTime(ProceedingJoinPoint proceedingJoinPoint) {
        Object result = null;
        try {
            long start = System.currentTimeMillis();
            result = proceedingJoinPoint.proceed();
            long end = System.currentTimeMillis();

            System.out.println("수행 시간 : "+ (end - start));
        } catch (Throwable throwable) {
            System.out.println("exception! ");
        }
        return result;
    }
}
~~~





![image](https://user-images.githubusercontent.com/22893111/78750103-83271800-79aa-11ea-963f-bb51e7d5e267.png)

@Around는 **어드바이스**입니다.

앞서 설명드린 것처럼 어드바이스는 애스펙트가 '무엇을', '언제' 할지를 의미하고 있습니다.

여기서 '무엇'은 calculatePerformanceTime() 메서드를 나타냅니다.

그리고 '언제'는 @Around가 되는데, 이 '언제'라는 시점의 경우 @Around만 존재하지 않고 총 5가지의 타입이 존재합니다.

- @Before (이전)
  - 어드바이스 타겟 메서드가 호출되기 전에 어드바이스 기능을 수행
- @After (이후)
  - 타겟 메서드의 결과에 관계없이(즉 성공, 예외 관계없이) 타겟 메서드가 완료되면 어드바이스 기능을 수행
- @AfterReturning (정상적 반환 이후)
  - 타겟 메서드가 성공적으로 결과값을 반환 후에 어드바이스 기능을 수행
- @AfterThrowing (예외 발생 이후)
  - 타겟 메서드가 수행 중 예외를 던지게 되면 어드바이스 기능을 수행
- @Around (메서드 실행 전후)
  - 어드바이스가 타겟 메서드를 감싸서 타겟 메서드 호출전과 후에 어드바이스 기능을 수행

예를 들어 타겟 메서드의 이전 시점에서만 어드바이스 메서드를 수행하고 싶다면,

~~~ java
@Before("포인트컷 표현식")
public void 어드바이스메서드() {
  ...
}
~~~

식으로 작성하면 됩니다.

여기서 주의해야할 점은 @Around의 경우 **반드시 proceed() 메서드가 호출**되어야 한다는 것입니다.

proceed() 메서드는 타겟 메서드를 지칭하기 때문에 proceed 메서드를 실행시켜야만 타겟 메서드가 수행이 된다는 것을 잊지 맙시다.



다음으로 알아볼것은 @Around 다음에 작성된 `"execution(* com.blogcode.board.BoardService.getBoards(..))"` 입니다.
어드바이스의 value로 들어간 이 문자열을 **포인트컷 표현식** 이라고 합니다.
포인트컷 표현식은 2가지로 나눠지는데, `execution`을 **지정자**라고 부르며
`(* com.blogcode.board.BoardService.getBoards(..))`는 **타겟 명세**라고 합니다.
포인트컷 지정자는 execution을 포함하여 총 9가지가 있습니다.

- args()
  - 메소드의 인자가 타겟 명세에 포함된 타입일 경우
  - ex) args(java.io.Serializable) : 하나의 파라미터를 갖고, 그 인자가 Serializable 타입인 모든 메소드
- @args()
  - 메소드의 인자가 타겟 명세에 포함된 어노테이션 타입을 갖는 경우
  - ex) @args(com.blogcode.session.User) : 하나의 파라미터를 갖고, 그 인자의 타입이 @User 어노테이션을 갖는 모든 메소드 (@User User user 같이 인자 선언된 메소드)
- execution()
  - 접근제한자, 리턴타입, 인자타입, 클래스/인터페이스, 메소드명, 파라미터타입, 예외타입 등을 전부 조합가능한 가장 세심한 지정자
  - 이전 예제와 같이 풀패키지에 메소드명까지 직접 지정할 수도 있으며, 아래와 같이 특정 타입내의 모든 메소드를 지정할 수도 있다.
  - ex) execution(* com.blogcode.service.AccountService.*(..) : AccountService 인터페이스의 모든 메소드
- within()
  - execution 지정자에서 클래스/인터페이스까지만 적용된 경우
  - 즉, 클래스 혹은 인터페이스 단위까지만 범위 지정이 가능하다.
  - ex) within(com.blogcode.service.*) : service 패키지 아래의 클래스와 인터페이스가 가진 모든 메소드
  - ex) within(com.blogcode.service..*) : service 아래의 모든 **하위패키지까지** 포함한 클래스와 인터페이스가 가진 메소드
- @within()
  - 주어진 어노테이션을 사용하는 타입으로 선언된 메소드
- this()
  - 타겟 메소드가 지정된 빈 타입의 인스턴스인 경우
- target()
  - this와 유사하지만 빈 타입이 아닌 타입의 인스턴스인 경우
- @target()
  - 타겟 메소드를 실행하는 객체의 클래스가 타겟 명세에 지정된 타입의 어노테이션이 있는 경우
- @annotation
  - 타겟 메소드에 특정 어노테이션이 지정된 경우
  - ex) @annotation(org.springframework.transaction.annotation.Transactional) : Transactional 어노테이션이 지정된 메소드 전부

다양한 지정자가 등장하였지만 실제로 execution과 @annotation을 주로 사용하는 것으로 알고 있습니다.



























































