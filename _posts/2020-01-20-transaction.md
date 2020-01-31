---
title: "transaction" 
date: 2020-01-08 
---

안녕하세요? 이번 주제는 트랜잭션 처리입니다. 
트랜잭션 처리가 제대로 안되어 많은 이슈를 만들어 내는 코드를 개선하면서 트랜잭션 처리에 대해 알아보았습니다.. 


## 트랜잭션 이란   
- 데이터베이스 내에서 한꺼번에 모두 수행되어야 할 작업 집합 (동시성 제어:Concurrency Control 의 기본 단위)
- 한 트랜젝션 내의 작업은 모두 반영 되거나,  오류가 생기면 모두 취소 되어야 함
- 작업이 성공적으로 종료 되었을 경우에는 commit, 실패 하였을 경우 rollback (all or nothing) 

### 즉.. 서버에서 하나의 기능이 동작할 때 여러 DB처리가 일어날 수 있는데 도중에 문제가 발생하여 로직이 중단되었을 경우 데이터가 절반만 처리되면 데이터가 꼬이기 때문에 트랜잭션이라는 기능이 한 작업 묶음에 대하여 최종 DB 반영 여부를 관리한다. 또한, 한 작업 내에서 트랜젝션을 적절히 분리해야할 경우도 있다. 때문에 스프링이 트랜잭션 어떻게 관리해주는지 알아두자. 가장 기본은 트렌잭션은 클래스 단위로 움직인다는 것. 같은 클래스 내에서 신규 트랜잭션을 생성해도 한 트랜잭션 내에서 움직인다!! 트랜잭션 분리를 위해서는 클래스 분리가 필요하다..

## 트랜잭션 기본 
### 클래스 단위로 작동 
<br> 내부 트랜잭션 메소드 호출 금지 
- 기본 프록시 모드에서는 클래스의 메소드에서 동일 클래스의 @Transactional 걸린 메소드를 호출하면 트랜잭션이 무시된다.
- 이는 AOP의 기본작동이 그렇기 때문이다.
- 따라서 항상 트랜잭션을 올바로 적용하려면 현재 클래스의 메소드가 아닌 다른 클래스의 메소드에 트랜잭션을 걸어야만 한다.
<br>참고) https://supawer0728.github.io/2018/03/22/spring-multi-transaction/ 
<br>참고) http://egloos.zum.com/springmvc/v/499291
<br>참고) https://kwonnam.pe.kr/wiki/springframework/transaction

## 트랜잭션 속성

| 속성 | 설명 | 예제 |
|:--------|:--------|:--------|
| isolation | Transaction의 isolation Level. 별도로 정의하지 않으면 DB의 Isolation Level을 따름. | @Transactional(isolation=Isolation.DEFAULT) |
| **propagation** | 트랜잭션 전파규칙을 정의, 트랜 잭션의 경계  | @Transactional(propagation=Propagation.REQUIRED) <br> - **REQUIRES_NEW** : 항상 새로운 트랜잭션을 시작한다.  이미 진행중인 트랜잭션이 있다면 잠깐 보류되고 해당 트랜잭션 경계가 종료 된 후 다시 시작된다.  <br> - **REQUIRED** : Defualt 속성이다.  트랜잭션의 시작 시점에 이미 진행중인 트랜잭션이 있으면 해당 트랜잭션에 참여하며 없을 경우 새로운 트랜잭션을 시작한다.  <br> - **SUPPORT** : 이미 진행중인 트랜잭션이 있으면 참여하고, 없을 경우 Non-transaction으로 시작한다.  <br> - **MANDATORY** : 이미 진행중인 트랜잭션이 반드시 있어야만 해당 경계를 넘어 시작할 수 있다.  없을 경우 Exception을 발생시킨다. <br> - **NOT_SUPPORT** : Non-transaction으로 시작하며, 이미 진행중인 트랜잭션이 있으면 잠시 보류시킨다. (부모 트랜잭션 내에서 실행될 경우 일시 정지 됩니다.)<br> - **NEVER** : Non-transaction 상태에서만 해당 경계를 넘어갈 수 있다.  이미 진행중인 트랜잭션(parent)가 있으면 예외를 발생시킨다. <br> - **NESTED** : 해당 메서드가 부모 트랜잭션에서 진행될 경우 별개로 커밋되거나 롤백될 수 있습니다. 둘러싼 트랜잭션이 없을 경우 REQUIRED와 동일하게 작동합니다. 이미 진행중인 트랜잭션(parent)이 있을 경우 중첩트랜잭션을 생성하여 시작한다.  생성된 중첩트랜잭션은 (parent)가 rollback되면 함께 되지만, 해당 트랜잭션안에서의 Commit/Rollback은 (parent)에 영향을 주지 않는다.  이미 진행중인 트랜잭션이 없을 경우 새로운 트랜잭션을 만든다. |
| readOnly | 해당 Transaction을 읽기 전용 모드로 처리 (Default = false) | @Transactional(readOnly = true) |
| rollbackFor | 정의된 Exception에 대해서는 rollback을 수행 | @Transactional(rollbackFor=Exception.class) |
| noRollbackFor | 정의된 Exception에 대해서는 rollback을 수행하지 않음. | @Transactional(noRollbackFor=Exception.class) |
| timeout | 지정한 시간 내에 해당 메소드 수행이 완료되지 않은 경우 rollback 수행. -1일 경우 no timeout (Default = -1) | @Transactional(timeout=10) |

## DBMS의 트랜젝션 관리

- PlatformTransactionManager 
<br> 스프링은 애플리케이션에서 사용하는 데이터 접근 기술에 따라 선택 가능한 몇 가지 PlatformTransactionManager 구현을 제공한다. 예를 들어, 데이터베이스 상호작용에 JDBC를 사용하는 애플리케이션에는 DataSourceTransactionManager가 적합하며, 하이버네이트를 사용하는 경우 HibernateTransactionManager, 그리고 JPA의 EntityManager를 사용하는 경우 JpaTransactionManager가 적합하다.
<br> 참고) https://wikibook.co.kr/article/transaction-management-using-spring/ 

- ReplicationRoutingDataSource
<br> transaction readOnly 속성으로 master, slave 구분하기

- Chained Transaction Manager 
<br> mysql과 jpa 동시에 사용하기

<br> ChainedTransactionManager Spring Data 에 추가된 다중 트랜잭션을 연결해서 관리하는 트랜잭션 매니저.
JTA 처럼 트랜잭션이 완전히 동기화 되는 것은 아니고, 여러 트랜잭션을 묶어서 순서대로 커밋/롤백을 진행하는 방식. JTA 수준은 아니더라도 여러 트랜잭션이 생성되는 애플리케이션에서 각자 @Transactional을 지정하지 않고 하나로 통일하고자 하는 경우에 괜찮을 듯. Chained Transaction Manager로 묶이는 각 트랜잭션의 Connection Pool 의 min/max 갯수는 동일하게 가져가야한다. 경험상, 서로 다른 동기화 시점을 가지는 트랜잭션은 묶지 않는 것이 좋다.
MQ 사용시, MQ로 받은 데이터를 저장하는데 DB 접속이 끊겼을 경우 retry 를 하고자 한다면 MQ 트랜잭션과 DB 트랜잭션을 함께 묶으면 이 작업이 안된다. DB 트랜잭션이 롤백 될때 MQ 트랜잭션까지 롤백 되어 버려서 재시도할 트랜잭션이 없어지기 때문이다.
<br>참고) https://blog.box.kr/?p=592
<br>참고) https://jogeum.net/3 

