## 프로젝트 생성
아래 주소에서 생성<br>
<https://start.spring.io> <br>
아래 JDK버전설정 참고
<https://study6-6.tistory.com/787>
<br>

## 간단한 예제
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

### hello.html 템플릿
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Hello</title>
</head>
<body>
    <h1>${data}</h1> <!-- Controller에서 주었던 addAttribute의 값으로 치환됨 -->
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
