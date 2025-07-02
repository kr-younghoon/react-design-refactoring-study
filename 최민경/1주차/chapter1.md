# chapter 1

## 1.2

### 주의해야할 서버 상태

- 비동기 특성
  - 데이터를 가져오는 것은 일반적으로 비동기 작업
- 오류 처리
  - 실패한 요청이거나 오류를 응답하거나
- 로딩 상태
  - 요청한 데이터가 도착하기까지 기다리는 동안 로딩중 상태를 다뤄야함
- 일관성
  - fe상태를 be와 동기화 하는것 실시간 어플리케이션 혹은 여러 사용자가 동일한 데이터를 변경하는 애플리케이션의 경우 다루기 어려우며 중요함
- 캐싱
  - 일부 서버 상태를 로컬에 저장하면 성능을 향상시킬 수 있지만 데이터 불일치와 무효화 같은 문제가 생길 수 있음
- 업데이트 및 낙관적 UI
  - 사용자가 상태를 변경했을때 응답을 받기전 미리 업데이트 하여 ux경험 향상 / 실패했을 경우 되돌리는 방법 필요

동적인 목록을 컴포넌트로 렌더링할 경우 index를 키로 사용하면 좋지 않음

⇒ 리액트는 각 리스트 아이템의 `Key` 를 기준으로 어떤 항목이 변경되었는지 판단해 최소한의 DOM 업데이트를 수행함. 근데 Index를 키로 사용할 경우 리스트가 변경되어 순서가 바뀌거나 삽입/삭제가 일어나도 키 값은 그대로이기 때문에 실제 변경된 항목을 정확히 인식하지 못하기 때문

## 1.3

### 다른 컴포넌트에서 발생한 오류

응답에 존재하지 않는 속성에 접근

리액트의 에러 바운더리기능은 자식 컴포넌트의 자바스크립트 오류를 잡아서 로그를 기록하고 애플리케이션 전체가 충돌하지 않도록 fallback UI를 표시

에러 바운더리는 렌더링 과정, 생명주기 메서드, 하위 트리 컴포넌트 생성자에서 발생하는 에러를 포착

### @ 에러바운더리란?

```jsx
import { ErrorBoundary } from 'react-error-boundary';

export function AddCommentContainer() {
  return (
    <ErrorBoundary fallback={<p>⚠️Something went wrong</p>}>
      <AddCommentButton />
    </ErrorBoundary>
  );
}
```

startTransition에 전달된 함수에서 오류가 발생하면
error boundary를 사용하여 사용자에게 오류를 표시할 수 있다.

error boundary를 사용하려면 useTransition을 호출하는 컴포넌트를
error boundary로 감싸면 됩니다. startTransition에 전달된 함수에서
오류가 발생하면 error boundary의 Fallback이 표시된다.

### @ transition / useTransition / startTransition 은?

✅ transition(트랜지션)이란?
React에서 **transition(트랜지션)**이란, 긴급하지 않은 상태 업데이트를 의미

React는 상태가 바뀌면 리렌더링을 수행하는데,
때로는 상태 업데이트가 긴급하지 않아서 즉각적인 UI 업데이트가 필요하지 않은 경우가 있음

이때 React에게 "이 상태 변경은 긴급하지 않으니 조금 천천히 처리해도 돼"라고 알려주는 것이 transition

✅ useTransition을 사용하면 좋은 상황:

- 대규모 목록 필터링: 입력창에 글자를 입력할 때마다 수천 개의 아이템을 필터링하는 경우
- 무거운 컴포넌트 재렌더링: 비용이 큰 UI 작업을 지연시키는 경우
- 상태 업데이트에 따라 데이터 fetching: 입력 시 데이터를 가져오는 요청을 지연하는 경우

즉, 사용자의 즉각적인 상호작용(UI)을 방해하지 않으면서 성능을 최적화하기 위해 사용합니다.

| 반환 값               | 의미                        | 설명                                                          |
| --------------------- | --------------------------- | ------------------------------------------------------------- |
| **`isPending`**       | 트랜지션이 진행 중인지 여부 | 트랜지션 상태 업데이트가 진행 중이면 `true`, 완료되면 `false` |
| **`startTransition`** | 트랜지션을 시작하는 함수    | 긴급하지 않은 상태 업데이트를 감싸는 함수                     |

```jsx
const [isPending, startTransition] = useTransition();
```

startTransition은 인자로 상태를 업데이트하는 함수를 전달받아 호출
이렇게 호출하면 React는 해당 업데이트를 긴급하지 않은(트랜지션) 상태로 처리

**참고**

https://react.dev/reference/react/useTransition#displaying-an-error-to-users-with-error-boundary

+) chatGPT

## 1.4 안티패턴 종류

### 1) prop drilling

prop이 부모 컴포넌트에서 자식 컴포넌트로 전달될 때 여러개의 중간 컴포넌트를 거쳐서 전달하게 되는 prop drilling 현상

<details>
<summary>책에 있는 예제 코드</summary>
	
<div markdown="1">       
	
```jsx
type Item = { id: string; name: string };

function SearchableList({
items,
onItemClick,
}: {
items: Item[]; // Item 배열
onItemClick: (id: string) => void; // Item 요소를 클릭했을때 이벤트 핸들러 함수
}) {
return (
<div className="searchable-list">
{/_ Potentially some search functionality here _/}
<List items={items} onItemClick={onItemClick} />
{/_ List 컴포넌트에 Item 배열과 이벤트 핸들러 함수를 전달 _/}
</div>
);
}

function List({
items,
onItemClick,
}: {
items: Item[];
onItemClick: (id: string) => void;
}) {
return (
<ul className="list">
{items.map((item) => (
<ListItem key={item.id} data={item} onItemClick={onItemClick} />
))}
</ul>
);
}

function ListItem({
data,
onItemClick,
}: {
data: Item;
onItemClick: (id: string) => void;
}) {
return (
<li className="list-item" onClick={() => onItemClick(data.id)}>
{data.name}
</li>
);
}

export default SearchableList;

```

</div>
</details>

⇒ List 컴포넌트는 해당 Prop을 사용하지는 않지만 ListItem에 전달해야함.

잠재적인 해결책 = Context API 사용


### 3) 뷰 영역의 복잡한 로직

리액트 같은 최신 프레임워크의 장점은 **관심사를 명확하게 분리**할 수 있음

컴포넌트는 비즈니스 로직을 신경쓰지 않고 프레젠테이션에 집중해야함!

but, 뷰 컴포넌트 안에서 비즈니스 로직을 다루는 함정에 종종 빠짐

```tsx
const filterExpensiveItems = (items: Item[]) => {
	return items.filter((item) => item.price > 100);
}
````

해당 함수는 비즈니스 로직의 일부이며, 뷰 컴포넌트 안에 위치함

문제점

- 유사한 필터가 필요할 경우 로직이 중복됨
- 렌더링 뿐만 아니라 비즈니스 로직도 테스트해야하므로 단위 테스트가 복잡해짐
- 더 많은 로직이 추가 되기 떄문에 유지보수가 어려워짐

**관심사 분리 원칙**을 지키는 것이 좋다.
⇒ 각각의 모듈 또는 함수가 애플리케이션의 하나의 기능에 대한 책임만 가져야 한다는 것

### 5) 중복된 코드

중복 배제 원칙 (Don’t Repeat Yourself, DRY)

참고 블로그
https://velog.io/@eunbinn/dry-the-common-source-of-bad-abstractions

### 6) 너무 많은 기능을 가진 컴포넌트

많은 prop 목록을 통해 다양한 기능을 담당할 경우, **단일 책임 원칙(Single-Responsibility Priciple, SRP)에 위배됨.**

핵심 기능을 분석하고 부가적인 지원 로직들을 더 작고 집중된 컴포넌트나 훅으로 분리하기
