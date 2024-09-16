## 검증 요구사항
사용자가 입력한 내용을 검증하고 유효하지 않으면 다시 폼으로 돌아오고 메시지를 띄움<br>
상품명 : 필수<br>
가격 : 1,000 ~ 1,000,000<br>
수량 : 1,000 ~ 9,999<br>
기타 : 가격 * 수량의 합은 10,000원 이상<br>
> 클라이언트 검증, 서버 검증 모두 이루어져야 함<br>
##  검증로직
#### 상품저장성공
사용자가 상품 등록 폼에서 정상 범위의 데이터를 입력하면 서버에서 검증 로직 통과, 상품 저장, 상품상세화면으로 redirect<br>

#### 상품저장실패
상품 등록 폼에서 검증에 실패한 경우 고객에게 다시 상품 등록 폼 보여주고 어떤 값이 잘못되었는지 알려줘야함<br>

## 코드
controller
```
 @PostMapping("/add")
 public String addItem(@ModelAttribute Item item, RedirectAttributes 
redirectAttributes, Model model) {
 //검증 오류 결과를 보관
Map<String, String> errors = new HashMap<>();
 //검증 로직
if (!StringUtils.hasText(item.getItemName())) {
        errors.put("itemName", "상품 이름은 필수입니다.");
    }
 if (item.getPrice() == null || item.getPrice() < 1000 || item.getPrice() > 1000000) {
        errors.put("price", "가격은 1,000 ~ 1,000,000 까지 허용합니다.");
    }
 if (item.getQuantity() == null || item.getQuantity() > 9999) {
        errors.put("quantity", "수량은 최대 9,999 까지 허용합니다.");
    }
 //특정 필드가 아닌 복합 룰 검증
if (item.getPrice() != null && item.getQuantity() != null) {
 int resultPrice = item.getPrice() * item.getQuantity();
 if (resultPrice < 10000) {
            errors.put("globalError", "가격 * 수량의 합은 10,000원 이상이어야 합니다. 현재 값 = " + resultPrice);
        }
    }
 //검증에 실패하면 다시 입력 폼으로
if (!errors.isEmpty()) {
        model.addAttribute("errors", errors);
 return "validation/v1/addForm";
    }
 //성공 로직
Item savedItem = itemRepository.save(item);
    redirectAttributes.addAttribute("itemId", savedItem.getId());
    redirectAttributes.addAttribute("status", true);
 return "redirect:/validation/v1/items/{itemId}";
 }
```
html
```
<div th:if="${errors?.containsKey('globalError')}">
    <p class="field-error" th:text="${errors['globalError']}">전체 오류 메시지</p>
</div>
<input type="text" id="quantity" th:field="*{quantity}" th:class="${errors?.containsKey('quantity')} ? 'form-control 
field-error' : 'form-control'" class="form-control" placeholder="수량을 입력하세요">
<div class="field-error" th:if="${errors?.containsKey('quantity')}" th:text="${errors['quantity']}">수량 오류</div>
```

> Safe Navigation Operator<br>
> > 만약 errors가 null이라면?<br>
> > 등록 폼에 진입한 시점에는 error가 없다 따라서 errors.containsKey()를 호출하면 NullPointerException이 발생<br>
> > 이때 문법중 errors?.는 error가  null일 때 NullPointerException대신 null을 반환하는 문법임<br>
> > 이에 결과로 th:if에서 null은 실패로 처리되므로 오류메시지가 출력되지 않음
<br>

> 정리<br>
> > 만약 검증 오류 발생 시 입력폼 다시보여줌<br>
> > 검증 오류를 고객에게 안내해 다시입력할 수 있도록<br>
> > 검증 오류가 발생해도 고객이 입력한 데이터가 유지됨
<br>

> 남은 문제점<br>
> > 뷰 템플릿 중복이 많음<br>
> > 타입오류처리가 안됨 - 숫자타입을 문자타입으로 설정하면 오류발생,<br>
> > 이는 스프링 MVC에서 컨트롤러에 진입하기 전 예외 발생하기 때문에 컨트롤러 호출되지 않고 400예외 발생<br>
> > 타입 오류 발생시 고객이 입력한 문자 화면에 남겨야함, 그러나 숫자 타입에 문자 입력시 문자 바인딩이 불가능 해서 보관할 수 없음,<br>
> > 결국 고객은 어떤 내용이 오류가 발생했는지 알기 어려움<br>
> > 고객이 입력한 값도 어딘에가 별도로 관리되어야 함
<br>

아래는 스프링이 제공하는 검증 방법들이다.

## Binding Result1
BindingResult 매개변수로 선언, model.addAttribute(errors)를 하지 않아도됨(각각의 조건문에서 binding함)<br>
문법 :  bindingResult.addError(new FieldError("item", "itemName", "상품 이름은 필수입니다."));<br>

> FieldError<br>
> > objectName = @ModelAttribute이름<br>
> > field = 오류가 발생한 필드 이름<br>
> > defaultMessage = 오류 기본 메시지<br>
> ObjectError<br>
> > objectName = @ModelAttribute이름<br>
> > defaultMessage = 오류 기본 메시지<br>
controller
```
@PostMapping("/add")
 public String addItemV1(@ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirectAttributes) {
  if (!StringUtils.hasText(item.getItemName())) {
   bindingResult.addError(new FieldError("item", "itemName", "상품 이름은 필수입니다."));
  }
 if (item.getPrice() == null || item.getPrice() < 1000 || item.getPrice() > 1000000) {
   bindingResult.addError(new FieldError("item", "price", "가격은 1,000 ~ 1,000,000 까지 허용합니다."));
 }
 if (item.getQuantity() == null || item.getQuantity() >= 10000) {
   bindingResult.addError(new FieldError("item", "quantity", "수량은 최대 9,999 까지 허용합니다."));
 }
 //특정 필드 예외가 아닌 전체 예외
 if (item.getPrice() != null && item.getQuantity() != null) {
   int resultPrice = item.getPrice() * item.getQuantity();
   if (resultPrice < 10000) {
     bindingResult.addError(new ObjectError("item", "가격 * 수량의 합은 10,000원 이상이어야 합니다. 현재 값 = " + resultPrice));
   }
 }
 if (bindingResult.hasErrors()) {
   log.info("errors={}", bindingResult);
   return "validation/v2/addForm";
 }
 //성공 로직
 Item savedItem = itemRepository.save(item);
 redirectAttributes.addAttribute("itemId", savedItem.getId());
 redirectAttributes.addAttribute("status", true);
 return "redirect:/validation/v2/items/{itemId}";
 }
 ```
html
 ```
 <div th:if="${#fields.hasGlobalErrors()}">
  <p class="field-error" th:each="err : ${#fields.globalErrors()}" th:text="${err}">글로벌 오류 메시지</p>
 </div>
<input type="text" id="quantity" th:field="*{quantity}" th:errorclass="field-error" class="form-control" placeholder="수량을 입력하세요">
  <div class="field-error" th:errors="*{quantity}">수량 오류</div>
 ```
> 주의 BindingResult bindingResult 파라미터의 위치는  @ModelAttribute Item item 다음에 와야 함

## Binding Result2
> binding 이 있으면 숫자 타입에 문자를 입력했을 때 400에러 발생하지 않고 폼으로 돌아오고 오류메시지 출력. 즉 컨트롤러 호출하게 해줌.
<br>

> 주의<br>
> > binding result는 순서가 중요. 검증할 대상인 @ModelAttribute Item item 바로 다음에 와야한다.
<br>

> Binding Result는 Errors 인터페이스를 상속받고 있음<br>
> Errors 인터페이스는 단순한 오류 저장과 조회 기능 제공<br>
> Binding Result는 더 추가적인 기능을 제공<br>
<br>

그런데 Binding Result로 변경하고 나서는 사용자가 잘못된 값을 입력해서 에러가 발생했을 때 입력값이 사라지는 이슈가 발생함<br>
이를 해결하기 위한 내용은 아래에 있음.<br>

## FieldError, ObjectError
### fieldError
> objectName = 오류가 발생한 객체 이름<br>
> field = 오류 필드<br>
> rejectedValue = 사용자가 입력한 값(거절된 값)<br>
> bindingFailure = 타입 오류 같은 바인딩 실패인지, 검증 실패인지 구분 값<br>
> codes = 메시지 코드<br>
> arguments = 메시지에서 사용자는 인자<br>
> defaultMessage = 기본오류메시지<br>
```
 @PostMapping("/add")
 public String addItemV2(@ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirectAttributes) {
   if (!StringUtils.hasText(item.getItemName())) {
     bindingResult.addError(new FieldError("item", "itemName", item.getItemName(), false, null, null, "상품 이름은 필수입니다."));
   }
   if (item.getPrice() == null || item.getPrice() < 1000 || item.getPrice() > 1000000) {
     bindingResult.addError(new FieldError("item", "price", item.getPrice(), false, null, null, "가격은 1,000 ~ 1,000,000 까지 허용합니다."));
   }
   if (item.getQuantity() == null || item.getQuantity() >= 10000) {
     bindingResult.addError(new FieldError("item", "quantity", 
     item.getQuantity(), false, null, null, "수량은 최대 9,999 까지 허용합니다."));
   }
   //특정 필드 예외가 아닌 전체 예외
   if (item.getPrice() != null && item.getQuantity() != null) {
     int resultPrice = item.getPrice() * item.getQuantity();
     if (resultPrice < 10000) {
       bindingResult.addError(new ObjectError("item", null, null, "가격 * 수량의 합은 10,000원 이상이어야 합니다. 현재 값 = " + resultPrice));
     }
   }
   if (bindingResult.hasErrors()) {
     log.info("errors={}", bindingResult);
     return "validation/v2/addForm";
   }
   //성공 로직
   Item savedItem = itemRepository.save(item);
   redirectAttributes.addAttribute("itemId", savedItem.getId());
   redirectAttributes.addAttribute("status", true);
   return "redirect:/validation/v2/items/{itemId}";
 }
```
### 오류 발생시 사용자가 입력하는 값 유지 해주는 코드 부분
item.getPrice()
```
 new FieldError("item", "price", item.getPrice(), false, null, null, "가격은 1,000 ~ 1,000,000 까지 허용합니다.")
```

### 타임리프 사용자의 입력값 유지
th:field="*{price}"<br>
타임리프의 th:field는 정상상황에서 모델 객체의 값을 사용하는데 오류가 발생하면 fieldError에서 보관한 값을 사용하여 값을 출력한다.<br>
따라서 타입 오류로 바인딩 실패하면 fieldError를 생성하면서 사용자가 입력한 값을 넣어두고, 해당 오류를 BindingResult에 담아 컨트롤러 호출한다.<br>
즉 바인딩 실패시에도 사용자의 오류 메시지를 정상 출력할 수 있다.<br>

## 오류메시지 처리
errors.properties 파일에서 에러메시지 입력<br>
> ex) application.properties<br>
> > spring.messages.basename=messages,errors<br>
> ex) errors.properties<br>
> >  required.item.itemName=상품 이름은 필수입니다.<br>
> > range.item.price=가격은 {0} ~ {1} 까지 허용합니다<br>
> >  max.item.quantity=수량은 최대 {0} 까지 허용합니다.<br>
> > totalPriceMin=가격 * 수량의 합은 {0}원 이상이어야 합니다. 현재 값 = {1}<br>
=> 전달된 값으로 치환된다<br>
코드 변경된 부분<br>
```
// range.item.price=가격은 {0} ~ {1} 까지 허용합니다.
  if (item.getPrice() == null || item.getPrice() < 1000 || item.getPrice() > 1000000) {
    bindingResult.addError(new FieldError("item", "price", item.getPrice(), false, new String[]{"range.item.price"},
    new Object[]{1000, 1000000}, null));
  }
 
// max.item.quantity=수량은 최대 {0} 까지 허용합니다.
 if (item.getQuantity() == null || item.getQuantity() > 10000) {
  bindingResult.addError(new FieldError("item", "quantity", item.getQuantity(), false, new String[]{"max.item.quantity"},
  new Object[]{9999}, null));
 }
// totalPriceMin=가격 * 수량의 합은 {0}원 이상이어야 합니다. 현재 값 = {1}
if (item.getPrice() != null && item.getQuantity() != null) {
 int resultPrice = item.getPrice() * item.getQuantity();
 if (resultPrice < 10000) {
   bindingResult.addError(new ObjectError("item", new String[]{"totalPriceMin"}, new Object[]{10000, resultPrice}, null));
        
```

## 오류 메시지 처리2
오류코드를 좀 더 자동화 시켜보자<br>
BindingResult는 검증해야할 객체인 target바로 다음에 오기 때문에 BindingResult는 이미 검증할 객체인 target을 알고 있다<br>
<br>
rejectValue()와, reject()를 사용하여 자동화 시켜볼 수 있다
```
if (!StringUtils.hasText(item.getItemName())) {
  bindingResult.rejectValue("itemName", "required");
}
if (item.getPrice() == null || item.getPrice() < 1000 || item.getPrice() > 1000000) {
  bindingResult.rejectValue("price", "range", new Object[]{1000, 1000000}, null);
}
if (item.getQuantity() == null || item.getQuantity() > 10000) {
  bindingResult.rejectValue("quantity", "max", new Object[]{9999}, null);
}
//특정 필드 예외가 아닌 전체 예외
if (item.getPrice() != null && item.getQuantity() != null) {
  int resultPrice = item.getPrice() * item.getQuantity();
  if (resultPrice < 10000) {
     bindingResult.reject("totalPriceMin", new Object[]{10000, resultPrice}, null);
  }
}
```

> rejectValue()<br>
> void rejectValue(@Nullable String field, String errorCode,<br>
> @Nullable Object[] errorArgs, @Nullable String defaultMessage);<br>
> > field = 오류필드명(html파일에서 지정)<br>
> > errorCode = 오류코드<br>
> > errorArgs = 오류 메시지에서 {0}을 치환하기 위한 값<br>
> > defaultMessage = 오류메시지를 찾을 수 없을 때 사용하는 기본 메시지<br>
> > bindingResult.rejectValue("price", "range", new Object[]{1000, 1000000}, null)<br>
> > >  FieldError() 를 직접 다룰 때는 오류 코드를 range.item.price와 같이 모두 입력했지만<br>
> > > rejectValue()를 사용하고부터는 간단하게 range만 입력했다 이에 대한 내용은 아래에서 자세히 설명한다.<br>

## 오류 메시지 처리3
위처럼 properties파일에 'required.item.itemName=상품 이름은 필수입니다.'라고 되어있다고 가정했을 때 <br>
전체 이름을 입력하지 않고 required라고만 입력해도 오류메시지가 출력되는 이유는 properties 파일이 아래와 같다고 가정했을 때 <br>
> 'required.item.itemName=상품 이름은 필수입니다.'<br>
> 'required=필수입니다'<br>
컨트롤러에서 아래와 같이 호출시<br>
> 'bindingResult.rejectValue(~~,"required")'<br>
reject는 가장먼저 'required.item.itemName'의 메시지를 찾고 만약 내용이 없다면 <br>
'required'의 메시지를 찾는다.<br>
즉 개발할 때 메시지의 내용을 더 세밀하게 바꾸어야 할 때 편리하게 메시지 내용을 변경할 수 있다.<br>
## MessageCodesResolver에 대하여
> properties 파일<br>
> required=필수값입니다.<br>
> required.java.lang.String=필수문자입니다.<br>
> required.item.itemName=상품명은 필수항목입니다.<br>
테스트 코드에서 실행<br>
아래코드를 실행해보면 messageCodes 안에는 "required.item","required"가 <br>
순서대로 들어있는 것을 확인할 수 있다.<br>
> MessageCodesResolver codesResolver = new DefaultMessageCodesResolver();<br>
> String[] messageCodes = codesResolver.resolveMessageCodes("required", "item");<br>
> assertThat(messageCodes).containsExactly("required.item", "required");<br>
아래코드를 실행해보면 테스트가 통과되는 것을 확인할 수 있다.<br>
> String[] messageCodes = codesResolver.resolveMessageCodes("required", "item", "itemName", String.class);<br>
> assertThat(messageCodes).containsExactly(<br>
> "required.item.itemName",<br>
> "required.itemName",<br>
> "required.java.lang.String",<br>
> "required"<br>
> );
**즉 세밀한 것부터 메시지를 배열에 담는다는 것을 알 수 있음!**
> 순서는 세밀 - 타입 - 기본<br>
> 타입이 붙었을 때 순서를 주의하자<br>
> (참고)다음과 같은 코드로도 작성할 수 있음 ValidationUtils.rejectIfEmptyOrWhitespace(bindingResult, "itemName", "required");<br>
정리<br>
> rejectValue()를 호출<br>
> MessageCodeResolver를 사용해 검증 오류 코드로 메시지 코드들을 생성<br>
> new FieldError()를 생성하면서 메시지 코드들 보관<br>
> th:errors에서 메시지 코드들로 메시지를 순서대로 찾고 출력<br>
## 스프링이 직접 만든 오류메시지 처리<br>
숫자에 문자열 입력했을 때 스프링에서 생성한 장문의 에러 메세지의 로그를 보면 <br>
typeMismatch.item.price, typeMismatch.price, typeMismatch.java.lang.Integer, typeMismatch라고 되어있음 <br>
위 중에서 원하는 것을 골라 properties 파일에 작성하면 됨<br>
> typeMismatch.item.price=숫자를 입력해주세요.<br>
> typeMismatch=타입오류입니다.<br>
## Validator 분리
1. 검증 로직이 너무 길어 정상 값이 입력 되었을 때 실행되는 코드를 찾기 어려움<br>
2. 위의 검증 로직을 따로 분리하는 컨트롤러를 생성<br>

Validator 클래스 생성
```
@Component
public class ItemValidator implements Validator {
  @Override
  public boolean supports(Class<?> clazz) {
    return Item.class.isAssignableFrom(clazz);
  }
  @Override
  public void validate(Object target, Errors errors) {
    Item item = (Item) target;
    ValidationUtils.rejectIfEmptyOrWhitespace(errors, "itemName", "required");
    if (item.getPrice() == null || item.getPrice() < 1000 || item.getPrice() > 1000000) {
      errors.rejectValue("price", "range", new Object[]{1000, 1000000}, null);
    }
    if (item.getQuantity() == null || item.getQuantity() > 10000) {
      errors.rejectValue("quantity", "max", new Object[]{9999}, null);
    }
    //특정 필드 예외가 아닌 전체 예외
    if (item.getPrice() != null && item.getQuantity() != null) {
      int resultPrice = item.getPrice() * item.getQuantity();
      if (resultPrice < 10000) {
      errors.reject("totalPriceMin", new Object[]{10000, resultPrice}, 
     null);
      }
    }
  }
}
```
코드설명<br>
> 이때 support메서드는 검증기를 지원하는 여부를 확인하는 메서드인데 이는 다음장에서 설명함<br>
> validate(Object target, Errors errors) : 검증 대상 객체와 BindingResult임<br>
> @Component를 통해 빈을 주입해 객체를 생성하지 않아도 호출가능하게 함<br>
> 컨트롤러에 있던 코드들 복사해서 가져오고, bindingResult.rejectValue를 errors.rejectValue로 변경<br>
> 이게 가능한 이유는 Errors 인터페이스가 BindingResult 인터페이스를 상속받고 있기 때문<br>
컨트롤러에서 Validator클래스 호출하기
```
private final ItemValidator itemValidator;
메서드(){
  itemValidator.validate(item, bindingResult);
}
```
다음과 같이 검증 대상 객체, bindingResult 순서대로 매개변수 입력<br>

## Validator를 더 간단하게 사용하기
컨트롤러에 아래 내용 추가
```
@InitBinder
public void init(WebDataBinder dataBinder) {
 log.info("init binder {}", dataBinder);
 dataBinder.addValidators(itemValidator);
}
```
위 검증기를 추가하면 해당 컨트롤러에서는 검증기를 자동으로 적용할 수 있다.<br>
@InitBinder 해당 컨트롤러에만 영향을 준다. 글로벌 설정은 별도<br>

컨트롤러에서 검증기를 수행할 객체 앞에 '@Validated' 어노테이션 추가<br>
> public String addItemV6(@Validated @ModelAttribute Item item)<br>
동작 방식 <br>
> @Validated 는 검증기를 실행하라는 애노테이션<br>
> 이 애노테이션이 붙으면 앞서 WebDataBinder 에 등록한 검증기(itemValidator)를 찾아서 실행<br>
> 여러 검증기를 등록한다면 그 중에 어떤 검증기가 실행되어야 할지 구분이 필요<br>
> 이때 supports() 가 사용, supports(Item.class) 호출되고 결과가 true이므로 ItemValidator 의 validate() 가 호출<br>
#### 글로벌 설정 - 모든 컨트롤러에 적용
실행클래스(application)에 아래 코드 추가(컨트롤러에 검증기 수행할 객체 앞에 @Vaildated는 꼭 붙이기)<br>
```
@Override
 public Validator getValidator() {
 return new ItemValidator();
}
```
참고<br>
> 글로벌 설정은 잘 사용되지 않는다.<br>
> 검증시 @Validated @Valid 둘다 사용가능하다.<br>
> 그러나 @Valid 사용하려면 의존관계 주입이 필요<br>
> implementation 'org.springframework.boot:spring-boot-starter-validation<br>
