# 더 자바, 코드를 조작하는 다양한 방법
> 아래 내용은 [더 자바, 코드를 조작하는 다양한 방법](https://www.inflearn.com/course/the-java-code-manipulation "더 자바, 코드를 조작하는 다양한 방법")을 참고 하였습니다.

## 1. JVM 이해하기

#### 1) 자바, JVM, JDK, JRE

* 다음 용어를 구분할 줄 알아야 한다.

    * `JVM (Java Virtual Machine) `
    
        * 자바 가상 머신으로 자바 바이트 코드(.class 파일)를 OS에 특화된 코드로 변환(인터프리터와 JIT 컴파일러를 사용)하여 실행한다.
         
        * 바이트 코드를 실행하는 표준(JVM 자체는 표준)이자 구현체(특정 밴더가 구현한 JVM)다. 
        
            * JVM 스펙: https://docs.oracle.com/javase/specs/jvms/se11/html/
            
            * JVM 벤더: 오라클, 아마존, Azul, ...
        
        * 특정 플랫폼에 종속적이다.
        
    * `JRE (Java Runtime Environment)` : JVM + 라이브러리
    
        * 자바 실행 환경으로 자바 애플리케이션을 실행할 수 있도록 구성된 배포판을 말한다.
            
        * `JVM`과 핵심 라이브러리 및 자바 런타임 환경에서 사용하는 프로퍼티 세팅이나 리소스 파일을 가지고 있다.
        
        * 개발 관련 도구는 포함하지 않는다. (그건 JDK에서 제공한다.)
        
    * `JDK (Java Development Kit)` : JRE + 개발 툴
    
        * JRE + 개발에 필요할 툴 (컴파일러...)
     
        * 소스 코드를 작성할 때 사용하는 자바 언어는 플랫폼에 독립적.
        
        * 오라클은 자바 11 부터 JDK만 제공하며 JRE를 따로 제공하지 않는다.
        
        * Write Once Run Anywhere
    
    * `자바 (Java)`
    
        * 프로그래밍 언어
     
        * JDK에 들어있는 자바 컴파일러(javac)를 사용하여 바이트 코드(.class 파일)로 컴파일 할 수 있다.
    
        * 자바 유료화? 
        
            * 자바가 유료화된 것은 아니다.
            
            * 오라클에서 만든 Oracle JDK가 11 버전 부터 상용으로 사용할 때 유료이다.
    
            * 예를 들어, 오라클에서 만든 Open JDK 11 버전은 무료이다. 
                   
            * https://medium.com/@javachampions/java-is-still-free-c02aef8c9e04
    
    * `JVM 언어`
    
        * JVM 기반으로 동작하는 프로그래밍 언어
        
        * 클로저, 그루비, JRuby, Jython, Kotlin, Scala, ...
    
#### 2) JVM 구조

* JVM 구조

    * 클래스 로더 시스템
    
        * 클래스 파일(.class)에서 바이트 코드를 읽고 메모리에 저장한다.
        
            * 로딩(loading) : 클래스 파일에서 바이트 코드를 읽어오는 과정
            
            * 링크(linking) : 레퍼런스를 연결하는 과정
            
            * 초기화(initialization) : static 값들을 초기화 및 변수에 할당하는 과정
            
    * 메모리(Runtime Data Area)
    
        * `메소드(Method) 영역 `
        
            * 클래스 수준의 정보 (클래스 이름, 부모 클래스 이름, 메소드, 변수)를 저장한다.
            
            * JVM 당 하나의 영역 밖에 존재하지 않으며 공유 자원이다. (모든 Thread가 공유한다.)
    
        * `힙(Heap) 영역`
        
            * 객체(인스턴스)를 저장한다.
            
            * JVM 당 하나의 영역 밖에 존재하지 않으며 공유 자원이다. (모든 Thread가 공유한다.)
    
        * `스택(Stack) 영역`
        
            * 쓰레드 마다 런타임 스택을 만들고, 그 안에 메소드 호출을 스택 프레임이라 부르는 블럭으로 쌓는다.
            
            * 쓰레드를 종료하면 런타임 스택도 사라진다.
            
        * `PC(Program Counter) 레지스터 `
        
            * 쓰레드 마다 현재 실행 중인 명령어(instruction)의 위치를 가리키는 포인터가 생성된다.
            
        * `네이티브 메소드 스택`
        
            * 네이티브(native) 메소드를 호출 할 때 사용하는 별도의 스택을 말한다.
            
            * `네이티브 메소드`는 Java가 아닌 C, C++로 구현된 메소드를 말한다.
            
            * 예를 들어, `Thread.currentThread()`가 있다. native 키워드가 사용 된 것을 확인 할 수 있다.
            
            * [참고] https://javapapers.com/core-java/java-jvm-run-time-data-areas/#Program_Counter_PC_ Register
            
    * 실행 엔진
    
        * `인터프리터(Interpreter)`
        
            * 바이트 코드를 한줄 씩 읽어서 네이티브 코드로 변환한 다음, 실행한다.
            
            * 똑같은 코드가 여러 번 나오더라도 매번 네이티브 코드로 변환해야되기 때문에 비효율적이다.
            
        * `JIT 컴파일러(JIT Compiler)`
        
            * 인터프리터가 반복되는 코드를 발견하면 JIT 컴파일러로 반복되는 코드를 전부 네이티브 코드로 바꿔 놓는다. 
            
            * 그 다음 부터 인터프리터는 반복되는 코드를 만나면 네이티브 코드로 컴파일된 코드를 바로 사용한다. (프로그램 실행 속도 향상 시킴)
    
        * `GC(Garbage Collector)`
        
            * 더 이상 참조되지 않는 객체를 모아서 정리한다.
            
    * `JNI(Java Native Interface)`
    
        * 자바 애플리케이션에서 C, C++, 어셈블리로 작성된 함수를 사용할 수 있는 방법을 제공한다.
         
        * https://medium.com/@bschlining/a-simple-java-native-interface-jni-example-in-java-and-scala-68fdafe76f5f  
    
    * `네이티브 메소드 라이브러리(Native Method Libraries)`
    
        * 실행 엔진에 필요한 네이티브 라이브러리를 모아놓은 것이다.
    
* 자바 실행 과정
 
    * ① 자바 컴파일러(javac)가 자바 소스코드(.java)를 바이트 코드(.class)로 컴파일 한다.
    
    * ② 클래스 로더가 바이트 코드(.class)를 읽어서 JVM의 메모리에 저장한다.

    * ③ 실행 엔진(Execution Engine)이 바이트 코드를 한줄 씩 읽어서 네이티브 코드로 변환한 다음, 실행한다.

        * 똑같은 코드를 여러 번 해석하는 것은 비효율적이므로 JIT 컴파일러를 사용한다.
        
        * 그리고 GC로 더 이상 참조되지 않는 객체를 모아서 정리한다.
        
        * 메모리 또는 실행 엔진이 네이티브 라이브러리를 사용한다면 JNI를 통해서 사용한다.
        
#### 3) 클래스 로더

* 클래스 로더

    * `로딩(loading)` → `링크(linking)` → `초기화(initialization)` 순으로 진행된다.
    
        * 로딩(loading)
        
            * 클래스 파일(.class)을 읽고, 바이너리 코드(binary code)로 변환 후, JVM의 `메소드 영역`에 클래스 정보를 저장한다.
            
            * 이때 메소드 영역에 저장하는 클래스 정보
            
                * `FQCN (Fully Qualified Class Name)` : 클래스가 속한 패키지명을 모두 포함한 이름을 말한다.
                
                * `클래스(class)`, `인터페이스(interface)`, `enum`
                
                * 메소드와 변수
                
            * 로딩이 끝나면 해당 클래스 타입의 `Class` 객체를 생성하여 `힙 영역`에 저장한다.
            
        * 링크(linking)
        
            * Verify, Prepare, Resolve 세 단계로 나눠져 있다.
            
                * `검증(Verify)` : 클래스 파일(.class)의 유효성을 체크한다. 검증에 실패하면 검증 에러가 발생하며 애플리케이션이 실행되지 않는다.
                
                * `준비(Prepare)` : 클래스 변수(static 변수)에 대해 메모리가 할당되고 기본 값이 저장된다.
                
                * `해석(Resolve)`
                
                   * 심볼릭 메모리 참조를 메소드 영역에 있는 실제 참조로 교체한다. (선택사항)
                
                   * 링크 과정에서 해당 교체가 이루어 질 수도 있고, 실제 해당 참조를 사용할 때, 교체가 이루어 질 수도 있다.

        * 초기화(initialization) 
        
            * static 변수의 값을 할당한다. (static 블럭이 있다면 이때 실행된다.)
   
* 클래스 로더의 종류

    * 클래스 로더는 계층 구조로 이루어져 있으며 기본적으로 세가지 클래스 로더가 제공된다.

    * `부트스트랩 클래스 로더(BootStrap ClassLoader)`
    
        * `JAVA_HOME\lib`에 있는 코어 자바 API를 제공한다. 최상위 우선순위를 가진 클래스 로더다.
        
    * `플랫폼 클래스 로더(Platform ClassLoader)`
    
        * `JAVA_HOME\lib\ext` 폴더 또는 `java.ext.dirs` 시스템 변수에 해당하는 위치에 있는 클래스를 읽는다.
    
        * 예전에는 확장 클래스 로더(Extension ClassLoader)라고 불렸음
    
    * `애플리케이션 클래스 로더(Application ClassLoader)`
    
        * 애플리케이션 클래스패스에서 클래스를 읽는다.
      
        * 클래스패스(classpath)
        
            * 클래스를 찾는 경로를 말한다.
            
            * 애플리케이션 실행할 때 주는 -classpath 옵션 또는 java.class.path 환경 변수의 값에 해당하는 위치
        
* `ClassLoader`가 클래스를 읽는 순서는 다음과 같다.

    * 최상위인 `BootStrap ClassLoader` → `Platform ClassLoader `→ `Application ClassLoader` 순서대로 읽게 된다.
    
    * 애플리케이션 클래스 로더(Application ClassLoader)에서도 클래스를 읽지 못한다면, `ClassNotFoundException` 예외가 발생한다.
    
    
      
  
    
   

    

            


    







    




    






        

        


    
    
    
    
    
    

    

    

    











  

  

  

  

  

    





     
