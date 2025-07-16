# 컴포넌트 설계 원칙 정리

## 1. 단일 책임 원칙 (SRP)

- 하나의 컴포넌트는 하나의 작업이나 기능만을 수행하는 것
- 코드 가독성이 높아지고, 유지보수와 테스트, 디버깅이 쉬워짐

---

## 2. 중복 배제 원칙 (DRY)

- 코드 안에서 중복을 줄이는 것이 목적
- 자바스크립트의 구조 분해 할당
  - 배열이나 객체의 속성을 해체하여 그 값을 개별 변수에 담을 수 있게 하는 표현식

```tsx
var a, b, rest;
[a, b] = [10, 20];
console.log(a); // 10
console.log(b); // 20
```

- 기능 변경을 한 곳에서 해결할 수 있기 때문에 유지보수가 단순해질 수 있음

---

## 3. 합성 활용하기

- 재사용이 가능한 작은 컴포넌트를 결합하여 큰 컴포넌트를 만든다는 것이 핵심
- 합성을 사용하여 단일 책임 원칙과 중복 배제 원칙을 온전하게 구현할 수 있음

---

## 4. 컴포넌트 설계 원칙의 결합

- 컴포넌트가 보통 5개 이상의 prop을 가진다면 분리가 필요함
  - 각 prop의 용도를 기억하기 쉽지 않을 수 있고, 잘못된 전달이나 순서가 틀릴 수 있음
- prop을 사용하는 방법에 따라 분류 가능
  - 정보가 서로 관련되어 있다면 하나의 그룹으로 묶어 새로운 컴포넌트를 만듦
- 컴포넌트를 prop으로 전달해서 내부에서 렌더링을 대체하는 방법
  - **컴포넌트를 prop으로 넘겨서 UI를 동적으로 구성하는 방식**
  - 유연하게 조합 가능

```tsx
// 1️⃣ ButtonWrapper.tsx
type ButtonWrapperProps = {
  Component: React.ElementType; // 또는 React.FC
};

export default function ButtonWrapper({ Component }: ButtonWrapperProps) {
  return (
    <div className="border p-4">
      <h2>버튼 감싸는 컴포넌트</h2>
      <Component /> {/* 👈 전달받은 컴포넌트 인스턴스 */}
    </div>
  );
}
```

```tsx
// 2️⃣ App.tsx
function RedButton() {
  return <button className="bg-red-500 text-white px-4 py-2">빨간 버튼</button>;
}
```

```tsx
function App() {
  return <ButtonWrapper Component={RedButton} />;
}
```

---
