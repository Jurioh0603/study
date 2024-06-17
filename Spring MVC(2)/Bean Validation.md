## Bean Validation 소개
검증 기능을 매번 코드로 작성하는 것은 번거로움<br>
간단한 검증 로직은 어노테이션으로 처리 할 수 있도록 기술 표준이 있다.<br>
Bean Validation을 구현한 기술중에 일반적으로 사용하는 구현체는 하이버네이트 Validator이다.<br>

## Bean Validation 예시
build.gradle에 의존관계 추가<br>
> implementation 'org.springframework.boot:spring-boot-starter-validation'<br>

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
검증 어노테이션 설명<br>
> @NotBlank : 빈값 + 공백만 있는 경우를 허용하지 않는다.<br>
> @NotNull : null 을 허용하지 않는다.<br>
> @Range(min = 1000, max = 1000000) : 범위 안의 값이어야 한다.<br>
> @Max(9999) : 최대 9999까지만 허용한다.<br>

참고<br>
> javax.validation 으로 시작하면 특정 구현에 관계없이 제공되는 표준 인터페이스이고, <br>
> org.hibernate.validator 로 시작하면 하이버네이트 validator 구현체를 사용할 때만 제공되는 검증 기능이다.<br>
> 실무에서 대부분 하이버네이트 validator를 사용하므로 자유롭게 사용해도 된다.<br>

이것이 실행 되는지 테스트 하는 코드는 pdf문서 참고하기<br>

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
dto파일에 선언한 어노테이션이 있기 때문에 검증이 수행된다.<br>
이때 어노테이션 뒤에 @NotBlank(message="공백이어서 안됩니다.")와 같이<br>
메시지를 지정하면 지정된 메시지가 출력됨<br>

주의<br>
> 스프링 부트는 자동으로 글로벌 Validator로 등록한다.<br>
> LocalValidatorFactoryBean 을 글로벌 Validator로 등록한다.<br>
> 이 Validator는 @NotNull 같은 애노테이션을 보고 검증을 수행한다.<br>
> 따라서 글로벌 vaildator이 있다면 제거해야한다.<br>
> 이렇게 글로벌 Validator가 적용되어 있기 때문에, @Valid , @Validated 만 적용하면 된다.<br>
> 검증 오류가 발생하면, FieldError , ObjectError 를 생성해서 BindingResult 에 담아준다.<br>

검증순서<br>
> 1. @ModelAttribute 각각의 필드에 타입 변환 시도<br>
> > - 성공하면 다음으로<br>
> > - 실패하면 typeMismatch 로 FieldError 추가<br>
> 2. Validator 적용<br>

## Bean Validation 오류 메시지 구체화하고 싶을 때 
properties파일에 아래 코드 추가하면 끝!!!
```
#Bean Validation 추가
NotBlank={0} 공백X 
Range={0}, {2} ~ {1} 허용
Max={0}, 최대 {1}
```
여기서 더 구체화하고 싶다면 이전에 공부했던대로 level을 높이기 -> Max.item과 같이 세밀하게 작성하면 됨<br>

## Bean Validation - 오브젝트 오류
특정 필드(html에서 지정한)의 오류가 아닌 여러 필드가 복합적으로 이루어진 조건으로 글로벌 오류를 생성하는 오프젝트 오류일 경우의 처리방법<br>
아래의 코드를 dto에 추가하면 된다.<br>
> @ScriptAssert(lang = "javascript", script = "_this.price * _this.quantity >= 10000")<br>

결론<br>
> 그러나 권장하는 방법은 아님, 위의 예제는 단순하지만 실제로 사용하려면 더 복잡함<br>
> 따라서 오브젝트 오류(글로벌 오류)의 경우 @ScriptAssert보다는 아래처럼 오브젝트 관련 부분만 자바코드로 작성하는 것을 권장함<br>
```
//특정 필드 예외가 아닌 전체 예외
 if (item.getPrice() != null && item.getQuantity() != null) {
   int resultPrice = item.getPrice() * item.getQuantity();
   if (resultPrice < 10000) {
   bindingResult.reject("totalPriceMin", new Object[]{10000, resultPrice}, null);
   }
 }
```

## 수정에도 검증 추가
수정 컨트롤러 메서드에서 모델 객체에 @Validated 추가<br>
검증오류가 발생하면 editForm으로 이동하는 코드
```
if (bindingResult.hasErrors()) {
 log.info("errors={}", bindingResult);
 return "validation/v3/editForm";
 }
```
html변경<br>
상품명, 가격, 수량 필드에 검증 기능 추가<br>
더 이상 errors맵에 담지 않아도 되니까<br>
th:class="${errors?.containsKey('price')} ? 'form-control field-error' : 'form-control'"에서<br>
th:errorclass="field-error" 로 변경<br>
```
<div th:if="${#fields.hasGlobalErrors()}">
 <p class="field-error" th:each="err : ${#fields.globalErrors()}" th:text="${err}">글로벌 오류 메시지</p>

<label for="itemName" th:text="#{label.item.itemName}">상품명</label>
 <input type="text" id="itemName" th:field="*{itemName}" th:errorclass="field-error" class="form-control" placeholder="이름을 입력하세요">
 <div class="field-error" th:errors="*{itemName}">상품명 오류</div>

<label for="price" th:text="#{label.item.price}">가격</label>
 <input type="text" id="price" th:field="*{price}" th:errorclass="field-error" class="form-control" placeholder="가격을 입력하세요">
 <div class="field-error" th:errors="*{price}">가격 오류</div>
```

## Bean Validation 한계
등록과 수정에서 조건이 다를 때 <br>
ex) 수정시에는 수량 9,999 이상의 값을 넣을 수 있다, id는 notnull이어야 한다. -> 등록이 작동되지 않음(id가 auto increament라서)<br>
다음장에서 해결해보자<br>

## Bean Validation groups
검증을 다르게 적용하고 싶은 등록과 수정 group을 인터페이스로 생성<br>
dto의 조건 어노테이션에 아래와 같이 groups로 명시<br>
@NotNull(groups = UpdateCheck.class)<br>
여러개일 경우<br>
@NotBlank(groups = {SaveCheck.class, UpdateCheck.class})<br>
컨트롤러에 @Validated(SaveCheck.class) 와 같이 원하는 인터페이스명.class 작성<br>

**그러나 복잡도가 올라가는 문제로 잘 사용되지 않음**
다음장에서 실무에서 주로 사용하는 쉬운 방법을 알아보자<br>

## form 전송 객체 분리
기존방법 = html Form -> Item -> Controller -> Item -> Repository<br>
간단하지만 검증이 중복될 수 있고, groups를 사용해야해서 복잡도가 증가됨<br>
폼 객체 분리 = html Form-> ItemSaveForm -> Controller -> Item 생성 -> Repository<br>
복잡한 폼 데이터도 맞춤형으로 처리해서 전달할 수 있음, 검증 중복 안됨<br>
폼 데이터를 기반으로 Item객체 생성하는 변환 과정이 추가됨<br>

#### 코드
Item(dto) 어노테이션 삭제<br>
ItemSaveForm, ItemUpdateForm dto 만들어 어노테이션 추가<br>
컨트롤러의 매개변수로 오는 객체 수정 - @Validated 있는 메서드(검증 메서드)에 저장폼메서드에는 ItemSaveForm, 수정폼메서드에는 ItemUpdateForm으로 수정<br>
주의<br>
> '@ModelAttribute ItemUpadateForm form' 으로 작성시 '@ModelAttribute("itemSaveForm") ItemUpadateForm form'으로 인식<br>
> 기존코드 수정하고 싶지 않으면 반드시 '@ModelAttribute("item") ItemUpadateForm form' 으로 작성<br>
> 또는 ("item") 작성하지 않고 html의 전달변수명? 수정<br>

기존 Item이 (ItemSaveForm으로 수정되어) 선언되어있지 않으므로 <br>
아래 성공로직에서 Item객체 생성하면 됨<br>

수정 코드도 동일하게 변경<br>

**이를 통해 폼 객체분리 완료**

## HTTP 메시지 컨버터
@RequestBody =  HTTP Body의 데이터를 객체로 변환할 때 사용, 주로 API JSON 요청을 다룰 때 사용<br>
@RestController = @RequestBody가 작성하지 않아도 적용됨<br>
컨트롤러 코드
```
if (bindingResult.hasErrors()) {
 log.info("검증 오류 발생 errors={}", bindingResult);
 return bindingResult.getAllErrors();
 }
```
즉 위 코드의 return bindingResult.getAllErrors(); 이 부분 결과가 자동으로 JSON 객체로 바뀜<br>

1. 포스트맨에서 아래와 같이 입력해 성공 여부 확인<br>
> POST http://localhost:8080/validation/api/items/add<br>
> {"itemName":"hello", "price":1000, "quantity": 10}<br><br>

2. 실패<br>
만약 숫자 넣는 부분에 문자형태로 입력하면?<br>
> {"itemName":"hello", "price":aaa, "quantity": 10}<br>
> 로그를 보면 객체 생성에 실패함을 알 수 있음<br>
> 객체를 만들지 못해 컨트롤러 자체가 호출되지 않음, 검증도 실행 안됨<br><br>

3. 검증 오류<br>
만약 검증 오류가 발생하도록 값을 넣으면?<br>
> {"itemName":"hello", "price":1000, "quantity": 10000}<br>
> 상단 컨트롤러 코드의 getAllErrors()가 JSON형태로 뿌려짐<br><br>

> 실제 개발할 때는 이 객체들을 그대로 사용하지 말고,<br>
> 필요한 데이터만 뽑아서 별도의 API 스펙을 정의하고 그에 맞는 객체를 만들어서 반환해야 한다.<br><br>

@ModelAttribute vs @RequestBody<br>
> @ModelAttribute 는 각각의 필드 단위로 세밀하게 적용해 특정 필드에 타입이 맞지 않는 오류가 발생해도 나머지 필드는 정상 처리할 수 있었다.<br>
> HttpMessageConverter 는 @ModelAttribute 와 다르게 각각의 필드 단위로 적용되는 것이 아니라, 전체 객체 단위로 적용된다.<br>
> @RequestBody 는 HttpMessageConverter 단계에서 JSON 데이터를 객체로 변경하지 못하면 <br>
> 이후 단계 자체가 진행되지 않고 예외가 발생한다. 컨트롤러도 호출되지 않고, Validator도 적용할 수 없다.<br>
> 이때 예외처리부분은 나중에 다룰 것이다.<br>
