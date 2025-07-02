# CHAPTER 2 리액트 필수 개념 이해

~~2.1 리액트 정적 컴포넌트~~  
~~2.2 prop이 있는 컴포넌트 만들기~~  
[2.3 UI를 여러 컴포넌트로 만들기](#23-ui를-여러-컴포넌트로-만들기)  
~~2.4 리액트 내부 상태관리~~  
[2.5 렌더링 과정 이해하기](#25-렌더링-과정-이해하기)  
[2.6 많이 사용되는 리액트 훅](#26-많이-사용되는-리액트-훅)

## 2.3 UI를 여러 컴포넌트로 만들기

### 컴포넌트를 나누는 기준

1. 재사용이 가능한 컴포넌트

- 보편적인 속성 (다른 컴포넌트가 가져가서 사용할 수 있도록 보편적인 속성을 갖고 있는가)
- HTML 요소 측면의 재사용성
- 중복을 고려한 재사용성

2. 복잡한 컴포넌트

- 컴포넌트가 여러 책임을 갖는 경우
- 컴포넌트에 비즈니스 로직이 있는 경우
- 서로 독립적인 상태가 한곳에 있는 경우 (하나의 상태값 변경이 다른 상태까지 렌더링 시키는 경우)

참고자료 - [구성요소를 분리하는 기준과 방법](https://medium.com/@junep/%ED%94%84%EB%A1%A0%ED%8A%B8%EC%97%94%EB%93%9C-%EC%95%84%ED%82%A4%ED%85%8D%EC%B2%98-%EC%BB%B4%ED%8F%AC%EB%84%8C%ED%8A%B8%EB%A5%BC-%EB%B6%84%EB%A6%AC%ED%95%98%EB%8A%94-%EA%B8%B0%EC%A4%80%EA%B3%BC-%EB%B0%A9%EB%B2%95-e7cf16bb157a)

## 2.5 렌더링 과정 이해하기

### 리엑트의 렌더링 과정 - Virtual DOM

- 초기 렌더링: 함수 컴포넌트가 처음 렌더링 된 이후, 컴포넌트 UI를 표현하는 가상 객체(UI 요소의 구조와 내용) 생성.
- 상태와 prop의 변화: 컴포넌트의 상태와 prop이 변경되면, 리액트는 함수 컴포넌트를 다시 호출. 이 과정에서 변경 전 함수의 실행 결과와 현재의 실행 결과를 비교(diffing)하는 알고리즘 수행
- 재조정(reconciliation): 비교 과정을 통해 어떤 UI 요소의 업데이트가 필요한지 결정. 필요한 부분만 업데이트.
- 리렌더링: 가상UI 요소를 업데이트하는 방식으로 리렌더링 수행. 이전의 가상 DOM을 새로운 가상 DOM으로 바꿈
- DOM 업데이트: 리액트가 실제 DOM에 효율적으로 가상 DOM의 변경 사항을 적용

##### :bulb: 브라우저의 렌더링 과정

참고자료 - [MDN-브라우저는 어떻게 동작하는가](https://developer.mozilla.org/ko/docs/Web/Performance/Guides/How_browsers_work)

## 2.6 많이 사용되는 리액트 훅

#### 2.6.1 useState

useState 훅은 컴포넌트 리렌더링이 발생하더라도 내부의 상태를 유지할 수 있게 한다.

##### :bulb: React의 상태 감지 방식

React는 상태가 바뀌었는지 판단할 때 `참조비교` 를 사용한다. 이는 객체나 배열의 내부 값이 변경된다고 해서 객체나 배열의 주소 자체는 변경되지 않고 유지되기 때문에 따라서 변경 감지를 위한 상태값은 `불변성(immutability)` 을 지녀야 한다.

```javascript
function Counter() {
  const [user, setUser] = useState({ name: 'Tom', age: 20 });

  const increaseAge = () => {
    user.age += 1;
    setUser(user); // 같은 참조 → 변경 감지 못함
    setUser((prev) => ({ ...prev, age: prev.age + 1 })); // 새로운 참조 → 리렌더링 진행
  };

  return (
    <div>
      <p>{user.age}</p>
      <button onClick={increaseAge}>나이 증가</button>
    </div>
  );
}
```

#### :bulb: useState는 비동기 함수일까?

```javascript
const [count, setCount] = useState(0);

const onClick = () => {
  setCount(count + 1);
  console.log(count); // 1 ?
};

return (
  <div>
    <button onClick={onClick}>click me</button>
  </div>
);
```

setState가 동기적 함수임에도 비동기 함수처럼 보이는 이유는 React의 리렌더링 원리가 비동기적으로 작동하기 때문이다.
React는 가상 돔이라는 자신만의 돔 이미지를 유지하고 있으며, 렌더링 함수를 호출하여 가상 돔을 업데이트한다.
렌더링 함수는 컴포넌트의 상태나 속성이 변경될 때마다 호출되며, 이렇게 하면 React는 가상 돔이 최신 상태로 유지되도록 한다.
그런 다음 React는 가상 돔과 실제 돔을 비교하여 실제 돔을 업데이트한다.

이때 React가 실제 돔에 반영하는 과정을 조정(reconciliation)이라고 하는데 fiber architecture가 조정 알고리즘을 구현 하는 방식이 이 비동기적 현상과 연관이 있다.
조정 알고리즘은 가상돔 트리를 render phase, commit phase 두가지로 나누어 변경된 부분을 찾고 실제 돔에 변경사항을 적용하는 작업을 나누어 진행한다.

- 렌더 단계: 가상돔 트리를 순회하면서 변경된 부분을 찾는다. 해당 fiber node의 effectTag를 UPDATE로 설정한다. 그리고 이펙트 리스트에 추가한다.
- 커밋 단계: 이펙트 리스트에 있는 fiber node의 effectTag에 따라 작업을 수행한다.
  - before mutation: 돔에 변화를 주기 전에 실행되는 생명주기 메서드들을 호출하는 단계. 예를 들어, getSnapshotBeforeUpdate 등.
  - mutation: 돔에 변화를 주는 단계. 이펙트 리스트에 저장된 작업들을 순서대로 실행. 예를 들어, 노드 삽입, 삭제, 수정 등.
  - layout: 돔에 변화를 준 후에 실행되는 생명주기 메서드들을 호출하는 단계. 예를 들어, componentDidMount, componentDidUpdate 등.

이러한 변경점을 찾고 실제 돔을 업데이트하는 과정을 동기적으로 진행한다면 렌더링 작업이 오래 걸린다면 메인 스레드가 차단되고 프레임 드롭이나 응답 지연이 발생하기 때문에 UX를 저해하는 요소가 된다.

참고자료 - [콘솔로그가 이상한건 setState가 비동기 함수여서가 아닙니다.](https://velog.io/@jay/setStateisnotasync)

#### :bulb: useState는 어떻게 최신 상태 값을 유지할 수 있을까

함수는 호출된 뒤 함수 내부에 선언된 변수는 더 이상 접근 할 수 없다. 함수 생명주기가 종료됨에 따라 함수의 컨텍스트도 제거되기 때문에 해당 컨텍스트에 선언된 변수도 제거된다.
React의 함수형 컴포넌트에서 내부에 선언된 변수는 컴포넌트의 생명주기가 종료되었을 때 같이 사라지며 리렌더링 될때 새로운 변수로 선언된다. 따라서 이전 상태를 유지할 수 없다.
이러한 문제를 해결하기 위해 상태값을 저장할 변수가 선언된 컨텍스트를 유지할 방법으로 클로저를 사용한다.

```javascript
let hooks = [];
let idx = 0;
function useState(initVal) {
  const state = hooks[idx] || initVal;
  const _idx = idx; // freeze idx using closure
  const setState = (newVal) => {
    hooks[_idx] = newVal;
  };
  idx++;
  return [state, setState];
}
```

상태 값이 여러개 존재하는 경우를 해결하기 위해 hooks 전역 배열을 활용한다. 따라서 Hooks 규칙이 존재한다.

- 최상위에서만 호출해야한다: 반복문, 조건문 혹은 중첩 함수 내에서 Hook을 호출할 수 없다. (조건에 따라 hook 실행 여부가 달라지면 컴포넌트가 렌더링 될 때마다 hook의 실행 순서가 달라짐)
- 오직 React 함수 내에서 호출해야 한다: React의 함수 컴포넌트 혹은 커스텀 훅 내부에서 Hook을 호출해야 한다. (컴포넌트의 렌더링 사이클과 관계없이 호출될 수 있음)

참고자료 - [useState 동작 원리와 클로저](https://jaehan.blog/posts/react/useState-%EB%8F%99%EC%9E%91-%EC%9B%90%EB%A6%AC%EC%99%80-%ED%81%B4%EB%A1%9C%EC%A0%80)

### 2.6.2 useEffect

사이드 이펙트란 컴포넌트 렌더링과 직접 관련되진 않지만 컴포넌트 바깥 스코프에 영향을 주는 코드로 API요청을 의미한다.
React Virtual DOM과 다른 DOM 수정, 이벤트 리스너 구독, 타이머 사용 등 외부 리소스와의 상호작용과 관련되어 있는 코드가 해당한다.
useEffect는 클래스형 컴포넌트에서 접근 가능했던 컴포넌트 생명주기에서의 사이드 이펙트 제어 기능을 함수형 컴포넌트에서도 가능하게 하는 훅이다.

#### :bulb: 컴포넌트 생명주기

크게 마운팅, 업데이트, 언마운팅으로 나눌 수 있음

- 마운팅 단계 : 컴포넌트가 생성되는 단계. 최초로 DOM에 삽입되며 애플리케이션에서 컴포넌트가 시작됨. 이 과정에서 호출되는 생명주기 메서드는 컴포넌트의 초기화와 설정을 수행함.
- 업데이트 단계: 컴포넌트의 props나 state 변경으로 인해 재렌더링이 발생할 때 일어나는 생명주기. 사용자 인터페이스가 애플리케이션의 최신 데이터의 상태를 반영하는데 필수적이며 업데이트 과정을 제어하고 사이드 이펙트 수행, 성능 최적화와 관련됨.
- 언마운팅 단계 : 컴포넌트의 생명주기가 끝나고 DOM에서 제거될 준비를 하는 단계. 설정된 타이머, 네트워크 요청, 이벤트 리스너 등이 제대로 해제되도록 보장하며 리소스 효율성을 보장하고 메모리 누수를 방지

#### :bulb: 언마운트 시 return을 적절히 사용하고 있을까? - cleanup이 필요한 경우

```javascript
// 이벤트 리스너 제거
useEffect(() => {
  const handleResize = () => console.log('윈도우 리사이즈됨');
  window.addEventListener('resize', handleResize);

  return () => {
    window.removeEventListener('resize', handleResize);
  };
}, []);

// setInterval, setTimeout 등 타이머 제거
useEffect(() => {
  const id = setInterval(() => {
    console.log('매초 실행');
  }, 1000);

  return () => {
    clearInterval(id);
  };
}, []);

// AbortController로 비동기 요청 취소
useEffect(() => {
  const controller = new AbortController();

  fetch('/api/data', { signal: controller.signal })
    .then((res) => res.json())
    .then((data) => console.log(data))
    .catch((err) => {
      if (err.name === 'AbortError') console.log('요청 취소됨');
    });

  return () => {
    controller.abort();
  };
}, []);

// WebSocket, WebRTC 등 연결종료
// Firebase, GrapQL RxJS 등 라이브 데이터 구독 해제
// requestAnimationFrame을 사용하여 javascript로 직접 에니메이션 제어 시 애니메이션 프레임 요청 제거
// IntersectionObserver, ResizeObserver 등 사용시 관찰자(Observer) 제거
```

### 2.6.3 useCallback

콜백 함수의 참조를 메모이제이션하고 최적화 할 때 사용한다. 특히 콜백함수를 자식 컴포넌트에 전달하거나, 콜백을 다른 훅의 의존성 목록으로 지정할 때 유용하다.
useCallback은 2개의 인자를 전달 받는다.

- callback: 메모이제이션 대상이 되는 함수. 인라인 함수 또는 함수참조(변수) 사용 가능.
- depedencies: 메모이제이션 하려는 콜백 함수의 의존성 배열. 의존성 배열에 명시된 값이 변하면 콜백 재생성.

useCallback을 사용하는 예시

```javascript
// 이벤트 핸들러 함수 처럼 컴포넌트가 리렌더링될 때마다 재생성되는 것을 방지
// 자식 컴포넌트의 props로 함수를 전달할 떄, 자식 컴포넌트의 리렌더링을 막기 (React.memo())
const Child = React.memo(({ onClick }) => {
  console.log('Child rendered');
  return <button onClick={onClick}>Click</button>;
});

function Parent() {
  const [count, setCount] = useState(0);

  const handleClick = useCallback(() => {
    console.log('Clicked!');
  }, []);

  return (
    <>
      <Child onClick={handleClick} />
      <button onClick={() => setCount(count + 1)}>Increase Count</button>
    </>
  );
}

// 함수 자체가 의존성 배열에 들어가는 경우
function UserFetcher({ userId }) {
  const [user, setUser] = useState(null);

  const fetchUser = useCallback(() => {
    fetch(`/api/users/${userId}`)
      .then((res) => res.json())
      .then((data) => setUser(data));
  }, [userId]); // userId에 따라 새로 만들어짐

  useEffect(() => {
    fetchUser(); // 함수가 의존성 배열에 들어감
  }, [fetchUser]);

  return <div>{user?.name}</div>;
}
```

### 2.6.4 Context API

여러 자식 컴포넌트와 공유해야 하는 전역 데이터가 있거나 중간 컴포넌트에서 필요 없이 말단 컴포넌트에 데이터 전달만 하는 경우에 사용하는 방식이다.(props drilling 문제 해결)
전역 상태값(State) 뿐만 아니라 상태값을 제어하는 함수(Action)도 같이 넘겨 하위 컴포넌트에서 상태값 제어를 가능하게 한다.

```javascript
// 예시 : 테마 (다크모드/라이트모드) 관리
// Provider 컴포넌트 - 전역으로 관리할 상태 선언부
const ThemeProvider = ({children}) => {
  const [theme, setTheme] = useState<"light" | "dark">("light");
  const toggleTheme = useCallback(() => {
    setTheme((prevTheme) => (prevTheme === "light" ? "dark" : "light"))
  }, [])

  return <ThemeContext.Provider value={{theme, toggleTheme}}>{children}</ThemeContext.Provider>
}

export default const Root = () => {
  return (
    <ThemeProvider>
      <App />
    </ThemeProvider>
  )
}

// Provider 컴포넌트의 모든 하위 컴포넌트에서 전역 상태에 접근 가능
export default const ThemeComponent = () => {
  const context = useContext(ThemeContext);
  const { theme } = context;

  return <div className={theme}>Current Theme: {theme}</div>
}

```
