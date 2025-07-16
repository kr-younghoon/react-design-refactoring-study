# CHAPTER 5 리액트 테스팅

[5.1 테스트가 필요한 이유](#51-테스트가-필요한-이유)  
[5.2 여러 종류의 테스트 알아보기](#52-여러-종류의-테스트-알아보기)  
[5.3 Jest로 하는 개별 단위 테스팅](#53-jest로-하는-개별-단위-테스팅)   
[5.4 통합 테스트 ](#54-통합-테스트)   
~~[5.5 Cypress를 이용한 E2E 테스트](#55-cypress를-이용한-e2e-테스트)~~

## 5.1 테스트가 필요한 이유
테스트의 장점
- `코드 검증`: 의도한 방향으로 동작하고 있음을 확실히 알려준다. 
- `회귀 방지`: 애플리케이션 확장시 새로운 코드의 추가로 인해 기존에 잘 동작하던 기능이 깨질 수 있으며(회귀), 자동화된 테스트는 회귀를 잡아주는 안정망 역할을 한다.
- `리팩터링과 유지보수의 용이함`: 리팩터링이나 레거시 코드를 업데이트 하는 과정에서 겪는 불안을 줄여준다. 
- `코드 품질에 대한 신뢰 향상`: 잘 작성된 테스트 코드가 뒷받침될 때 코드의 품질을 정량적으로 측정할 수 있다.
- `문서화`: 테스트는 문서로 쓰일 수 있다. 테스트를 통해 함수나 컴포넌트의 수행 동작을 명확히 이해할 수 있고, 새로운 팀 동료가 프로젝트 기능 동작을 이해하는데 도움이 된다.


## 5.2 여러 종류의 테스트 알아보기
- 단위 테스트(Unit test): 개별 컴포넌트나 함수의 기능을 격리하여 테스트
- 통합 테스트: 다른 모듈과 서비스들이 원활하게 상호작용하여 잘 동작하는지 테스트
- E2E 테스트: 전체 애플리케이션의 흐름을 시작부터 종료까지 실제 사용자 행동을 묘사하여 테스트
- 시각적 회귀 테스트: 각기 다른 시점에서 웹페이즈 또는 컴포넌트의 스크린샷이나 스냅샷을 찍어 필섹 단위로 비교하여 시각적 차이점을 찾아내는 테스트
- 정적 검사: 오류 잡기, 코딩 표준 보장을 위해 코드를 실행시키지 않고 구문 분석을 하는 테스트


## 5.3 Jest로 하는 개별 단위 테스팅
### 5.3.1 테스트 작성하기
```javascript
import {add} from './math'
text('add adds numbers correctly',()=> {
  expect(add(1,2)).toBe(3);
})
```

#### :bulb: BDD 개발 방법론 - 행동주도
- TDD에서 파생된 개발방법론으로 TDD에서 기능 중심의 '테스트케이스'를 작성하는데 한계를 극복하기 위해 등장
- '사용자의 행위'를 기반으로 '사용자 시나리오'를 구성하여 테스트케이스를 작성하고 개발 진행

> 요구사항 명세화 > 테스트케이스 작성 > 테스트 수행 > 코드 구현 > 테스트 확인 > 리팩토링

테스트 시나리오 작성하는 방법 : Given, When, Then
- Given - 주어진 환경
  - '사용자 행위'를 수행하기 위해 주어진 '환경'
  - 시스템이나 애플리케이션의 초기 상태, 환경, 입력 등
  - ex) 사용자가 페이지에 접속해 있는 상황에서

- When - 행위
  - 실제 사용자 행위에 대해 서술
  - 시스템이나 애플리케이션 내에 발생하는 특정 조건이나 이벤트들에 대해 서술
  - ex) 사용자 아이디 필드에 'admin54' 를 입력하고 비밀번호 필드에 '12345'를 입력 한 뒤 '로그인' 버튼을 누르면

- Then - 기대 결과
  - 행위에 따른 기대 결과 서술
  - 예상되는 동작이 실제로 발생하는지 확인하고 기대한 결과가 나오는지에 대한 검증을 수행
  - ex) 로그인이 성공하고 메인 페이지로 이동

> (Given)사용자가 페이지에 접속해 있는 상황에서    
> (When)사용자 아이디 필드에 'admin54' 를 입력하고 비밀번호 필드에 '12345'를 입력 한 뒤 '로그인' 버튼을 누르면   
> (Then)로그인이 성공하고 메인 페이지로 이동

- 기획 누락의 발견
- 사전에 예외 케이스를 찾아서 추후 발생할 문제를 줄이기

### 5.3.2 테스트 그룹 묶기
```javascript
// describe로 기능 영역 구분하여 가독성을 높이자
describe('math function', () => {
  it('adds positive numbers correctly', () =. {
    expect(add(1, 2).toBe(3));
  });

  it('adds negative numbers correcttly', () => {
    expect(add(-1, -2)).toBe(-3);
  });
  // 그 외 테스트 코드
})
```

### 5.3.3 리액트 컴포넌트 테스트
- 리액트 테스팅 라이브러리(RTL)을 활용하여 컴포넌트 테스팅

```javascript
// 테스트 할 컴포넌트
const Section = ({heading, content}) => {
  return (
    <article>
      <h1>{heading}</h1>
      <p>{content}</p>
    </article>
  )
}

// 컴포넌트 테스트 코드
describe("Section", () => {
  it("render a section with heading and content", () => {
    // 컴포넌트 렌더링
    render(<Section heading="Basic" content="Hello world" />)

    // DOM 중에서 특정 text를 가진 요소를 조회
    expect(screen.getByText("Basic")).toBeInTheDocument();
    expect(screen.getByText("Hello world")).toBeInTheDocument();
  })
})
```

## 5.4 통합 테스트
- 하나의 기능단위를 넘어 기능의 흐름과 기능에 연관된 여러 요소들의 상호작용을 테스트
```javascript
describe('Terms and Conditions', () => {
  it("renders learn react link", () => {
    render(<TermsAndConditions />); // 컴포넌트 렌더링
    const button = screen.getByText('Next'); // Next 텍스트를 가진 버튼 조회
    expect(button).toBeDisabled(); // 버튼 비활성화 검사

    const checkbox = screen.getByRole('checkbox'); // checkbox 조회

    // 사용자 행동 시뮬레이션
    act(() => {
      userEvent.click(checkbox);
    })

    expect(button).toBeEnabled();
  })
})
```

## 5.5 Cypress를 이용한 E2E 테스트
