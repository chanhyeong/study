# Coroutine
- computer program components that generalize subroutines for non-preemptive multitasking, by allowing execution to be suspended and resumed [wikepedia](https://en.wikipedia.org/wiki/Coroutine)
- subroutine = function
  - 진입하면 activation record 가 stack 에 할당되어 local variable 등이 초기화됨
- non-preemptive (비선점형)
  - 어떤 프로세스가 CPU를 할당 받으면 그 프로세스가 종료되거나 입출력 요구가 발생하여 자발적으로 중지될 때까지 계속 실행되도록 보장합니다. 순서대로 처리되는 공정성이 있고 다음에 처리해야 할 프로세스와 관계없이 응답 시간을 예상할 수 있으며, 선점 방식보다 스케줄러 호출 빈도 낮고 문맥 교환에 의한 오버헤드가 적습니다. [tistory](https://jwprogramming.tistory.com/14)
- multiple subroutines that work in collaboration with each other to exchange execution
  - 서로 협력해서 실행을 주고받으면서 작동하는 여러 서브루틴
- **generator**
  - function A 가 실행되다가 generator function B 를 호출하면, A 가 실행되던 thread 에서 B 가 실행됨
  - B 가 yield 를 호출하면 A 에 다시 넘김
  - A 는 coroutine 을 호출한 바로 다음에서 실행하다가 다시 B 를 호출
  - B 는 이전에 yield 했던 시점부터 다시 실행 (local variable 등의 새로운 초기화 없이)

## common coroutines
- generator, async/await, ...
- 1.3 버전에서는 기본으로 들어갔으나, 하위 버전에서는 의존성 추가 필요

### various coroutines
- coroutine builder: 원하는 동작을 lambda 로 넘겨서 coroutine 을 만들어 실행하는 방식

#### launch
- coroutine 을 `Job` 반환, 만들어진 코루틴은 디폴트로 즉시 실행
- `cancel()` 호출하여 중단 가능
- 아래 yieldExample 로 볼 수 있는 결과
  - `launch` 는 즉시 return
  - `runBlocking` 은 내부 coroutine 이 전부 끝난 후 return
  - `delay()` 사용 시 설정한 시간이 끝날 때까지 다른 coroutine 에 양보
    - 두 번째 delay() 로 인해 delay() 상태로 유지되어 3456 이 아닌 3546 으로 출력됨

```kotlin
fun runBlockingExample() {
  runBlocking { // coroutine 의 실행이 끝날 때까지 thread 를 block 하는 함수
    launch {
      log("launch started")
    }
  }
}

fun main() {
  log("main() started.")
  runBlockingExample()
  log("runBlockingExample() executed")
  log("main() terminated")
}

fun yieldExample() {
  runBlocking {
    launch {
      log("1")
      yield()
      log("3")
      yield()
      log("5")
      yield()
    }
  }
  log("after first launch")
  runBlocking {
    launch {
      log("2")
      delay(1000L)
      log("4")
      delay(1000L)
      log("6")
      delay(1000L)
    }
  }
  log("after second launch")
}

/**
 * after first launch
 * after second launch
 * 1
 * 2
 * 3
 * 5
 * 4
 * 6
 */
```

#### async
- launch 와 동작이 같음
- launch - Job, async - **Deffered**

```kotlin
fun sumAll() {
  runBlocking {
    val d1 = async { delay(1000L) ; 1 }
    log("after async(d1)")
    val d2 = async { delay(2000L) ; 2 }
    log("after async(d2)")
    val d3 = async { delay(3000L) ; 3 }
    log("after async(d3)")

    log("1+2+3 = ${d1.await() + d2.await() + d3.await()}") // 3초만 걸림, thread 는 1개만 사용
    log("after await all & add")
  }
}
```

### coroutine context & dispatcher
- launch, async 는 **CoroutineScope** 의 extension function
- CoroutineScope 에는 **CoroutineContext** 필드 하나만 있음
  - coroutine 이 실행 중인 여러 job 과 dispatcher 를 저장하는 일종의 map
  - kotlin runtime 이 이걸 사용해 다음에 실행할 작업 선정, thread 배정 결정

```kotlin
launch { // parent context 를 사용 (main)
}

launch(Dispatchers.Unconfined) { // 특정 thread 에 종속되지 않음 ? main
}

launch(Dispatcher.Default) { defautl dispatcher 사용
}

launch(newSingleThreadContext("MyOwnThread")) { // 새로운 thread 사용
}
```

### coroutine buidler, suspending function

#### coroutine builder
- coroutine 을 만들어주는 함수
- launch, async, runBlocking
- produce: 정해진 채널로 데이터를 스트림으로 보내는 coroutine 생성. ReceiveChannel<> 반환
- actor: 정해진 채널로 메시지를 받아 처리하는 actor 를 coroutine 으로 생성. SenderChannel<> 을 반환

#### suspending function
- delay, yield
- withContext: 다른 context 로 coroutine 전환
- withTimeout: coroutine 이 정해진 시간 내에 실행되지 않으면 exception
- withTimeoutOrNull: null 을 반환
- awaitAll: 모든 job 의 성공을 기다림. 하나라도 실패하면 실패
- joinAll: 모든 job 이 끝날 때까지 현재 작업을 일시 중단

## suspend keyword & suspending function compile
- 일반 함수에서 `delay` 나 `yield` 를 사용하면 compile 단에서 에러
- suspending function 을 어떻게 만들 수 있나? -> `suspend` 키워드 제공

```kotlin
suspend fun yieldThreeTimes() {
  log("1")
  delay(1000L)
  yield()
  ...
}

fun suspendExample() {
  GlobalScope.launch { yieldThreeTimes() } // launch 내의 함수가 긴 경우 이와 같이 분리 가능
}
```
- suspend function 의 동작 필요 사항  
1) coroutine 에 진입할 때와 coroutine 에서 나갈 때 coroutine 이 실행 중이던 상태를 저장하고 복구하는 등의 작업이 가능해야 함
2) 현재 실행 중이던 위치를 저장하고, 다시 코루틴이 재개될 때 해당 위치부터 실행해야 함
3) 다음에 어떤 coroutine 을 실행할지 결정 -> 이건 Dispatcher 에 의해 수행됨

### CPS (Continuation Passing Style)
- 프로그램의 실행 중 특정 시점 이후에 진행해야 하는 내용을 별도의 함수로 뽑고
- 그 함수에게 현재 시점까지 실행한 결과를 넘겨서 처리하게 만듦
- callback style 과 유사
- 자세하게 보려면 별도 공부가 필요

## coroutine builder 만들기
- 생략