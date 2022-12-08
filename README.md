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
 #### 설정 : 주문 서비스의 application.xml에 hystrix enabled = true와 임계치를 설정한다.
 ![image](https://user-images.githubusercontent.com/61446346/206486704-1e3eadd4-404e-4b41-8b51-5858d285f386.png)
 
 #### 구현 : 오더주문시 지연처리 되도록 설정한다.
 ![image](https://user-images.githubusercontent.com/61446346/206487332-54119229-9539-4a6b-998f-5cd8f980bf86.png)

 #### 확인 : 서비스 실행시, 장시간 처리되지 않고 임계치 도달시 오류처리되는것을 확인한다
 ![image](https://user-images.githubusercontent.com/61446346/206488225-c5b3379b-52a8-43ab-8a01-43656645c0f3.png)


### 6. Gateway / Ingress
 #### 구현 
  - ![image](https://user-images.githubusercontent.com/61446346/205819140-dc60d97e-0355-4d72-96af-8fe8560089df.png)

 #### 실행
  - ![image](https://user-images.githubusercontent.com/61446346/205817463-37fa0a22-7f8b-4e2d-8be3-aa76d4aabb92.png)





