## Bean Validation 소개
검증 기능을 매번 코드로 작성하는 것은 번거로움
간단한 검증 로직은 어노테이션으로 처리 할 수 있도록 기술 표준이 있다.
Bean Validation을 구현한 기술중에 일반적으로 사용하는 구현체는 하이버네이트 Validator이다.

## Bean Validation 예시
build.gradle에 의존관계 추가
> implementation 'org.springframework.boot:spring-boot-starter-validation'

dto파일에 어노테이션 추가
```
@Data
public class Item {
   private Long id;
   @NotBlank
   private String itemName;
   @NotNull
   @Range(min = 1000, max = 1000000)
   private Integer price;
   @NotNull
   @Max(9999)
   private Integer quantity;
   public Item() {
   }
}
```
검증 어노테이션 설명
> @NotBlank : 빈값 + 공백만 있는 경우를 허용하지 않는다.
> @NotNull : null 을 허용하지 않는다.
> @Range(min = 1000, max = 1000000) : 범위 안의 값이어야 한다.
> @Max(9999) : 최대 9999까지만 허용한다.

참고
> javax.validation 으로 시작하면 특정 구현에 관계없이 제공되는 표준 인터페이스이고, 
> org.hibernate.validator 로 시작하면 하이버네이트 validator 구현체를 사용할 때만 제공되는 검증 기능이다.
> 실무에서 대부분 하이버네이트 validator를 사용하므로 자유롭게 사용해도 된다.

이것이 실행 되는지 테스트 하는 코드는 pdf문서 참고하기

## Bean Validation 스프링에 적용
기존에 작성했던 validator클래스 삭제, 컨트롤러에서 관련 코드 삭제(아래코드 참고)
```
private final ItemValidator itemValidator;
@InitBinder
public void init(WebDataBinder dataBinder) {
 log.info("init binder {}", dataBinder);
 dataBinder.addValidators(itemValidator);
}
```
dto파일에 선언한 어노테이션이 있기 때문에 검증이 수행된다.
이때 어노테이션 뒤에 @NotBlank(message="공백이어서 안됩니다.")와 같이
메시지를 지정하면 지정된 메시지가 출력됨

주의
> 스프링 부트는 자동으로 글로벌 Validator로 등록한다.
> LocalValidatorFactoryBean 을 글로벌 Validator로 등록한다.
> 이 Validator는 @NotNull 같은 애노테이션을 보고 검증을 수행한다.
> 따라서 글로벌 vaildator이 있다면 제거해야한다.
> 이렇게 글로벌 Validator가 적용되어 있기 때문에, @Valid , @Validated 만 적용하면 된다.
> 검증 오류가 발생하면, FieldError , ObjectError 를 생성해서 BindingResult 에 담아준다.

검증순서
> 1. @ModelAttribute 각각의 필드에 타입 변환 시도
> > - 성공하면 다음으로
> > - 실패하면 typeMismatch 로 FieldError 추가
> 2. Validator 적용

## Bean Validation 오류 메시지 구체화하고 싶을 때 
properties파일에 아래 코드 추가하면 끝!!!
```
#Bean Validation 추가
NotBlank={0} 공백X 
Range={0}, {2} ~ {1} 허용
Max={0}, 최대 {1}
```
여기서 더 구체화하고 싶다면 이전에 공부했던대로 level을 높이기 -> Max.item과 같이 세밀하게 작성하면 됨

## Bean Validation - 오브젝트 오류
특정 필드(html에서 지정한)의 오류가 아닌 여러 필드가 복합적으로 이루어진 조건으로 글로벌 오류를 생성하는 오프젝트 오류일 경우의 처리방법
아래의 코드를 dto에 추가하면 된다.
> @ScriptAssert(lang = "javascript", script = "_this.price * _this.quantity >= 10000")

결론
> 그러나 권장하는 방법은 아님, 위의 예제는 단순하지만 실제로 사용하려면 더 복잡함
> 따라서 오브젝트 오류(글로벌 오류)의 경우 @ScriptAssert보다는 아래처럼 오브젝트 관련 부분만 자바코드로 작성하는 것을 권장함
```
//특정 필드 예외가 아닌 전체 예외
 if (item.getPrice() != null && item.getQuantity() != null) {
   int resultPrice = item.getPrice() * item.getQuantity();
   if (resultPrice < 10000) {
   bindingResult.reject("totalPriceMin", new Object[]{10000, resultPrice}, null);
   }
 }
```