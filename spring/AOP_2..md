# AOP


### 적용 배경

- 외부 api 통신 데이터를 사내 rds에 insert 하는 과정

- rds에서 deadlock 이슈가 발생함에 따라 여러방면으로 해결해보려 했으나
  계속 regression
- redis cache를 사용하여 필요할 때마다 데이터를 가져오기로 의사결정
- deadlock 해결에 대한 시도 사례는 추후에 다루기로

---
```kotlin
implementation("org.springframework.boot:spring-boot-starter-aop")
implementation("org.springframework.boot:spring-boot-starter-data-redis")
```
먼저 aop 와, redis 의존성을 추가해준다.


```kotlin
@Target(AnnotationTarget.FUNCTION)
@Retention(AnnotationRetention.RUNTIME)
annotation class RedisCacheable(
    val keys: Array<String> = [],
    val timeout: String = "Infinity"
)
```
@RedisCacheable 이라는 어노테이션을 정의하기로 한다.

timeout 을 두어 캐시의 수명 설정

---

```kotlin
@Aspect
@Component
class RedisCacheAspect(
    private val redisTemplate: RedisTemplate<String, Any>
) {
    @Around("@annotation(RedisCacheable)")
    fun cacheableProcess(joinPoint: ProceedingJoinPoint): Any?
    
    private fun generateKey(
      redisCacheable: RedisCachable,
      joinPoint: ProceedingJoinPoint
    ): String  
}
```
- class RedisCacheAspect
  - @Aspect : 이 클래스가 Aspect 를 나타내는 클래스라는 것을 명시
  - @Component: 스프링 빈 등록
- cacheableProcess()
  - @Around : 타겟 메서드를 감싸서 특정 Advice 를 실행


- generateKey()
  - return "$target.$method($key)"
  - target : target 의 simpleName 으로 class 명을 가져온다.
  - method : signature 의 method 명을 가져온다.
  - key : parameterNames 를 joinToString()으로 파싱
---

