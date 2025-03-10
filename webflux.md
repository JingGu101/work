## Reactive Programing
- Non-blocking, 
  - WebMVC
    - 一个请求处理会占用一个线程
      - 当某些操作(比如数据库查询, 外部API调用, 文件读取等)需要等待时, 该线程会停下来,直至草错完成后才继续执行
      - 在高并发的情况下, 大量线程被阻塞(blocked)等待资源, 导致线程上下文切换频繁, 增加资源消耗,降低吞吐量.
      -  the thread is waiting for recourses, eg. waiting for query data from db
   - reactive programing
     - 请求的处理不会阻塞线程, 线程可以在等待资源返回结果时同时处理其他任务
     - 在i/o 操作 (比如数据库查询或者HTTP 调用)没有完成时, 线程不会被挂起, 而是立刻返回控制权, 允许其他线程使用同一线程
     - 一旦操作完成, 框架会通过事件驱动机制通知应用程序, 继续处理该请求的后续逻辑
    - while Reactive programing, the thread will work on other request, when data are finished, the framework is able to 
- iterator is pull-based, reactive streams are push-based
  - in iterator/iterable, user decides when to access `next()`
  - in reactive streams, it is publisher-subscriber. Publisher notifies Subscriber of newly available values as they come.
  - onNext x 0...N [onError | onComplete]
- endpoint
```
@Slf4j
@Controller("raw")
@Tag(name = "Copernicus raw API")
public class CopernicusRawController{
    private final ApiCommonController apiCommonController;

    @Post(consumes = MediaType.APPLICATION_JSON, produces = MediaType.APPLICATION_JSON)
    @Status(value = HttpStatus.OK)
    public Mono<MuttableHttpResponse<APIResponseWrapper>> rawQuery(
        @Valid @Body ApiRequestDTO apiEventRequest,
        HttpHeaders headers,
        @QueryValue(defaultValue = "") String type
    ) {
        validateHeaders(headers);
        validateBody(apiEventRequest);

        ...
        return Mono.fromCallable(()->{...})
            .subscribeOn(Schedulers.boundedElastic())
            .onErrorMap(SomeException.class, e -> {...})
            .doOnError(SomeException.class, e -> {...})
            .timeout(....)
            .map(... -> ...)
            .single()
            .cache()


    }
}
```

- debug
  - `log("DEBUG")`
    - provides detailed logs for each step in the Mono or Flux pipeline. This includes subscription, request, data emission, and cancellation.
  - `doOnNext()` `doOnError`, `doFinally`
```
1. log("debug")
2. doOnNext(), 
```