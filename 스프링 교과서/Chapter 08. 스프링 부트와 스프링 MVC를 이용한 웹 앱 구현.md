# Chapter 08. 스프링 부트와 스프링 MVC를 이용한 웹 앱 구현

- 템플릿 엔진: 컨트롤러가 전송하는 가변 데이터를 쉽게 가져와 표시할 수 있게 해 주는 의존성이다.

## 8.1 동적 뷰를 사용한 웹 앱 구현

- 동일한 웹 페이지지만, 사용자마다 다른 뷰를 제공하는 것을 동적 뷰라 한다.
- 동적 뷰를 정의하려면 컨트롤러는 뷰에 데이터를 전송해야 한다.

### 8.1.1 HTTP 요청에서 데이터 얻기

> 대부분은 HTTP 요청으로 데이터를 전송할 때 다음 방식 중 하나를 사용한다.
> #### HTTP 요청 매개변수(request parameter)
>   - 키-값 쌍으로 된 형식으로 클라이언트에서 서버로 값을 전송하는 간단한 방식.
>   - **쿼리 매개변수**라고도 한다.
>   - 소량의 데이터를 전송할 때만 사용해야 한다.
> #### HTTP 요청 헤더(request header)
>   - 요청 헤더가 HTTP 헤더로 전송된다는 점에서 요청 매개변수와 유사하다. URI에 표시되지 않는다는 것이 차이점이다.
>   - 소량의 데이터를 전송할 때 사용한다.
> #### 경로 변수(path variable)
>   - 요청 경로 자체를 이용하여 데이터를 전송한다.
>   - 소량의 데이터를 전송할 때 사용한다.
> ####  HTTP 요청 본문(request body)
>   - 대량의 데이터를 전송하는 데 사용된다.

### 8.1.2 클라이언트에서 서버로 데이터를 전송하려고 요청 매개변수 사용

다음과 같은 시나리오에서 사용된다.
- 전송하는 데이터양이 많지 않을 때
- 필수가 아닌 데이터를 보내야 할 때

```java
// 요청 매개변수로 값 얻기

@Controller
public class MainController {

    @RequestMapping("/home")
    public String home(@RequestParam String color, Model page) {
        page.addAttribute("username", "katy");
        page.addAttribute("color", color);
        return "home.html";
    }
}
```

### 8.1.3 경로 변수로 클라이언트에서 서버로 데이터 전송

- 경로 변수는 키로 값을 식별하지 않는다.
- 경로 변수로 값 두 개를 제공할 수 있지만, 일반적으로 복수 값은 사용하지 않는 편이 좋다.
- 필수 매개변수에만 경로 변수를 사용하는 편이 좋다.

```java
// 경로 변수 사용하기

@Controller
public class MainController {

    @RequestMapping("/home/{color}")
    public String home(@PathVariable String color, Model page) {
        page.addAttribute("username", "katy");
        page.addAttribute("color", color);
        return "home.html";
    }
}
```

## 8.2 HTTP GET과 POST 메서드 사용

- 설계된 목적과 다르게 HTTP 메서드를 사용할 수 있지만 이는 올바르지 않다.

> 웹 앱에서 자주 사용하는 기본 HTTP 메서드
> - GET: 클라이언트 요청은 데이터 조회만 한다.
> - POST: 클라이언트 요청은 서버가 추가할 새로운 데이터를 전송한다.
> - PUT: 클라이언트 요청은 서버에 있는 데이터 레코드를 변경한다.
> - PATCH: 클라이언트 요청은 서버에 있는 데이터 레코드를 부분적으로 변경한다.
> - DELETE: 클라이언트 요청은 서버에 있는 데이터를 삭제한다.

- @RequestMapping 어노테이션은 기본적으로 HTTP GET을 사용한다.
- 컨트롤러 동적의 매개변수로 Model이 아닌 직접 정의한 클래스를 컨트롤러 액션의 매개변수로 직접 사용할 수 있다. 다만 매개변수의 인스턴스를 생성할 수 있도록 기본 생성자가 있어야 한다. 

## 8.3 요약

- 동적 페이지는 요청에 따라 다른 콘텐츠를 표시할 수 있다.
- 동적 뷰는 표시할 정보를 알려고 컨트롤러에서 변수 데이터를 가져온다.
- 스프링 앱에서 동적 페이지를 구현하는 쉬운 방법은 타임리프(Thymeleaf) 같은 템플릿 엔진을 사용하는 것이다. 타임리프 외엔 머스태치(Mustache), 프리마커(FreeMarker), 자바 서버 페이지(JSP)가 있다.
- 템플릿 엔진은 컨트롤러가 전송하는 데이터를 쉽게 가져와서 뷰에 표시할 수 있는 기능을 앱에 제공하는 의존성이다.
- 클라이언트가 전송하는 필수 데이터에 대해 경로 변수를 사용해야만 한다.
- 경로와 HTTP 메서드로 HTTP 요청을 식별한다. 프로덕션 앱에서 자주 볼 수 있는 필수 HTTP 메서드는 GET, POST, PUT, PATCH, DELETE다.
