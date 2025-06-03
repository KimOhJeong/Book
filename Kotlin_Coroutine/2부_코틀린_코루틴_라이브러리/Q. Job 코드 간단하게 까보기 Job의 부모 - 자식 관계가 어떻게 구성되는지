
# 📘 Kotlin Coroutine의 `Job` 정리

## 1️⃣ `Job` 클래스 계층 구조

```kotlin
interface Job : CoroutineContext.Element
interface CompletableJob : Job

// 실제 구현체
open class JobSupport(...) : CompletableJob, CoroutineScope
class JobImpl(...) : JobSupport
class SupervisorJobImpl(...) : JobSupport
```

| 클래스/인터페이스 | 설명 |
|------------------|------|
| `Job` | 코루틴 생명주기 관리 인터페이스 (취소, 완료 등) |
| `CompletableJob` | `Job`에 수동 `complete()` 기능 추가 |
| `JobSupport` | 상태 관리, 부모-자식 관계, 핸들러 등록 등 핵심 구현 |
| `JobImpl` | 일반 `Job` 구현체 |
| `SupervisorJobImpl` | 자식 실패를 무시하는 Supervisor 역할 |

---

## 2️⃣ 부모-자식 Job 구조화

### ✅ 생성 시점

- `CoroutineScope.launch` 또는 `async` 호출 시 현재 `CoroutineContext`의 `Job`이 부모가 됨
- 자식 Job은 `parent.attachChild(child)`로 연결됨
- 연결은 내부적으로 `ChildHandleNode` 형태로 부모의 `NodeList`에 추가됨

```kotlin
val parentJob = Job()
val scope = CoroutineScope(parentJob)

val child1 = scope.launch { /* ... */ }
val child2 = scope.launch { /* ... */ }

println("Children: ${parentJob.children.toList()}")
```

### 🛑 해제 시점

- 자식이 **완료되거나 취소**되면 `parentHandle.dispose()` 호출
- `parent.children`은 실행 중인 자식만 포함
- 완료된 Job의 경우, `parent` 참조는 제거됨 → `null` 반환됨

---

## 3️⃣ 전파 구조와 효과

| 전파 방향 | 동작 | 설명 |
|-----------|------|------|
| 부모 → 자식 | ✅ 자동 취소 | 부모가 취소되면 모든 자식도 취소됨 |
| 자식 → 부모 | ✅ 예외 전파 | 자식이 실패하면 부모도 취소됨 (`Job`) |
| 자식 → 부모 | ❌ 무시됨 | `SupervisorJob`이면 자식 실패 무시됨 |

```kotlin
ParentJob
 ├── child1: Job
 └── child2: Job

parent.cancel() → child1.cancel(), child2.cancel()
child1 throws → parent.cancel() (Job인 경우)
```

---

## 4️⃣ `NodeList`를 통한 자식 관리 구조

- `JobSupport`의 상태가 `Incomplete`일 때만 `NodeList`를 사용
- 자식은 `ChildHandleNode(child)` 형태로 추가됨
- `NodeList`는 일반 핸들러도 함께 포함

```
JobSupport(state=Incomplete)
 └── NodeList
      ├── ChildHandleNode(child1)
      ├── ChildHandleNode(child2)
      └── CompletionHandlerNode(...)
```

- 논리적으로는 트리(Tree) 구조
- 자료구조적으로는 **LinkedList + 참조 기반 트리**

---

## 5️⃣ 핵심 함수: `attachChild`, `childCancelled`

### 🔧 `attachChild(child: ChildJob): ChildHandle`

- 부모 Job에 자식을 연결함
- 자식 완료 시 `dispose()` 호출해야 메모리 누수 방지
- 내부 API (`@InternalCoroutinesApi`) → 외부에서 직접 호출 ❌

### 🚨 `childCancelled(cause: Throwable): Boolean`

- 일반 Job: 자식 예외 → `cancelImpl()` 호출하여 부모도 취소
- SupervisorJob: **오버라이드**하여 무조건 `false` 반환 → 자식 실패 무시

```kotlin
// 일반 Job
override fun childCancelled(cause: Throwable): Boolean {
    cancelImpl(cause)
    return true
}

// SupervisorJob
override fun childCancelled(cause: Throwable): Boolean {
    return false // 자식 실패 무시
}
```

---

## 6️⃣ `children: Sequence<Job>`

- 현재 Job의 모든 **실행 중 자식 Job**을 반환
- 완료된 자식은 포함되지 않음

```kotlin
val parentJob = Job()
val scope = CoroutineScope(parentJob)

val child1 = scope.launch { ... }
val child2 = scope.launch { ... }

println(parentJob.children.toList()) // child1, child2 (완료 전)
```

---

## ✅ 요약 표

| 항목 | 설명 |
|------|------|
| `Job` | 코루틴 생명주기 관리 인터페이스 |
| `JobImpl` | 자식 예외 → 부모로 전파 |
| `SupervisorJobImpl` | 자식 예외를 무시 |
| `attachChild()` | 자식을 부모에 연결 (내부 API) |
| `dispose()` | 자식 완료 시 부모와의 연결 제거 |
| `children` | 부모가 추적 중인 자식 목록 |
| 전파 구조 | 부모 → 자식: 취소 전파<br>자식 → 부모: 예외 전파 (`Job`만) |


## 참고
https://github.com/Kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/common/src/Job.kt

https://dongx2.tistory.com/143

https://github.com/semin0151/Books/blob/main/Kotlin_Coroutine%3ADeep_Dive/Part_2.md
