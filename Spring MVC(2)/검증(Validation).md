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
```

> Safe Navigation Operator
> 만약 errors가 null이라면?
> 등록 폼에 진입한 시점에는 error가 없다 따라서 errors.containsKey()를 호출하면 NullPointerException이 발생
> 이때 문법중 errors?.는 error가  null일 때 NullPointerException대신 null을 반환하는 문법임
> 이에 결과로 th:if에서 null은 실패로 처리되므로 오류메시지가 출력되지 않음
<br>
> 정리
> 만약 검증 오류 발생 시 입력폼 다시보여줌
> 검증 오류를 고객에게 안내해 다시입력할 수 있도록
> 검증 오류가 발생해도 고객이 입력한 데이터가 유지됨
<br>
> 남은 문제점
> 뷰 템플릿 중복이 많음
> 타입오류처리가 안됨 - 숫자타입을 문자타입으로 설정하면 오류발생,
> 이는 스프링 MVC에서 컨트롤러에 진입하기 전 예외 발생하기 때문에 컨트롤러 호출되지 않고 400예외 발생
> 타입 오류 발생시 고객이 입력한 문자 화면에 남겨야함, 그러나 숫자 타입에 문자 입력시 문자 바인딩이 불가능 해서 보관할 수 없음,
> 결국 고객은 어떤 내용이 오류가 발생했는지 알기 어려움
> 고객이 입력한 값도 어딘에가 별도로 관리되어야 함
<br>
아래는 스프링이 제공하는 검증 방법들이다.
