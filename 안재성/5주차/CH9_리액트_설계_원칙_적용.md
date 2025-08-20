# CHAPTER 9 리액트 설계 원칙 적용

[9.1 단일 책임 원칙](#91-단일-책임-원칙)   
[9.2 의존관계 역전 원칙](#92-의존관계-역전-원칙)   
[9.3 명령과 조회 책임 분리 원칙](#93-명령과-조회-책임-분리-원칙)   

## 9.1 단일 책임 원칙
컴포넌트의 주요 역할을 정확히 파악하고, 컴포넌트가 수행해야 할 핵심 기능을 분리하여 부가적인 기능들을 리팩터링하고 추상화하자.
1. render prop: 리액트 컴포넌트 간 코드 공유를 위해 함수 prop을 이용. 컴포넌트가 자체적으로 렌더링 로직을 구현하는 대신 리액트 요소를 반환하고 이를 호출하는 함수 사용
2. composition: 작고 재사용 가능한 컴포넌트들을 만들어 조합으로 더 복잡한 UI를 만듦

### 9.1.1 render prop 패넡을 통한 단일 책임 원칙 적용
```javascript
// react node를 반환하는 함수 children으로 Title외부에 title에 대한 상세 구현을 위임
const Title = ({
  title,
  children
}: {
  title: string,
  children: (s: string) => React.ReactNode;
}) => <div>{children(title)}</div>

// title에 적용할 포매팅 스타일을 Title이 선언되는 곳에서 지정할 수 있다.
// 렌더링 역할만 Title에 위임하고 스타일 적용은 외부로 분리하게 된다.
<Title title='This is a title'>
  {(s: string) => {
    const formatted = s.toUpperCase();
    return <h3>{formatted}</h3>
  }}
</Title>
```

### 9.1.2 합성을 통한 단일 책임 원칙 적용
```javascript
const Avatar = ({name, role, url}: AvatarProps) => {
  if (name) {
    return (
      <Tooltip name={name} role={role}>
        <div className='rounded'>
          <img src={ulr} alt={`${name}'s profile`} />
        </div>
      </Tooltip>
    )
  }
}

// Avatar 컴포넌트가 이미지를 보여주는 역할에 충실하도록 분리
const Avatar = ({name = '', url}: AvatarProps) => (
  <div className='rounded'>
    <img src={url} alt={name} title={name} />
  </div>
);

// Avatar와 Tootip 컴포넌트의 합성으로 사용할 수 있음
const MyAvatar = ({name, role, url}) => (
  <Tooltip name={name} role={role}>
    <Avatar name={name} url={ulr}/>
  </Tooltip>
)

```

## 9.2 의존관계 역전 원칙
`의존관계 역전 원칙 (DIP)`은 유지보수 가능하며 유연하고 확장 가능한 소프트웨어를 만들기 위해 필요한 SOLID 5원칙 중 하나로, 구체적인 구현보다는 추상화에 초점을 맞춤.   

단단하게 결합된 모듈로 인해 발생하는 문제를 해결한다. 상위 레벨의 모듈이 하위 레벨의 모듈에 의존관계를 가지면, 하위 레벨 모듈을 조금만 변경해도 광범위한 영향을 미쳐 시스템 전체 변경이 필요해짐.


### 9.2.1 의존관계 역전 원칙의 원리
```javascript
// Application은 EmailNotification과 결합되어 있어 Application의 확장성이 닫혀있음
// email 말고 sms방식으로 변경하려면? -> application 부분의 수정이 필요함
class EmailNotification {
  send(message: string, type: string) {
    console.log(`메세지를 담은 이메일을 전송합니다. message:${message}, type:${type}`);
  }
}

class Application {
  private emailNotification: EmailNotification;

  constructor(emailNotification: EmailNotification) {
    this.emailNotification = emailNotification;
  }

  process() {
    this.emailNotification.send('이벤트가 발생했습니다', 'info');
  }
}

const app = new Application(new EmailNotification());
app.process

```
```javascript
// Application의 수정 없이 확장성을 가질 수 있도록 인터페이스 계층을 추가하자
// Notification 규격을 따르는 구현체 제공으로 Application의 확장성을 보장
interface Notification {
  send(message: string, type: string): void;
}

class EmailNotification implemenets Notification {
  send(message: string, type: string) {
    console.log(`메세지를 담은 이메일을 전송합니다. message:${message}, type:${type}`);
  }
}

class SMSNotification {
    send(message: string, type: string) {
    console.log(`메세지를 담은 SMS를 전송합니다. message:${message}, type:${type}`);
  }
}

class Application {
  private notifier: Notification;

  constructor(notifier: Notification) {
    this.notifier = notifier;
  }

  process() {
    this.notifier.send('이벤트 발생', 'info');
  }
}

const emailApp = new Application(new EmailNotification());
const smsApp = new Application(new SMSNotification());
```


### 9.2.2 버튼 클릭 로그 수집에 의존관계 역전 원칙 적용하기


## 9.3 명령과 조회 책임 분리 원칙
`명령과 조회 책임 분리 원칙 (CQRS)`은 소프트웨어 설계에서 메서드나 함수는 시스템의 상태를 수정하는 명령이거나 시스템 상태에 대한 정보를 조회하여 반환하는 쿼리 둘 중에 하나여야 하며, 2가지가 동시에 수행되지 않아야 한다는 원칙.

`명령 메서드`는 `액션 또는 객체의 상태 변경을 수행하며 값을 반환하지 않음`. 반면 `조회 메서드`는 `객체의 상태를 변경 없이 읽기만 함`. 명령과 조회를 분리하면 컴포넌트 사이에 결합을 분리하여 테스트와 유지보수 및 코드 변경을 쉽게 만들며, 동작에 대한 추론이 쉬워져 시스템 전반적인 설계를 개선할 수 있다.


### 9.3.1 useReducer 훅
useReducer훅은 함수 컴포넌트에서 상태 관리를 하는데 사용. 특히 다음 상태가 이전 상태에 의존하여 변하거나, 복잡한 상태 로직일 때 유용.

useReducer훅은 reduce함수와 초기상태를 2개의 인자로 받음. 그리고 현재 상태와 업데이트를 수행하기 위한 dispatch 메서드를 반환.
```javascript
// 현재 상태와 변경하는 액션 타입을 전달 받아 동작
const shoppingCartReducer = (
  state: ShoppingCartState = initState,
  action: ActionType
) => {
  switch(action.type) {
    case "ADD_ITEM": {
      const item = {
        ...action.payload,
        uniqKey: `${action.payload.id}-${Date.now()}`,
      };
      return {...state, items: [...state.items, item]}
    }

    case "REMOVE_ITEM": {
      const newItems = state.items.filter(
        (item) => item.uniqKey !== action.payload.uniqKey
      );
      return {...state, items:newItems};
    }

    default:
      return state;
  }
}

const item = {
  id: 'p1',
  name: 'iPad',
  price: 666,
};

let x = shoppingCartReducer(initState, {
  type: "ADD_ITEM",
  payload: item,
});

/*{
  "item": [
    {
      "id": "p1",
      "name": "iPad",
      "price": 666,
      "uniqKey": "p1-1696059737801"
    }
  ],
  "totalPrice": 0
}*/

```

### 9.3.2 컨텍스트 안에서 reducer 함수 사용하기
