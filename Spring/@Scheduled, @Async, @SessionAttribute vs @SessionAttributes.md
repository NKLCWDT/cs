# @Scheduled, @Async, @SessionAttribute vs @SessionAttributes

## @Scheduled

스케줄러를 사용하기 위해서는 아래 조건을 다 만족해야 한다.

- Application 클래스에서 `@EnableScheduling` 활성화
- 스케줄러를 제공할 클래스를 빈으로 등록
- 스케줄러에 @Scheduled 사용

```kotlin
@Slf4j
@Component
class TimeScheduler {

    private val log = logger()
    private var formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss")

    /** 5초 마다 현재 시간을 로그로 찍는 스케줄러 */
    @Scheduled(fixedRate = 5000)
    fun reportCurrentTime() {
        log.info("The time is now {}", LocalDateTime.now().format(formatter))
    }
}
```

@Scheduled 는 다양한 옵션들을 지원한다. 특히 특정 날짜, 시간에 동작하게끔 할 수도 있다.

```
@Scheduled("0 * * * * MON-FRI")
```

- __@Scheduled cron options__
  - second
  - minute
  - hour
  - day of month
  - month
  - day of week

## @Async

