# Chapter4 리액트 컴포넌트 설계하기

## 4.1 단일 책임 원칙 (SRP, Single Responsibility Principle)

하나의 컴포넌트는 하나의 작업이나 기능만을 수행하는 것

```jsx
1) 재사용이나 변경의 여지가 있는가?
2) 함수명이 표현식보다 훨씬 더 가독성이 있는가?
3) 반대로 함수표현으로 인해 전체 파이프라인을 왔다 갔다 하게 되어 이해하는데 방해가 되는가?
3) 하나의 파이프 라인에 속한 함수들이 각각의 모듈로 쪼개어져 있어 응집도가 떨어지는가?
```

• 응집도는 모듈의 독립성을 나타내는 개념으로, 모듈 내부 구성요소 간 연관 정도 (높을수록 good)

https://youtu.be/edWbHp_k_9Y?si=6wMtyWTuET2pM_wE

---

## 4.2 중복 배제 원칙(DRY, Dont repeat yourself)

https://dj-min43.medium.com/dry%EC%9B%90%EC%B9%99%EC%9D%84-%EA%BC%AD-%EC%A7%80%ED%82%AC-%ED%95%84%EC%9A%94%EB%8A%94-%EC%97%86%EC%8A%B5%EB%8B%88%EB%8B%A4-d05f137105e0

- DRY 원칙이 지켜지지 않아질때를 표현하는 방법 중 하나입니다
- “write every time”, “we enjoy typing” or “waste everyone’s time” 라고 표현합니다. 두 번 쓰면 많은 시간낭비가 일어납니다
- 99 Bottles of OOP의 저자 Sandi Metz는 DRY원칙을 지키기 위해 많이 선택하는 방법인 추상화가 적절하게 이루어지지 않으면 반복되어지는 코드보다 더 큰 비용이 든다고 생각입니다

⇒ DRY원칙과 추상화를 ‘왜’ 지키고 실행 하는지 이해 하는것이 중요

무조건적으로 원칙들을 따르기 보다는, 왜 그렇게 하는지, 그리고 지금의 상황에 어울리는지 생각하고 이해를 하는게 무척이나 중요함 → 그렇지 않으면 기술이 나를 집어삼킨다

---

## 4.3 합성 활용하기

**@ SOLID - OCP(Open Closed Principle)**

소프트웨어 엔티티는 확장을 위해 열려야 하지만 수정을 위해 닫혀 있어야 한다

개방-폐쇄 원칙은 원본 소스 코드를 변경하지 않고 확장할 수 있는 방식으로 구성 요소를 구조화

```jsx
// BAD
// if 페이지가 더 추가된다면?

const Header = () => {
  const { pathname } = useRouter();

  return (
    <header>
      <Logo />
      <Actions>
        {pathname === '/dashboard' && (
          <Link to='/events/new'>Create event</Link>
        )}
        {pathname === '/' && <Link to='/dashboard'>Go to dashboard</Link>}
      </Actions>
    </header>
  );
};
```

```jsx
// GOOD

const Header = ({ children }) => (
  <header>
    <Logo />
    <Actions>{children}</Actions>
  </header>
);

const HomePage = () => (
  <>
    <Header>
      <Link to='/dashboard'>Go to dashboard</Link>
    </Header>
    <OtherHomeStuff />
  </>
);
```

컴포넌트 자체를 수정하지 않고도 원하는 내용을 자유롭게 입력할 수 있도록 하기 위해 합성을 활용

`children`을 통해 기본적인 확장성을 제공할 수 있으며, 여러 확장 지점이 필요한 경우 별도의 props를 추가로 만들어 유연성을 확보할 수 있음

이는 OCP(개방-폐쇄 원칙)를 따르는 방식으로, 컴포넌트를 수정하지 않고도 기능을 확장할 수 있게 함

---

왜 이렇게 코드를 작성했는지 항상 고민하고,

현재 상황에 맞는 적절한 추상화인지 검토하여

깔끔하고 유지보수가 쉬운 코드 짜기 😊
