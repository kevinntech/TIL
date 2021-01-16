# 백기선님의 스프링과 JPA 기반 웹 애플리케이션 개발
> 아래 내용은 [스프링과 JPA 기반 웹 애플리케이션 개발](https://www.inflearn.com/course/스프링-JPA-웹앱# "스프링과 JPA 기반 웹 애플리케이션 개발") 강좌를 정리한 내용 입니다.

## 1. 프로젝트 만들기

* https://start.spring.io/ 에서 다음과 같이 프로젝트를 생성 할 수 있다. 

    ![image 1](images/img1.png)
    
    * 스프링 부트
    * 스프링 웹 MVC
    * 타임리프 (뷰 템플릿)
    * 스프링 시큐리티
    * 스프링 데이터 JPA
    * H2
    * PostgreSQL
    * 롬복
    * 스프링 mail
    * 스프링 부트 devtools

## 2. 스프링

#### 1) 회원가입

* (1) 계정 도메인 만들기

    * `domain` 패키지를 생성한다.

    * 해당 패키지에 Account 클래스를 작성한다.
    
         ```java
        @Entity
        @Getter @Setter @EqualsAndHashCode(of = "id") // 무한루프 방지
        @Builder @AllArgsConstructor @NoArgsConstructor
        public class Account {
        
            @Id @GeneratedValue
            private Long id;
        
            @Column(unique = true)
            private String email;
        
            @Column(unique = true)
            private String nickname;
        
            private String password;
        
            private boolean emailVerified;
        
            // 이메일을 검증할 때, 사용 할 토큰 값을 저장하는 필드를 선언
            private String emailCheckToken;
        
            // 가입 날짜
            private LocalDateTime joinedAt;
        
            // 짧은 자기 소개
            private String bio;
        
            private String url;
        
            private String occupation;
        
            // 살고 있는 지역
            private String location;
        
            @Lob @Basic(fetch = FetchType.EAGER)
            private String profileImage;
        
            private boolean studyCreatedByEmail;
        
            private boolean studyCreatedByWeb;
        
            private boolean studyEnrollmentResultByEmail;
        
            private boolean studyEnrollmentResultByWeb;
        
            private boolean studyUpdatedByEmail;
        
            private boolean studyUpdatedByWeb;
        
        }
         ```

* (2) 컨트롤러 만들기

    *  `account` 패키지를 생성한다.

    *  AccountController를 작성한다.
    
         ```java
        @Controller
        public class AccountController {
        
            @GetMapping("/sign-up")
            public String signUpForm(Model model){
                return "account/sign-up";
            }
        
        }
         ```
 
* (3) 뷰 만들기 (templates/account/sign-up) 

 ```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
Sign Up page
</body>
</html>
 ```

* (4) 스프링 시큐리티 설정하기

    * `config` 패키지를 생성한다. 
    
    * `SecurityConfig` 클래스를 작성한다.
    
         ```java
        @Configuration
        @EnableWebSecurity
        public class SecurityConfig extends WebSecurityConfigurerAdapter {
        
            @Override
            protected void configure(HttpSecurity http) throws Exception {
        
                http.authorizeRequests()
                        // 권한 확인 없이 접근 가능해야 함
                        .mvcMatchers("/", "/login", "/sign-up", "/check-email", "/check-email-token",
                                "/email-login", "/check-email-login", "/login-link").permitAll()
                        // 프로필은 HTTP GET 요청만 가능하며
                        .mvcMatchers(HttpMethod.GET, "/profile/*").permitAll()
                        // 나머지는 로그인을 해야 사용 할 수 있다.
                        .anyRequest().authenticated();
            }
        
        }
         ```
      
        * `@EnableWebSecurity`는 스프링 시큐리티 설정을 활성화한다.
        
        * `authorizeRequests()`는 시큐리티 처리에 HttpServletRequest를 이용한다는 것을 의미한다.
        
        * 스프링 시큐리티 설정은 `WebSecurityConfigurerAdapter` 클래스를 상속 받은 `SecurityConfig`를 작성한 다음, `HttpSecurity` 타입을 파라미터로 가지는 메서드를 오버라이딩 하여 지정한다.
        
        * `mvcMatchers()` 는 특정 경로를 지정한다. 
        
        * `permitAll()` 는 모든 사용자가 접근 할 수 있다.
        
        * `hasRole()` 는 특정 권한을 가진 사용자만 접근 할 수 있다.
        
        * `.anyRequest().authenticated()` 는 나머지 요청에 대해서는 인증된 사용자만 접근할 수 있다.
        
    * 테스트 코드 작성하기
    
        ```java
        @SpringBootTest
        @AutoConfigureMockMvc
        class AccountControllerTest {
        
            @Autowired private MockMvc mockMvc;
            
            @DisplayName("회원 가입 화면 보이는지 테스트")
            @Test
            void signUpForm() throws Exception {
                mockMvc.perform(get("/sign-up"))
                        .andDo(print())
                        .andExpect(status().isOk())
                        .andExpect(view().name("account/sign-up"));
            }      
      
        }
        ```
 
    
