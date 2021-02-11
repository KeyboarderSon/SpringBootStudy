###템플릿 엔진을 사용하여

<br>

```@GetMapping("hello")```
: /hello로 들어가면 웹어플리케이션에서  ```@GetMapping``` 하단부 메서드를 호출한다.<br>

```return "hello"```
: templates/hello.html 파일을 찾아 실행시켜라

```java

/*  src/main/java/hello-spring/controller(package)/HelloController(class) 생성  */
package hello.hellospring.controller;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class HelloController {

    @GetMapping("hello")
    public String hello(Model model){
        model.addAttribute("data", "hello!!");
        return "hello";

    }

}
```
<br>

---
<br>

```${data}```
:  위 contorller의 ```model.addAttribute("data", "hello!!");```에서의 key값에 해당한다. value에 해당하는 후자 hello!!로 치환된다.

 ```html
 <!-- src/main/resources/templates/hello.html -->

<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Hello</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8"/>
</head>
<body>
<p th:text="'안녕하세요. ' + ${data}">안녕하세요. 손님</p>
</body>
</html>
```
<br>
####실행화면
![사진](C:/Users/user/Desktop/output.JPG)