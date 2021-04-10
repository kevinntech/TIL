# 김영한님의 스프링 프레임워크 핵심 원리
> 아래 내용은 [스프링 프레임워크 핵심 원리](https://www.inflearn.com/course/스프링-핵심-원리-기본편# "스프링 프레임워크 핵심 원리") 강좌를 정리한 내용 입니다.

#### 1) 좋은 객체 지향 설계의 5가지 원칙(SOLID)

* 좋은 객체 지향 설계의 5가지 원칙을 정리 (SOLID)

    * `단일 책임 원칙(SRP, Single Responsibility Principle)` : 한 클래스는 하나의 책임만 가져야 한다.
    
        * 하나의 책임이라는 것은 모호하다. 클 수도 있고 작을 수도 있다.
        
        * 중요한 기준은 변경이다.
        
            * 변경이 있을 때 파급 효과가 적다면 단일 책임 원칙을 잘 따른 것이다.
            
            * Ex) UI 변경, 객체의 생성과 사용을 분리  
      
    * `개방-폐쇄 원칙(OCP, Open Closed Principle)` : 소프트웨어 요소는 확장에는 열려 있으나 변경에는 닫혀 있어야 한다.

        * 다형성을 활용해서 인터페이스를 구현한 새로운 클래스를 하나 만든 다음, 새로운 기능을 구현하자.

            ```java
            public class MemberService{
                /*
                * 기존 코드  
                * */
                //private MemberRepository memberRepository = new MemoryMemberRepository(); 
          
                /*
                * 변경 코드 : MemberRepository 인터페이스를 구현한 새로운 클래스 JdbcMemberRepository를 작성 하였다.
                * */
                private MemberRepository memberRepository = new JdbcMemberRepository(); 
            } 
            ```
          
            * MemberService 클라이언트가 구현 클래스를 직접 선택한다. 
            
            * 구현 객체를 변경하려면 MemberService 클라이언트의 코드를 변경해야 한다.
            
            * 분명히 다형성을 사용 했지만 `OCP` 원칙을 지킬 수 없다.
            
                * 이 문제를 어떻게 해결해야 할까?
                
                * 객체를 생성하고, 연관관계를 맺어주는 별도의 조립, 설정자가 필요하다.

    * `리스코프 치환 원칙(LSP, Liskov Substitution Principle)` : 프로그램의 객체는 프로그램의 정확성을 깨뜨리지 않으면서 하위 타입의 객체로 바꿀 수 있어야 한다.
    
        * 다형성에서 하위 클래스는 인터페이스 규약을 다 지켜야 한다는 것이다. 
    
        * Ex) 자동차 인터페이스의 엑셀은 앞으로 가라는 기능이며 뒤로 가도록 구현하면 LSP 원칙을 위반하게 된다. 느리더라도 앞으로 가야한다. 
    
    * `인터페이스 분리 원칙(ISP, Interface Segregation Principle)` : 특정 클라이언트를 위한 인터페이스 여러 개가 하나의 범용 인터페이스 보다 낫다.
    
        * 자동차 인터페이스 -> 운전 인터페이스, 정비 인터페이스로 분리
        
        * 사용자 클라이언트 -> 운전자 클라이언트, 정비사 클라이언트로 분리
        
            * 분리하면 정비 인터페이스 자체가 변해도 운전자 클라이언트에 영향을 주지 않음
            
            * 인터페이스가 명확해지고, 대체 가능성이 높아진다.
    
    * `의존 역전 원칙(DIP, Dependency Inversion Principle)` : 구체화에 의존하기 보다는 추상화에 의존해야 한다.
    
        * 즉, 클라이언트 코드가 구현 클래스를 바라보지 말고 인터페이스를 바라보라는 것이다.
        
        * 앞에서 이야기한 역할(Role)에 의존하게 해야 한다는 것과 같다. 
        
            * 객체 세상도 클라이언트가 인터페이스에 의존해야 유연하게 구현체를 변경할 수 있다.         