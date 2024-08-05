## AOP가 필요한 상황
- 모든 메서드의 호출 시간 측정을 원할 때
- 호출 시간이 공통 관심 사항일 때(cross-cutting concern)
- 회원 가입 시간, 회원 조회 시간을 측정하고 싶을 때

### 문제
- 회원가입, 회원 조회에 시간을 측정하는 기능은 핵심관심 사항(core concern)이 아니다.
- 시간을 측정하는 로직과 핵심 비즈니스 로직이 섞이면 유지보수가 어렵다.
- 시간을 측정하는 로직을 별도의 공통 로직으로 만들기 어렵다.
- 시간을 측정하는 로직을 변경할 때 모든 로직을 찾아서 변경해야 한다.

## AOP 적용
- AOP : Aspect Oriented Programming
- 공통 관심사항과 핵심 관심사항 분리
### 시간 측정 AOP 등록
```
@Component
@Aspect
public class TimeTraceAop {
	// 					패키지.하위패키지.하위패키지 ex)service ....	(현재 상태는 hellospring 하위 모든 클래스 대상)
	@Around("execution(* hello.hellospring..*(..))")
    public Object execute(ProceedingJoinPoint joinPoint) throws Throwable {
    	long start = System.currentTimeMillis();
        
        System.out.println("START: " + joinPoint.toString());
        
        try {
        	return joinPoint.proceed();
        } finally {
        	long finish = System.currentTimeMillis();
            long timeMs = finish - start;
            
            System.out.println("END: " + joinPoint.toString() + " " + timeMs + "ms");
        }
    }
}
```
> `AOP`는 필수로 사용되는 기능이 아니다. 따라서 `@Component`를 통해 스프링에 등록하기 보다, `SpringConfig` 에서 아래 코드와 같이 직접 `Bean`을 명시적으로 등록 해주는 것이 유지보수 및 관리적 측면에서 더 좋다.

```
@Bean
public TimeTraceAop timeTraceAop() {
	return new TimeTraceAop();
}
```


### 결론
- 회원가입, 회원 조회 등 핵심 관심사항과 시간을 측정하는 공통 관심사항을 분리한다.
- 시간을 측정하는 로직을 별도의 공통 로직으로 만들었다.
- 변경이 필요하면 공통 관심사항 로직만 변경하면 된다.
- 원하는 적용 대상을 선택할 수 있다.

### 동작 방식 설명
#### AOP 적용 전
`memberController` -> `memberService`

#### AOP 적용 후
`memberController` -> <U>프록시 `memberService` -> 실제 `memberService`</U>


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;↑

<br>

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;joinPoint.proceed()

#### AOP 적용 후 전채적 그림
![](https://velog.velcdn.com/images/julioh0603/post/2a7efa16-7b63-4d57-aba2-80d91459faa3/image.png)
