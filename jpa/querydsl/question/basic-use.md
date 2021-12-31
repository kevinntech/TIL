# 스프링 데이터 JPA에 Querydsl 적용하기

## 1. Querydsl를 적용한 조회 화면 만들기

* (1) 프로젝트 생성하기

    * ① https://start.spring.io/ 에서 프로젝트를 생성한다.

    * ② `application.yml` 파일을 생성한다.

        ```yaml
        # src/main/resources/application.yml
      
        spring:
          jpa:
            hibernate:
              ddl-auto: create
            properties:
              hibernate:
                #show_sql: true
                format_sql: true
                use_sql_comments: true
        
        logging.level:
          org.hibernate.SQL: debug
        #  org.hibernate.type: trace
        ```

* (2) Gradle 설정하기

    * `build.gradle`에 다음과 같은 Querydsl 관련 내용을 추가한다.

        ```
        // Gradle 5 버전 이상 
      
        // 추가
        buildscript {
            ext {
                queryDslVersion = "5.0.0"
            }
        }
        
        plugins {
            ...
            
            // 추가
            id "com.ewerk.gradle.plugins.querydsl" version "1.0.10"
            
            ...
        }
        
        ...
        
        dependencies {
            ...
            
            // querydsl 추가
            implementation "com.querydsl:querydsl-jpa:${queryDslVersion}"
            implementation "com.querydsl:querydsl-apt:${queryDslVersion}"
            implementation "com.querydsl:querydsl-core:${queryDslVersion}"
        
            ...
        }
        
        test {
            useJUnitPlatform()
        }
        
        // [START] querydsl 빌드 관련 설정 
        def querydslDir = "$buildDir/generated/querydsl"
        
        querydsl {
            jpa = true
            querydslSourcesDir = querydslDir
        }
        sourceSets {
            main.java.srcDir querydslDir
        }
        compileQuerydsl{
            options.annotationProcessorPath = configurations.querydsl
        }
        configurations {
            compileOnly {
                extendsFrom annotationProcessor
            }
            querydsl.extendsFrom compileClasspath
        }
        // [END] querydsl 빌드 관련 설정 
        ```

* (3) Querydsl 설정하기

    * ① 엔티티를 작성한다.
    
    * ② DTO를 작성한다.

        * 검색 조건으로 사용할 DTO를 작성한다.

            ```java
            @Data
            @NoArgsConstructor
            public class OrderSearchCond {
            
                private CodeType codeType;
            
                private String searchCode;
            
                private OrderStatus orderStatus;
            
                @DateTimeFormat(iso = DateTimeFormat.ISO.DATE)
                private LocalDate startDate;
            
                @DateTimeFormat(iso = DateTimeFormat.ISO.DATE)
                private LocalDate endDate;
            
            }
            ```

        * 조회 결과를 담는 용도로 사용할 DTO를 작성한다.

            ```java
            @Data
            @NoArgsConstructor
            public class OrderDetailsDto {
            
                private Long id;
            
                private String productCode;
            
                private String productName;
            
                private Long orderPrice;
            
                private Long orderQuantity;
            
                private Long orderAmount;
            
                private OrderStatus orderStatus; // 주문 상태 [ORDER, CANCEL]
            
                private DeliveryStatus deliveryStatus; // 배송 상태
            
                private String orderedName; // 주문한 사람 이름
            
                private String deliveredName; // 받는 사람 이름
            
                private String address; // 주소
            
                private String phoneNumber; // 전화번호
            
                private LocalDateTime orderedAt;
            
                private Long displayOrderDetailId;
            
                @QueryProjection
                public OrderDetailsDto(Long id, String productCode, String productName, Long orderPrice, Long orderQuantity, Long orderAmount, OrderStatus orderStatus, DeliveryStatus deliveryStatus, String orderedName, String deliveredName, String address, String phoneNumber, LocalDateTime orderedAt) {
                    this.id = id;
                    this.productCode = productCode;
                    this.productName = productName;
                    this.orderPrice = orderPrice;
                    this.orderQuantity = orderQuantity;
                    this.orderAmount = orderAmount;
                    this.orderStatus = orderStatus;
                    this.deliveryStatus = deliveryStatus;
                    this.orderedName = orderedName;
                    this.deliveredName = deliveredName;
                    this.address = address;
                    this.phoneNumber = phoneNumber;
                    this.orderedAt = orderedAt;
                }
            
            }
            ```

    * ② Query Type (Q Type)를 생성한다.

        * `Gradle` → `Tasks` → `other` → `compileQuerydsl`를 클릭한다.

    * ③ Gradle인 경우에 `build/generated/querydsl` 디렉토리에 Q 타입이 생성 되었는지 확인한다.

* (4) Spring Data JPA 프로젝트에 Querydsl 적용하기

    * ① Querydsl용 자바 설정 파일을 만든다.
    
        ```java
        @Configuration
        public class QuerydslConfig {
        
            @Autowired
            private EntityManager entityManager;
        
            @Bean
            public JPAQueryFactory jpaQueryFactory() {
                return new JPAQueryFactory(entityManager);
            }
      
        }
        ```
    
        * JPAQueryFactory는 동시성 문제에 대해서 걱정하지 않아도 된다.
    
            * 그 이유는 여기서 스프링이 주입하는 엔티티 매니저는 프록시용 가짜 엔티티 매니저이기 때문이다.
    
            * 이 가짜 엔티티 매니저는 실제 사용하는 시점에 트랜잭션 단위로 진짜 엔티티 매니저(영속성 컨텍스트)를 할당해준다.
    
                * 정리하자면, 트랜잭션 마다 별도의 영속성 컨텍스트를 제공하기 때문에 동시성 문제가 없다.
    
    * ② JpaRepository를 상속받은 스프링 데이터 JPA 리포지토리를 작성한다.
    
        ```java
        public interface OrderRepository extends JpaRepository<Order, Long> {
        
        }
        ```
    
    * ③ JPAQueryFactory를 주입 받아서 사용하는 OrderQueryRepository를 작성한다.
    
        ```java
        @Repository
        @RequiredArgsConstructor
        public class OrderQueryRepository {
        
            private final JPAQueryFactory queryFactory;
        
            public List<Order> getOrders() {
                return queryFactory
                        .selectFrom(order)
                        .fetch();
            }
        
            public List<OrderDetailsDto> search(OrderSearchCond searchCond) {
                List<OrderDetailsDto> result = queryFactory
                        .select(new QOrderDetailsDto(
                                order.id, product.code, product.name, orderProduct.orderPrice, orderProduct.orderQuantity,
                                orderProduct.orderPrice.multiply(orderProduct.orderQuantity), order.orderStatus, order.deliveryStatus,
                                order.orderedName, order.deliveredName, order.address, order.phoneNumber, order.createdAt))
                        .from(orderProduct)
                        .join(orderProduct.order, order)
                        .join(orderProduct.product, product)
                        .where(codeTypeAndSearchCodeEq(searchCond.getCodeType(), searchCond.getSearchCode()),
                                orderStatusEq(searchCond.getOrderStatus()),
                                orderDateTimeBetween(searchCond.getStartDate(), searchCond.getEndDate()))
                        .orderBy(order.id.asc())
                        .fetch();
        
                return result;
            }
        
            private BooleanExpression orderDateTimeBetween(LocalDate startDate, LocalDate endDate) {
                if (startDate == null || endDate == null)
                    return null;
        
                return order.createdAt.between(startDate.atStartOfDay(), endDate.atTime(23, 59));
            }
        
            private BooleanExpression orderStatusEq(OrderStatus orderStatus) {
                return orderStatus != null ? order.orderStatus.eq(orderStatus) : null;
            }
        
            private BooleanExpression codeTypeAndSearchCodeEq(CodeType codeType, String searchCode) {
                if (codeType == null || !StringUtils.hasLength(searchCode)) {
                    return null;
                }
        
                if (codeType.getValue().equals(CodeType.ORDER_NUMBER.getValue())) {
                    // 숫자 형식이 아니면 조건을 적용하지 않음
                    if (!searchCode.matches("[0-9]+")) {
                        return null;
                    }
        
                    return order.id.eq(Long.valueOf(searchCode));
                } else if (codeType.getValue().equals(CodeType.PRODUCT_CODE.getValue())) {
                    return product.code.eq(searchCode);
                } else {
                    return null;
                }
            }
        
        }
        ```
    
    * ④ OrderController를 작성한다.
    
        ```java
        @Controller
        @RequestMapping("/orders")
        @RequiredArgsConstructor
        public class OrderController {
        
            private final OrderService orderService;
        
            // ...
        
            @GetMapping("/list")
            public String ordersList(@ModelAttribute("orderSearchCond") OrderSearchCond orderSearchCond, Model model){
                List<OrderDetailsDto> orders = orderService.findOrderDetails(orderSearchCond);
        
                model.addAttribute("orders", orders);
                model.addAttribute("orderStatuses", OrderStatus.values());
                model.addAttribute("codeTypes", CodeType.values());
        
                return "orders/list";
            }
        
        }
        ```

    * ⑤ OrderService를 작성한다.
    
        ```java
        @RequiredArgsConstructor
        @Transactional
        @Service
        public class OrderService {
        
            // ...
            
            private final OrderQueryRepository orderQueryRepository;
        
            // ...
        
            @Transactional(readOnly = true)
            public List<OrderDetailsDto> findOrderDetails(OrderSearchCond orderSearchCond) {
                List<OrderDetailsDto> orderDetails = orderQueryRepository.search(orderSearchCond);
        
                // 디스플레이용 순번 매기기
                processOrderDetailId(orderDetails);
        
                return orderDetails;
            }
        
            private void processOrderDetailId(List<OrderDetailsDto> orderDetails) {
                Map<Long, Long> orderDetailIds = new HashMap<>(); // <OrderId, LastOrderDetailId>
        
                for (OrderDetailsDto orderDetail : orderDetails) {
                    long orderId = orderDetail.getId();
                    long orderDetailId = orderDetailIds.getOrDefault(orderId, 0L) + 1;
                    orderDetailIds.put(orderId, orderDetailId);
        
                    orderDetail.setDisplayOrderDetailId(orderDetailId);
                }
            }
        
        }
        ```

    * ⑥ 조회 화면(View)을 작성한다.
    
        ```html
        <!DOCTYPE html>
        <html lang="en"
              xmlns:th="http://www.thymeleaf.org">
        <head th:replace="fragments/layout/header :: header"></head>
        <body class="bg-light">
            <div th:replace="fragments/layout/navigation :: main-nav"></div>
            <div class="container-fluid">
                <div class="row mt-5 justify-content-center">
                    <div class="col-2">
                        <div th:replace="fragments/layout/admin-menu :: admin-menu(currentMenu='order-list')"></div>
                    </div>
                    <div class="col-10">
                        <div class="col-11">
                            <div class="row mt-2">
                                <div class="col">
                                    <h2 class="mb-3">주문 내역</h2>
                                    <!-- 검색 조건 -->
                                    <form id="searchForm" th:object="${orderSearchCond}" method="get" class="needs-validation" novalidate>
                                        <div class="row mt-5">
                                            <div class="col-md-3  align-self-end">
                                                <div class="row">
                                                    <div class="col-sm-5">
                                                        <label for="codeType">번호 / 코드</label>
                                                        <select id="codeType" th:field="*{codeType}" class="form-control custom-select d-block w-100" required>
                                                            <option value="">전체</option>
                                                            <option th:each="codeType : ${codeTypes}"
                                                                    th:value="${codeType}"
                                                                    th:text="${codeType.getValue()}">
                                                            </option>
                                                        </select>
                                                    </div>
                                                    <div class="col-sm-7 align-self-end">
                                                        <input id="searchCode" th:field="*{searchCode}" type="text" class="form-control" required>
                                                    </div>
                                                </div>
                                            </div>
                                            <div class="col-md-2">
                                                <label for="orderStatus">주문 상태</label>
                                                <select id="orderStatus" th:field="*{orderStatus}" class="form-control custom-select d-block w-100" required>
                                                    <option value="">전체</option>
                                                    <option th:each="status : ${orderStatuses}"
                                                            th:value="${status}"
                                                            th:text="${status.getValue()}">
                                                    </option>
                                                </select>
                                            </div>
                                            <div class="col-md-2">
                                                <label for="startDate">시작일</label>
                                                <input th:field="*{startDate}" type="date" class="form-control" id="startDate" placeholder="" value="" required>
                                            </div>
                                            <div class="col-md-2">
                                                <label for="endDate">종료일</label>
                                                <input th:field="*{endDate}" type="date" class="form-control" id="endDate" placeholder="" value="" required>
                                            </div>
                                            <div class="col-md-2 align-self-end">
                                                <div class="row">
                                                    <button type="button" class="btn btn-outline-danger mr-2" id="init"/>초기화</button>
                                                    <button type="submit" class="btn btn-outline-success" id="search">검색</button>
                                                </div>
                                            </div>
                                        </div>
                                    </form>
                                </div>
                            </div>
        
                            <div class="row mt-2">
                                <div class="table-responsive">
                                    <table class="mt-5 table table-hover border-bottom">
                                        <thead>
                                            <tr>
                                                <th>주문번호</th>
                                                <th>받는 사람 이름</th>
                                                <th>주소</th>
                                                <th>전화번호</th>
                                                <th>주문일자</th>
                                                <th>주문 상태</th>
                                                <th>상품코드</th>
                                                <th>상품명</th>
                                                <th>단가</th>
                                                <th>수량</th>
                                                <th>금액</th>
                                            </tr>
                                        </thead>
                                        <tbody>
                                            <tr th:each="order : ${orders}">
                                                <!-- 주문번호 -->
                                                <td th:text="${order.id}" th:if="${order.displayOrderDetailId} == 1"></td>
                                                <td th:unless="${order.displayOrderDetailId} == 1"></td>
                                                <!-- 받는 사람 이름 -->
                                                <td th:text="${order.deliveredName}" th:if="${order.displayOrderDetailId} == 1"></td>
                                                <td th:unless="${order.displayOrderDetailId} == 1"></td>
                                                <!-- 주소 -->
                                                <td th:text="${order.address}" th:if="${order.displayOrderDetailId} == 1"></td>
                                                <td th:unless="${order.displayOrderDetailId} == 1"></td>
                                                <!-- 전화번호 -->
                                                <td th:text="${order.phoneNumber}" th:if="${order.displayOrderDetailId} == 1"></td>
                                                <td th:unless="${order.displayOrderDetailId} == 1"></td>
                                                <!-- 주문일자 -->
                                                <td th:text="${#temporals.format(order.orderedAt, 'yyyy년 M월 d일')}" th:if="${order.displayOrderDetailId} == 1"></td>
                                                <td th:unless="${order.displayOrderDetailId} == 1"></td>
                                                <!-- 주문 상태 -->
                                                <td th:text="${order.orderStatus.getValue()}" th:if="${order.displayOrderDetailId} == 1"></td>
                                                <td th:unless="${order.displayOrderDetailId} == 1"></td>
                                                <!-- 상품코드 -->
                                                <td th:text="${order.productCode}"></td>
                                                <!-- 상품명 -->
                                                <td th:text="${order.productName}"></td>
                                                <!-- 단가 -->
                                                <td th:text="${#numbers.formatInteger(order.orderPrice, 0, 'COMMA')} + 원"></td>
                                                <!-- 수량 -->
                                                <td th:text="${#numbers.formatInteger(order.orderQuantity, 0, 'COMMA')} + 개"></td>
                                                <!-- 금액 -->
                                                <td th:text="${#numbers.formatInteger(order.orderAmount, 0, 'COMMA')} + 원"></td>
                                            </tr>
                                        </tbody>
                                    </table>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        
            <!-- Bootstrap js -->
            <script th:replace="fragments/script/main-script :: jquery-script"></script>
            <script th:replace="fragments/script/main-script :: bootstrap-script"></script>
            <script type="text/javascript">
                 // Form 초기화
                 $(document).ready(function() {
                    // 처음 로딩 시, 검색 텍스트 필드를 읽기 전용으로 한다.
                    if ($("#codeType").val() == '') {
                        $("#searchCode").attr("readonly", true);
                        $("#searchCode").val('');
                    }
        
                    // 번호 드롭다운 리스트를 변경할 때 마다 검색 텍스트 필드의 읽기 전용 속성을 활성화 / 비활성화 한다.
                    $("#codeType").change(function(){
                        if ($(this).val() == '') {
                            $("#searchCode").attr("readonly", true);
                            $("#searchCode").val('');
                        } else {
                            $("#searchCode").attr("readonly", false);
                        }
                    });
        
                    $("#init").click(function() {
                        $("#numberType").val('');
                        $("#searchCode").val('');
                        $("#orderState").val('');
                        $("#startDate").val('');
                        $("#endDate").val('');
                    });
                 });
            </script>
        
        </body>
        </html>
        ```