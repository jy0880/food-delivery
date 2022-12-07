# 과제

## 서비스 시나리오

###기능적 요구사항
1. 고객이 메뉴를 선택하여 주문한다.
1. 고객이 선택한 메뉴에 대해 결제한다.
1. 주문이 되면 주문 내역이 입점상점주인에게 주문정보가 전달된다
1. 상점주는 주문을 수락하거나 거절할 수 있다
1. 상점주는 요리시작때와 완료 시점에 시스템에 상태를 입력한다
1. 고객은 아직 요리가 시작되지 않은 주문은 취소할 수 있다
1. 요리가 완료되면 고객의 지역 인근의 라이더들에 의해 배송건 조회가 가능하다
1. 라이더가 해당 요리를 Pick한 후, 앱을 통해 통보한다.
1. 고객이 주문상태를 중간중간 조회한다
1. 주문상태가 바뀔 때 마다 카톡으로 알림을 보낸다
1. 고객이 요리를 배달 받으면 배송확인 버튼을 탭하여, 모든 거래가 완료된다


###비기능적 요구사항
1. 장애격리
 - 상점관리 기능이 수행되지 않더라도 주문은 365일 24시간 받을 수 있어야 한다 Async (event-driven), Eventual Consistency
 - 결제시스템이 과중되면 사용자를 잠시동안 받지 않고 결제를 잠시후에 하도록 유도한다 Circuit breaker, fallback
2. 성능
 - 고객이 자주 상점관리에서 확인할 수 있는 배달상태를 주문시스템(프론트엔드)에서 확인할 수 있어야 한다 CQRS
 - 배달상태가 바뀔때마다 카톡 등으로 알림을 줄 수 있어야 한다 Event driven


## 완성된 1차 모형
![image](https://user-images.githubusercontent.com/61446346/206143689-14f04447-700b-4ac0-822f-ca2c3ef64b0c.png)

### 시나리오 적용
![image](https://user-images.githubusercontent.com/61446346/206149719-cb7a2d68-5f6d-478e-995e-717b95c769c9.png)
```
1. 고객이 메뉴를 선택하여 주문한다. (ok)
2. 고객이 선택한 메뉴에 대해 결제한다.(ok)
6. 고객은 아직 요리가 시작되지 않은 주문은 취소할 수 있다(ok)
9. 고객이 주문상태를 중간중간 조회한다(ok)
```

![image](https://user-images.githubusercontent.com/61446346/206151197-7b8c9b4e-0b3f-4f5b-b4e2-f07e6066ab06.png)
```
3. 주문이 되면 주문 내역이 입점상점주인에게 주문정보가 전달된다 (ok)
4. 상점주는 주문을 수락하거나 거절할 수 있다 (ok)
5. 상점주는 요리시작때와 완료 시점에 시스템에 상태를 입력한다 (ok)
7. 요리가 완료되면 고객의 지역 인근의 라이더들에 의해 배송건 조회가 가능하다 (ok)
```

![image](https://user-images.githubusercontent.com/61446346/206153621-7bedc111-ec8c-4209-80d1-177e95c6e122.png)
```
8. 라이더가 해당 요리를 Pick한 후, 앱을 통해 통보한다.
10. 주문상태가 바뀔 때 마다 카톡으로 알림을 보낸다
11. 고객이 요리를 배달 받으면 배송확인 버튼을 탭하여, 모든 거래가 완료된다
```

### 모델 수정/추가 적용사항
![image](https://user-images.githubusercontent.com/61446346/206154463-9d8b0b65-8375-42e6-8643-0dfd130e5946.png)

```
1. order상태변경시 고객에게 비동기적으로 SMS알람처리를 하고 부하발생시 알람처리를 중단한다.
2. 배달완료가 되면 고객 등급을 상향한다. 
```   

## 체크포인트
### 1. Saga (Pub / Sub)

  #### 구현 : Order커맨드로 주문시 주문정보는 kafka에 저장되며 store에서는 해당 오더정보를 확인할 수 있다.
  - ![image](https://user-images.githubusercontent.com/61446346/205812757-49e1c8be-4159-4254-b2a5-e3b7eadb3c70.png)
  
  #### 실행
  - Order커맨드 실행
  - ![image](https://user-images.githubusercontent.com/61446346/205813102-0f9f9a12-7f77-495c-a055-af23e0da81e8.png)
  - Store에서 오더정보 확인
  - ![image](https://user-images.githubusercontent.com/61446346/205835532-0b51f886-871e-4151-afcc-720e605cf599.png)


  #### kafka 확인
  - ![image](https://user-images.githubusercontent.com/61446346/205813194-69878c3d-f958-4399-ae41-6200670551c3.png)


### 2. CQRS

  - 구현 : 오더주문시 orderView 정보를 생성한고, 각 단계전진시 orderStatus상태를 현행화 관리한다.
  - ![image](https://user-images.githubusercontent.com/61446346/205814117-7aa5d785-2d93-4d1a-90bd-8eb501648efe.png)

  - 확인 : 오더주문시 생성된 데이터 
  - ![image](https://user-images.githubusercontent.com/61446346/205814569-86d3e309-477c-40b1-8c9e-665be1c90f42.png)

### 3. Compensation / Correlation

  #### 구현 : 오더주문커맨드 실행시 오더정보 kafka에 적재, 오더캔슬커맨드 실행시 오더정보를 삭제한다.
  - 오더주문 
  - ![image](https://user-images.githubusercontent.com/61446346/205846913-33451e38-0195-4ba1-a04e-4199dd09fd93.png)

  - 오더취소
  - ![image](https://user-images.githubusercontent.com/61446346/205847021-47ff372f-da6d-409c-9ec3-d19fe2859461.png)

  #### 실행 : 
  - 오더주문
  - ![image](https://user-images.githubusercontent.com/61446346/205847148-833f5e98-7df1-4639-b246-6c8c3b2dd445.png)

  #### kafka 확인 : 
  - 오더주문
  - ![image](https://user-images.githubusercontent.com/61446346/205847939-52f2d793-e957-4a17-883f-b6c008893146.png)

  - 오더취소
  - ![image](https://user-images.githubusercontent.com/61446346/205848028-f28981bb-3cec-48a1-babb-c0a153ea8248.png)

  - 오더정보 확인 : orderId = 5
  - ![image](https://user-images.githubusercontent.com/61446346/205848158-51867ac1-c3d2-4edf-8b55-ca709a3faa43.png)

  
### 4. Request / Response

  #### 구현 : 주문정보 조회시 http GET 구현
  - ![image](https://user-images.githubusercontent.com/61446346/205844002-80b55f91-6d3c-48f7-9e2e-38e48e8d2f5c.png)

  #### 실행 : 
  -   ![image](https://user-images.githubusercontent.com/61446346/205843890-597798c8-1605-4fd9-a70e-0763b2afa0d6.png)


### 5. Circuit Breaker



### 6. Gateway / Ingress

  - 구현
  - ![image](https://user-images.githubusercontent.com/61446346/205819140-dc60d97e-0355-4d72-96af-8fe8560089df.png)

  - 실행
  - ![image](https://user-images.githubusercontent.com/61446346/205817463-37fa0a22-7f8b-4e2d-8be3-aa76d4aabb92.png)





## 헥사고날 아키텍처 다이어그램 도출
    
![image](https://user-images.githubusercontent.com/487999/79684772-eba9ab00-826e-11ea-9405-17e2bf39ec76.png)


    - Chris Richardson, MSA Patterns 참고하여 Inbound adaptor와 Outbound adaptor를 구분함
    - 호출관계에서 PubSub 과 Req/Resp 를 구분함
    - 서브 도메인과 바운디드 컨텍스트의 분리:  각 팀의 KPI 별로 아래와 같이 관심 구현 스토리를 나눠가짐


# 구현:

분석/설계 단계에서 도출된 헥사고날 아키텍처에 따라, 각 BC별로 대변되는 마이크로 서비스들을 스프링부트와 파이선으로 구현하였다. 구현한 각 서비스를 로컬에서 실행하는 방법은 아래와 같다 (각자의 포트넘버는 8081 ~ 808n 이다)

```
cd app
mvn spring-boot:run

cd pay
mvn spring-boot:run 

cd store
mvn spring-boot:run  

cd customer
python policy-handler.py 
```

## DDD 의 적용

- 각 서비스내에 도출된 핵심 Aggregate Root 객체를 Entity 로 선언하였다: (예시는 pay 마이크로 서비스). 이때 가능한 현업에서 사용하는 언어 (유비쿼터스 랭귀지)를 그대로 사용하려고 노력했다. 하지만, 일부 구현에 있어서 영문이 아닌 경우는 실행이 불가능한 경우가 있기 때문에 계속 사용할 방법은 아닌것 같다. (Maven pom.xml, Kafka의 topic id, FeignClient 의 서비스 id 등은 한글로 식별자를 사용하는 경우 오류가 발생하는 것을 확인하였다)

```
package fooddelivery;

import javax.persistence.*;
import org.springframework.beans.BeanUtils;
import java.util.List;

@Entity
@Table(name="결제이력_table")
public class 결제이력 {

    @Id
    @GeneratedValue(strategy=GenerationType.AUTO)
    private Long id;
    private String orderId;
    private Double 금액;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }
    public String getOrderId() {
        return orderId;
    }

    public void setOrderId(String orderId) {
        this.orderId = orderId;
    }
    public Double get금액() {
        return 금액;
    }

    public void set금액(Double 금액) {
        this.금액 = 금액;
    }

}

```
- Entity Pattern 과 Repository Pattern 을 적용하여 JPA 를 통하여 다양한 데이터소스 유형 (RDB or NoSQL) 에 대한 별도의 처리가 없도록 데이터 접근 어댑터를 자동 생성하기 위하여 Spring Data REST 의 RestRepository 를 적용하였다
```
package fooddelivery;

import org.springframework.data.repository.PagingAndSortingRepository;

public interface 결제이력Repository extends PagingAndSortingRepository<결제이력, Long>{
}
```
- 적용 후 REST API 의 테스트
```
# app 서비스의 주문처리
http localhost:8081/orders item="통닭"

# store 서비스의 배달처리
http localhost:8083/주문처리s orderId=1

# 주문 상태 확인
http localhost:8081/orders/1

