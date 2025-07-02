# CHAPTER 1 리액트 안티패턴 소개

~~1.1 UI 구축의 어려움에 대한 이해~~  
[1.2 상태 관리의 이해](#12-상태-관리의-이해)  
[1.3 예외 흐름 탐색하기](#13-예외-흐름-탐색하기)  
[1.4 리액트의 일반적인 안티패턴 살펴보기](#14-리액트의-일반적인-안티패턴-살펴보기)

## 1.2 상태 관리의 이해

### 프론트 개발시 주의해야 할 서버의 상태

- 비동기 특성: 원격 소스에서 데이터를 가져올 때 발생. 특히 가져온 여러개의 원격 데이터를 동기화 할 때 주의.
- 오류 처리: 원격 소스 연결 실패, 서버에서 오류를 응답하는 경우.
- 로딩 상태: 원격 소스에서 데이터가 도착할 때 까지 '로딩 중' 상태를 효과적으로 표시해야함. 로딩 표시기와 이전 데이터로 표시하기, 로딩 실패시 fallbackUI를 활용.
- 일관성: 백엔드와 프런트엔드 상태를 동기화. 실시간 앱, 다수 사용자의 데이터 접근의 경우 특히 주의.
- 캐싱: 성능 향상을 위해 특정 시간동안 로컬에 저장된 데이터를 표시하는 방법. 데이터 일관성 문제를 고려해야 하며 서버에서 변경이 발생하면 로컬 상태를 갱신해야함.
- 업데이트 및 낙관적 UI: 로딩 완료 전에 UI를 미리 갱신해 사용자 경험을 향상시키는 방법. 서버 응답 실패의 경우를 고려해야함.

#### :bulb: 반복문을 활용한 JSX 선언시 key에 index를 주면 안되는 이유?

React에서 key는 리스트에서 각 컴포넌트를 고유하게 식별하기 위한 것.
이 key를 기준으로 React는 가상 DOM 비교(=reconciliation)를 수행하며 어떤 요소가 추가/삭제/이동/갱신 되었는지를 파악함.

이때 index를 key로 썼을 경우 생기는 문제

1. 리스트 순서가 바뀌거나 요소가 삽입/삭제될 경우

```javascript
const items = ['a', 'b', 'c'];
// index를 key로 사용하는 경우
items.map((item, index) => <div key={index}>{item}</div>);
```

`'b'` 를 삭제하면 배열은 `['a', 'c']` 가 된다. 이때 React는 다음과 같이 처리한다.

- index 0 → 'a' → 동일하니까 유지
- index 1 → 'c' → 원래는 'b'였는데 지금은 'c', 하지만 key는 같아서 'b'를 'c'로 착각하고 재사용함

2. 입력값이나 상태가 꼬이는 경우
   리스트 안의 요소가 내부적으로 input, form, animation, state 등을 갖는 경우
   index 기반 key를 사용하면 컴포넌트가 바뀌어도 React는 같은 컴포넌트로 간주하고 상태를 유지시켜버릴 수 있다.
   실제로는 새로운 값이지만, React는 “같은 key니까 이전 컴포넌트를 재사용해야겠다”고 판단해 버그 발생.

-> 따라서 데이터의 변경을 추적할 수 있는 데이터의 고유 식별자 사용이 안전

## 1.3 예외 흐름 탐색하기

### 1.3.1 다른 컴포넌트에서 발생한 오류

서버로부터 데이터를 요청할 경우 항상 성공한다는 보장이 없으므로 오류를 응답할 경우를 고려해야함.
전체 화면에서 작은 부분을 차지하는 컴포넌트에서 서버에 데이터를 요청하고 오류를 응답받았을 경우 `에러 바운더리` 로 격리시키지 않으면 애플리케이션 전체가 충돌하여 멈출 수 있음.
그렇다고 모든 경계마다 에러 바운더리를 두는것은 적절치 않으니 적절한 경계를 생각해볼 것.

```javascript
// 에러 바운더리 예제
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  // 에러 발생 시 호출
  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  // 에러 정보 로깅 등
  componentDidCatch(error, errorInfo) {
    console.error('Error caught by ErrorBoundary:', error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return this.props.fallback;
    }

    return this.props.children;
  }
}

...

// 사용방법
<ErrorBoundary fallback={<p>Something went wrong</p>}>
  <Profile />
</ErrorBoundary>
```

클래스형 컴포넌트만 지원하기 때문에 함수형 컴포넌트에서는 `react-error-boundary` 라이브러리를 사용하는 방법도 있다.

에러 바운더리가 잡을 수 있는 에러

- render() 안에서 발생한 예외
- componentDidMount() 안에서 발생한 에러
- 자식 컴포넌트 안에서의 에러

에러 바운더리가 잡을 수 없는 에러

- 이벤트 핸들러 내부의 에러
- 비동기 코드 내부의 에러
- 서버사이드 렌더링 중의 에러

:bulb: 이 방식은 왜 에러 바운더리 역할을 하지 못할까?

```javascript
function ProblemComponent() {
  throw new Error('문제 발생!');
  return <div>정상 출력</div>;
}

function App() {
  try {
    return (
      <div>
        <ProblemComponent />
      </div>
    );
  } catch (e) {
    // 이 블록은 호출되지 않음
    return <div>에러 발생!</div>;
  }
}
```

JSX로 작성된 컴포넌트들은 React가 비동기로 각 컴포넌트가 독자적인 render과정을 가진다.
따라서 외부 try-catch에서는 이미 하위 컴포넌트가 렌더링 되기 전에 성공으로 처리후 종료되어 버린다.
반대로 이벤트 핸들러의 경우 동기로 처리되기 때문에 try-catch로 검출이 가능하다.

### 1.3.2 예측하지 못한 사용자 행동

사용자는 항상 예상하지 못한 방식으로 시스템을 사용한다. 이를 대비할 수 있는 UI를 설계하야 하지만, 유효성 검사와 안전 장치를 위해 추가 구현을 하다보면 UI코드는 복잡해진다.
'예외 흐름'에 대한 이해와 적절한 처리는 견고하고 유연하며 사용자 친화적인 인터페이스를 만드는데 매우 중요하다.

```javascript
// 사용자 입력에 대한 폼 컴포넌트의 고려사항
const Form = () => {
  const [value, setValue] = useState<string>('');

  // 입력이 변경될 때마다 영숫자가 아닌 입력에 대해서는 모두 제거한 뒤 상태를 업데이트
  const handleChange = (event) => {
    const inputValue = event.target.value;
    const sanitizedValue = inputValue.replace(/[^\w\s]gi, '')
    setValue(sanitizedValue)
  }

  return(
    <form>
    <label>특수문자 없이 입력하세요: <input type='text' value={value} onChange={handleChange} /></label>
    </form>
  )
}
```

## 1.4 리액트의 일반적인 안티패턴 살펴보기

### 1.4.1 Prop Drilling

prop이 부모 컴포넌트에서 자식 컴포넌트로 전달될 때 여러개의 중간 컴포넌트를 거쳐서 전달하게 되는 현상으로 복잡도를 높이고 유지보수성을 떨어뜨린다.
이는 개발자가 데이터 흐름에 대한 이해를 방해하고 디버깅 하기 어렵게 한다.
Context API 혹은 전역 상태관리 라이브러리를 활용하여 컴포넌트 간에 직접 데이터와 함수를 공유하는 방법으로 이를 해결한다.

```javascript
//prop drilling 예시
type Item = { id: string, name: string };

function SearchableList({ items, onItemClick }) {
  return (
    <div className='searchable-list'>
      <List items={items} onItemClick={onItemClick} />
    </div>
  );
}

function List({ items, onItemClick }) {
  return (
    <ul className='list'>
      {items.map((item) => (
        <ListItem key={item.id} data={item} onItemClick={onItemClick} />
      ))}
    </ul>
  );
}

function ListItem({ data, onItemClick }) {
  return (
    <li className='list-item' onClick={() => onItemClick(data.id)}>
      {data.name}
    </li>
  );
}
```

### 1.4.2 컴포넌트 내 데이터 변환

외부 API나 백엔드에서 전달받은 데이터가 프론트엔드에 적합하지 않은 형태일 경우 컴포넌트 안에서 직접 변환 작업을 수행하는 경우가 있다.
이렇게 복잡한 데이터 변환 로직을 컴포넌트 내부에 직접 구현하면 다음과 같은 문제가 발생한다.

- 명확하지 않음: 데이터 가져오기와 변환, 렌더링 작업이 하나의 컴포넌트 안에서 이루어지므로 이 컴포넌트의 역할을 알기 어려워진다.
- 재사용성이 떨어짐: 다른 컴포넌트에서 유사한 변환이 필요한 경우, 로직의 중복이 발생한다.
- 테스트하기 어려움: 테스트를 하려면 변환로직을 고려해야 하므로 테스트 코드가 더 복잡해진다.

이러한 안티 패턴 방지를 위해 `유틸리티 함수` 나 `사용자 정의 훅` 을 이용해 데이터 변환 로직을 컴포넌트와 분리하는 것이 좋다.

```javascript
// 컴포넌트 내 데이터 변환 예시
function UserProfile({ userId }) {
  const [user, setUser] = (useState < User) | (null > null);

  useEffect(() => {
    fetch(`/api/user/${userId}`)
      .the((response) => response.json())
      .then((data: RemoteUser) => {
        // 컴포넌트 안에서 데이터 변환
        const transformedUser = {
          name: `${data.firstName} ${data.lastName}`,
          age: data.age,
          address: `${data.addressLine1}, ${data.city}, ${data.country}`,
        };
        setUser(transformedUser);
      });
  });

  ...
}
```

### 1.4.3 뷰 영역의 복잡한 로직

뷰 컴포넌트 안에서 비즈니스 로직을 다루는 경우 다음과 같은 문제가 발생한다.

- 재사용성: 다른 컴포넌트에서 유사한 로직이 필요한 경우 로직이 중복된다.
- 테스팅: 렌더링뿐만 아니라 비즈니스 로직도 테스트해야 하므로 단위 테스트가 복잡해진다.
- 유지보수성: 애플리케이션이 커지면서 더 많은 로직이 추가되기 때문에 유지보수하기 어려워진다.

컴포넌트를 재사용 가능하고 유지보수하기 쉽게 만들기 위해서는 `관심사 분리 원칙` (각각의 모듈 또는 함수가 애플리케이션의 하나의 기능에 대한 책임만을 가진다) 을 지키는 것이 좋다.

```javascript
// 뷰 영역에 복잡한 로직을 구현하는 예시
function PriceListView({ items }) {
  // 뷰 내부의 비즈니스 로직
  const filterExpensiveItems = (items) => {
    return items.filter((item) => item.price > 100);
  };

  const expensiveItems = filterExpensiveItems(items);

  return (
    <div>
      {expensiveItems.map((item) => (
        <div key={item.id}>
          {item.name}: ${item.price}
        </div>
      ))}
    </div>
  );
}
```

### 1.4.4 테스트 부족

계속해서 변화하는 영역과 여러 개발 로직이 교차하는 부분에서 개발자가 의도한대로 구현되고 있는지 확인이 필요하며 테스트 코드는 이를 쉽게한다.
테스트 코드를 먼저 작성하고, 실제 컴포넌트 로직을 나중에 작성하는 `테스트 주도 개발(TDD)` 방식을 사용하면 오류를 조기에 발견하기 쉽다.
또한 구조화, 유지보수 용이한 코드 작성이 가능하고 구현 코드 수정이나 기능추가에도 정확성을 보장한다.

### 1.4.5 중복된 코드

유사하거나 동일한 코드 조각이 애플리케이션의 여로 부분에 흩어져 있는 경우가 흔하다. 이러한 중복 코드는 코드량을 부풀리면서 잠재적 문제를 가져올 수 있다.
또한 버그 발견 혹은 개선이 필요한 경우 중복된 코드를 모두 변경해야 하므로 오류 발생 가능성이 높아진다.
`중복 배제 원칙` 을 따라서 코드를 작성하고, 공통 로직을 유틸리티 함수나 고차 컴포넌트로 모아서 관리하여 해결할 수 있다.
같은 데이터를 여러곳에서 사용하는 경우 `단일 진실 공급원` (모든 정보 요소를 하나의 출처에서만 공급하는 것) 으로 변경이 용이하게 할 수 있다.

### 1.4.6 너무 많은 기능을 가진 컴포넌트

하나의 컴포넌트가 여러개의 역할을 수행하면 복잡해지고 유지보수, 테스트가 힘들어진다.
컴포넌트는 하나의 구성 요소가 하나의 기능만을 수행해야 한다는 `단일 책임 원칙` 을 따르는 것이 권장된다.

### 1.4.7 안티패턴을 없애기 위한 접근 방식

- 다양한 디자인 패턴(render prop, 고차 컴포넌트, 커스텀 훅 등): 로직과 데이터, 프레젠테이션을 일관된 방식으로 나누고 코드를 간소화 할 수 있다. 또한 리액트 애플리케이션의 지속가능성을 높이며 개발자들의 효과적인 팀워크를 위한 기반을 마련한다.

- 인터페이스 지향 프로그래밍: 인터페이스를 통해 소프트웨어 모듈 간에 발생하는 상호작용을 중심으로 소프트웨어를 구성한다. 모듈화된 운영방식은 소프트웨어 모듈을 쉽게 변경하면서 동시에 일관성을 유지한다.

- 헤드리스 컴포넌트 패러다임: 사용하는 컴포넌트에 UI 렌더링의 역할을 넘겨주어 다양한곳에 재사용하기 편리하다.

- TDD와 지속적인 리팩토링: 잠재적인 불일치에 대한 즉각적인 피드백 루프를 제공하며 코드를 계속해서 최적화하고 개선시킨다.
