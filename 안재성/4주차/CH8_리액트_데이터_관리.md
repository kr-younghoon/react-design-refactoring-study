# CHAPTER 8 리액트 데이터 관리

[8.1 비즈니스 로직 누수 현상](#81-비즈니스-로직-누수-현상)   
[8.2 ACL오류 방지 계층](#82-acl오류-방지-계층)   
[8.3 Prop Drilling 문제 살펴보기](#83-prop-drilling-문제-살펴보기)   
[8.4 Context Api를 통한 Prop Drilling 문제 해결](#84-context-api를-통한-prop-drilling-문제-해결)   

### 8.1 비즈니스 로직 누수 현상
> 비즈니스 로직과 관련 없는 컴포넌트나 애플리케이션 영역으로 흘러 들어가는 현상
```javascript
// 가져온 데이털르 변환한 뒤 그 값을 user 상태 변수에 저장 
// JSON 응답을 프론트엔드 에플리케이션에 적합한 이름으로 변환
function UserProfile({id}: {id:string}){
  const[user, setUser] = useState<User | null>(null)

  useEffect(() => {
    async function fetchUser() {
      const response = await fetch(`/api/users/${id}`);
      const data = await response.json();

      setUser({
        id: data.user_identification,
        name: data.user_full_name,
        isPremium: data.ispremium_user,
        subscription: data.subscription_details.level,
        expire: data.subscription_details.expiry,
      });
    }

    fetchUser();
  }, [id])

  if (!user) {
    return <div>Loading...</div>
  }

  return (
    <div data-testid='user-profile'>
      <h1>{user.name}</h1>
    </div>
  )
}
```
이러한 변환은 애플리케이션 전반에서 발생할 수 있으며, 리액트 컴포넌트 뿐만 아니라 훅이나 다른 곳에서도 발생한다. 백엔드로부터 전달받은 데이터 형태가 api마다 달라질 경우 코드 영역에서 데이터 변환이 필요하기 때문에 데이터 구조가 변경될 때 일부 영역에서 업데이트를 놓칠 수도 있다.

### 8.2 ACL(오류 방지 계층)
- 프론트엔드로 연결되는 서브시스템 사이의 중재 역할
- 프론트엔드 개발시 일관성이 부족한 데이터 형식을 다루는 경우 ACL 기법으로 통합 인터페이스를 구축
- 캐시 처리, 오류 변환과 같은 문제를 처리하는 전략 계층으로 활용 가능 (중복 배제)

#### 8.2.1 일반적인 ACL 사용법

```javascript
// type RemoteUser: 외부(백엔드 서버)로부터 받은 데이터 타입
// type User: 프론트엔드에서 사용할 데이터 타입
export const transormUser = (remoteUser: RemoteUser): User => {
  return {
    id: remoteUser.user_identification,
    name: remoteUser.user_full_name,
    isPremiun: remoteUser.is_premium_user,
    subscription: remoteUser.subscription_details.level as UserSubscription,
    ...
  }
}


// fetchUserData 헬퍼 함수로 외부로부터 들어온 데이터 객체에 대해 내부를 격리
// 외부 데이터 타입이 변환되더라도 변경 범위를 헬퍼 함수까지로 제한할 수 있음
async function fetchUserData<T>(id:string) {
  const reponse = await fetch(`/api/users/${id}`);
  const rawData = await response.json();

  return transformUse(rawData) as T;
}
```

#### 8.2.2 예외 상황 대응 또는 기본값
- 원격 서버로부터 전달받은 데이터 타입에 대한 예외 처리와 에러 대응를 헬퍼함수에서 해보자
```javascript
// 원격 데이터에 포함되어 있을 값이 없는 경우 null 혹은 undefined로 처리되기 때문에
// 미리 기본값을 지정해두어 애플리케이션이 터지는 것을 방지
export const transormUser = (remoteUser: RemoteUser): User => {
  return {
    id: remoteUser.user_identification ?? 'N/A', 
    name: remoteUser.user_full_name ?? 'Unknown User',
    isPremiun: remoteUser.is_premium_user ?? false,
    subscription: (remoteUser.subscription_details.level ?? Basic ) as UserSubscription,
    ...
  }
}

```

### 8.3 Prop Drilling 문제 살펴보기
> 데이터가 필요하지 않은 여러 계층으로 이루어진 컴포넌트를 통과하여 하위 계층으로 데이터를 전달해야 할 때 발생하는 문제

### 8.4 Context API를 통한 Prop Drilling 문제 해결
> 동일한 부모 컴포넌트에 속한 모든 하위 컴포넌트가 공용으로 사용하는 컨테이너를 만들자
```javascript
// 상태를 저장할 스토어(컨테이너)를 생성
...
const SearchableListContext = createContext<SearchableListContextType>({
  onSearch: noop,
  onItemClicked: noop,
});


// 위의 컨텍스트로 감싼 하위 컴포넌트 어디에서나 컨텍스트 내부에 정의된 값과 함수에 접근할 수 있다
...
return (
  <SearchableListContext.Provider value={{onSearch, onItemClicked}}>
    <SearchInput onSearch={handleSearch} />
    <List items={filteredItems} />
  </SearchableListContext.Provider>
)
...

// 직접 props로 전달해주지 않아도 자식 컴포넌트에서 onSearch, onItemClicked에 접근 가능
const List = ({ items }: {items: Item[]}) => {
  return (
    <section data-testid="searchable-list">
      <ul>
        {items.map((item)=> (
          <ListItem item={item} />
        ))}
      </ul>
    </section>
  )
}

const ListItem = ({ item }: {item:Item}) => {
  const { onItemClicked } = useContext(SearchableListContext);

  return (
    <li onClick={() => onItemCliked(item)}>
      <h2>{item.name}</h2>
      <p>{item.description}</p>
    </li>
  )
}

```