# CHAPTER 6 일반적인 리팩터링 기법 살펴보기

[6.1 리팩터링 이해하기]()
[6.2 ]()
[6.]()
[6.]()
[6.]()


## 6.1 리팩터링 이해하기
>`리팩터링`은 체계화돤 절차와 방법에 따라 기존의 코드베이스에서 `기능 동작의 외적인 변화 없이 구조를 개선`하는 것
- 한번에 완벽하게 리팩터링 하는 것은 불가능
- 반복적이고 점진적으로 지속해서 수행하는것
- 리팩터링과 리스트럭쳐링을 혼동하지 말것
    - 리팩터링: 가독성을 높여 작업이 용이하도록 개선하는 것. 기능 추가 X
    - 리스트럭처링: 큰 범위, 외부 기능의 변화, 설계와 데이터 모델, 인터페이스 등의 변경을 포함

## 6.2 리팩터링 전 테스트 추가하기
- 리팩터링은 기능 변화가 없어야 하기 때문에 현재 동작을 보장해줄 테스트 코드가 필요
- 개발 단계에 맞춰 초기 설정한 변수명을 적절하게 변경 시키는 것으로 가독성을 올릴 수 있음

## 6.3 변수 추출하기
- 상수 추출: 하드코딩 된 값을 상수로 뽑아내 가독성과 유지 보수성을 높임

## 6.4 반복문을 파이프라인으로 바꾸기
- 고차 함수
- map, filter, reduce 메서드와 체이닝 기법을 활용하여 파이프라인 형태로 구현
```javascript
// 단순 반복문
calculateTotal() {
  let total = 0;
  for (let i =0; i<this.times.length; i++) {
    let item = this.items[i];
    let subTotal = item.price * item.quantity;
    if (item.quantity > 10) {
      subTotal *= DISCOUNT_RATE;
    }
    total += subTotal;
  }
  return total
}

// reduce함수를 활용한 파이프라인 방식
// 불필요한 변수 선언부를 없애 코드 수를 줄임
calculateTotal() {
  return this.items.reduce((total, item) => {
    let subTotal = item.price * item.quantity;
    return total + (item.quantity > 10 ? subTotal * DISCOUNT_RATE : subTotal)
  }, 0);
}
```

## 6.5 함수 추출하기
- 함수 일부를 새로운 함수로 추출하고 의미를 담아 이름을 지으면 함수 자체가 기능을 설명할 수 있어 문서화 효과
- 작은 단위로 나눌수록 재사용성 향상
```javascript
function applyDiscountIfEligible(item: Item, subTotal: number) {
  return item.quantity > 10 ? subTotal * DISCOUNT_RATE : subTotal;
}

class ShoppingCart {
  calculateTotal() {
    return this.items.reduce((total, item) => {
      let subTotal = item.price * item.quantity;
      return total + applyDiscountIfEligible(item, subTotal)
    })
  }
}
```

## 6.6 매개변수 객체 도입
- 함수가 많은 수의 매개변수를 가질 때
- 여러개의 함수가 같은 매개변수를 공유할 때
```javascript
class ShoppingCart{
  items: Item[] = [];

  addItemToCart({id, price, quantity}:Item) {
    this.items.push({ id, price, quantity })
  }
}
```
- 도메인 개념을 상기
- 매개변수를 별도의 클래스로 사용하여 캡슐화, 객체 지향적 달성 가능

## 6.7 조건문 분해하기
- if-else, switch의 조건 분기점을 별도 함수로 추출


## 6.8 함수 이동하기
- 클래스 내부 코드가 늘어날 때 함수들을 모듈화 시켜 별도로 분리하기
- 기능의 연관성이 높거나 필요한 곳으로 옮겨 응집도 상승


### :bulb: Tanstack Query를 활용한 리팩터링 실전연습
#### 1. 커스텀 훅 분리하기
```javascript
// UI 컴포넌트로부터 비동기 로직을 분리하기
// 분리 전
const { data } = useQuery({ queryKey: ['todolist'], queryFn: fetchTodolist})


// 분리 후
// hooks/useTodolist.ts
export const useTodolist = () => useQuery({ queryKey: ['todolist'], queryFn: fetchTodolist})

// 컴포넌트에서 사용하기
const {data} = useTodolist();
``` 

#### 2. 공통 Query Option 추출
```javascript
const defaultQueryOptions = {
  staleTime: 1000 * 60, // 1분
  cacheTime: 1000 * 300, // 5분
  retry: 1,
};

useQuery({
  queryKey: ['data'],
  queryFn: fetchData,
  ...defaultQueryOptions,
});

// 또는 QueryProvier에서 defaultOption으로 설정하기
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 60_000,
      retry: 1,
    },
  },
});
``` 

#### 3. mutation 로직 추상화
```javascript
// 공통 로직 invalidate, toast, optimisitc update 등 재사용하기
export const useCustomMutation = (mutationFn, queryKey) => {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn,
    onSuccess: () => queryClient.invalidateQueries({ queryKey }),
  });
};

``` 

#### 4. Query Key 상수화
```javascript
// 협업시 쿼리키 관리가 용이하고 계층화된 쿼리키 관리하기 쉬움
const STUDY_QUERY_KEYS = {
  'all': () => ['study'],
  'detail': (id: string) => [...STUDY_QUERY_KEYS.all(), pagination, id],
};

useQuery({ queryKey: STUDY_QUERY_KEYS.detail(1), queryFn: fetchTodo });
``` 

#### 5. select로 가공 로직 분리
```javascript
// 데이터 처리를 컴포넌트에서 분리, 쿼리레벨에서 처리하기
// MSW를 활용할 경우 이후 백엔드와 연동시 유리
useQuery({
  queryKey: ['todos'],
  queryFn: fetchTodos,
  select: (data) => data.filter((todo) => !todo.done),
});
``` 

#### 6. enabled 조건분기
```javascript
// id가 있는 경우에만 Query 보낼수 있도록 하기 - 별로 if문 없이 가능
useQuery({
  queryKey: ['todo', id],
  queryFn: () => fetchTodo(id),
  enabled: !!id, // id가 있어야 실행
});
```

#### 7. Error, Loading, Empty 처리 UI 추상화
```javascript
// 같은 UI로 Loading, Error를 처리하는 경우 패턴 제거
function QueryStateWrapper({ isLoading, error, children }) {
  if (isLoading) return <Spinner />;
  if (error) return <ErrorMessage />;
  return children;
}
```
