# 더 자바, Java 8
> 아래 내용은 [더 자바, Java 8](https://www.inflearn.com/course/the-java-code-manipulation "더 자바, Java 8")을 참고 하였습니다.

## 6. Date와 Time

#### 1) 소개

* (1) 자바 8에 새로운 날짜와 시간 API가 생긴 이유

    * ① 이전까지 사용하던 `java.util.Date` 클래스는 변경 가능(mutable) 하기 때문에 Thread-Safe 하지 않다.
    
        * 즉, 멀티 스레드 환경에서 안전하지 않다.

    * ② 클래스 이름이 명확하지 않다. 
      
        * Date인데 시간까지 다룬다.

            ```java
            Date date = new Date();
            long time = date.getTime();
            ```

    * ③ 버그를 발생할 여지가 많다. 
      
        * 타입 안정성(type-safe)이 없고, 월이 0 부터 시작하는 등의 문제가 있다.

            ```java
            public GregorianCalendar(int year, int month, int dayOfMonth) {
                this(year, month, dayOfMonth, 0, 0, 0, 0);
            }
            ```
          
            * 생성자의 매개변수 타입이 특정 Enum을 받도록 했다면 타입 안정성이 있었겠지만 int를 받도록 되어 있다.
    
            * 이러한 경우에 타입 안정성(type-safe)이 없다고 한다. 즉, 잘못된 값이 들어올 수 있다.

    * 위와 같은 불편함 때문에 날짜와 시간 처리가 복잡한 애플리케이션에서는 보통 `Joda Time`을 사용 했었다.

* (2) 자바 8에서 제공하는 Date-Time API

    * ① JSR-310 스펙의 구현체를 제공한다.
    
    * ② 디자인 철학은 다음과 같다.
    
        * `Clear` : API가 명확해야한다는 의미다. 
          
            * 이전처럼 Date인데 시간까지 다루거나 하지 않는다는 의미다.
          
        * `Fluent` : null를 파라미터로 받거나 null 값을 리턴하지 않기 때문에 체이닝을 할 수 있다는 의미다.
          
        * `Immutable` : 날짜와 시간에 대한 연산을 하면 새로운 인스턴스가 만들어진다는 의미다.
          
        * `Extensible` : 확장성이 있다는 의미다.

* (3) 기계용 시간, 인류용 시간

    * 자바 8의 Date-Time API는 크게 `기계용 시간(machine time)`과 `인류용 시간(human time)`으로 나눌 수 있다.
    
        * `기계용 시간`은 EPOCK TIME (1970년 1월 1일 0시 0분 0초) 부터 현재까지의 타임스탬프를 표현하는 것이다.
    
            * Ex) 메소드의 실행 시간을 측정할 때 사용 할 수 있다.
        
        * `인류용 시간`은 우리가 흔히 사용하는 연,월,일,시,분,초 등을 표현하는 것이다.

* (4) 주요 API

    * 기계용 시간(타임 스탬프)
      
        * `Instant`를 사용한다.
    
    * 인류용 시간

        * 특정 날짜(`LocalDate`), 시간(`LocalTime`), 일시(`LocalDateTime`)를 사용 할 수 있다.

    * 기간을 표현하기
      
        * 시간 기반(`Duration`) 과 날짜 기반(`Period`)를 사용 할 수 있다.

    * 형식화(Formatting) 하기

        * `DateTimeFormatter`를 사용해서 일시를 특정한 문자열로 형식화 할 수 있다.

#### 2) Date와 Time API

* (1) 기계용 시간을 표현하는 방법

    * `Instant` : EPOCK TIME (1970년 1월 1일 0시 0분 0초 UTC) 부터 현재까지의 타임스탬프를 표현한다.

        * 특징
    
            * `Instant`는 항상 UTC를 기준으로 한다.

            * `UTC(Universal Time Coordinated : 세계 협정시)`와 `GMT(Greenwich Mean Time : 그리니치 평균시)`는 거의 같다.
            
                * 영국 런던에 위치한 그리니치 천문대를 기준으로 한 시간을 리턴한다고 생각하면 된다.
        
        * 주요 메소드

            * `Instant.now()` : 현재 UTC (GMT)를 리턴한다.

            * `Instant.ofEpochSecond()`

        * 예시

            ```java
            Instant instant = Instant.now();
            System.out.println(instant);
            System.out.println(instant.atZone(ZoneId.of("UTC")));
            
            // 현재 시스템의 시간대를 기준으로 기계용 시간을 표현한다.
            ZoneId zone = ZoneId.systemDefault(); // 현재 시스템 상의 시간대를 반환한다.
            ZonedDateTime zonedDateTime = instant.atZone(zone);
            System.out.println(zonedDateTime);
            ```
          
            * `Instant`는 항상 UTC(`+00:00`)를 기준으로 하기 때문에 `LocalTime`가 차이가 있을 수 있다.

                * 한국은 시간대가 `+09:00`이므로 `LocalTime`과 9 시간의 차이가 있다.

                    * `Instant` : 2021-06-01 09:00:00

                    * `LocalTime` : 2021-06-01 18:00:00

            * 시간대를 표현할 때는 `ZoneId`를 사용한다. 이것을 이용하면 현재 시스템의 시간대를 기준으로 기계용 시간을 표현 할 수 있다.
    
* (2) 인류용 일시를 표현하는 방법

    * `LocalDate` : 날짜를 표현하는 클래스다.

        * 주요 메소드

            * `LocalDate.now()` : 현재 날짜를 반환한다.
            
            * `LocalDate.of(year, month, dayOfMonth)` : 특정 날짜를 반환한다.

        * 예시

            ```java
            LocalDate now = LocalDate.now();
            LocalDate of = LocalDate.of(2020, 5, 1);
            
            System.out.println(now);
            System.out.println(of);
            ```

    * `LocalTime` : 시간을 표현하는 클래스다.

        * 주요 메소드
          
            * `LocalTime.now()` : 현재 시간을 반환한다.
    
            * `LocalTime.of(hour, minute)` : 특정 시간을 반환한다.

            * `atTime()` : 시간 정보를 추가한다.
            
                * `LocalDateTime`을 얻을 수 있다.

        * 예시
        
            ```java
            LocalTime now = LocalTime.now();
            LocalTime of = LocalTime.of(10, 10);
            
            System.out.println(now);
            System.out.println(of);
            ```

    * `LocalDateTime` : 일시를 표현하는 클래스다.

        * 주요 메소드

            * `LocalDateTime.now()`: 현재 일시를 반환한다.
        
                * 즉, 현재 시스템 Zone에 해당하는 일시를 리턴한다. (로컬 일시)
            
                * [Tip] 메소드의 파라미터 타입이 `TemporalUnit`이면 `ChronoUnit`을 사용하면 된다.
    
            * `LocalDateTime.of(year, month, dayOfMonth, hour, minute, second)` : 특정 일시를 리턴한다.

            * `atZone()` : 시간대 정보를 추가한다.
    
                * ZonedDateTime을 얻을 수 있다.

        * 예시
        
            ```java
            LocalDateTime now = LocalDateTime.now();
            LocalDateTime of = LocalDateTime.of(2020, Month.JUNE, 1, 0, 0);
            
            System.out.println(now);
            System.out.println(of);
            
            ZonedDateTime zonedDateTime = now.atZone(ZoneId.of("Asia/Seoul"));
            ```
          
    * `ZonedDateTime` : 일시와 시간대(time-zone)를 표현하는 클래스다.

        * 주요 메소드

            * `ZonedDateTime.now()` : 현재 일시와 시간대를 반환한다.
        
            * `ZonedDateTime.of(year, month, dayOfMonth, hour, minute, second, nanoOfSecond, ZoneId)` : 특정 시간대의 일시를 리턴한다.

            * 날짜와 시간에 관련된 다른 클래스로 변환하는 메소드를 제공한다.

                * `toLocalDate()`, `toLocalTime()`, `toLocalDateTime()` ...

        * 예시

            ```java
            ZonedDateTime now = ZonedDateTime.now();
            ZonedDateTime of = ZonedDateTime.of(2020, 3, 1,
                                                10, 10, 10,
                                                0, ZoneId.of("Asia/Seoul"));
            System.out.println(now);
            System.out.println(of);
            ```

* (3) 날짜와 시간 관련 메소드 정리

    * 날짜와 시간의 특정 필드 값 가져오기
    
        * `get(TemporalField)` , `getXXX()` : 특정 필드의 값 가져오기
    
            ```java
            LocalDateTime now = LocalDateTime.now();
            int year = now.get(ChronoField.YEAR);
            Month month = now.getMonth();
            
            System.out.println(year);
            System.out.println(month);
            ```
          
            * 해당 클래스가 지원하지 않는 필드를 사용하면, 예외(UnsupportedTemporalTypeException)가 발생한다.

    * 날짜와 시간의 특정 필드 값 변경하기
    
        * 필드를 변경하는 메소드들은 항상 새로운 객체를 생성해서 반환한다는 것을 알고 있어야 한다. (반환하지 않으면 아무일도 하지 않는 것과 같음)
    
            * `withXXX()` : 특정 필드의 값을 변경한다.
            
                ```java
                LocalDateTime now = LocalDateTime.now();
                LocalDateTime modifiedNow = now.withHour(1); // 특정 필드 값을 변경한 다음, 변수에 저장해야 의미가 있다.
              
                System.out.println(modifiedNow);
                ```
              
            * `plus()` : 특정 필드의 값을 더한다.

                ```java
                LocalDateTime now = LocalDateTime.now();
                LocalDateTime modifiedNow = now.plus(1, ChronoUnit.MONTHS);
                
                System.out.println(modifiedNow);
                ```

            * `minus()` : 특정 필드의 값을 뺀다.
            
                ```java
                LocalDateTime now = LocalDateTime.now();
                LocalDateTime modifiedNow = now.minus(1, ChronoUnit.HOURS);
                System.out.println(modifiedNow);
                ```

            * `truncatedTo()` : 지정된 필드 보다 작은 단위를 0으로 만든다.
            
                ```java
                LocalDateTime now = LocalDateTime.now(); // 현재 19시 40분 10초라고 가정
                LocalDateTime modifiedNow = now.truncatedTo(ChronoUnit.HOURS); // 시간(Hour) 보다 작은 단위를 0으로 만든다.
                System.out.println(modifiedNow);
                ```
              
                * LocalDate의 년, 월, 일은 0이 될 수 없으므로 `truncatedTo()`가 존재하지 않는다.

    * 날짜와 시간의 비교

        * `isBefore()` : 주어진 날짜 또는 시간이 이전이면 true 아니면 false를 반환한다.
          
        * `isAfter()` : 주어진 날짜 또는 시간이 이후이면 true 아니면 false를 반환한다.
    
        * `isEqual()` : 주어진 날짜 또는 시간이 같으면 true 아니면 false를 반환한다. (연대까지 같아야 함)
        
            ```java
            LocalDateTime june = LocalDateTime.of(2021, Month.JUNE, 1, 19, 40);
            LocalDateTime now = LocalDateTime.now();
            
            System.out.println(june.isBefore(now)); // true
            System.out.println(june.isAfter(now)); // false
            System.out.println(june.isEqual(now));
            ```
    
* (4) 기간을 표현하는 방법
  
    * `Period` : 날짜의 차이를 표현하는 클래스다.

        ```java
        LocalDate today = LocalDate.now();
        LocalDate thisYearBirthday = LocalDate.of(2021, Month.JULY, 15);
        
        Period period = Period.between(today, thisYearBirthday);
        System.out.println(period.getDays());
        
        Period until = today.until(thisYearBirthday);
        System.out.println(until.get(ChronoUnit.DAYS));
        ```
      
        * `between()` : 날짜의 차이를 나타내는 Period를 반환한다.

        * `until()` : 날짜의 차이를 나타내는 Period를 반환한다.

            * `between()`는 static 메소드이며 `until()`는 인스턴스 메소드라는 차이가 있다.

    * `Duration` : 시간의 차이를 표현하는 클래스다.

        ```java
        Instant now = Instant.now();
        Instant plus = now.plus(10, ChronoUnit.SECONDS);
        Duration between = Duration.between(now, plus); // now와 plus의 차이를 나타내는 Duration
        System.out.println(between.getSeconds());
        ```

        * `between()` : 시간의 차이를 나타내는 Duration를 반환한다.

        * `until()` : 시간의 차이를 나타내는 Duration를 반환한다.
    
* (5) 파싱 또는 포맷

    * 포맷
      
        * `DateTimeFormatter` : 날짜와 시간을 원하는 형식으로 출력할 때 사용한다.

            * `ofLocalizedDateTime()`, `ofLocalizedDate()`, `ofLocalizedTime()` : 로케일에 종속적인 포매터를 생성한다.

                ```java
                DateTimeFormatter formatter = DateTimeFormatter.ofLocalizedTime(FormatStyle.SHORT);
                ```

            * `ofPattern()` : 원하는 출력 형식을 직접 작성한다.

                ```java
                LocalDateTime now = LocalDateTime.now();
                DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy/MM/dd");
                
                System.out.println(now.format(formatter));
                ```

    * 문자열을 날짜와 시간으로 파싱하기
    
        * `parse()` : 문자열을 날짜 또는 시간으로 변환할 때 사용한다.
    
            ```java
            DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy/MM/dd");
            LocalDate parse = LocalDate.parse("2019/01/01", formatter);
            System.out.println(parse);
            ```

* (6) TemporalAdjusters

    * `TemporalAdjusters` : 자주 쓰일만한 날짜 계산을 위한 메소드가 정의되어 있는 클래스다. 

        ```java
        LocalDate now = LocalDate.now();
        LocalDate firstDayOfMonth = now.with(TemporalAdjusters.firstDayOfMonth()); // 이번 달의 첫 날
        
        System.out.println(firstDayOfMonth);
        ```

* (7) 레거시 API 지원

    * `GregorianCalendar`와 `Date` 타입의 인스턴스를 `Instant`나 `ZonedDateTime`으로 변환 가능.
    
    * `java.util.TimeZone`에서 `java.time.ZoneId`로 상호 변환 가능.

        ```java
        Date date = new Date();
        Instant instant = date.toInstant(); // legacy -> new
        Date newDate = Date.from(instant);  // new -> legacy
        
        GregorianCalendar gregorianCalendar = new GregorianCalendar();
        ZonedDateTime zonedDateTime = gregorianCalendar.toInstant().atZone(ZoneId.systemDefault());
        GregorianCalendar from = GregorianCalendar.from(zonedDateTime);
        
        ZoneId newZoneAPI = TimeZone.getTimeZone("PST").toZoneId();
        TimeZone legacyZoneAPI = TimeZone.getTimeZone(newZoneAPI);
        ```

## 7. CompletableFuture

#### 1) 자바 Concurrent 프로그래밍 소개

* (1) `Concurrent 소프트웨어` : 동시에 여러 작업을 할 수 있는 소프트웨어를 말한다.
  
    * Ex) 웹 브라우저로 유튜브를 보면서 키보드로 문서에 타이핑을 할 수 있다.

* (2) 자바에서 지원하는 Concurrent 프로그래밍 
  
    * 멀티 프로세싱 (ProcessBuilder)
    
    * 멀티 쓰레드

        * 쓰레드를 작성하는 방법은 다음 2 가지가 있다. 

            * Thread를 상속받는다.
    
                ```java
                public class App {
                
                    public static void main(String[] args) {
                        MyThread myThread = new MyThread();
                        myThread.start();
                
                        System.out.println("Hello: " + Thread.currentThread().getName());
                    }
                
                    static class MyThread extends Thread {
                        @Override
                        public void run() {
                            System.out.println("Thread: " + Thread.currentThread().getName());
                        }
                    }
                
                }
                ```

            * Runnable를 구현하거나 람다식을 이용한다.

                ```java
                public class App {
                
                    public static void main(String[] args) {
                        Thread thread = new Thread(() -> {
                            System.out.println("Thread: " + Thread.currentThread().getName());
                        });
                        thread.start();
                
                        System.out.println("Hello: " + Thread.currentThread().getName());
                    }
                
                }
                ```

        * 쓰레드의 주요 기능
    
            * 현재 쓰레드 멈춰두기 (sleep) : 다른 쓰레드가 처리할 수 있도록 기회를 주지만 그렇다고 락을 놔주진 않는다. (잘못하면 데드락 걸릴 수 있겠죠.)
              
            * 다른 쓰레드 깨우기 (interupt) : 다른 쓰레드를 깨워서 interruptedExeption을 발생 시킨다.
              
                * 그 에러가 발생했을 때 할 일은 코딩하기 나름. 종료 시킬 수도 있고 계속 하던 일 할 수도 있고.
              
            * 다른 쓰레드 기다리기 (join) : 다른 쓰레드가 끝날 때까지 기다린다.
    
        * 실습하기
    
            * `sleep()`

                ```java
                public class App {
                
                    public static void main(String[] args) {
                        Thread thread = new Thread(() -> {
                            try {
                                Thread.sleep(1000L);
                            } catch (InterruptedException e) {
                                e.printStackTrace();
                            }
                
                            System.out.println("Thread: " + Thread.currentThread().getName());
                        });
                        thread.start();
                
                        System.out.println("Hello: " + Thread.currentThread().getName());
                    }
                
                }
                ```

            * `interrupt()`

                ```java
                public class App {
                
                    public static void main(String[] args) throws InterruptedException {
                        Thread thread = new Thread(() -> {
                            while(true){
                                System.out.println("Thread: " + Thread.currentThread().getName());
                
                                try {
                                    Thread.sleep(1000L);
                                } catch (InterruptedException e) {
                                    System.out.println("exit!");
                                    return; // 쓰레드 종료
                                }
                            }
                
                
                        });
                        thread.start();
                
                        System.out.println("Hello: " + Thread.currentThread().getName());
                        Thread.sleep(3000L);
                        thread.interrupt();
                    }
                
                }
                ```

            * `join()`

                ```java
                public class App {
                
                    public static void main(String[] args){
                        Thread thread = new Thread(() -> {
                            System.out.println("Thread: " + Thread.currentThread().getName());
                
                            try {
                                Thread.sleep(3000L);
                            } catch (InterruptedException e) {
                                throw new IllegalStateException(e);
                            }
                        });
                        thread.start();
                
                        System.out.println("Hello: " + Thread.currentThread().getName());
                        try {
                            thread.join(); // 해당 쓰레드(thread)가 끝날 때까지 기다린다.
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                        System.out.println(thread + " is finished");
                    }
                
                }
                ```
              
                * 프로그래머가 직접 여러 개의 쓰레드를 만들고 관리하는 것은 힘들다. 

#### 2) Executors

* (1) 고수준 (High-Level) Concurrency 프로그래밍 

    * 애플리케이션에서 쓰레드를 만들고 관리하는 작업을 분리해서 `Executors`에게 위임한다.
      
    * `Executors`는 애플리케이션에서 실행되는 작업(TASK)을 비동기로 처리 할 수 있도록 thread-pool과 API를 제공한다.
    
* (2) Executors가 하는 일

    * ① `쓰레드 만들기` : 애플리케이션이 사용할 쓰레드 풀을 만들어 관리한다.
      
    * ② `쓰레드 관리` : 쓰레드의 생명 주기를 관리한다.
      
    * ③ `작업(TASK) 제출 및 실행` : 쓰레드로 실행할 작업을 제공할 수 있는 API를 제공한다.
    
* (3) 주요 인터페이스

    * `Executor` : execute(Runnable)
      
    * `ExecutorService` : Executor를 상속 받은 인터페이스로, Callable도 실행할 수 있으며, Executor를 종료 시키거나, 여러 Callable을 동시에 실행하는 등의 기능을 제공한다.
    
    * `ScheduledExecutorService` : ExecutorService를 상속 받은 인터페이스로 특정 시간 이후에 또는 주기적으로 작업을 실행할 수 있다.
    
* (4) ExecutorService로 작업 실행하기

    ```java
    /*
    * ExecutorService로 작업을 실행하면 다음 작업이 들어오기 전까지 계속해서 대기한다.
    * 따라서 명시적으로 종료를 해야한다.
    * */
    ExecutorService executorService = Executors.newSingleThreadExecutor();
    executorService.execute(() -> {
        System.out.println("Thread " + Thread.currentThread().getName());
    });
    ```

* (5) ExecutorService로 멈추기

    ```java
    executorService.shutdown(); // 처리중인 작업을 기다렸다가 종료한다.
    //executorService.shutdownNow(); // 당장 종료한다.
    ```

* (6) Executors.newFixedThreadPool

    ```java
    public class App {
    
        public static void main(String[] args){
            ExecutorService executorService = Executors.newFixedThreadPool(2);
            executorService.execute(getRunnable("Hello"));
            executorService.execute(getRunnable("The"));
            executorService.execute(getRunnable("Java"));
            executorService.execute(getRunnable("Thread"));
    
            executorService.shutdown();
        }
    
        private static Runnable getRunnable(String message) {
            return () -> System.out.println(message + " " + Thread.currentThread().getName());
        }
    
    }
    ```

* (7) Executors.newSingleThreadScheduledExecutor

    ```java
    ScheduledExecutorService executorService = Executors.newSingleThreadScheduledExecutor();
    executorService.scheduleAtFixedRate(getRunnable("Hello"), 1, 2, TimeUnit.SECONDS);
    ```

* (8) Fork/Join 프레임워크

    * Fork/Join 프레임워크 : ExecutorService의 구현체로 손쉽게 멀티 프로세서를 활용 할 수 있도록 도와준다.

#### 3) Callable과 Future

* (1) Callable과 Future

    * `Callable` : Runnable과 유사하지만 작업한 결과를 반환한다.
    
    * `Future` : 비동기적인 작업의 현재 상태를 조회하거나 결과를 가져올 수 있다.
    
        * Future는 내부적으로 Thread-Safe 하도록 구현되었기 때문에 synchronized block을 사용하지 않아도 된다.

* (2) Future의 주요 메소드

    * `get()` : 결과를 가져오기

        * 블록킹 콜이다.
    
            * 블록킹 콜 : 결과 값을 가져올 때까지 기다려야 한다는 것을 의미한다.

        * 타임아웃(최대한으로 기다릴 시간)을 설정할 수 있다.

        * 이미 취소된 작업에 get()를 호출하면 예외(CancellationException)가 발생한다.

    * `cancel()` : 작업 취소하기

        * 취소 했으면 true 못했으면 false를 리턴한다.

        * parameter로 true를 전달하면 현재 진행중인 쓰레드를 interrupt하고 그러지 않으면 현재 진행중인 작업이 끝날때까지 기다린다.

    * `invokeAll()` : 여러 작업 동시에 실행하기

        * 동시에 실행한 작업 중에 제일 오래 걸리는 작업 만큼 시간이 걸린다.

    * `invokeAny()` : 여러 작업 중에 하나라도 먼저 응답이 오면 끝내기

        * 동시에 실행한 작업 중에 제일 짧게 걸리는 작업 만큼 시간이 걸린다.

        * 블록킹 콜이다.

* (3) Callable과 Future 사용하기

    * 예시 1
    
        ```java
        ExecutorService executorService = Executors.newSingleThreadExecutor();
        
        // 리턴 타입을 제네릭 타입으로 지정한다.
        Callable<String> hello = () -> {
            Thread.sleep(2000L);
            return "Hello";
        };
        
        Future<String> future = executorService.submit(hello);
        System.out.println("Started!");
        
        future.get(); // get() : 결과 값을 가져올 때까지 기다린다. (블록킹 콜)
        
        System.out.println("End!!");
        executorService.shutdown();
        ```

    * 예시 2

        ```java
        ExecutorService executorService = Executors.newSingleThreadExecutor();
        
        // 여러 작업을 실행하기
        Callable<String> hello = () -> {
            Thread.sleep(2000L);
            return "Hello";
        };
        
        Callable<String> java = () -> {
            Thread.sleep(3000L);
            return "Java";
        };
        
        Callable<String> user = () -> {
            Thread.sleep(1000L);
            return "User";
        };
        
        List<Future<String>> futures = executorService.invokeAll(Arrays.asList(hello, java, user));
        for (Future<String> f : futures) {
            System.out.println(f.get());
        }
        
        executorService.shutdown();
        ```

    * 예시 3

        ```java
        ExecutorService executorService = Executors.newFixedThreadPool(4);
        
        // 여러 작업을 실행하기
        Callable<String> hello = () -> {
            Thread.sleep(2000L);
            return "Hello";
        };
        
        Callable<String> java = () -> {
            Thread.sleep(3000L);
            return "Java";
        };
        
        Callable<String> user = () -> {
            Thread.sleep(1000L);
            return "User";
        };
        
        String s = executorService.invokeAny(Arrays.asList(hello, java, user));
        System.out.println(s);
        
        executorService.shutdown();
        ```

#### 4) CompletableFuture

* (1) Future로 하기 어려웠던 작업들

    * Future를 외부에서 완료 시킬 수 없다.

    * 블로킹 코드(`get()`)를 호출하기 전에 콜백을 정의 할 수 없다.

    * 여러 Future를 조합 할 수 없다. 
      
        * Ex) Event 정보를 가져온 다음, Event에 참석하는 회원 목록 가져오기
      
    * 예외 처리용 API를 제공하지 않는다.

* (2) CompletableFuture

    * `CompletableFuture` : 비동기(Asynchronous) 프로그래밍을 가능하도록 하는 클래스다.

        * CompletableFuture는 Future와 CompletionStage를 구현한 클래스다.
    
        * 작업이 완료 되었을 때 콜백을 실행 할 수 있다.

    * CompletableFuture 생성하기

        * 1번 

            ```java
            CompletableFuture<String> future = new CompletableFuture<>();
            future.complete("user");
            
            System.out.println(future.get());
            ```

        * 2번

            ```java
            CompletableFuture<String> future = CompletableFuture.completedFuture("user");
            System.out.println(future.get());
            ```

    * 주요 메소드
    
        * `complete()` : Future에 값을 설정한다.
    
            * 즉, get()에 의해서 반환되는 값을 설정한다.

        * 비동기로 작업 실행하기

            * `runAsync() `: 비동기로 작업을 실행한다. (리턴 값이 없는 경우)

                ```java
                CompletableFuture<Void> future = CompletableFuture.runAsync(() -> {
                    System.out.println("Hello " + Thread.currentThread().getName());
                });
                future.get();
                ```

            * `supplyAsync()` : 비동기로 작업을 실행한다. (리턴 값이 있는 경우)

                ```java
                CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
                    System.out.println("Hello " + Thread.currentThread().getName());
                    return "Hello";
                });
                System.out.println(future.get());
                ```

            * 원하는 Executor(쓰레드 풀)를 사용해서 실행 할 수도 있다. 
    
                ```java
                ExecutorService executorService = Executors.newFixedThreadPool(4);
                CompletableFuture<Void> future = CompletableFuture.supplyAsync(() -> {
                    // ...
                }, executorService).thenRun(() -> {
                    // ...
                });
                ```

                * 기본은 ForkJoinPool.commonPool()를 사용한다.

        * 콜백 제공하기
          
            * `thenApply(Function)` : 리턴 값을 받아서 다른 값으로 변경하는 콜백

                ```java
                CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
                    System.out.println("Hello " + Thread.currentThread().getName());
                    return "Hello";
                }).thenApply((s) -> {
                    System.out.println(Thread.currentThread().getName());
                    return s.toUpperCase();
                });
                ```
              
                * `콜백 함수(Callback Function)` : 어떤 함수가 실행을 완료한 이후에 호출하는 함수를 말한다.
    
                    * 위의 예시에서 콜백은 스레드가 작업을 완료하면 지정한 함수를 자동으로 실행하는 기법을 의미한다. 

            * `thenAccept(Consumer)` : 리턴 값을 받아서 또 다른 작업을 처리하는 콜백

                ```java
                CompletableFuture<Void> future = CompletableFuture.supplyAsync(() -> {
                    System.out.println("Hello " + Thread.currentThread().getName());
                    return "Hello";
                }).thenAccept((s) -> {
                    System.out.println(Thread.currentThread().getName());
                    System.out.println(s.toUpperCase());
                });
                
                future.get();
                ```

            * `thenRun(Runnable)` : 리턴 값을 받지 않고 다른 작업을 처리하는 콜백

                ```java
                CompletableFuture<Void> future = CompletableFuture.supplyAsync(() -> {
                    System.out.println("Hello " + Thread.currentThread().getName());
                    return "Hello";
                }).thenRun(() -> {
                    System.out.println(Thread.currentThread().getName());
                });
                
                future.get();
                ```

            * 콜백 자체를 또 다른 쓰레드에서 실행 할 수 있다.

        * 조합하기

            * `thenCompose()` : 두 작업이 서로 이어서 실행하도록 조합
              
            * `thenCombine()` : 두 작업을 독립적으로 실행하고 둘 다 종료 했을 때 콜백 실행
              
            * `allOf()` : 여러 작업을 모두 실행하고 모든 작업 결과에 콜백 실행

            * `anyOf()` : 여러 작업 중에 가장 빨리 끝난 하나의 결과에 콜백 실행

        * 예외 처리
          
            * `exceptionally(Function)`
              
            * `handle(BiFunction)`