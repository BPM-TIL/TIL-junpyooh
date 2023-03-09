# AOP


### 적용 배경

- 외부 api 통신 데이터를 사내 rds에 insert 하는 과정

- rds에서 deadlock 이슈가 발생함에 따라 여러방면으로 해결해보려 했으나
  계속 regression
- redis cache를 사용하여 필요할 때마다 데이터를 가져오기로 의사결정
- deadlock 해결에 대한 시도 사례는 추후에 다루기로

```kotlin
private fun generateKey(
    redisCachable: RedisCachable, 
    joinPoint: ProceedingJoinPoint
)
```