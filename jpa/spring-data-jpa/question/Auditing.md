# 스프링 데이터 JPA에 Auditing 기능 적용하기

## 1. 스프링 시큐리티를 이용한 Auditing 기능 적용

* ① 스프링 부트의 메인 클래스(`DataJpaApplication`)에 `@EnableJpaAuditing`를 적용한다.

    ```java
    @EnableJpaAuditing(auditorAwareRef = "springSecurityAuditorAware")
    @SpringBootApplication
    public class Application {
    
        public static void main(String[] args) {
            SpringApplication.run(Application.class, args);
        }
    
    }
    ```
  
    * `@EnableJpaAuditing` : Auditing 기능을 활성화한다.
    
        * `auditorAwareRef = "springSecurityAuditorAware"` : 등록자, 수정자를 자동으로 입력하는 역할을 할 AuditorAware의 이름이다.

* ② `BaseEntity` 클래스를 작성한다.
 
    ```java
    @EntityListeners(AuditingEntityListener.class)
    @MappedSuperclass
    @Getter
    public class BaseEntity {
    
        @CreatedDate
        @Column(updatable = false)
        private LocalDateTime createdAt;
    
        @LastModifiedDate
        private LocalDateTime modifiedAt;
    
        @CreatedBy
        @Column(updatable = false)
        private String createdBy;
    
        @LastModifiedBy
        private String modifiedBy;
    
    }
    ```

    * `@EntityListeners(AuditingEntityListener.class)` : 해당 클래스에 Auditing 기능을 포함시킨다.
    
    * `@MappedSuperclass` : 자식 클래스에게 공통 매핑 정보를 제공하는 부모 클래스를 작성할 때 사용한다.
    
        * 여기서 공통 매핑 정보는 createdDate, lastModifiedDate, createdBy, lastModifiedBy를 말한다.

    * `@Column(updatable = false)` : 해당 컬럼의 값은 처음 값을 지정한 이후, 수정하지 못하도록 한다.

    * 등록일, 수정일, 등록자, 수정자 관련 애노테이션
      
        * `@CreatedDate` : 엔티티가 생성되어 저장될 때, 시간이 자동 저장된다.
        
        * `@LastModifiedDate` : 조회한 엔티티의 값을 변경할 때, 시간이 자동 저장된다. 
        
        * `@CreatedBy` : 엔티티가 생성되어 저장될 때, 생성자가 자동 저장된다.
        
        * `@LastModifiedBy` : 조회한 엔티티의 값을 변경할 때, 수정자가 자동 저장된다. 

* ③ AuditorAware를 구현한 클래스를 작성한다. 

    ```java
    @Component
    public class SpringSecurityAuditorAware implements AuditorAware<String> {
    
        @Override
        public Optional<String> getCurrentAuditor() {
            Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
    
            if (authentication == null || !authentication.isAuthenticated()) {
                return null;
            }
    
            UserCustom userCustom = (UserCustom) authentication.getPrincipal();
    
            return Optional.of(userCustom.getUsername());
        }
    
    }
    ```
    
    * 스프링 시큐리티 로그인 정보에서 유저 정보를 가져와서 등록자, 수정자에 자동으로 설정한다.
    
* ④ BaseEntity 클래스를 상속 받는다.
    
    ```java
    public class Product extends BaseEntity {}
    ```

* ⑤ 앞서 작성한 내용을 확인하는 테스트 코드를 작성한다.

    ```java
    @SpringBootTest
    @Transactional
    class BaseEntityTest {
    
        @Autowired ProductRepository productRepository;
    
        // 모든 테스트를 실행 할 때 마다 UserSaveRequestDto를 만든다.
        @BeforeEach
        void beforeEach(){
            Product product = Product.builder()
                    .code("NO-1")
                    .name("운동화")
                    .brand("브랜드")
                    .price(10000)
                    .description("운동화 입니다.")
                    .build();
    
            productRepository.save(product);
        }
    
        // 이메일이 중복되어 생성되지 않도록 삭제를 한다.
        @AfterEach
        void afterEach(){
            productRepository.deleteAll();
        }
    
        @Test
        @WithUser("kevin") // 웹 페이지에서 로그인을 한 것과 동일한 기능을 함
        @DisplayName("Auditing 테스트")
        void testAuditing() {
            Product findProduct = productRepository.findOneByCode("NO-1");
    
            assertThat(findProduct.getCode()).isEqualTo("NO-1");
            assertThat(findProduct.getName()).isEqualTo("운동화");
            assertThat(findProduct.getBrand()).isEqualTo("브랜드");
            assertThat(findProduct.getPrice()).isEqualTo(10000);
            assertThat(findProduct.getDescription()).isEqualTo("운동화 입니다.");
            assertThat(findProduct.getCreatedAt()).isNotNull();
            assertThat(findProduct.getCreatedBy()).isEqualTo("kevin");
            assertThat(findProduct.getModifiedAt()).isNotNull();
            assertThat(findProduct.getModifiedBy()).isEqualTo("kevin");
        }
    
    }
    ```