# CHAPTER 7 리액트에서의 테스트 주도 개발

[7.1 TDD 이해하기](#71-tdd-이해하기)   
[7.2 태스킹 이해하기](#72-태스킹-이해하기)   
[7.3 온라인 피자 가게 애플리케이션](#73-온라인-피자-가게-애플리케이션)   
[7.4 애플리케이션 요구사항 세분화](#74-애플리케이션-요구사항-세분화)  
[7.5 애플리케이션 헤드라인 구현](#75-애플리케이션-헤드라인-구현)  
[7.6 메뉴 목록 구현](#76-메뉴-목록-구현)  
[7.7 장바구니 만들기](#77-장바구니-만들기)  
[7.8 장바구니에 아이템 담기](#78-장바구니에-아이템-담기)  
[7.9 애플리케이션 리팩터링](#79-애플리케이션-리팩터링)  

### 7.1 TDD 이해하기
#### `레드-그린-리팩터 루프`
- `레드`: 새로운 기능 또는 기능의 수정 사항을 확인하기 위한 테스트를 작성. 아직 기능 구현이 되지 않은 상태에서 작성하기 때문에 초기에는 테스트가 실패할 수밖에 없다. 
- `그린`: 테스트를 통과시키기 위한 최소한의 코드를 작성. 되도록 적은 코드를 작성하여 테스트를 통과하고, 텍스트를 녹색으로 바꾸는 것으로 충분하다.
- `리팩터`: 기능 동작을 유지하면서 코드를 정리. 기능을 그대로 유지하면서 코드의 효율성과 가독성을 높이는 작업을 진행. 리팩터링이 완료된 후에도 테스트는 통과해야만 한다.

#### TDD의 장점
- `집중적으로 문제 해결`: 특정 기능에 대한 테스트를 먼저 작성함으로써 한 번에 하나의 문제만 집중하고 개발 부담을 덜 수 있다.
- `예측 가능한 다음 단계`: 테스트 주도 방식을 따르면, 다음 단계에서 해야할 일은 테스트를 통과하는 것이다. 이는 인지 부하를 줄여 직면한 작업 자체에 집중할 수 있게 한다.
- `단순하고 유지보수하기 쉬운 설계`: 테스트를 통과하는 데 필요한 가장 단순한 코드를 작성하게 한다. 최소한의 설계를 구성하므로 이해하고 유지보수하기 쉽다.
- `사고의 흐름 유지`: TDD는 코딩에 대한 구조화된 접근 방식으로 사고의 흐름을 유지하게 한다. 이는 생산적인 코딩의 흐름을 방해하는 잦은 컨텍스트 전환을 중여주어 하나의 작업에만 집중할 수 있게 도와준다.
- `자동화된 테스트 커버리지`: TDD는 애플리케이션이 탄탄한 테스트 커버리지를 이미 가지고 있음을 보장한다. 테스트를 작업 이후에 추가하는 게 아니라 개발 과정에 포함되어 있기 때문에, 보다 안정적인 코드베이스를 만든다.

#### 7.1.1 여러 종류의 TDD
- `기본 TDD`
  - 단위 테스트에 집중: 개별 함수나 메서드와 같은 가장 작은 단위의 코드에 대해 테스트 작성
  - 코드베이스의 각 부분들이 격리된 상태에서 잘 동작하는지를 확인하는 것이 목표
  - 로직과 알고리즘을 테스트하기에는 강력
  - 리액트처럼 복잡한 UI 프레임워크에서는 여러 부분이 어떻게 상호작용을 하며 동작하는지 찾아내기 어려움

- `ATDD`(Acceptance Test Driven Development)
  - 사용자 승인 테스트로 개발 과정을 시작하는 것을 추가하여 TDD 확장
  - 어떤 코드를 작성하기 전에 이해관계자들과 협업하여 사용자 관점에서 완료 상태를 정의, 이후 기능 개발의 기초 자료로 활용
  - 사용자가 원하는 것과 요구하는 사항들을 확실하게 반영할 수 있는 방법

- `BDD`(Behavior Driven Development)
  - 애플리케이션에 주어지는 입력에 대한 반응에 주목
  - 시스템이 특정 조건에서 예상한 대로 동작하는지를 확인
  - 보다 서술적인 언어를 사용해 이해관계자 중 비개발자도 이해할 수 있게 함
  - Cucumber 도구를 사용

#### 7.1.2 사용자 가치에 집중하기
- 사용자는 상태관리나 효율적 생명주기 메서드 관리 등에는 관심 없고 오로지 버튼이 의도한대로 동작하는지, 폼 양식을 제출하면 결과가 의도한대로 나오는지가 중요하다.   
- 사용자 경험에 집중하기 위해서 테스트는 소프트웨어를 사용하는 방식을 닮을수록 좋다.
- 세부 구현보다 상호작용의 결과에 집중하는 BDD, ATDD 역시 사용자 중심 접근법과 같은 맥락.


### 7.2 태스킹 이해하기
- 태스킹은 사용자 스토리 또는 기능을 작고 다루기 편한 태스크로 나누어 테스트 케이스의 기본 단위로 활용
- 태스킹의 목적은 코드로 무엇을 작성할지, 어떻게 테스트할지, 어떤 순서로 진행할지를 분명히 결정하는데 있음

#### 큰 요구사항을 작은 단위로 쪼갤 때의 장점
- 명확해지는 범위: 무엇을 완료해야 하는지, 어떤 방법으로 접근할지 이해하기 쉬워짐
- 문제의 단순화: 복잡한 문제를 작은 태스크로 나우어 다루기 용이
- 우선순위 결정: 태스크가 나뉘면, 중요도와 논리적 구축에 따라 작업의 우선순위를 정할 수 있음
- 개별 작업에 집중: 태스킹은 작성하는 테스트가 명확하고 즉각적인 목적을 수행하도록 보장
- 협업 증대: 팀원들은 개별 태스크를 맡아 작업할 수 있음. 이때 모든 작업이 큰 단위 기능으로 결합되어 기여한다는 점을 알고 각자 작업 진행 가능

#### 태스킹 방법
1. 사용자 스토리와 요구사항 검토: 사용자 스토리를 검토하거나 구현할 기능에 대해 이해
2. 논리적인 구성 요소 식별: 스토리를 도메인 개념, 비즈니스 규칙, 사용자 작업의 단위와 같은 로직 단위로 나눔
3. 태스크 목록 작성: 태스크는 15~30분 정도로 짧은 시간 안에 구현 가능할 만큼 작은 단위
4. 태스크 순서 배열: 모든 것이 순조롭게 오류 없이 동작하는 기본 시나리오 '정상 흐름'으로 시작하고, 이후에 예외 케이스를 다룬 다음 오류를 처리
5. 태스크를 테스트로 처리: 태스크 각각의 기능을 보장하는 테스트 케이스 식별


### 7.3 온라인 피자 가게 애플리케이션
- 피자메뉴: 8가지 종류 피자 구성 메뉴. 피자 이름, 가격 표시
- Add 버튼: 각각의 옵션 옆에 Add 버튼 위치, 선택한 피잘르 장바구니에 담음
- 장바구니: 화면의 지정된 영역에 장바구니 표시. 선택한 피자의 이름과 가격 표시
- 장바구니 수정: 장바구니에 담긴 아이템 추가/삭제
- 전체 주문: 장바구니에서 선택한 아이템 전체 결제 가격 계산
- Place my order 버튼: 최종 주문 버튼. 배달 또는 픽업으로 주문 처리

### 7.4 애플리케이션 요구사항 세분화
#### 상향식 TDD
- 작고 기초적인 기능들의 테스트 코드를 작성하고 기능을 구현
- 개별 단위 또는 클래스부터 만들어가면서 상위 컴포넌트로 결합하여 구성하기 전에 철저히 테스트
- 시스템의 개별 부분들을 강력하게 검증하여 견고한 기반을 만드는데 도움
- 개별 단위 간 상호작용에 대한 고려와 전체 시스템을 통합적으로 보는 관점이 부족할 경우, 통합 작업 시 어려울 수 있음

피자가게 페이지 세부 작업 나누기
- 피자 이름을 포함하는 하나의 PizzaItem 컴포넌트 구현
- PizzaItem에 가격 추가
- PizzaItem에 버튼 추가
- PizzaList 컴포넌트 추가
- 간단한 기능의 ShoppingCart 컴포넌트와 버튼 구현
- ShoppingCart 컴포넌트에 아이템 추가 및 삭제 기능 지원
- 전체 피자 개수 계산
- 개별 컴포넌트를 통합하여 전체 애플리케이션 구현

#### 하향식 TDD
- 시스템의 전체 구조와 기능부터 시작
- 메인 컴포넌트의 기능을 구현하는 것부터 시작, 이후 세부적인 기능들로 점진적 확장
- 시스템의 주된 목적과 작업의 흐름을 초반부터 확립할 수 있고, 명확한 개발 로드맵을 알 수 있음
- 개발되기 전인 하위 단계 컴포넌트의 동작을 파악하기 위해 임시로 스텁이나 모킹이 필요할 수 있음

피자가게 페이지 세부 작업 나누기
- 페이지 제목 구현
- 피자의 이름을 포함하는 메뉴 목록 구현
- 버튼만 있는 ShoppingCart 컴포넌트 구현
- ShoppingCart 버튼이 활성화 되었을 때 아이템 목록에서 버튼을 누르면 추가되는 기능 구현
- ShoppingCart 컴포넌트에 가겨 추가
- ShoppingCart에 선택한 아이템의 전체 개수를 표시
- ShoppingCart의 아이템을 제거하고, 전체 개수를 그에 맞게 변경


### 7.5 애플리케이션 헤드라인 구현
```javascript
// 페이지 제목 구현하기 테스트
// 테스트를 먼저 작성했기 때문에 '레드 단계' - 테스트 실패
desxribe('Code oven Application', () => {
  it('renders application heading', () => {
    render(<PizzaShopApp />);
    const heading = screen.getByText('The Code Oven');
    expect(heading).toBeInTheDocument();
  });
});

// 위의 테스트를 통과하기 위한 코드를 작성하자
// '그린 단계' - 테스트 통과

// App.tsx
function PizzaShopApp() {
  return <>The Code Oven</>
}

```

> TDD는 반복되는 과정이므로 초기 코드가 완벽할 필요가 없다! 계속해서 좋은 테스트 코드로 점검하면서 나은 코드로 바꿔 나갈 수 있다.

### 7.6 메뉴 목록 구현
```javascript
// 테스트 코드를 먼저 작성 - 테스트 실패
it('renders menu list', () => {
  render(<PizzaShopApp />);
  const menuList = screen.getByRole('list'); // list역할 HTML 요소 찾기
  const menuItems = within(menuList).getAllByRole('listitem'); // within으로 검색 목록 좁히기

  expect(menuItems.length).toEqual(8); // 피자 가게에서 제공하는 피자 메뉴수 8개와 동일한지 확인
});


// 테스트 코드에 따라 코드 구현 - 테스트 성공
// 완벽하지는 않지만 테스트는 통과함 -> 이후 개선할 기회를 찾자
export function PizzShopApp() {
  return <>
    <h1>The Code Oven</h1>
    <ol>
      <li></li>
      <li></li>
      <li></li>
      <li></li>
      <li></li>
      <li></li>
      <li></li>
      <li></li>
    </ol>
  </>
}

// 이러한 방식은 코드가 언제든지 배포 가능한 상태를 유지한다는 장점이 있다.
// 리팩토링 해기

export function PizzaShopApp() {
  return <>
    <ol>
      {new Array(8).fill(0).map(x => <li></li>)}
    </ol>
  </>
}
```

```javascript
// 테스트 코드 추가하기
it('renders menu list', () => {
  render(<PizzaShopApp />);
  const menuList = screen.getByRole('list'); 
  const menuItems = within(menuList).getAllByRole('listitem'); 

  expect(menuItems.length).toEqual(8); 

  expect(within(menuItems[0]).getByText('Margherita Pizza')).toBeInTheDocument();
  expect(within(menuItems[1]).getByText('Pepperoni Pizza')).toBeInTheDocument();
  expect(within(menuItems[2]).getByText('Veggie Supreme Pizza')).toBeInTheDocument();
});

// 추가된 테스트를 통과하기 위해 코드 추가
const pizzas = [
  "Margherita Pizza",
  "Pepperoni Pizza",
  "Veggie Supreme Pizza",
  "Chicken BBQ Pizza",
  "Spicy Meat Feast Pizza",
  "Pasta Primavera",
  "Caesar Salad",
  "Chocolate Lava Cake"
]

export function PizzaShopApp() {
  return <>
    <h1>The Code Oven</h1>
    <ol>
      {new Array(8).fill(0).map(x => <li></li>)}
    </ol>
  </>
}
```

### 7.7 장바구니 만들기
```javascript
// 컨테이너와 버튼이 있는지 확인하는 단순 테스트 부터 시작하기
it('renders a shopping cart', () => {
  render(<PizzaShopApp />);

  const shoppingCartContainer = screen.getByTestId('shopping-cart');
  const placeOrderButon = within(shoppingCartContainer).getByRole('button');
  
  expect(placeOrderButton).toBeInTheDocument();
})

// 테스트 코드를 통과하기 위한 코드 추가
export function PizzaShopApp() {
  return <>
    <h1>The Code Oven</h1>
    <ol>
      {new Array(8).fill(0).map(x => <li></li>)}
    </ol>

    <div data-testid="shopping-cart">
      <button></button>
    </div>
  </>
}

// 테스트 코드 추가하기
it('renders a shopping cart', () => {
  render(<PizzaShopApp />);

  const shoppingCartContainer = screen.getByTestId('shopping-cart');
  const placeOrderButon = within(shoppingCartContainer).getByRole('button');
  
  expect(placeOrderButton).toBeInTheDocument();
  expect(placeOrderButton).toHaveTextContent('Place my order');
  expect(placeOrderButton).toBeDisabled();
})


// 추가한 테스트 코드 충족시키기
// 테스트 코드를 통과하기 위한 코드 추가
export function PizzaShopApp() {
  return <>
    <h1>The Code Oven</h1>
    <ol>
      {new Array(8).fill(0).map(x => <li></li>)}
    </ol>

    <div data-testid="shopping-cart">
      <button disabled>Place my order</button>
    </div>
  </>
}
```

### 7.8 장바구니에 아이템 담기
```javascript
// 장바구니 테스트 코드 작성
it('adds menu item to shopping cart', async () => {
  render(<PizzaShopApp />);

  // 메뉴 목록 리스트내 각 아이템 찾기
  const menuList = screen.getByTestId('menu-list');
  const menuItems = within(menuList).getAllByRole('listitem');

  // 메뉴 0번의 버튼을 찾고 클릭하기
  const addButton = within(menuItems[0]).getByRole('button');
  await userEvent.click(addButton);

  // 장바구니 컨테이너 내 주문하기 버튼 찾기
  const shoppingCartContainer = screen.getByTestId('shopping-cart');
  const placeOrderButton = within(shoppingCartContainer).getByRole('button');

  // 0번 메뉴인 마르게리따 피자가 들어있는지 확인하고 주문 버튼 활성화인지 확인
  expect(within(shoppingCartcontainer).getByText('Margherita Pizza')).toBeInTheDocument();
  expect(placeOrderButton).toBeEnabled();
});

// 테스트 코드에 맞게 코드 작성하기
export function PizzaShopApp() {
  const [cartItems, setCartItems] = useState<string[]>([]);

  const addItem = (item: string) => {
    setCartitems([...cartItems, item])
  }

  return <>
    <h1>The Code Oven</h1>
    <div data-testid='menu-list'>
      <ol>
        {pizzas.map((x)= > <li>
          {x}        
          <button onClick={() => addItem(x)}>Add</button>
        </li>)}
      </ol>
    </div>

    <div data-testid="shopping-cart">
      <ol>
        {cartItems.map((x) => <li>{x}</li>)}
      </ol>
      <button disabled>Place my order</button>
    </div>
  </>
}
```


### 7.9 애플리케이션 리팩터링
