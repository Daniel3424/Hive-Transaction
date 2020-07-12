# 락(Lock)
트랜잭션과 락은 동시성을 지원하는 장치입니다. 트랜잭션은 작업의 논리적인 단위입니다. 락은 트랜잭션을 처리할 때 테이블, 파티션에 접근을 제어하는 용도로 사용합니다.

# 락의 종류: 공유 잠금, 배타적 잠금
락은 Shared(S), Exclusive(X)가 있습니다. 데이터베이스에서 락은 공유 잠금(S)과 배타적 잠금(X)이 있습니다. 공유 잠금은 읽기 잠금(Read Lock)이라고도 불립니다. 다른 트랜잭션에서 데이터를 읽으려고 할 때 다른 공유 잠금은 허용되지만, 배타적 잠금은 허용되지 않습니다.

배타적 잠금은 쓰기 잠금(Write Lock)이라고도 불립니다. 데이터를 변경(INSERT, UPDATE, DELETE)하려고 할 때 다른 트랜잭션에서 데이터를 읽거나 변경하지 못하게 배타적 잠금을 설정합니다. 배타적 잠금이 걸리면 공유 잠금, 배타적 잠금을 설정할 수 없습니다.

# 락의 획득
락은 논파티션 테이블과 파티션 테이블에서 따로 동작합니다. 논파티션 테이블은 직관적으로 동작합니다. 테이블을 읽을 때는 S잠금을 획득하고, 다른 작업에서는 X잠금을 획득합니다.

파티션 테이블은 읽을 때 테이블에 S잠금, 파티션에 S잠금을 획득합니다. 다른 작업에서는 테이블에 S잠금, 파티션에 X잠금을 획득합니다.

락을 획득하는 방법은 다음과 같습니다.

Hive Command |	Locks Acquired
--|--
select .. T1 partition P1	|S on T1, T1.P1
insert into T2(partition P2) select .. T1 partition P1	|S on T2, T1, T1.P1 and X on T2.P2
insert into T2(partition P.Q) select .. T1 partition P1|	S on T2, T2.P, T1, T1.P1 and X on T2.P.Q
alter table T1 rename T2	|X on T1
alter table T1 add cols	|X on T1
alter table T1 replace cols	|X on T1
alter table T1 change cols|	X on T1
alter table T1 concatenate	|X on T1
alter table T1 add partition P1|	S on T1, X on T1.P1
alter table T1 drop partition P1	|S on T1, X on T1.P1
alter table T1 touch partition P1|	S on T1, X on T1.P1
alter table T1 set serdeproperties	|S on T1
alter table T1 set serializer	|S on T1
alter table T1 set file format	|S on T1
alter table T1 set tblproperties	|X on T1
alter table T1 partition P1 concatenate|	X on T1.P1
drop table T1	|X on T1

# 락 확인
락을 확인하는 명령어입니다.
```
SHOW LOCKS <TABLE_NAME>;
SHOW LOCKS <TABLE_NAME> EXTENDED;
SHOW LOCKS <TABLE_NAME> PARTITION (<PARTITION_DESC>);
SHOW LOCKS <TABLE_NAME> PARTITION (<PARTITION_DESC>) EXTENDED;
```
하이브에서 락은 아래와 같이 확인합니다.
```
hive (default)>  show locks;
OK
Lock ID Database    Table   Partition   State   Blocked By  Type    Transaction ID  Last Heartbeat  Acquired At UseHostname Agent Info
406.1   default table_name  NULL    ACQUIRED                    SHARED_READ 96  0   1584422197000   hadoop  home    hadoop_20200317051637_e6f8965b-eb5d-4281-b60a-8dc7f499c7d5
Time taken: 0.014 seconds, Fetched: 2 row(s)
```
