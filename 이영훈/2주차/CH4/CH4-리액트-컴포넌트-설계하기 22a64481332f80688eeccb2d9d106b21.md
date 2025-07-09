# CH4-리액트-컴포넌트-설계하기

생성자: 영훈
생성 일시: 2025년 7월 9일 오전 1:33
카테고리: 패턴으로익히고설계로완성하는리액트
최종 편집자:: 영훈
최종 업데이트 시간: 2025년 7월 9일 오후 6:42

# 개요

리액트 컴포넌트를 설계하는데 있어 가장 중요한 내용

- 리액트 컴포넌트를 설계할 때 피해야하는 거대한 단일 컴포넌트
- Props Drilling 같은 안티패턴에 대해..
- 리액트 유지보수 및 확장에 방해가 되는 함정과 이를 피하는 방법
- 단일 책임 원칙(SRP)을 소개
    - 각각 컴포넌트는 하나의 특정한 목적을 가져야 함을 강조하는 원칙
    - 컴포넌트를 테스트하고 유지보수하기 쉬우며, 코드를 더 읽기 쉽게 관리 가능
- 중복 배제 원칙
    - 효과적인 프로그래밍을 위한 핵심 원칙
    - 중복을 최소화, 재사용성 향상
    - 리액트에서 간소화되고 효율적이며 유지보수 가능한 코드베이스를 만드는 중요한 열쇠
- 컴포넌트 합성 원칙
    - 단순하고 재사용 가능한 컴포넌트를 조합하여 복잡한 UI를 구성
    - 리액트에서 상속봐 합성을 선호하며, 합성을 통해 더욱 유연하고 다루기 쉬운 컴포넌트를 만들 수 있다.

# 4.1 단일 책임 원칙(single responsibility principle)

- 소프트웨어 공학의 기본 개념 중 하나
    - 함수, 클래스, 리액트 컴포넌트는 변경해야 할 이유가 단 하나만 있어야 함을 의미
    - 하나의 컴포넌트 ⇒ 하나의 작업이나 기능을 수행 하는 것.
    - 코드 가독성이 향상, 유지보수, 테스트, 디버깅이 쉬워진다.
- 예제
    - p93 블로그 포스트 데이터를 불러와서 포스트를 표시하고 사용자가 ‘좋아요’를 누르는 기능 까지 모두 하나의 BlogPost 컴포넌트 안에서 수행하는 예시
    
    ```jsx
    import React, { useState, useEffect } from "react";
    import fetchPostById from "./fetchPostById";
    interface PostType {
    id: string;
    title: string;
    summary: string;
    }
    const BlogPost = ({ id }: { id: string }) => {
    	const [post, setPost] = useState<PostType>(EmptyBlogPost);
    	const [isLiked, setIsLiked] = useState(false);
    	
    	useEffect(() => {
    		fetchPostById(id).then((post) => setPost(post));
    	}, [id]);
    	
    	const handleClick = () => {
    		setIsLiked(!isLiked);
    	};
    
    return (
    	<div>
    		<h2>{post.title}</h2>
    		<p>{post.summary}</p>
    		<button onClick={handleClick}>
    			{isLiked ? "Unlike" : "Like"}
    		</button>
    	</div>
    	);
    };
    
    export default BlogPost;
    ```
    
    - 이 코드는 단일 책임 원칙을 위반한다.
        - 데이터 가져오기, 블로그 포스트 표시, 좋아요 기능 관리 까지 3가지 동작을 하나의 컴포넌트에서 수행하기 때문.
- 리팩토링 예시
    
    ```jsx
    const useFetchPost = (id: string): PostType => {
    	const [post, setPost] = useState<PostType>(EmptyBlogPost);
    
    	useEffect(() => {
    		fetchPostById(id).then((post) => setPost(post));
    	}, [id]);
    
    	return post;
    };
    
    const LikeButton: React.FC = () => {
    	const [isLiked, setIsLiked] = useState(false);
    
    	const handleClick = () => {
    		setIsLiked(!isLiked);
    	};
    	
    	return <button onClick={handleClick}> {isLiked ? "Unlike" : "Like"}</button>;
    };
    
    const BlogPost = ({ id }: { id: string }) => {
    	const post = useFetchPost(id);
    
    	return (
    		<div>
    			<h2>{post.title}</h2>
    			<p>{post.summary}</p>
    			<LikeButton />
    		</div>
    	);
    };
    ```
    
    - useFetchPost는 블로그 포스트 데이터를 가져오는 사용자 정의 훅이다.
    - LikeButton : 좋아요 기능을 다루는 컴포넌트
    - BlogPost : 이제 블로그 포스트 내용과 LikeButton 렌더링만 맡게 된다.

# 4.2 중복 배제 원칙(Don’t repeat yourself)

- 소프트웨어 개발의 기본 개념
    - 코드 안에서 중복을 줄이는 것
    - 유지보수성과 가독성이 높아지고 테스트하기 쉬워지며 로직 중복으로 인해 발생하는 버그 방지
- 예시
    - 쇼핑 사이트에서 상품 목록 보여주고 [Add to Cart] 버튼을 나란히 표현한다 가정
    
    ![image.png](CH4-%E1%84%85%E1%85%B5%E1%84%8B%E1%85%A2%E1%86%A8%E1%84%90%E1%85%B3-%E1%84%8F%E1%85%A5%E1%86%B7%E1%84%91%E1%85%A9%E1%84%82%E1%85%A5%E1%86%AB%E1%84%90%E1%85%B3-%E1%84%89%E1%85%A5%E1%86%AF%E1%84%80%E1%85%A8%E1%84%92%E1%85%A1%E1%84%80%E1%85%B5%2022a64481332f80688eeccb2d9d106b21/image.png)
    
    - 코드로 구현시 아래와 같다
        
        ```jsx
        type Product = {
        	id: string;
        	name: string;
        	image: string;
        	price: number;
        };
        
        const ProductList = ({
        	products,
        	addToCart,
        }: {
        	products: Product[];
        	addToCart: (id: string) => void;
        }) => (
        	<div>
        		<h2>Product List</h2>
        		{products.map((product) => (
        			<div key={product.id} className="product">
        				<img src={product.image} alt={product.name} />
        				<div>
        					<h2>{product.name}</h2>
        					<p>${product.price}</p>
        					<button onClick={() => addToCart(product.id)}>Add to Cart
        						</button>
        				</div>
        			</div>
        		))}
        	</div>
        );
        
        export default ProductList;
        ```
        
    - Cart 컴포넌트는 유사한 구조를 가진다.
    - 아이템 콜백함수가 필요하고, [Remove from Cart] 텍스트 버튼을 보여준다.
    - 사용자가 버튼을 누르면 전달받은 콜백함수를 호출한다.
    
    ```jsx
    const Cart = ({
    		cartItems,
    		removeFromCart,
    }: {
    		cartItems: Product[];
    		removeFromCart: (id: string) => void;
    }) => (
    	<div>
    		<h2>Shopping Cart</h2>
    		{cartItems.map((item) => (
    		<div key={item.id} className="product">
    			<img src={item.image} alt={item.name} />
    		<div>
    			<h2>{item.name}</h2>
    			<p>${item.price}</p>
    			<button onClick={() => removeFromCart(item.id)}>
    				Remove from Cart
    			</button>
    		</div>
    	</div>
    	))}
    </div>
    );
    ```
    
    - 중복을 줄이고 각각의 컴포넌트가 하나의 역할만 할 수 있도록 `LineItem` 컴포넌트를 분리한다.
    
    ```jsx
    import { Product } from "./types";
    
    const LineItem = ({
    	product,
    	performAction,
    	label,
    }: {
    	product: Product;
    	performAction: (id: string) => void;
    	label: string;
    }) => {
    	const { id, image, name, price } = product;
    	
    	return (
    		<div key={id} className="product">
    			<img src={image} alt={name} />
    			<div>
    			<h2>{name}</h2>
    			<p>${price}</p>
    			<button onClick={() => performAction(id)}>{label}</button>
    		</div>
    	</div>
    	);
    };
    
    export default LineItem;
    ```
    
    - 기존의 코드 중복을 없앨 수 있다
    
    ```jsx
    const ProductList = ({
    	products,
    	addToCart,
    }: {
    	products: Product[];
    	addToCart: (id: string) => void;
    }) => (
    	<div>
    		<h2>Product List</h2>
    		{products.map((product) => (
    		<LineItem
    			key={product.id}
    			product={product}
    			performAction={addToCart}
    			label="Add to Cart"
    		/>
    	))}
    	</div>
    );
    ```
    
    - Cart 컴포넌트도 LineItem 컴포넌트을 재사용하여 상품 상세 정보를 렌더링하는 비슷한 구조를 가진다
    
    ```jsx
    const Cart = ({
    	cartItems,
    	removeFromCart,
    }: {
    	cartItems: Product[];
    	removeFromCart: (id: string) => void;
    }) => (
    	<div>
    		<h2>Shopping Cart</h2>
    		{cartItems.map((item) => (
    		<LineItem
    			key={item.id}
    			product={item}
    			performAction={removeFromCart}
    			label="Remove from Cart"
    		/>
    	))}
    	</div>
    );
    ```
    
    - 중복배제 원칙에 따라 유지보수하기 더 쉽고 재사용성을 높인 접근법입니다.
    - 변경은 한 곳에서만 일어나므로 버그 발생할 가능성이 줄어든다.
    - 만약 `LineItem` 에 새로운 기능을 추가한다면 하나의 컴포넌트만 수정하면 됌
- 마무리
    - 코드의 중복을 제거하여 불일치와 버그의 가능성을 줄여준다.
    - 코드 중복을 피하면, 기능 변경을 한 곳에서 해결할 수 있기 때문에 유지보수가 단순해질 수 있었다.
    - 리액트의 핵심 개념인 합성을 사용하여 컴포넌트를 구조화해보자

# 4.3 합성 활용하기

- 리액트에서 합성(composition)은 컴포넌트를 설계하는 자연스러운 패턴
- JSX 마크업 언어 문법은 div와 h2 태그 결합하기 위한 별도 문법 없이도 자연스럽게 연결해준다.
- 예시 - 사용자 정보를 표시하는 UserDashboard 컴포넌트

```jsx
type User = {
	name: string;
	avatar: string;
	friends: string[];
};

type Post = {
	author: string;
	summary: string;
};

type UserDashboardProps = {
	user: User;
	posts: Post[];
};

function UserDashboard({ user, posts }: UserDashboardProps) {
	return (
		<div>
			<h1>{user.name}</h1>
			<img src={user.avatar} alt="profile" />
			<h2>Friends</h2>
			<ul>
				{user.friends.map((friend) => (
					<li key={friend}>{friend}</li>
				))}
			</ul>
			<h2>Latest Posts</h2>
				{posts.map((post) => (
			<div key={post.author}>
			<h3>{post.author}</h3>
			<p>{post.summary}</p>
		</div>
		))}
	</div>
	);
}

export default UserDashboard;
```

- UserDashboard는 사용자의 프로필 정보, 친구목록, 최근 포스트 목록을 표시하며, 단일 책임 원칙을 위반한다.
    - 이것을 더욱 작은 컴포넌트로 쪼개고 각각 하나의 책임만을 가져가야 한다.
    - 먼저 UserProfile 컴포넌트 분리.
    
    ```jsx
    const UserProfile = ({ user }: { user: User }) => {
    	return (
    		<>
    			<h1>{user.name}</h1>
    			<img src={user.avatar} alt="profile" />
    		</>
    	);
    };
    ```
    
    - 두번째, FriendList 컴포넌트 분리
    
    ```jsx
    const FriendList = ({ friends }: { friends: string[] }) => {
    	return (
    		<>
    			<h2>Friends</h2>
    			<ul>
    				{friends.map((friend) => (
    					<li key={friend}>{friend}</li>
    				))}
    			</ul>
    		</>
    	);
    };
    ```
    
    - 마지막, PostList 컴포넌트 분리
    
    ```jsx
    const PostList = ({ posts }: { posts: Post[] }) => {
    	return (
    		<>
    			<h2>Latest Posts</h2>
    			{posts.map((post) => (
    				<div key={post.author}>
    					<h3>{post.author}</h3>
    					<p>{post.summary}</p>
    				</div>
    			))}
    		</>
    	);
    };
    ```
    
- 리팩토링한 UserDashBoard 컴포넌트의 여러가지 장점
    - 관심사 분리 : 단일 책임 원칙 때문에 분리해서 각각의 컴포넌트가 하나의 역할을 담당한다… 덕분에 유지보수가 쉽다.
    - 더 나은 가독성  : `UserDashboard` 는 가독성이 높아 이해하기 쉽다. 사용자 프로필, 친구 목록, 포스트 목록 등 컴포넌트가 어떤 것을 렌더링 할지 명확하다. 각 영역이 어디서 어떻게 렌더링 되는지 세세하게 볼 피룡가 없다.
    - 높은 재 사용성 UserProfile, FriendList, PostList 컴포넌트를 애플리케이션 다른 영역에서도 사용할 수 있기 떄문에 중복을 줄일 필요가 있따.
    - 테스트의 용이함. 의존하는 코드가 적어 테스트하기가 쉽다.

# 4.4 컴포넌트 설계 원칙의 결합

- 실전 코딩 상황은 생각보다 더 복잡하다. 코드 가독성과 유지보수성을 높이기 위해 여러 원칙을 동시에 적용해야만 한다.
- 예제 생략
- 커다란 컴포넌트를 작게 나누는 방법은 다양하다
    - 정보가 서로 관련되어 있다면, 하나의 그룹으로 묶고 이 단위로 새로운 컴포넌트를 만든다.
- 예제 초기 당시 지나치게 많은 역할을 맡고 있어서 prop은 늘어날 수 밖에 없었다.
- 추가로 많은 데이터를 여러 계층에 전달해야 하는 Props Drilling 문제가 있었다.
- 이 설계는 구성하기 복잡할 뿐만 아니라 유지보수하기도 어려웠다.
- 이러한 문제점들을 해결고자 여러 기본 설계원칙들을 적용하였다.
    - 단일 책임 원칙을 기반으로, 거대한 Page 컴포넌트를 다루기 쉬운 Header Sidebar, Main 컴포넌트로 분리하는 리팩토링 과정을 거쳤다
    - 서브 컴포넌트가 각자의 역할을 담당하게 하고 그에 필요한 prop만을 전달받아 구조가 단순해졌다.
- 컴포넌트를 분리한 뒤에 합성을 통해 Page 컴포넌트가 분리한 서브 컴포넌트를 prop으로 전달받도록 수정했다.
- 이 방법은 Page 컴포넌트를 사용하는 곳에서 서브 컴포넌트를 prop으로 제공하도록 하여 Prop Drilling 문제를 확연하게 줄였다.