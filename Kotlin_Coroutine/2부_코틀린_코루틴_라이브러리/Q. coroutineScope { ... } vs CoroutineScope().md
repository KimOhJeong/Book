# CoroutineScope vs coroutineScope

---

## ✅ 개념 비교 요약

| 항목 | `coroutineScope` | `CoroutineScope` |
|------|------------------|------------------|
| 타입 | **함수 (suspend)** | **인터페이스 / 클래스** |
| 사용 맥락 | **코루틴 빌더 (중첩 scope)** | **스코프를 명시적으로 만들 때** |
| 반환 | 마지막 구문의 결과 | 없음 (`Job` 등 관리용) |
| 취소 전파 | **내부 coroutine이 모두 끝날 때까지 suspend** | 외부에서 수명 관리 |
| 예외 전파 | 내부 예외 → 외부로 전파됨 | 예외 전파는 스코프의 Job 설정에 따라 다름 |
| 주요 용도 | 구조화된 동시성 / 내부 작업 그룹 | 명시적 수명 관리 (UI, 백그라운드 등) |

---

## 🔧 1. `coroutineScope` (suspend builder function)

- **suspend 함수** 내에서 **구조화된 동시성**을 보장
- 내부 코루틴들이 모두 완료될 때까지 **현재 코루틴을 중단** (`suspend`)
- 내부에서 예외가 발생하면 외부로 **전파됨**

```kotlin
suspend fun someWork() = coroutineScope {
    launch { delay(1000); println("A") }
    launch { delay(500); println("B") }
    // 여기는 두 launch가 모두 완료되어야 실행됨
    println("All done")
}
```

### 📌 특징:
부모 코루틴과 자식 코루틴이 강하게 연결됨
내부 코루틴이 실패하면 전체 scope 취소됨

## 🧰 2. CoroutineScope (인터페이스/클래스)
Job과 Dispatcher를 합쳐서 코루틴 실행 컨텍스트를 지정

코루틴의 수명 범위를 명시할 때 사용

```kotlin
val scope = CoroutineScope(Dispatchers.Default + Job())

scope.launch {
    delay(1000)
    println("Hello from scoped coroutine")
}
## 📌 특징:

명시적으로 수명 관리가 필요할 때 유용 (예: Android ViewModel, 서버 핸들러 등)

scope.cancel()로 전체 코루틴 트리 중단 가능
```

🆚 함께 쓰일 때의 예
```kotlin
class MyClass {
    private val scope = CoroutineScope(Dispatchers.IO + SupervisorJob())

    fun doWork() {
        scope.launch {
            coroutineScope {
                launch { delay(500); println("child1") }
                launch { delay(1000); println("child2") }
            }
            println("All internal coroutineScope work done")
        }
    }

    fun clear() {
        scope.cancel()
    }
}
```

## 요약
| 항목       | `coroutineScope {}` | `CoroutineScope(...)` |
| -------- | ------------------- | --------------------- |
| 역할       | 코루틴 그룹 빌더           | 수명 관리 컨테이너            |
| 종류       | suspend 함수          | 인터페이스/객체              |
| 예외 전파    | 내부 예외 → 외부 전파       | Job 종류에 따라 다름         |
| 주로 사용 위치 | 함수 내 중첩 작업          | 클래스, 객체 외부 수명 관리      |
