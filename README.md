# msa 4조
![image](https://user-images.githubusercontent.com/35085704/160336168-704f0b03-5ce5-4d24-84b1-41a363904cf5.png)

# 예제 - 온라인 도서 구매 시스템
본 예제는 MSA/DDD/Event Storming/EDA 를 포괄하는 분석/설계/구현/운영 전단계를 커버하도록 구성한 예제입니다. 이는 클라우드 네이티브 애플리케이션의 개발에 요구되는 체크포인트들을 통과하기 위한 예시 답안을 포함합니다.

체크포인트 : https://workflowy.com/s/assessment-check-po/T5YrzcMewfo4J6LW

# Table of Contents 
+ 서비스 시나리오
+ 체크포인트
+ 분석/설계
+ 구현
  + DDD 의 적용
  + 폴리글랏 퍼시스턴스
  + 폴리글랏 프로그래밍
  + 동기식 호출 과 Fallback 처리
  + 비동기식 호출 과 Eventual Consistency
+ 운영
  + CI/CD 설정
  + 동기식 호출 / 서킷 브레이킹 / 장애격리
  + 오토스케일 아웃
  + 무정지 재배포

# 서비스 시나리오

### 기능적 요구사항
  1. 서점 점주가 책을 등록/수정/삭제한다.
  2. 구매자가 책을 선택하여 구매한다.
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
![image](https://user-images.githubusercontent.com/35085704/160333984-b35900c7-3066-4c96-846d-5ec0a0356496.png)

## TO-BE 조직 (Vertically-Aligned)
![image](https://user-images.githubusercontent.com/35085704/160335058-408f670e-46f3-4e57-8a77-c969ba126009.png)

# Event Storming 결과
+ MSAEz 로 모델링한 이벤트 스토밍 결과: https://labs.msaez.io/#/storming/RWvnPeZmdwhH2VUyVhPmZ5pZgLw2/4e8fde73cfb97272d06117d1e84f642d

## 이벤트 도출 
![image](https://user-images.githubusercontent.com/35085704/160337656-ec1eb8f5-7704-4599-87b7-a19bce853c18.png)

## 부적격 이벤트 탈락
![image](https://user-images.githubusercontent.com/35085704/160337865-99e4a56d-1bde-403a-a66e-5c515a3f2622.png)

```
- 과정중 도출된 잘못된 도메인 이벤트들을 걸러내는 작업을 수행함
    - 주문시>주문전달완료, 주문시>책선택완료 :  UI 의 이벤트이지, 업무적인 의미의 이벤트가 아니라서 제외
```

## 액터, 커맨드 부착하여 읽기 좋게 
![image](https://user-images.githubusercontent.com/35085704/160338111-a11cc01d-76e8-4f21-9aa2-77b9b2ad0d96.png) ![image](https://user-images.githubusercontent.com/35085704/160338158-bdae7868-0e33-4dfd-a011-a7d86fbd4ceb.png) ![image](https://user-images.githubusercontent.com/35085704/160338221-a6a333c2-52b0-4767-b39b-c9d99e6cf94e.png) 

## 어그리게잇으로 묶기 
![image](https://user-images.githubusercontent.com/35085704/160338265-4ef2675f-6c2e-45ae-8596-09eb0ff30de2.png) ![image](https://user-images.githubusercontent.com/35085704/160338278-dd654553-2f92-49ef-b98b-220d8587c3d3.png) ![image](https://user-images.githubusercontent.com/35085704/160338316-3c44bf02-c708-4671-b84b-52ed5bf6fb46.png)

```
- front의 Order, store 의 주문처리, payment 의 결제이력, delivery의 배송현황은 그와 연결된 command 와 event 들에 의하여 트랜잭션이 유지되어야 하는 단위로 그들 끼리 묶어줌
```

## 바운디드 컨텍스트로 묶기
![image](https://user-images.githubusercontent.com/35085704/160338374-5d1fa9bc-5fa4-4362-84c0-952752bf487b.png) ![image](https://user-images.githubusercontent.com/35085704/160338394-a999f49b-de1a-4691-885f-9d24be03f597.png) 

```
- 도메인 서열 분리 
    - Core Domain:  app(front), store : 없어서는 안될 핵심 서비스이며, 연간 Up-time SLA 수준을 99.999% 목표, 배포주기는 app 의 경우 1주일 1회 미만, store 의 경우 1개월 1회 미만
    - Supporting Domain:   marketing, customer : 경쟁력을 내기위한 서비스이며, SLA 수준은 연간 60% 이상 uptime 목표, 배포주기는 각 팀의 자율이나 표준 스프린트 주기가 1주일 이므로 1주일 1회 이상을 기준으로 함.
    - General Domain: pay : 결제서비스로 3rd Party 외부 서비스를 사용하는 것이 경쟁력이 높음 
```

## 폴리시 부착
![image](https://user-images.githubusercontent.com/35085704/160339429-1760c79e-e2f5-4e5b-852f-1bf9d3a7299f.png)

## 완성된 1차 모형
![image](https://user-images.githubusercontent.com/35085704/160339612-0078086d-2b7c-4a1a-81b6-a8cbe2da0e41.png)

```
- View Model 및 폴리시의 이동과 컨텍스트 매핑 추가
```

## 1차 완성본에 대한 기능적/비기능적 요구사항을 커버하는지 검증
![image](https://user-images.githubusercontent.com/35085704/160340044-8344cc1a-a433-4b39-bbe6-dd2c1ea85bd5.png)

```
### 기능적 요구사항
  1. 서점 점주가 책을 등록/수정/삭제한다. (ok)
  2. 구매자가 책을 선택하여 구매한다. (ok)
  3. 주문과 동시에 결제가 진행된다. (ok)
  4. 결제가 되면 구매 내역 (Message)이 전달된다. (not yet)
  5. 판매자가 주문을 확인하여 배송을 시작한다. (ok)
  6. 고객이 주문을 취소할 수 있다. (ok)
  7. 주문이 취소되면 배송이 취소된다. (not yet)
  8. 주문이 취소될 경우 취소 내역 (Message)이 전달된다. (not yet)
  9. 배송상태가 바뀔 때 마다 알림을 보낸다. (not yet)
```


## 모델 수정
![image](https://user-images.githubusercontent.com/35085704/160341656-3b89ce47-5c5d-44cf-8618-5399dbe7397a.png)
```
- 수정된 모델은 모든 요구사항을 커버함
```

## 비기능 요구사항에 대한 검증
```
- 마이크로 서비스를 넘나드는 시나리오에 대한 트랜잭션 처리
    - 고객 주문시 결제처리:  결제가 완료되지 않은 주문은 미처리됨에 따라, ACID 트랜잭션 적용. 주문완료시 결제처리에 대해서는 Request-Response 방식 처리.
    - 결제 완료시 점주연결 및 배송처리:  App(front) 에서 Store 마이크로서비스로 주문요청이 전달되는 과정에 있어서 Store 마이크로 서비스가 별도의 배포주기를 가지기 때문에 Eventual Consistency 방식으로 트랜잭션 처리함.
    - 나머지 모든 inter-microservice 트랜잭션: 주문상태, 배달상태 등 모든 이벤트에 대해 카톡을 처리하는 등, 데이터 일관성의 시점이 크리티컬하지 않은 모든 경우가 대부분이라 판단, Eventual Consistency 를 기본으로 채택함.
```

## 헥사고날 아키텍처 다이어그램 도출

```
- Chris Richardson, MSA Patterns 참고하여 Inbound adaptor와 Outbound adaptor를 구분함
- 호출관계에서 PubSub 과 Req/Resp 를 구분함
- 서브 도메인과 바운디드 컨텍스트의 분리:  각 팀의 KPI 별로 아래와 같이 관심 구현 스토리를 나눠가짐
```

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

## CQRS
주문상태와 배송상태 등 총 Status에 대해서 확인 할 수 있도록 CQRS로 구현하였다.
비동기식으로 처리되어 발행된 이벤트 기반 Kafka를 통해 수신/처리 되어 별도 OrderStatus table에서 관리한다.
+ OrderStatus

![image](https://user-images.githubusercontent.com/77971366/160515900-2d93c9f1-7c0d-4888-a04d-e5d6f2e03bb5.png)

+ OrderView 서비스의 PolicyHandler를 통해 구현
+ OrderPlaced, DeliveryStarted, OrderCancelled. DeliveryCancelled 이벤트 발생시, Pub/Sub 기반으로 별도 OrderStatus 테이블에 저장

![image](https://user-images.githubusercontent.com/77971366/160516067-c82a0a47-1abf-439d-a5b1-509be67c363a.png)
![image](https://user-images.githubusercontent.com/77971366/160516122-8c193862-edfe-48b4-a68c-704010833b4d.png)

+ OrderStatus 조회시 주문상태/배달상태 등의 정보를 종합적으로 알 수 있다.
- 책 주문
<img width="413" alt="HTTP1 1 201" src="https://user-images.githubusercontent.com/77971366/160517424-7a10b354-d459-4d32-ad88-a2c8d9bf2457.png">
![image](https://user-images.githubusercontent.com/77971366/160516279-47f052c6-005a-4ea6-9027-822a2c5dc09a.png)

- 책 주문취소, 배달취소
![image](https://user-images.githubusercontent.com/77971366/160516331-16237f0c-3460-435a-bae0-d00160c5e8be.png)

## API Gateway
Spring Gateway 서비스를 추가후 application.yaml 내에 각 마이크로서비스의 routes를 추가함

![image](https://user-images.githubusercontent.com/77971366/160516379-da586d99-6e89-4d9a-9766-6ad7bc4d58dc.png)

## Correlation
BookStore 프로젝트에서는 PolicyHandler에서 처리 시 어떤 건에 대한 처리인지를 구별하기 위한 Correlation-key 구현을 이벤트 클래스 안의 변수로 전달받아 서비스간 연관된 처리를 정확하게 구현하고 있다.
주문(Order)을 하면 동시에 배송(Delivery)등의 서비스 상태가 변경되고, 주문취소를 수행하면 다시 배송(Delivery)등의 서비스 상태값 등의 데이터가 적당한 상태로 변경된다.

![image](https://user-images.githubusercontent.com/77971366/160517019-7abca190-01c3-4edf-b67c-54d9ab4dfafd.png)
![image](https://user-images.githubusercontent.com/77971366/160517028-17e9db06-da6c-4360-9ce9-070b47870e6e.png)


## DDD의 적용

## 동기식 호출 (Sync) 와 Fallback 처리
**Req/Res 연동**

1. 주문 생성
http localhost:8081/orders productId=1 quantity=3 customerId="sunghan.lee@hanwha.com" customerName="Lee" customerAddr="seoul"
![image](https://user-images.githubusercontent.com/102270635/160507856-40a69811-e01d-430f-a2f5-e2313cc2608b.png)



2. 배송 확인
http localhost:8082/deliveries
![image](https://user-images.githubusercontent.com/102270635/160507815-d00e48e4-dcc1-47c6-acd3-750818c1047a.png)


3. 부하 툴을 사용하여 주문 생성
siege -c2 -t10S  -v --content-type "application/json" 'http://localhost:8081/orders POST {"productId":2, "quantity":1}

<부하 전>
![image](https://user-images.githubusercontent.com/102270635/160507930-4c679192-cbf2-419b-a275-30496384bcd1.png)

<부하 후>
![image](https://user-images.githubusercontent.com/102270635/160507945-1201381f-8d3e-4787-8c26-7810eded63b0.png)


4. fallback 처리를 하여 유연하게 대처
<fallback 처리 후>
![image](https://user-images.githubusercontent.com/102270635/160507989-f5f3462d-ca00-4aa8-8310-73dce9b3d419.png)
## 비동기식 호출 / 시간적 디커플링 / 장애격리 / 최종 (Eventual) 일관성 테스트

# 운영
## Deploy
1. AWS CodeBuild 를 통한 store, delivery 서비스 배포
<img width="1338" alt="스크린샷 2022-03-29 오전 10 58 44" src="https://user-images.githubusercontent.com/54835264/160518739-c13292be-827d-4098-be67-6a05230f4c94.png">
<img width="1338" alt="스크린샷 2022-03-29 오전 10 57 59" src="https://user-images.githubusercontent.com/54835264/160518779-b748c2c3-3369-4d24-96fc-84de25ba10d3.png">

## 동기식 호출 / 서킷 브레이킹 / 장애격리

## AutoScale (HPA)
1. 컨터이너 리소스 CPU 200m 설정
<img width="312" alt="스크린샷 2022-03-28 오후 9 43 28" src="https://user-images.githubusercontent.com/54835264/160518898-2e808e95-382e-41cb-a940-9a1fa9b5306f.png">

2. HPA(Horizonal Pod Autoscale)를 cpu임계치=20%, 최대Pod수=3, 최저Pod수=1 로 설정
```
kubectl autoscale deployment team4-store --cpu-percent=20 --min=1 --max=3
```

3. siege 컨테이너 배포 후 siege 컨테이너 내부에서 동시사용자 1명, 20초간 부하 테스트를 실행
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
siege -c1 -t10S -v http://team4-store:8080/profile/products
```

<img width="526" alt="스크린샷 2022-03-28 오후 9 41 25" src="https://user-images.githubusercontent.com/54835264/160518943-1d828460-eabc-443e-9dd8-4ac8eb4bc97b.png">

4. CPU 리소스가 20%를 넘어서자 HPA 이벤트가 발생하고 Replicas=3 으로 AutoScale 실행 
<img width="631" alt="스크린샷 2022-03-28 오후 9 33 54" src="https://user-images.githubusercontent.com/54835264/160518979-07cd3bc4-7453-4d68-8e96-914daf8ad46b.png">

5. 쿨타임 이후 다시 3->1 로 Replica Set 변경
<img width="631" alt="스크린샷 2022-03-28 오후 9 40 24" src="https://user-images.githubusercontent.com/54835264/160518959-acbdc790-ac32-45d3-9846-61fe35efffd5.png">


## Self-healing (Liveness Probe)
1. Liveness Probe 설정 후 재배포 실행
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

2. Spring-Boot 설정문제로 재배포 된 컨테이너 내부 오류 발생
<img width="1157" alt="스크린샷 2022-03-28 오후 10 35 15" src="https://user-images.githubusercontent.com/54835264/160519238-625d5083-d332-4a09-83ce-0f392fdfb847.png">
<img width="1143" alt="스크린샷 2022-03-28 오후 10 32 21" src="https://user-images.githubusercontent.com/54835264/160519188-d2a85efc-e847-41da-81d8-30da0b4ef9a4.png">

3. Liveness Probe 실패로 컨테이너 재시작 이벤트 발생
<img width="669" alt="스크린샷 2022-03-28 오후 10 33 29" src="https://user-images.githubusercontent.com/54835264/160519225-4a0b47f6-91c1-4edb-a643-7ef9ba0e9ba5.png">

## Zero-Downtime Deploy (Readiness Probe)
1. HPA 제거
```
kubectl delete hpa team4-store
```

2. Readiness Probe 미설정 배포 시 siege 테스트 결과가 Availability 69.94% 임을 확인
<img width="379" alt="스크린샷 2022-03-28 오후 9 53 24" src="https://user-images.githubusercontent.com/54835264/160519270-2d68512b-2b95-40a7-aaf1-88347542b249.png">


3. Readiness Probe 설정 후 재배포 실행
```
readinessProbe:
  httpGet:
    path: /actuator/health
    port: 8080
  initialDelaySeconds: 10
  timeoutSeconds: 2
  periodSeconds: 5
  failureThreshold: 10
```

4. Readiness Probe 설정 이후 배포 시 siege 테스트 결과가 Availability 100% 임을 확인
<img width="981" alt="스크린샷 2022-03-28 오후 10 00 03" src="https://user-images.githubusercontent.com/54835264/160519277-39c82daa-3a59-4794-8670-eae92f55a328.png">



