# 이성한 개인평가
![image](https://user-images.githubusercontent.com/102270635/162107476-f46a7466-4ae4-4809-b66b-9ff594fddca7.png)


# 예제 - 헬스 용품 온라인 구매 시스템
본 예제는 MSA/DDD/Event Storming/EDA 를 포괄하는 분석/설계/구현/운영 전단계를 커버하도록 구성한 예제입니다. 이는 클라우드 네이티브 애플리케이션의 개발에 요구되는 체크포인트들을 통과하기 위한 예시 답안을 포함합니다.

체크포인트 : https://workflowy.com/s/assessment-check-po/T5YrzcMewfo4J6LW

# Table of Contents 
+ 서비스 시나리오
+ 체크포인트
+ 분석/설계
+ 구현
+ 운영


# 서비스 시나리오

### 기능적 요구사항
  1. 사장님이 헬스 온라인 쇼핑몰에 헬스용품을 등록/수정/삭제한다.
  2. 구매자가 헬스용품을 선택하여 구매한다.
  3. 주문과 동시에 결제가 진행된다.
  4. 결제가 되면 구매 내역 (Message)이 전달된다.
  5. 판매자가 주문을 확인하여 배송을 시작한다.
  6. 고객이 주문을 취소할 수 있다.
  7. 주문이 취소되면 배송이 취소된다.
  8. 주문이 취소될 경우 취소 내역 (Message)이 전달된다.
  9. 배송상태가 바뀔 때 마다 알림을 보낸다

### 비기능적 요구사항
  1. 트랜잭션
     + 결제가 되지 않은 주문건은 거래가 성립되지 성립되지 않아야 한다. (Sync 호출)

  2. 장애격리
     + 상점관리 기능이 수행되지 않더라도 주문은 365일 24시간 받을 수 있어야 한다. [Async (event-driven), Eventual Consistency]
     + 결제시스템이 과중되면 사용자를 잠시동안 받지 않고 결제를 잠시후에 하도록 유도한다. [Circuit breaker, fallback]

  3. 성능
     + 구매자가 상점관리에서 확인할 수 있는 구매 정보 및 배송상태 등을 주문시스템에서 한번에 확인할 수 있어야 한다 [CQRS]
     +  배송상태가 상태가 바뀔 때마다 메시지로 알림을 줄 수 있어야 한다 [Event driven]

# 체크포인트
+ 분석 설계

  + 이벤트스토밍:

    + 스티커 색상별 객체의 의미를 제대로 이해하여 헥사고날 아키텍처와의 연계 설계에 적절히 반영하고 있는가?
    + 각 도메인 이벤트가 의미있는 수준으로 정의되었는가?
    + 어그리게잇: Command와 Event 들을 ACID 트랜잭션 단위의 Aggregate 로 제대로 묶었는가?
    + 기능적 요구사항과 비기능적 요구사항을 누락 없이 반영하였는가?
  
  + 서브 도메인, 바운디드 컨텍스트 분리

    + 팀별 KPI 와 관심사, 상이한 배포주기 등에 따른  Sub-domain 이나 Bounded Context 를 적절히 분리하였고 그 분리 기준의 합리성이 충분히 설명되는가?
    + 적어도 3개 이상 서비스 분리
    + 폴리글랏 설계: 각 마이크로 서비스들의 구현 목표와 기능 특성에 따른 각자의 기술 Stack 과 저장소 구조를 다양하게 채택하여 설계하였는가?
    + 서비스 시나리오 중 ACID 트랜잭션이 크리티컬한 Use 케이스에 대하여 무리하게 서비스가 과다하게 조밀히 분리되지 않았는가?

  + 컨텍스트 매핑 / 이벤트 드리븐 아키텍처

    + 업무 중요성과  도메인간 서열을 구분할 수 있는가? (Core, Supporting, General Domain)
    + Request-Response 방식과 이벤트 드리븐 방식을 구분하여 설계할 수 있는가?
    + 장애격리: 서포팅 서비스를 제거 하여도 기존 서비스에 영향이 없도록 설계하였는가?
    + 신규 서비스를 추가 하였을때 기존 서비스의 데이터베이스에 영향이 없도록 설계(열려있는 아키택처)할 수 있는가?
    + 이벤트와 폴리시를 연결하기 위한 Correlation-key 연결을 제대로 설계하였는가?
  
  + 헥사고날 아키텍처

    + 설계 결과에 따른 헥사고날 아키텍처 다이어그램을 제대로 그렸는가?

+ 구현

    + [DDD] 분석단계에서의 스티커별 색상과 헥사고날 아키텍처에 따라 구현체가 매핑되게 개발되었는가?

      + Entity Pattern 과 Repository Pattern 을 적용하여 JPA 를 통하여 데이터 접근 어댑터를 개발하였는가
      + [헥사고날 아키텍처] REST Inbound adaptor 이외에 gRPC 등의 Inbound Adaptor 를 추가함에 있어서 도메인 모델의 손상을 주지 않고 새로운 프로토콜에 기존 구현체를 적응시킬 수 있는가?
      + 분석단계에서의 유비쿼터스 랭귀지 (업무현장에서 쓰는 용어) 를 사용하여 소스코드가 서술되었는가?

    + Request-Response 방식의 서비스 중심 아키텍처 구현

      + 마이크로 서비스간 Request-Response 호출에 있어 대상 서비스를 어떠한 방식으로 찾아서 호출 하였는가? (Service Discovery, REST, FeignClient)
      + 서킷브레이커를 통하여  장애를 격리시킬 수 있는가?

    + 이벤트 드리븐 아키텍처의 구현

      + 카프카를 이용하여 PubSub 으로 하나 이상의 서비스가 연동되었는가?
      + Correlation-key: 각 이벤트 건 (메시지)가 어떠한 폴리시를 처리할때 어떤 건에 연결된 처리건인지를 구별하기 위한 Correlation-key 연결을 제대로 구현 하였는가?
      + Message Consumer 마이크로서비스가 장애상황에서 수신받지 못했던 기존 이벤트들을 다시 수신받아 처리하는가?
      + Scaling-out: Message Consumer 마이크로서비스의 Replica 를 추가했을때 중복없이 이벤트를 수신할 수 있는가
      + CQRS: Materialized View 를 구현하여, 타 마이크로서비스의 데이터 원본에 접근없이(Composite 서비스나 조인SQL 등 없이) 도 내 서비스의 화면 구성과 잦은 조회가 가능한가?

    + 폴리글랏 플로그래밍

      + 각 마이크로 서비스들이 하나이상의 각자의 기술 Stack 으로 구성되었는가?
      + 각 마이크로 서비스들이 각자의 저장소 구조를 자율적으로 채택하고 각자의 저장소 유형 (RDB, NoSQL, File System 등)을 선택하여 구현하였는가?

   + API 게이트웨이

      + API GW를 통하여 마이크로 서비스들의 집입점을 통일할 수 있는가?
      + 게이트웨이와 인증서버(OAuth), JWT 토큰 인증을 통하여 마이크로서비스들을 보호할 수 있는가?

+ 운영

    + SLA 준수
      + 셀프힐링: Liveness Probe 를 통하여 어떠한 서비스의 health 상태가 지속적으로 저하됨에 따라 어떠한 임계치에서 pod 가 재생되는 것을 증명할 수 있는가?
      + 서킷브레이커, 레이트리밋 등을 통한 장애격리와 성능효율을 높힐 수 있는가?
      + 오토스케일러 (HPA) 를 설정하여 확장적 운영이 가능한가?
      + 모니터링, 앨럿팅
      
    + 무정지 운영 CI/CD (10)
      + Readiness Probe 의 설정과 Rolling update을 통하여 신규 버전이 완전히 서비스를 받을 수 있는 상태일때 신규버전의 서비스로 전환됨을 siege 등으로 증명
      + Contract Test : 자동화된 경계 테스트를 통하여 구현 오류나 API 계약위반를 미리 차단 가능한가?

# 분석/설계
## AS-IS 조직 
![image](https://user-images.githubusercontent.com/102270635/162105049-8c180158-9d2d-4ba7-b9f2-7d0325dbd020.png)

## TO-BE 조직 (Vertically-Aligned)
![image](https://user-images.githubusercontent.com/102270635/162107224-9b4e9248-f77f-4287-92de-8969e6d6fdc2.png)

# Event Storming 결과
> MSAEz 로 모델링한 이벤트 스토밍 결과: 
> https://labs.msaez.io/#/storming/dTF4zdMrxveJgEJ6Yh8n3YazyJf2/66c443fb41f9b19754c89205ab14b055

## 이벤트 도출 
![image](https://user-images.githubusercontent.com/35085704/160337656-ec1eb8f5-7704-4599-87b7-a19bce853c18.png)

## 부적격 이벤트 탈락
![image](https://user-images.githubusercontent.com/102270635/162103631-df897e6f-a27e-40cd-8c0b-e1b5a5a9aeaf.png)

> 과정중 도출된 잘못된 도메인 이벤트들을 걸러내는 작업을 수행함
> 주문시>주문전달완료, 주문시> 상품 선택완료 :  UI 의 이벤트이지, 업무적인 의미의 이벤트가 아니라서 제외

## 액터, 커맨드 부착하여 읽기 좋게 
![image](https://user-images.githubusercontent.com/102270635/162118276-3565bac3-20e5-4032-a875-2f87c6e5d418.png)![image](https://user-images.githubusercontent.com/102270635/162118481-c2d52fe8-8171-4ed2-a403-d3cf1f9d57ba.png)![image](https://user-images.githubusercontent.com/102270635/162118645-76e71f82-d1cb-4796-900c-0c1eb7930bad.png)



## 어그리게잇으로 묶기 
![image](https://user-images.githubusercontent.com/102270635/162118309-6c433c13-9fdc-45a4-beac-fdbe763384d2.png)![image](https://user-images.githubusercontent.com/102270635/162118551-072fe83d-1ed0-472d-824f-da690b67324e.png)![image](https://user-images.githubusercontent.com/102270635/162118712-1a1f2444-9137-4155-8dd0-b1e2bd817d64.png)



> front의 Order, store 의 주문처리, payment 의 결제이력, delivery의 배송현황은 그와 연결된 command 와 event 들에 의하여 트랜잭션이 유지되어야 하는 단위로 그들 끼리 묶어줌

## 바운디드 컨텍스트로 묶기
![image](https://user-images.githubusercontent.com/102270635/162118918-857d6508-deeb-4f16-bffc-685f1f469169.png)![image](https://user-images.githubusercontent.com/102270635/162118990-544fd267-6935-4f55-8e9b-5483606bd479.png)![image](https://user-images.githubusercontent.com/102270635/162119010-76279310-21df-4461-adc0-eeec9c0fb5c1.png)

> 도메인 서열 분리 
>  - Core Domain:  app(front), store : 없어서는 안될 핵심 서비스이며, 연간 Up-time SLA 수준을 99.999% 목표, 배포주기는 app 의 경우 1주일 1회 미만, store 의 경우 1개월 1회 미만
>  - Supporting Domain:   marketing, customer : 경쟁력을 내기위한 서비스이며, SLA 수준은 연간 60% 이상 uptime 목표, 배포주기는 각 팀의 자율이나 표준 스프린트 주기가 1주일 이므로 1주일 1회 이상을 기준으로 함.
>  - General Domain: pay : 결제서비스로 3rd Party 외부 서비스를 사용하는 것이 경쟁력이 높음 

## 폴리시 부착
![image](https://user-images.githubusercontent.com/102270635/162119206-fe1ce057-38b3-43e5-a2f2-bdde1d22e7c1.png)![image](https://user-images.githubusercontent.com/102270635/162119222-9cf7abbc-da8f-4a93-8ff7-da6a4c48b801.png)![image](https://user-images.githubusercontent.com/102270635/162119238-5a503515-5df3-4838-a6b4-1aff5d4abf76.png)![image](https://user-images.githubusercontent.com/102270635/162119261-818e97ba-2198-4ea3-8972-752418ce08be.png)



## 완성된 1차 모형
![image](https://user-images.githubusercontent.com/102270635/162120039-278fa31e-676e-4d87-9921-95115a2b6226.png)



> View Model 및 폴리시의 이동과 컨텍스트 매핑 추가


## 1차 완성본에 대한 기능적/비기능적 요구사항을 커버하는지 검증
![image](https://user-images.githubusercontent.com/102270635/162133989-45a34fdb-f151-47d2-a62e-549bf6947c2f.png)

> 기능적 요구사항
>  1. 온라인 쇼핑몰 사장이 헬스용품을 등록/수정/삭제한다. (ok)
>  2. 구매자가 헬스용품을 선택하여 구매한다. (ok)
>  3. 주문과 동시에 결제가 진행된다. (ok)
>  4. 결제가 되면 구매 내역 (Message)이 전달된다. (not yet)
>  5. 판매자가 주문을 확인하여 배송을 시작한다. (ok)
>  6. 고객이 주문을 취소할 수 있다. (ok)
>  7. 주문이 취소되면 배송이 취소된다. (not yet)
>  8. 주문이 취소될 경우 취소 내역 (Message)이 전달된다. (not yet)
>  9. 배송상태가 바뀔 때 마다 알림을 보낸다. (not yet)


## 모델 수정
![image](https://user-images.githubusercontent.com/102270635/162119296-12b90d46-a9ab-417f-b84a-81b70b6c3c9f.png)
> 수정된 모델은 모든 요구사항을 커버함

## 비기능 요구사항에 대한 검증
> 마이크로 서비스를 넘나드는 시나리오에 대한 트랜잭션 처리
>  - 고객 주문시 결제처리:  결제가 완료되지 않은 주문은 미처리됨에 따라, ACID 트랜잭션 적용. 주문완료시 결제처리에 대해서는 Request-Response 방식 처리.
>  - 결제 완료시 점주연결 및 배송처리:  App(front) 에서 Store 마이크로서비스로 주문요청이 전달되는 과정에 있어서 Store 마이크로 서비스가 별도의 배포주기를 가지기 때문에 Eventual > Consistency 방식으로 트랜잭션 처리함.
>  - 나머지 모든 inter-microservice 트랜잭션: 주문상태, 배달상태 등 모든 이벤트에 대해 카톡을 처리하는 등, 데이터 일관성의 시점이 크리티컬하지 않은 모든 경우가 대부분이라 판단, Eventual Consistency 를 기본으로 채택함.

## 헥사고날 아키텍처 다이어그램 도출
<img width="868" alt="image" src="https://user-images.githubusercontent.com/54835264/160537929-f9e1d9b8-dae3-4b73-b569-e8b1502b04b7.png">


> - Chris Richardson, MSA Patterns 참고하여 Inbound adaptor와 Outbound adaptor를 구분함
> - 호출관계에서 PubSub 과 Req/Resp 를 구분함
> - 서브 도메인과 바운디드 컨텍스트의 분리:  각 팀의 KPI 별로 아래와 같이 관심 구현 스토리를 나눠가짐


# 구현:
분석/설계 단계에서 도출된 헥사고날 아키텍처에 따라, 각 BC별로 대변되는 마이크로 서비스들을 스프링부트와 자바로 구현하였다. 구현한 각 서비스를 로컬에서 실행하는 방법은 아래와 같다 (각자의 포트넘버는 8081 ~ 808n 이다)

```
cd app
mvn spring-boot:run

cd pay
mvn spring-boot:run 

cd store
mvn spring-boot:run  

cd delivery
mvn spring-boot:run  
```

## Saga 

1. order 서비스의 이벤트 Publish
http localhost:8081/orders productId=1 productName="Dumbbell" qty=3

2. OrderApplication에서 Run 실행 후 kafka Consumer에서 이벤트 확인
![image](https://user-images.githubusercontent.com/102270635/162133699-ac740f6b-9caa-458e-8317-cb530a9feff0.png)
![image](https://user-images.githubusercontent.com/102270635/162133815-32e0ea1b-2d5d-4085-b055-6b80bdbca9f6.png)
![image](https://user-images.githubusercontent.com/102270635/162133843-fb0212e6-c51e-43f7-b692-8b6b23273414.png)


## CQRS
> 주문상태와 배송상태 등 총 Status에 대해서 확인 할 수 있도록 CQRS로 구현하였다.
> 비동기식으로 처리되어 발행된 이벤트 기반 Kafka를 통해 수신/처리 되어 별도 OrderStatus table에서 관리한다.

1. OrderView 서비스의 PolicyHandler를 통해 구현
OrderPlaced, DeliveryStarted, OrderCancelled, DeliveryCancelled 이벤트 발생시, Pub/Sub 기반으로 별도 OrderStatus 테이블에 저장
![image](https://user-images.githubusercontent.com/102270635/162134408-81a9d950-8742-4685-b7d3-26397cd1a9d4.png)
![image](https://user-images.githubusercontent.com/102270635/162134496-d9b8ce00-171c-4651-ab57-6d3454e0151d.png)


2. OrderStatus 조회시 주문상태/배달상태 등의 정보를 종합적으로 확인
* 헬스용품 주문

![image](https://user-images.githubusercontent.com/102270635/162138922-ef6bda95-eca6-4832-a2cf-57ca831641b3.png)


* 헬스용품 주문취소, 배달취소 - (안됨)



## Correlation
> 배송서비스에서 주문 삭제시 배송을 취소하는 작업을 위해 PolicyHandler 수정
![image](https://user-images.githubusercontent.com/102270635/162139402-ecb10c5a-57ca-4ac4-b989-73c3c7a07846.png)


1. 주문생성
 
![image](https://user-images.githubusercontent.com/102270635/162139537-9078c823-a336-4f0f-aede-85ccd473478e.png)

2. 주문취소 

![image](https://user-images.githubusercontent.com/102270635/162139606-75f7bea9-b9db-4f9d-b6ce-f514b08e4870.png)

 * 주문(Order)을 하면 동시에 배송(Delivery) 서비스 상태가 변경됨(Kafka를 통해 확인)
 
 * 주문취소를 수행하면 다시 배송(Delivery) 서비스 상태가 변경됨(Kafka를 통해 확인)
 
![image](https://user-images.githubusercontent.com/102270635/162139686-f462a5a9-2138-47bc-a10f-e26a53c02830.png)



## Req/Resp / Circuit Breaker

1. 주문 생성

http localhost:8081/orders productId=1 quantity=3 customerId="sunghan.lee@hanwha.com" customerName="Sunghan" customerAddr="seoul"

![image](https://user-images.githubusercontent.com/102270635/162145649-6146327d-e95d-49db-b281-8dccceaa62f5.png)


2. 기존 Monolith 구현체 제거, FeignClient 활성화

![image](https://user-images.githubusercontent.com/102270635/162142678-2aca9d6b-b731-4a1e-bf37-17fb2108cafc.png)


3. 배송 확인

http localhost:8082/deliveries

![image](https://user-images.githubusercontent.com/102270635/162145717-7a51f158-ff55-4792-87a3-247f273747f9.png)



4. 부하 툴을 사용하여 주문 생성

siege -c2 -t10S  -v --content-type "application/json" 'http://localhost:8081/orders POST {"productId":2, "quantity":1}

* 서킷브레이커 설정

![image](https://user-images.githubusercontent.com/102270635/162146529-1bf2a7ea-abc8-4fcc-9003-a4dea2f83136.png)


* Delivery.java에 딜레이 설정

![image](https://user-images.githubusercontent.com/102270635/162146678-60d8b723-6a0a-4d7d-a768-e5fb7bd966dd.png)


<부하 전>

![image](https://user-images.githubusercontent.com/102270635/162146915-b31cc683-d8b6-4962-9226-7b378f2d5292.png)


<부하 후>

![image](https://user-images.githubusercontent.com/102270635/162147333-531e33dc-6a5a-42ec-a67e-d3469637ac2a.png)



4. fallback 처리를 하여 유연하게 대처

* 폴백옵션 추가

![image](https://user-images.githubusercontent.com/102270635/162147694-d6ccfe56-7ad0-4480-86f0-116613b28c44.png)


<fallback 처리 후> 

![image](https://user-images.githubusercontent.com/102270635/162148203-06725b63-4691-41c0-8ec1-3813ed02b3d8.png)



## API Gateway
1. Spring Gateway 서비스를 추가후 application.yaml 내에 각 마이크로서비스의 routes를 추가함

```
server:
  port: 8088

---

spring:
  profiles: default
  cloud:
    gateway:
      routes:
        - id: front
          uri: http://localhost:8081
          predicates:
            - Path=/orders/**, /payments/** 
        - id: store
          uri: http://localhost:8082
          predicates:
            - Path=/storeOrders/**, /menus/** 
        - id: delivery
          uri: http://localhost:8083
          predicates:
            - Path=/deliveries/** 
        - id: frontend
          uri: http://localhost:8080
          predicates:
            - Path=/**
      globalcors:
        corsConfigurations:
          '[/**]':
            allowedOrigins:
              - "*"
            allowedMethods:
              - "*"
            allowedHeaders:
              - "*"
            allowCredentials: true
```


# 운영
## Deploy
1. AWS CodeBuild 를 통한 health store, delivery 서비스 배포

![image](https://user-images.githubusercontent.com/102270635/162180962-c76d4c56-c886-40df-b566-4e2bcedfc61f.png)

![image](https://user-images.githubusercontent.com/102270635/162177703-cfb8469b-05f7-4e23-bf24-cf227c6e23ba.png)


## AutoScale (HPA) - 
1. HPA(Horizonal Pod Autoscale)를 cpu임계치=50%, 최대Pod수=10, 최저Pod수=1 로 설정
```
kubectl autoscale deployment php-apache --cpu-percent=50 --min=1 --max=10
```

![image](https://user-images.githubusercontent.com/102270635/162341481-72b43dcf-7488-4412-a2fb-2189120d4549.png)


3. siege 컨테이너 배포 후 siege 컨테이너 부하 테스트를 실행
```
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: siege
spec:
  containers:
  - name: siege
    image: apexacme/siege-nginx
EOF
```

```
kubectl exec -it siege -- /bin/bash
siege -c30 -t30S -v http://php-apache 
```

![image](https://user-images.githubusercontent.com/102270635/162343371-4f2fe5fa-eaf4-47de-983b-ca085449312e.png)

cpu임계치=20%, 최대Pod수=3, 최저Pod수=1 로 재설정 및 테스트 수행
kubectl autoscale deployment php-apache --cpu-percent=20 --min=1 --max=3


## Self-healing (Liveness Probe)
1. Liveness Probe 설정 후 재배포 실행

kubectl apply -f exec-liveness.yaml

```
livenessProbe:
  httpGet:
    path: /actuator/health
    port: 8080
  initialDelaySeconds: 120
  timeoutSeconds: 2
  periodSeconds: 5
  failureThreshold: 5
```

* Liveness Probe가 적용된 주문 마이크로서비스를 배포

kubectl apply -f https://raw.githubusercontent.com/acmexii/demo/master/edu/order-liveness.yaml

* Order 서비스 생성

kubectl expose deploy order --type=LoadBalancer --port=8080

* EXTERNAL-IP으로 Livenss Prove 설정 및 확인

-- Liveness Probe 확인
http 218.236.22.35:8080/actuator/health
-- Liveness Probe Fail 설정 및 확인
http put 218.236.22.35:8080/actuator/down
http 218.236.22.35:8080/actuator/health

![image](https://user-images.githubusercontent.com/102270635/162345188-003af6bf-1a95-46fa-80ce-21d3db8cc578.png)

* Probe Fail에 따른 쿠버네티스 동작 확인

kubectl describe pod/health-order-f4f9849d8-d6n67

![image](https://user-images.githubusercontent.com/102270635/162345410-5c026d4c-5b37-4a6d-8d93-364fff77f711.png)


## Zero-Downtime Deploy (Readiness Probe)

1. 배송 마이크로서비스를 배포

kubectl apply -f https://raw.githubusercontent.com/acmexii/demo/master/edu/delivery-rediness-v1.yaml
kubectl expose deploy delivery --port=8080

2. Readiness Probe 설정 후 재배포 실행

kubectl apply -f https://raw.githubusercontent.com/acmexii/demo/master/edu/delivery-no-rediness-v2.yaml

![image](https://user-images.githubusercontent.com/102270635/162347448-7b6d8ec5-5a88-41a8-8097-1f8b3fddb074.png)


3. Readiness Probe 설정 이후 배포 시 siege 테스트 결과가 Availability 100% 임을 확인

![image](https://user-images.githubusercontent.com/102270635/162346880-b4f140b5-5fc5-4df2-8049-ec103d79f516.png)



## ConfigMapig

* ConfigMap 생성
 
 kubectl create configmap health-store  --from-literal=language=java
 
 * ConfigMAp 확인

![image](https://user-images.githubusercontent.com/102270635/162364584-33066226-2ca2-4f7b-9141-d7efc7e3e93d.png)

* configmap을 통한 마이크로서비스 배포(AWS)

 - 도커이미지 빌드 & Push

docker build -t 979050235289.dkr.ecr.ca-central-1.amazonaws.com/cm-sandbox:v1 .

![image](https://user-images.githubusercontent.com/102270635/162364919-b85f748f-fff4-43ca-b76c-2141a61568d8.png)


aws ecr create-repository --repository-name cm-sandbox --region ca-central-1

![image](https://user-images.githubusercontent.com/102270635/162364953-a67d0272-2cd2-42b1-95b3-9d768da518b1.png)


docker push  979050235289.dkr.ecr.ca-central-1.amazonaws.com/cm-sandbox:v1

![image](https://user-images.githubusercontent.com/102270635/162365334-8f8553c2-1258-4d37-a23f-9ee3d3ef0822.png)

컨테이너 및 서비스 생성

kubectl create -f cm-deployment.yaml
kubectl create -f cm-service.yaml

* Secret 생성 및 설정

kubectl create secret generic my-password --from-literal=password=mysqlpassword

![image](https://user-images.githubusercontent.com/102270635/162366495-ac7dec67-e3f8-4341-8e1b-39cf066cc8d1.png)





