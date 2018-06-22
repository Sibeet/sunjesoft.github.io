---
title: Goldilocks Cluster 시작하기 
sidebar: guides_sidebar
permalink: guides_gentle_guide_to_goldilocks_cluster.html
folder: guides
---

## 시작하기 

Goldilocks Cluster 에 대해서 간략하게 소개합니다. 기본적인 개념과 용어에 대하여 설명하고, 실습하는 과정을 통해서 Goldilocks Cluster 를 이해하는데 도움이 되기 위하여 작성되었습니다. 

## 개념과 용어

### Cluster System

Cluster System 은 하나 이상의 Cluster Member 로 구성되어있는 Cluster Group 으로 이루어진 시스템으로, 물리적으로 분리된 데이터베이스를 Cluster 로 묶어 커다란 하나의 데이터베이스 처럼 사용하게 하는 시스템입니다. 

![Cluster Overview](/images/guides_gentle_guide_to_goldilocks_cluster/cluster_overview.png )

위의 그림은 3개의 Cluster Group 과 각 Cluster Group 이 2개의 Cluster Member 를 가진 Cluster System 을 나타냅니다. 위에서 표현된 Application 들은 각자 다른 Member 에 접속하여 데이터를 처리할 수 있고, G3N2 에 접속한 Application 이 G1N2 에 존재하는 데이터에 접근할 수 있습니다. 이때 DBMS 에서 지원하는 트렌젝션 특성인 ACID 가 모두 완벽하게 지원됩니다. 

### Cluster Group

Goldilocks Cluster 를 구성하는 가장 큰 단위로, 하나 이상의 Cluster Member 로 구성됩니다.  테이블의 데이터의 일부 또는 전체를 배치하는 단위입니다. Cluster 시스템에서 하나의 테이블은 여러개의 Shard 로 분리할 수 있고, 하나의 그룹은 하나이상의 샤드를 가질 수 있습니다. 결과적으로 하나의 테이블은 여러개의 GROUP 에 분리되어 저장될 수 있습니다. 다음 그림은 각각의 Group 에 데이터가 나뉘어서 저장되는 방식을 설명합니다. 

![Cluster Group](/images/guides_gentle_guide_to_goldilocks_cluster/cluster_group_concept.png )

### Cluster Member 

Cluster Member 는 Cluster Group 을 구성하는 단위입니다. Cluster Group 을 구성하기 위해서는 하나이상의 Cluster Member 가 필요합니다. 동일 Cluster Group 에 존재하는 Cluster Member 들의 데이터는 논리적으로 서로 동일합니다. Cluster Member 는 Goldilocks Cluster 시스템의 가용성을 의미합니다. 시스템의 가용성을 높이기 위해서  하나의 물리적인 서버가 하나의 Cluster Member 로 사용됩니다.  다음 그림과 같이 각 Group 별로 하나의 Cluster Member 만 동작가능하여도 전체 Cluster 시스템은 정상적으로 동작합니다. 

![Cluster Fault Tolerant](/images/guides_gentle_guide_to_goldilocks_cluster/cluster_fault_tolerant.png )

{% include note.html content="Golidlocks Cluster 시스템을 표현하는 단위로 M x N 이라는 용어를 사용합니다. M 는 Group 수 , N 은 Member 수로 4x2 라면 각 2개의 Member 를 가진 4개의 Group 으로 이루어진 시스템을 의미합니다. 이러한 표기법은 Goldilocks Cluster 시스템을 간략하게 표시하는 방법입니다" %}

### Shard 

하나의 테이블의 데이터를 서로 중복되지 않게 규칙에 의해 분리한다면, 각각의 분리된 단위를 Shard 라고 합니다. 전통적인 DBMS 에서 제공하는 파티션드 테이블의 파티션의 개념과 유사합니다.   파티션은 서로 물리적으로 분리된 DB 에 걸쳐서 존재할 수 없지만 Shard 는 가능하다는 게 차이점입니다. Shard 는 Cluster Group 에 배치하는 데이터 단위로써, 하나의 매우 큰 테이블을 여러개의 Shard 로 쪼개서 Cluster Group 에 배치 할 수 있습니다. 

하나의 테이블을 여러개의 Shard 로 분리하는 방식에는 다음과 같이 3가지가 있습니다. 

| 구분  | 설명   |
|----|---|
| LIST  | Shard Key 로 정의된 컬럼의 값을 기준으로, 컬럼 값의 나열에 따라 Sharding 을 하는 규칙입니다  |
| RANGE  | Shard Key 로 정의된 컬럼의 값을 기준으로, 컬럼 값의 범위에 따라 Sharding 하는 규칙입니다.  |
| HASH  | Shard Key 의 값에 Goldilocks 가 제공하는 HASH 함수를 씌워서 해당 HASH 함수의 값으로 Sharding 하는 규칙입니다.  |

## 실제 구성을 통해 테스트 해보기 

### 준비사항

여러분은 Linux 시스템이 설치된 4대의 물리적인 하드웨어를 가지고 있고, 이 장비들은 서로 동일한 네트워크에 위치하여 서로 통신 가능하다고 가정합니다. 각 장비의 명칭과 IP는 다음과 같이 정해져있다고 합시다. Virtual Box 나 Docker 등의 가상 시스템으로 구성해도 물론 문제 없이 동작합니다. 

| 이름    |  IP          |
|---------|--------------|
| Node1   |  10.10.10.1  |
| Node2   |  10.10.10.2  |
| Node3   |  10.10.10.3  |
| Node4   |  10.10.10.4  |

이제 모든 준비가 끝났습니다. 공유디스크, Clusterware, Custom Kernel, 특별한 하드웨어장치와 같은 어떤 부가적인 것도 필요하지 않습니다. 하나의 네트워크 안에 서로 연결된 Linux 시스템들만 준비되면 Goldilocks Cluster 를 꾸밀 수 있습니다. 물론 성능적인 향상을 위해서 NIC 를 듀얼로 구성하는 등의 작업은 필요할 수 있습니다만 실제 Cluster 를 구성하는데 필수 불가결한 요소는 아닙니다. 

### 패키지 설치 

일단 NODE1 에 계정을 만들고 Goldilocks 를 설치합니다. 다음과 같은 절차로 설치합니다. 

1. NODE1 에 db 계정을 생성합니다. 
2. db 계정에 Godlilocks 패키지를 업로드하고, 압축을 해제합니다. 
3. 환경 변수에 필요한 값을 세팅합니다. 

위와 같은 절차는 매우 간략화된 설치 순서이고, 자세한 설치 순서는 GOLDILOCKS 설치 가이드 문서에 잘 나와있습니다. 해당 문서에는 DB 를 설치하기 전에 체크하여야할 환경 설정등도 기술하고 있기때문에 반드시 해당 문서를 참고하여 Goldilocks 패키지를 설치합니다. 

### Cluster 생성 

Cluster를 지원하기 위하여 다음과 같은 명령을 통해서 DB 를 생성합니다. 위에서 만든 db 계정에서 수행하여야 합니다. 

```sh
$ gcreatedb --cluster --member=G1N1 --host=10.10.10.1 --port 10101
```  

위 명령은 cluster 모드로 DB 를 구성할것이며 앞으로 이 DB 의 이름은 클러스터 내에서 G1N1 으로 불릴 것이고, IP는 무엇이다는 것을 지정하고 있습니다. IP 를 지정하는 이유는 두개 이상의 NIC 로 구성된 시스템에서 Cluster 가 사용할 NIC 를 지정하기 위해서 입니다. Cluster 가 내부적으로 사용하는 Network 과 응용프로그램과 연결되는 네트워크는 물리적으로 분리하는 것이 권고사항입니다. 본 예제는 간단히 하기 위해 서로 동일하게 진행하였습니다. 

데이터베이스 생성이 모두 완료되었다면 다음과 같이 DB 를 GLOBAL OPEN 상태로 전환합니다. 

```
$ gsql sys gliese --as sysdba <<EOF 
startup; 
ALTER SYSTEM OPEN GLOBAL DATABASE ;
quit;
EOF
```

이제 Cluster GROUP 을 생성할 차례입니다. 다음과 같이 생성합니다. 

```
$ gsql sys gliese --as sysdba <<EOF 
create cluster group g1 cluster member G1N1 host '10.10.10.1' port 10101;
quit;
EOF
```

Cluster Group 을 생성합니다. G1N1 이라는 MEMBER 를 가진 CLUSTER GROUP 를 생성합니다. 여기까지 단일 그룹과 단일 MEMBER 를 가진 1x1 클러스터 시스템을 구성하였습니다. 이제 Performance View 를 생성합니다. 

```
$ gsql sys gliese --as sysdba -i $GOLDILOCKS_HOME/admin/cluster/DictionarySchema.sql
$ gsql sys gliese --as sysdba -i $GOLDILOCKS_HOME/admin/cluster/InformationSchema.sql
$ gsql sys gliese --as sysdba -i $GOLDILOCKS_HOME/admin/cluster/PerformanceViewSchema.sql
```

### 테이블 생성

이제 Cluster 시스템에 테이블을 만들 차례입니다. 실제 확장하는 모습을 표현하기 위해서 고객의 자산을 관리하는 가상의 시스템이 있다고 가정합니다. 다음과 같은  테이블로 표현할 수 있을 것 같습니다. 

```
CREATE TABLE CUSTOMER 
(
    CUST_NO  INTEGER PRIMARY KEY , -- 고객번호 
    CUST_NAME VARCHAR(50) NOT NULL, -- 고객명 
    CUST_BALANCE INTEGER NOT NULL DEFAULT 0 -- 고객자산 
);
```

이 시스템은 고객이 엄청나게 많이 늘어날 수 있는 시스템입니다만 현재로는 적은 숫자의 사용자를 대상으로 서비스 하고 있습니다. 굳이 미래의 잠재고객 때문에 미리부터 시스템을 크게 가지고 가고 싶지 않아서 일단 1 x 1 시스템을 구축하여 하나의 DB 로 서비스를 수행하고 있다고 가정합시다. 

위와 같은 성격의 데이터를 Goldilocks Cluster 시스템에 적용하는 전략은 일단 하나의 장비에 모두 Customer 테이블을 유지하다가 고객이 점점 늘수록 새로운 장비를 추가하여 데이터를 새로 추가한 장비로 분산하여 저장하고 서비스 하는 방법입니다. 

위와 같이 하기 위해서는 일단 SHARD 테이블로 CUSTOMER 를 구성하는 것이 좋겠습니다. 왜냐하면 나중에 부하나 데이터가 늘어날때 SHARD 단위로 Group 에 옮길 수 있기 때문입니다. 

위에서 말한 내용을 고려해서 다음과 같이 CUSTOMER 테이블을 바꾸어 봅니다. 

```
CREATE TABLE CUSTOMER 
(
    CUST_NO  INTEGER PRIMARY KEY , -- 고객번호 
    CUST_NAME VARCHAR(50) NOT NULL, -- 고객명 
    CUST_BALANCE INTEGER NOT NULL DEFAULT 0 -- 고객자산 
) ShardING BY RANGE ( CUST_NO )
Shard S1 VALUES LESS THAN (1000000) AT CLUSTER GROUP G1, 
Shard S2 VALUES LESS THAN (2000000) AT CLUSTER GROUP G1, 
Shard S_DEFAULT VALUES LESS THAN (MAXVALUE) AT CLUSTER GROUP G1;
```

CUST_NO 를 기준으로 S1 샤드와 S2 샤드, 그리고 S_DEFAULT 로 나누어서 저장하도록 했습니다. 현재는 물리적인 하드웨어가 1대 뿐인 환경이라 모두 위에서 만든 G1 Cluster Group 에 존재하도록 만들었습니다. 

{% include note.html content="최초 Create Table 시점에 Shard 를 모두 기술하지 않고 ALTER TABLE 을 통하여 Default Shard 를 Split 하는 방식으로 Shard 를 추가할 수 있습니다. 단 아직까지 위에서 예로 든 RANGE 방식의 Sharding 방식은 Shard 추가를 지원하지 않습니다. 그래서 가능한 한 Shard 를 미리 많이 만들어놔야 할 필요가 있습니다." %}

### HA 설정 

우리가 만든 G1 이라는 그룹에 모든 고객 데이터가 존재합니다. 그런데 G1 그룹에는 한개의 Member 만 존재하기 떄문에 1개의 물리적인 장비의 Failure 가 발생하면 해당 시스템의 데이터는 유실됩니다. 이러한 문제를 해결하기 위해서 G1 그룹에 새로운 G1N2 맴버를 추가하여 SPF ( Single Point of Failure ) 구성을 회피해봅시다. 새로운 Member 를 추가하는 것은 온라인 중에 가능합니다. 

위에서 말한 Node 2 를 G1N2 로 사용할 것 입니다. 일단 해당 Node 에 Database  를 설치하고, Database 를 Global Open 단계까지 진행합니다. 물론 gcreatedb 를 할때 IP 는 Node2의 IP를 설정하고, 이름은 G1N2로 설정 해야 합니다. Global Open 까지 데이터 베이스를 오픈했다면, Node 1 에서 다음과 같은 구문으로 Cluster Group 에 새로운 Member 를 추가합니다 .

```
$ gsql sys gliese --as sysdba <<EOF 
alter cluster group g1 cluster member G1N2 host '10.10.10.2' port 10101;
quit;
EOF
```

G1 그룹에 G1N2 Member 를 추가하는 구문입니다. 위와 같이 MEMBER 를 추가한 후 데이터 동기화를 위해 다음과 같은 구문을 수행합니다. 

```
$ gsql sys gliese --as sysdba <<EOF 
alter database rebalance; 
quit;
EOF
```

여기가지 수행하면 G1 그룹에 새로운 Member 를 추가하는 것이 완료되었습니다. 두개의 Cluster Member 는 모두 동일한 데이터를 가지고 있습니다. 보통 장애 발생시 Cluster Group 의 데이터 유실을 막기 위해서 한 Group 은 두개 이상의 Member 로 구성할 것을 권고합니다. 

일반적인 DBMS 시스템의 HA 와 Goldilocks Cluster HA 시스템의 차이는 다음과 같습니다. 

* Replication Lag 이 존재하지 않고 그렇기 때문에 순간적인 데이터 불일치가 발생하지 않습니다. 엄밀히 말하면 불일치가 발생할 수 있으나 해당 불일치가 사용자에게 보여지지 않습니다. 
* 사용자가 어느 Member 를 접근하던지 동일한 일관성을 제공합니다. 
* 완벽한 Active-Active 환경입니다. 어느 멤버에서 변경을 수행해도 데이터 일관성에 문제가 없습니다. 
* DDL 또한 완벽하게 동기화 됩니다. 
* Failover 가 즉각적으로 이루어집니다. 

### 시스템 확장 

위에서 만든 시스템이 성공해서 고객이 계속 늘고, 부하도 늘어서 하나의 단일 서버에서 계속 업무를 수행하는 것이 불가능해졌습니다. 이런 문제를 해결하기 위하여 Goldilocks Cluster 를 확장할 것입니다. 즉 새로운 Cluster Group 인 G2 를 생성하고, CUSTOMER 테이블을 G1 과 G2 에 나누어 저장하는 방식을 사용할 것입니다.

위에서 설명한 대로, CUSTOMER 테이블은 S1,S2, DEFAULT Shard 로 이루어져 있고, S2 Shard 를 G1 에서 G2 로 옮겨서 저장할 것입니다. 

먼저 Node 3 에 Goldilocks 를 설치하고, 역시 Global Open 까지 진행합니다. 역시 gcreatedb 할때 IP는 Node 3의 IP를 이름은 G2N1 으로 지정합니다.

Node 1 이나 Node 2 중 아무 곳에서나 다음과 같은 명령을 통해서 Node3 을 Cluster 로 추가합니다. 

```
$ gsql sys gliese --as sysdba <<EOF 
create cluster group g2 cluster member G2N1 host '10.10.10.3' port 10101;
quit;
EOF
```

이제 CUSTOMER 테이블에 S2 Shard 를 G2 로 옮겨줍니다. 

```
ALTER TABLE CUSTOMER MOVE SHARD S2 TO CLUSTER GROUP G2; 
```


역시 G1N2 를 추가할때와 마찬가지로 G2N2 를 추가하여 G2 의 데이터에 대한 HA 설정을 완료합니다. 

#### 어플리케이션 변경

위와 같이 G1과 G2 로 데이터를 분리하였다면 사용자 Application 또한 어느정도 변경이 필요합니다. Goldilocks Cluster 시스템이 고객번호를 SHARD 기준으로 나누어 물리적으로 나누었기 때문에 Application 도 자신이 처리할 고객번호를 판단하여 해당 데이터가 존재하는 쪽으로 접속해야합니다. 물론 그렇게 하지 않더라도 Goldilocks Cluster 시스템은 오류를 리턴하지 않습니다. 그러나 데이터를 물리적으로 분리해서 기대했던 성능 확장을 위해선 Application 고려도 필요합니다. 


## 마무리 

위와같이 1X1 으로 최초 작게 시스템을 구축해서 2X2 Cluster 환경까지 확장하는 부분에 대해서 기술했습니다. 보시다시피 Goldilocks Cluster 시스템은 최초 사이징에 따른 부담없이 현재 데이터에 맞는 규모의 시스템을 구성하고, 향후 시스템을 자유롭게 확장할 수 있도록 개발되었습니다. 이는 과거 전통적인 DBMS 시스템에서 수평 확장을 지원하지 않아서 현재 데이터가 적더라도 10년뒤에 쓸것으로 예측되는 데이터 사이즈에 맞추어 구성하여야 했던 것과는 크게 대비되는 Goldilocks Cluster 의 특성입니다. 

끝!
