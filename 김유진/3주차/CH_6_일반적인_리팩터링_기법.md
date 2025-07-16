# ♻️ 리팩터링 정리

## ✨ 리팩터링이란?

- **리팩터링(Refactoring)**: 기능의 **외적인 변화 없이** 내부 구조를 개선하는 것
- 목적은 **가독성 향상**, **중복 제거**, **유지보수 용이성** 확보
- **작고 점진적인 개선**을 장기적으로 반복하면서 코드 품질을 높임

---

## 💡 리팩터링의 이점

1. **기능은 그대로 유지하면서 구조 개선**
2. **버그 없이 신규 기능을 더 쉽게 추가**할 수 있음
3. **요구사항 변경에 빠르고 유연하게 대응**
4. **팀 내 협업 생산성 향상** (코드가 읽기 쉬워지기 때문)

---

## ⚠️ 리팩터링할 때 주의할 점

### ❗️리팩터링 vs 리스트럭처링

- `리팩터링`: 기능 변화 없이 코드 구조 개선
- `리스트럭처링(Restructuring)`: **기능이나 시스템 전체 구조 변경**

✅ *리팩터링 전에는 반드시 테스트 코드 작성해두기!*  
→ 기능 변화가 없어야 하므로, 리팩터링 전후 결과가 동일한지 확인해야 함

---

## 🧰 주요 리팩터링 기법

### 1. 변수 이름 바꾸기 (`Rename Variable`)

- 변수의 목적이 바뀌었을 때, **더 명확한 이름**으로 변경
- ✅ 가독성과 유지보수성 향상

---

### 2. 변수 추출하기 (`Extract Variable`)

- 복잡한 계산식 → 의미 있는 변수로 분리

```ts
// Before
if (order.amount * order.price > 10000) { ... }

// After
const orderTotal = order.amount * order.price;
if (orderTotal > 10000) { ... }
```


# ♻️ 리팩터링 주요 기법 예시

## 3. 반복문을 파이프라인으로 바꾸기
`for` 반복문 대신 `map`, `filter`, `reduce` 사용하여 명확하고 선언적인 코드로 개선

```ts
// Before
const completed = [];
for (const task of tasks) {
  if (task.done) completed.push(task);
}

// After
const completed = tasks.filter(task => task.done);
```

---

## 4. 함수 추출하기 (Extract Function)
복잡한 로직을 의미 있는 이름의 함수로 분리  
✅ 재사용성 ↑ / 테스트 용이성 ↑

---

## 5. 매개변수 객체 도입하기 (Introduce Parameter Object)
매개변수가 너무 많을 때, 하나의 객체로 묶어서 전달

```ts
// Before
function createUser(name, email, age, gender) { ... }

// After
function createUser({ name, email, age, gender }) { ... }
```

---

## 6. 조건문 분해하기 (Decompose Conditional)
복잡한 조건 로직을 함수로 추출하여 의도 명확하게 표현

```ts
// Before
if (user.isActive && user.points > 100) { ... }

// After
function isPremiumUser(user) {
  return user.isActive && user.points > 100;
}
if (isPremiumUser(user)) { ... }
```

---

## 7. 함수 이동하기 (Move Function)
함수의 역할이 현재 위치와 맞지 않을 경우, 관련 있는 모듈이나 클래스로 이동

```ts
// Before: 유틸함수인데 컴포넌트 내부에 정의되어 있는 경우
// After: 유틸 모듈로 옮겨서 재사용 및 관리 용이
```

---

## ✅ 리팩터링 진행 순서 예시

1. 테스트 코드 먼저 작성  
2. 작고 단순한 변경부터 시작 (이름 변경, 추출 등)  
3. 테스트 돌려서 기존 기능이 깨지지 않았는지 확인  
4. 반복하며 점진적으로 구조 개선
