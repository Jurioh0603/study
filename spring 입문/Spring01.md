## 프로젝트 생성
아래 주소에서 생성<br>
<https://start.spring.io> <br>
아래 JDK버전설정 참고 <br>
<https://study6-6.tistory.com/787>
<br>

## Welcome page 만들기
### HelloController 생성

```
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class HelloController {

    @GetMapping("/hello")
    public String hello(Model model) {
        // Model 객체 가져와 attributeName과 value값 지정
        model.addAttribute("data", "Hello, World!");
        return "hello"; // hello.html 템플릿으로 반환
    }
}
```
<br>

### index.html 템플릿
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
Hello
<a href="/hello">hello</a>
</body>
</html>
```
<br>

### hello.html 템플릿
```
<!DOCTYPE html>
<html xmlns:th="http://thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Hello</title>
</head>
<body>
Hello
<p th:text="'안녕하세요. ' + ${data}" >안녕하세요. 손님</p>
</body>
</html>
```
<br>

### 스프링 부트 템플릿 엔진 설정
스프링 부트의 템플릿 엔진은 resources/templates/ 디렉토리에서 뷰를 찾습니다.

<br>

### 참고
* spring-boot-devtools 라이브러리를 추가하면, html 파일을 컴파일만 해주면 서버 재시작 없이 View 파일 변경이 가능합니다.
* 인텔리제이에서 컴파일 방법: 메뉴 build -> Recompile

<br>

## 치환
http://localhost:8081/hello-mvc?name=Spring!!
<br>
다음 주소를 입력하면 주소 뒤의 name=~~가 ${name}자리에 출력된다
```
<!DOCTYPE html>
<html xmlns:th="http://thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<p th:text="'hello ' + ${name}" >hello! empty</p>
</body>
</html>
```

