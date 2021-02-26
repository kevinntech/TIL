# 타임리프(Thymeleaf)
> 아래 내용은 타임리프에서 자주 사용되는 부분을 정리 하였습니다.

## 1. 프로젝트 만들기

* https://start.spring.io/ 에서 다음과 같이 프로젝트를 생성 할 수 있다. 

    ![image 1](images/img1.png)
    
## 2. 타임리프 동작 확인하기

* (1) 컨트롤러 작성

    ```java
    @Controller
    public class HelloController {
    
        @GetMapping("/hello")
        public String hello(Model model){
            model.addAttribute("hello", "Hello Thymeleaf !!!");
    
            return "hello";
        }
    
    }
    ```

* (2) 템플릿 페이지 작성하기

    * ① 타임리프 네임 스페이스를 선언
    
        ```html
        <html lang="en" xmlns:th="http://www.thymeleaf.org">
        ```

    * ② 타임리프를 이용한 템플릿 페이지(`hello.html`)를 작성한다.

        ```html
        <!DOCTYPE html>
        <html lang="en" xmlns:th="http://www.thymeleaf.org">
        <head>
            <meta charset="UTF-8">
            <title>Thymeleaf Test</title>
        </head>
        <body>
        <h1 th:text="${hello}">Thymeleaf</h1>
        </body>
        </html>
        ```

## 3. 타임리프 문법 살펴보기

* (1) 준비하기
 
    * 앞으로 살펴볼 예제에서 사용할 코드를 작성한다. 
 
        * ① `SignUpForm`을 작성한다. 
    
            ```java
            @Data
            @AllArgsConstructor
            public class SignUpForm {
            
                private String id;
            
                private String password;
            
                private String name;
            
                private LocalDateTime createdDate;
            
            }
    
            ```
    
        * ② `MemberController` 클래스를 작성한다.
        
            ```java
            @Controller
            public class MemberController {
            
                @GetMapping("/sign-up")
                public String hello(Model model){
            
                    SignUpForm signUpForm = new SignUpForm(1L, "123456", "관리자", LocalDateTime.now());
            
                    model.addAttribute("signUpForm", signUpForm);
            
                    return "sign-up";
          
                }
            
            }
            ```
          
* (2) 객체를 화면에 출력하기 (`sign-up.html`)

    * `th:text`는 태그의 내용물로 문자열을 출력한다.
  
        ```html
        <!DOCTYPE html>
        <html lang="en" xmlns:th="http://www.thymeleaf.org">
        <head>
            <meta charset="UTF-8">
            <title>Thymeleaf Test</title>
        </head>
        <body>
        <h1 th:text="${signUpForm}">데이터 없음</h1>
        </body>
        </html>
        ```
      
* (3) HTML 출력하기 (`sign-up.html`)

    * `th:utext`는 태그의 내용물로 HTML 자체를 출력한다.
  
        ```html
        <div th:utext='${"<h1>" + signUpForm.name + "</h1>"}'></div>
        ```

* (4) 리스트를 화면에 출력하기 (`sign-up.html`)

    * `MemberController`에서 여러 개의 `signUpForm` 객체를 리스트에 담아서 뷰로 전달한다.
  
        ```java
        @Controller
        public class MemberController {
        
            @GetMapping("/sign-up")
            public String hello(Model model){
        
                List<SignUpForm> signUpForms = new ArrayList<>();
        
                for(long i = 0; i < 100; i++){
                    SignUpForm signUpForm = new SignUpForm(i, "123456", "사용자 " + i, LocalDateTime.now());
                    signUpForms.add(signUpForm);
                }
        
                model.addAttribute("signUpForms", signUpForms);
        
                return "sign-up";
            }
        
        }
        ```

    * 모델(Model)에 담아서 뷰로 전달한 리스트는 `th:each`를 이용하여 반복해서 출력한다.
  
        ```html
        <!DOCTYPE html>
        <html lang="en" xmlns:th="http://www.thymeleaf.org">
        <head>
            <meta charset="UTF-8">
            <title>Thymeleaf Test</title>
        </head>
        <body>
            <table border="1">
                <tr>
                    <td>아이디</td>
                    <td>패스워드</td>
                    <td>이름</td>
                    <td>생성일</td>
                </tr>
        
                <tr th:each="signUpForm : ${signUpForms}">
                    <td th:text="${signUpForm.id}">아이디</td>
                    <td th:text="${signUpForm.password}">패스워드</td>
                    <td th:text="${signUpForm.name}">이름</td>
                    <td th:text="${signUpForm.createdDate}">생성일</td>
                </tr>
            </table>
        </body>
        </html>
        ```
      

