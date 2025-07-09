# CHAPTER 4 리액트 컴포넌트 설계하기

[4.1 단일 책임 원칙](#41-단일-책임-원칙srp)  
[4.2 중복 배제 원칙](#42-중복-배제-원칙)  
[4.3 합성 활용하기](#43-합성-활용하기)  
[4.4 컴포넌트 설계 원칙의 결합](#44-컴포넌트-설계-원칙의-결합)

## 4.1 단일 책임 원칙(SRP)

`단일 책임 원칙은 함수, 클래스 혹은 리액트 컴포넌트는 변경해야 할 이유가 단 하나만 있어야 함`을 의미한다.

```javascript
// BlogPost 컴포넌트가 데이터 가져오기, 블로그 포스트 표시, 좋아요 기능 관리
// 3가지 동작을 수행하기 때문에 단일 책임 원칙을 위반
const BlogPost = ({ id }) => {
  const [post, setPost] = useState(EmptyBlogPost);
  const [isLiked, setIsLiked] = useState(false);

  useEffect(() => {
    fetchPostById(id).then((post) => setPost(post));
  }, [id]);

  const handleClick = () => {
    setIsLiked(!isLiked);
  };

  return (
    <div>
      <h2>{post.id}</h2>
      <p>{post.summary}</p>
      <button onClick={handleClick}>{isLiked ? 'Unlike' : 'Like'}</button>
    </div>
  );
};

// 단일 책임 원칙에 따라 리팩토링
// 블로그 포스트 데이터 가져오는 커스텀훅
const useFetchPost = (id) => {
  const [post, setPost] = useState(EmptyBlogPost);

  useEffect(() => {
    fetchPostById(id).then((post) => setPost(post));
  }, [id]);

  return post;
};

// '좋아요'기능 컴포넌트
const LikeButton = () => {
  const [isLiked, setIsLiked] = useState(false);

  const handleClick = () => {
    setIsLiked(!isLiked);
  };

  return <button onClick={handleClick}>{isLiked ? 'Unlike' : 'Like'}</button>;
};

// 블로그 포스트 '내용'과 LikeButton 두 컨텐츠의 렌더링 만을 담당
const BlogPost = ({ id }) => {
  const post = useFetchPost(id);

  return (
    <div>
      <h2>{post.id}</h2>
      <p>{post.summary}</p>
      <LikeButton />
    </div>
  );
};
```

## 4.2 중복 배제 원칙

`중복 배제(DRY) 원칙은 코드 안에서 중복을 줄이는 것이 목적`
같은 내용과 구조를 사용하는 영역을 별도의 컴포넌트로 분리하고 재사용 하는 방식으로 구현

```javascript
const ProductList = () => {
  ...

  return (
    <h2>Product List</h2>
    {products.map((product) => (
      <Item
        key={product.id}
        product={product}
        performAction={addToCart}
      />
    ))}
  )
}

const Cart = () => {
  return (
    <h2>Shopping Card</h2>
    {products.map((product) => (
      <Item
        key={product.id}
        product={product}
        performAction={removeFromCard}
      />
    ))}
  )
}

const Item = ({product, performAction}) => {
  const {name, id, price, image} = product
  return (
    <div key={id}>
      <img src={image} alt={name}/>
      <div>
        <h2>{name}</h2>
        <p>{price}</p>
      </div>
    </div>
  )
}
```

## 4.3 합성 활용하기

1. 하나의 컴포넌트 내에 관련된 관심사 별로 묶어 하위 컴포넌트로 분리하기
2. 내부 구현은 분리된 하위 컴포넌트에서 담당하고 상위 래퍼 컴포넌트에서는 배치와 조합만을 고려

```javascript
function UserDashboard({user,posts}) {
  return (
    <div>
      <UserProfile user={user}>
      <FriendList friends={user.friends}>
      <PostList posts={posts}/>
    </div>
  )
}
```

## 4.4 컴포넌트 설계 원칙의 결합

1. 컴포넌트가 보통 5개 이상의 prop을 가진다면 분리가 필요하다.

   - prop을 사용하는 방법에 따라 분류하기
   - 정보가 서로 관련되어 있는 것 끼리 하나의 컴포넌트로 묶기

2. 세분화한 하위 컴포넌트를 이용한 합성 방식을 사용하자.

   - Prop이 많아질 경우 각 하위 컴포넌트의 구현부를 외부에 위임할 수 있다
   - 이 방식으로 Prop Drilling 문제 해결 가능

```javascript
/*
방식 1: props로 데이터를 받아와 직접 구현하는 방식
장점
  - Page 컴포넌트가 내부 구현을 주도하여 어떤 UI를 구성할지 명확
  - 자식 컴포넌트에 필요한 데이터만 넘기므로 의존성 낮음
  - 일관된 방식으로 구성하기 쉬움

단점
  - Page가 하위 컴포넌트 구조에 강하게 결합됨
  - 상위 컴포넌트에서 어떤 컴포넌트가 사용되는지 알기 어려움
  - UI의 유연성이 떨어짐 (Header를 다른 방식으로 교체하기 어려움)

주 사용처
  - 단일 페이지 또는 매우 단순한 구조
  - 페이지마다 구조가 제각각: Header가 있는 페이지와 없는 페이지를 합성으로 구현하려면 불필요한 조건문이 더 늘어날 수 있음
*/

function Page({headerProp, sidebarProp, mainProp}) {
  return (
    <div>
      <Header headerProp={headerProp} />
      <Sidebar sidebarProp={sidebarProp}/>
      <Main sidebarProp={mainProp}/>
    </div>
  )
}

/*
방식 2: 컴포넌트를 직접 주입하는 방식 (Composition 방식)
장점
  - 유연성이 높음. Page는 UI의 컨테이너 역할만 하고, 실제 내용을 상위에서 자유롭게 조립 가능.
  - 테스트나 재사용성이 높음. 각 부분을 교체하거나 조건부 렌더링하기 쉬움.
  - 관심사의 분리가 잘 됨: Page는 “어디에 렌더링할지”, MyPage는 “무엇을 렌더링할지”에 집중.

단점
  - Page가 단순히 컨테이너 역할만 하므로 구조에 대한 정보가 분산될 수 있음.
  - 상위에서 컴포넌트를 직접 조립해야 하므로 번거로울 수 있음.
  - 내부 구현에서 실수로 header, sidebar 등을 잘못 렌더링하면 구조가 깨질 위험이 있음.

주 사용처
  - 레이아웃 자체가 자주 사용되는 경우: hedaer, sidebar, main 구성요소를 가지지만 세부 구현만 다른 경우
  - 관심사를 구조(Wrapper)와 내용(Content)으로 나누고 싶을 때
*/

function Page({haeader ,sidebar, main}) {
  return (
    <div>
      {header}
      {sidebar}
      {main}
    </div>
  )
}

function MyPage() {
  return (
    <Page
      header={
        <Header headerProp={headerProp} />
      }
      sidebar={<Sidebar sidebarProp={sidebarProp}/>}
      main={<Main {mainProp}/>}
    >
  )
}
```

각 방식의 필요성을 따지기 보다 내가 불편함을 느끼는 방향을 우선 정의해보는게 좋다

- 어떤 역할을 분리하고 싶은가?
- 얼마나 반복되는가?
