# 에러 및 해결책 정리

## 1. 타임리프

* org.thymeleaf.exceptions.TemplateInputException: Error resolving template 에러 발생

    * 로컬 개발 환경에서는 정상적으로 동작 하였지만, 리눅스 서버에 배포하면 컨트롤러에서 뷰 이름을 리턴할 때, 맨 앞에 `/`가 있어서 에러가 발생했다.
    
         ```java
        @GetMapping("/board/{id}/modify")
        public String updateBoardForm(@CurrentAccount Account account, @PathVariable Long id, Model model) {
            …
        
            return “/board/modify"
        }
         ```
      
    * 그래서 다음과 같이 맨 앞에 있는 `/`를 제거하여 해결하였다.
    
         ```java
        @GetMapping("/board/{id}/modify")
        public String updateBoardForm(@CurrentAccount Account account, @PathVariable Long id, Model model) {
            …
        
            return “board/modify"
        }
         ```

## 2. 데이터베이스

* `role "kevin" does not exist`라는 에러가 발생

    * 본인 컴퓨터에 먼저 `PostgreSQL`를 설치한 상태에서 (설치한 것을 잊고) 도커를 이용하여 `PostgreSQL`를 설치하고 접속한다면 해당 에러가 발생 할 수 있다.
    
    * 본인 컴퓨터에 설치 했던 `PostgreSQL`를 Stop 하거나 도커를 이용하여 PostgreSQL를 설치할 때 포트를 변경하여 설치하자.
    
## 3. 스프링 부트 

* 스프링 시큐리티 status: 999 페이지로 리다이렉트

    * 로그인을 할 때, 로그인은 되는데 `/error`로 리다이렉트 되며 아래와 같은 에러가 발생한다. 
    
        ```
        {"timestamp":"2021-03-13T08:36:50.301+0000","status":999,"error":"None","message":"No message available"}
        ```
  
    * 로그인을 할 때 사용되는 정적 리소스를 찾지 못하면 에러 페이지로 리다이렉트 된다고 한다. 스프링 시큐리티 설정에서 다음 내용을 추가해서 해결하자. 
    
        ```
        @Configuration
        @EnableWebSecurity
        @RequiredArgsConstructor
        public class SecurityConfig extends WebSecurityConfigurerAdapter {
        
            @Override
            public void configure(WebSecurity web) throws Exception {
                // 스택 오버 플로우 답변
                //web.ignoring().antMatchers("/favicon.ico", "/resources/**", "/error");
      
                // 내 프로젝트에 실제 추가한 코드
                web.ignoring().antMatchers("/error");
            }
        }
        ```
        
        * [참고] https://stackoverflow.com/questions/61029340/spring-security-redirects-to-page-with-status-code-999/61029341
         
