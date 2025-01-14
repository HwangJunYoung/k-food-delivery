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

# 체크포인트
## Microservice Implementation
### 1. Saga (Pub / Sub)

  #### 구현 : Order커맨드로 주문시 주문정보는 kafka에 저장되며 store에서는 해당 오더정보를 확인할 수 있다.
   - ![image](https://user-images.githubusercontent.com/13827032/219181502-8f33ee91-5f3c-44fb-bdd9-8e0b10ab756e.png)
  
### 2. CQRS
  #### 구현 : 오더주문시 orderView 정보를 생성한고, 각 단계전진시 orderStatus상태를 현행화 관리한다.
  - ![image](https://user-images.githubusercontent.com/13827032/219182613-30b37b93-bcb2-466e-a6f1-2bafbbc56f2d.png)

### 3. Compensation / Correlation

  #### 구현 : 오더주문커맨드 실행시 오더정보 kafka에 적재, 오더캔슬커맨드 실행시 오더정보를 삭제한다.
  - 오더주문 
  - ![image](https://user-images.githubusercontent.com/13827032/219181502-8f33ee91-5f3c-44fb-bdd9-8e0b10ab756e.png)

  - 오더취소
  - ![image](https://user-images.githubusercontent.com/13827032/219183570-9be5b875-ff7f-45d8-aa7c-ef2ff989dd9a.png)
  
## Microservice Orchestration
### 1. Deploy to EKS Cluster
  #### 서비스 목록
  ![image](https://user-images.githubusercontent.com/13827032/219187909-85cafabb-5102-430c-b357-2d661280592e.png)

### 2. Gateway & Service Router 설치
 #### 서비스 목록
 ![image](https://user-images.githubusercontent.com/13827032/219188072-59bc89a7-e609-41cd-afe1-e9d43c1a03eb.png)
 
### 3.  Autoscale (HPA)
 #### 부하 발생 전
  ![image](https://user-images.githubusercontent.com/13827032/219223756-a296c0c7-9afc-4171-9ae2-8d45dff99fa1.png)
 #### 부하 발생 후
  ![image](https://user-images.githubusercontent.com/13827032/219224311-38fb7b9f-0604-4040-8f4c-cad8d96feb5c.png)
 #### pod 증가
  ![image](https://user-images.githubusercontent.com/13827032/219224443-a01133cd-ad27-4441-aaab-18e25acf84fa.png)


