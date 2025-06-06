`Dispatchers.Main`에는 `.immediate`가 존재하는데,

`Dispatchers.Default`나 `Dispatchers.IO`에는 **`.immediate`가 없는 이유는 개념적으로 의미가 없기 때문**이에요.

---

### ✅ `Dispatchers.Main.immediate`가 존재하는 이유

UI 스레드에서 **이미 실행 중일 때**, 다시 Main Dispatcher로 `withContext(Dispatchers.Main)` 하게 되면 **한번 더 디스패치**가 발생해요. 이때:

- `Dispatchers.Main`: 무조건 **디스패치** (즉, queue에 넣고 나중에 실행)
- `Dispatchers.Main.immediate`: **현재 스레드가 Main이면 즉시 실행**, 아니면 디스패치

즉, `.immediate`는 **디스패치 비용을 줄여서 성능 최적화**를 하기 위해 제공돼요.

→ `LiveData`, `StateFlow` 같은 UI 관련 구조에서 중요하죠.

---

### ❌ 왜 `Dispatchers.Default`, `IO`에는 `.immediate`가 없을까?

1. **Main 스레드는 싱글스레드이고, "현재 스레드가 Main인지" 판단이 가능**
    
    → 그래서 immediate 실행 조건이 성립함
    
2. 반면 `Default`, `IO`는 **스레드 풀 기반**이에요.
    
    → 현재 스레드가 Default/IO 풀의 일부라고 해도, 어떤 코루틴이 어디서 실행될지 보장할 수 없음
    
    → 즉, "즉시 실행 가능성(immediate)"이라는 개념 자체가 **애매**해짐
    

---

### 📌 예시 비교

```kotlin
kotlin
복사편집
// Main.immediate
withContext(Dispatchers.Main.immediate) {
    // 이미 Main에서 실행 중이면 즉시 실행
}

```

```kotlin
kotlin
복사편집
// Default는 이렇게 동작할 수 없음
withContext(Dispatchers.Default.immediate) // ❌ 존재하지 않음

```

---

### 🔚 결론

- `immediate`는 **디스패치 여부를 최적화하기 위한 기능**이고,
- **Main 스레드처럼 '현재 실행 중인지' 확인 가능한 환경**에서만 의미 있어요.
- 따라서 **멀티스레드 풀 기반인 Default/IO에는 적용 불가**합니다.
