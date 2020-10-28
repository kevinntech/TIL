# 김영한님의 실전! 스프링 데이터 JPA

## 1. 프로젝트 환경설정

* `@PersistenceContext` : EntityManager를 주입 받을 때 사용한다.

* JPA 기본편 내용 복습

    * JPA의 모든 데이터 변경은 트랜잭션 안에서 이루어져야 한다.

    * JPA는 같은 트랜잭션 안에서 영속성 컨텍스트의 동일성을 보장한다.

    * (예시에서 `member == findMember` 이다.)

* `@Transactional`

    * 트랜잭션을 처리 할 때 사용한다.
    
    * @Transactional를 테스트 코드에 사용하면 기본적으로 테스트가 종료된 다음, 바로 롤백을 한다.
    
    * @Rollback(false)를 추가하면 롤백을 하지 않고 커밋을 한다. 
