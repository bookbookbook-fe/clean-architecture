# 클린 아키텍처 4~6장 정리

> 📖 = 책 내용 | 💭 = 개인 생각

## 📋 목차

- [2부: 벽돌부터 시작하기: 프로그래밍 패러다임](https://github.com/bookbookbook-fe/clean-architecture/blob/main/week-1-%EC%95%84%ED%82%A4%ED%85%8D%EC%B2%98%EA%B8%B0%EC%B4%88/seyeong.md#2%EB%B6%80-%EB%B2%BD%EB%8F%8C%EB%B6%80%ED%84%B0-%EC%8B%9C%EC%9E%91%ED%95%98%EA%B8%B0-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D-%ED%8C%A8%EB%9F%AC%EB%8B%A4%EC%9E%84)
  - [4장: 구조적 프로그래밍](#4장-구조적-프로그래밍)
  - [5장: 객체지향 프로그래밍](#5장-객체-지향-프로그래밍)
  - [6장: 함수형 프로그래밍](#6장-함수형-프로그래밍)
- [정리 및 다음 학습 방향](#-정리-및-다음-학습-방향)

---

## 4장: 구조적 프로그래밍

### 배경과 문제의식

- **문제**: 프로그램의 작은 세부사항이라도 간과하면 예상 외의 방식으로 실패하는 문제를 해결하려고 함
- **해결 접근**: 수학적 증명의 원리를 적용하여 프로그램의 정확성을 보장하고자 함

### 유클리드 계층구조와 goto문의 문제

- **유클리드 계층구조**: 입증된 구조를 사용하여 코드의 올바름을 수학적으로 증명하는 체계
- **goto문의 문제점**: 분할정복 접근법을 통한 합리적 증명을 방해함
  - 임의의 위치로 점프할 수 있어 프로그램의 흐름을 예측하기 어려움
  - 코드의 논리적 구조를 파괴하여 증명이 불가능해짐

### 구조적 프로그래밍의 핵심: 3가지 제어 구조

**순차(Sequence), 분기(Selection), 반복(Iteration)**만으로 모든 프로그램 표현 가능

1. **순차**:

   - 단순 열거법으로 입증
   - 각 구문의 입력과 출력을 수학적으로 추적 가능

2. **분기**:

   - 열거법 재적용
   - 모든 경로를 열거하고 각 경로가 올바른 결과를 만드는지 증명

3. **반복**:
   - 귀납법 사용
   - 기본 케이스(1)가 올바름을 증명하고, N이 올바르다면 N+1도 올바름을 증명

### 제어 흐름의 변화

- **과거 (포트란, 코볼)**: 제약 없이 직접 제어 흐름 전환 가능 (goto문)
- **현재**: 구조적 프로그래밍으로 제어 흐름을 예측 가능하게 제한

### 구조적 프로그래밍의 의의

- **기능적 분해**: 모듈을 증명 가능한 더 작은 단위로 재귀적 분해
- **계층적 설계**: 고수준 기능 → 저수준 함수로 끝없이 분해 가능
- **대규모 시스템 관리**: 모듈과 컴포넌트를 입증 가능한 작은 기능으로 세분화

### 수학적 증명에서 과학적 방법으로의 전환

**수학적 증명의 한계**

- 프로그램에서 유클리드 계층구조는 실제로 만들어지지 못함
- 엄밀한 증명이 고품질 소프트웨어 생산에 적절하지 않음이 판명

**과학적 방법의 채택**

- **수학**: 참임을 증명 (증명 가능)
- **과학**: 거짓임을 증명 (반증 가능하지만 증명 불가)
- 소프트웨어 개발은 수학보다는 **과학에 가까움**

### 테스팅과 구조적 프로그래밍

- **테스트의 한계**: "버그가 있음을 보여줄 뿐, 버그가 없음을 보여줄 수는 없다"
- **과학적 접근**: 올바르지 않음을 증명하는데 실패함으로써 충분히 참이라고 여김

### 구조적 프로그래밍의 핵심 가치

- **반증 가능한 단위 생성**: 테스트를 통해 프로그램의 올바름을 검증할 수 있는 단위로 분해
- **증명 가능한 세부 기능**: 각 기능을 독립적으로 테스트하여 거짓임을 증명하려 시도
- **현대 언어의 기반**: goto문을 제거하고 구조적 제어만 지원하는 이유
- **아키텍처 원칙**: 기능적 분해가 최고의 실천법이 되는 근거 제공

## 5장: 객체 지향 프로그래밍

### 개요

객체지향의 전통적인 3요소: **캡슐화, 상속, 다형성**
하지만 이 세 가지가 정말 객체지향의 본질일까?

### 1. 캡슐화 (Encapsulation)

#### 캡슐화란?

- 응집력 있게 집단을 구성하고 다른 집단과 구분선을 그음
- 구분선 바깥에서 데이터는 은닉되고, 일부 함수만 외부에 노출

#### C언어에서의 완벽한 캡슐화

```c
// point.h
struct Point;
struct Point* makePoint(double x, double y);
double distance(struct Point *p1, struct Point *p2);
```

```c
// point.c
#include "point.h"
#include <stdlib.h>
#include <math.h>

struct Point {
    double x, y;
};

struct Point* makePoint(double x, double y) {
    struct Point* p = malloc(sizeof(struct Point));
    p->x = x;
    p->y = y;
    return p;
}

double distance(struct Point *p1, struct Point *p2) {
    double dx = p1->x - p2->x;
    double dy = p1->y - p2->y;
    return sqrt(dx*dx + dy*dy);
}
```

**완벽한 캡슐화**: 사용자는 `makePoint()`, `distance()` 함수만 호출 가능하고, Point 구조의 내부 구현은 전혀 알 수 없음

#### 객체지향 언어에서의 캡슐화 약화

```cpp
// point.h (C++)
class Point {
public:
    Point(double x, double y);
    double distance(const Point& p) const;
private:
    double x;  // 헤더에 노출됨!
    double y;  // 헤더에 노출됨!
};
```

**문제점**:

- 컴파일러가 클래스 크기를 알아야 하므로 멤버변수를 헤더에 선언해야 함
- 사용자가 멤버변수 존재를 알게 됨
- 멤버변수 이름이 바뀌면 재컴파일 필요 → 캡슐화 깨짐

#### 왜 헤더 노출이 캡슐화를 깨뜨리는가?

**C언어의 진정한 캡슐화**:

```c
// 사용자는 Point가 어떻게 구현되었는지 전혀 모름
struct Point;  // 전방 선언만 있음
struct Point* makePoint(double x, double y);
```

**C++의 노출된 구조**:

```cpp
// 사용자가 내부 구조를 모두 볼 수 있음
class Point {
private:
    double x;  // 이 변수가 존재한다는 것을 알 수 있음
    double y;  // 이 변수가 존재한다는 것을 알 수 있음
};
```

**캡슐화가 깨지는 구체적 상황들**:

1. **구현 세부사항 노출**:

   ```cpp
   // 개발자가 내부 구현을 추측할 수 있음
   // "아, Point는 x, y 좌표로 구현되어 있구나"
   // "double을 사용하는구나"
   ```

2. **컴파일 의존성 발생**:

   ```cpp
   // Point.h가 변경되면
   class Point {
   private:
       double x, y;
       string name;  // 새 멤버 추가
   };

   // 이 클래스를 사용하는 모든 파일이 재컴파일됨
   // C언어였다면 Point.c만 재컴파일하면 됐을 것
   ```

3. **진정한 데이터 은닉 불가능**:

   ```c
   // C언어 방식 - 진짜 숨김
   // point.c에서만 구조 정의
   struct Point {
       double x, y;        // 또는
       float radius, angle; // 극좌표 방식으로 바꿔도
   };                      // 사용자는 전혀 모름

   // C++ 방식 - 노출됨
   class Point {
   private:
       double x, y;  // 사용자가 직교좌표계 사용을 알 수 있음
   };
   ```

#### "볼 수 있다" vs "봐야만 한다"의 차이

**C언어의 캡슐화**:

```c
// 개발자가 Point를 사용할 때
#include "point.h"  // 여기에는 구현 정보가 없음

struct Point* p = makePoint(1.0, 2.0);  // 사용 가능
// 내부가 어떻게 되어있는지 몰라도 사용 가능
// 궁금하면 point.c를 "의도적으로" 열어봐야 함
```

**C++의 노출된 구조**:

```cpp
// 개발자가 Point를 사용할 때
#include "Point.h"  // 여기에 구현 정보가 강제로 노출됨

class Point {
private:
    double x, y;  // 원하든 원하지 않든 보게 됨
};

Point p(1.0, 2.0);  // 사용하려면 무조건 위 정보를 봐야 함
```

#### 진짜 문제: 컴파일러 의존성

**C언어**:

```c
// point.c 내부를 이렇게 바꿔도
struct Point {
    float radius, angle;  // 극좌표로 변경
};

// 클라이언트 코드는 재컴파일 불필요!
// point.h는 변경되지 않았기 때문
```

**C++**:

```cpp
// Point.h를 이렇게 바꾸면
class Point {
private:
    float radius, angle;  // 극좌표로 변경
};

// 이 헤더를 include한 모든 파일이 재컴파일 필요!
// private이어도 헤더에 있으면 컴파일러가 알아야 함
```

#### 라이브러리 배포 관점에서의 문제

**C언어 라이브러리**:

```c
// point.h (배포용 헤더)
struct Point;  // 사용자는 내부 구조를 모름
struct Point* makePoint(double x, double y);

// point.so (컴파일된 라이브러리)
// 사용자는 헤더만 보고 라이브러리를 사용
// 내부 구현이 바뀌어도 사용자 코드는 영향받지 않음
```

**C++ 라이브러리**:

```cpp
// Point.h (배포용 헤더)
class Point {
private:
    double x, y;  // 사용자가 보게 됨
public:
    Point(double x, double y);
};

// 내부 구현이 바뀌면 헤더가 바뀜
// → 사용자가 재컴파일해야 함
// → 라이브러리 업데이트가 어려워짐
```

**결론**: C언어는 "**의도적으로 찾아봐야 알 수 있는**" 캡슐화를 제공하지만, C++은 "**사용하려면 반드시 봐야 하는**" 구조로, 진정한 정보 은닉과 컴파일 독립성을 제공하지 못함

### 2. 상속 (Inheritance)

#### 상속이란?

변수와 함수를 하나의 유효범위로 묶어서 재정의하는 것

#### C언어에서의 상속 흉내내기

```c
// namedPoint.h
struct NamedPoint;
struct NamedPoint* makeNamedPoint(double x, double y, char* name);
void setName(struct NamedPoint* np, char* name);
char* getName(struct NamedPoint* np);
```

```c
// namedPoint.c
#include "namedPoint.h"
#include "point.h"

struct NamedPoint {
    double x, y;  // Point와 동일한 순서
    char* name;
};

struct NamedPoint* makeNamedPoint(double x, double y, char* name) {
    struct NamedPoint* np = malloc(sizeof(struct NamedPoint));
    np->x = x;
    np->y = y;
    np->name = name;
    return np;
}
```

```c
// main.c
int main() {
    struct NamedPoint* origin = makeNamedPoint(0.0, 0.0, "origin");
    struct NamedPoint* upperRight = makeNamedPoint(1.0, 1.0, "upperRight");

    // Point로 타입 캐스팅하여 사용
    double dist = distance((struct Point*)origin, (struct Point*)upperRight);

    return 0;
}
```

**핵심**: NamedPoint가 Point의 "가면"을 쓰고 동작할 수 있음 (멤버변수 순서 유지)

#### C언어의 수동 타입 캐스팅 vs 객체지향의 자동 업캐스팅

**C언어에서는 수동 캐스팅 필요**:

```c
struct NamedPoint* np = makeNamedPoint(0.0, 0.0, "origin");
// 명시적으로 타입 캐스팅해야 함
double dist = distance((struct Point*)np, (struct Point*)upperRight);
//                    ^^^^^^^^^^^^^^^^  위험한 수동 캐스팅
```

**문제점**:

- 개발자가 캐스팅을 깜빡할 수 있음
- 잘못된 타입으로 캐스팅할 위험
- 컴파일러가 타입 안전성을 보장하지 못함

**객체지향에서는 자동 업캐스팅**:

```cpp
class Point {
public:
    Point(double x, double y);
    double distance(const Point& other);
};

class NamedPoint : public Point {
    string name;
public:
    NamedPoint(double x, double y, string name) : Point(x, y), name(name) {}
};

// 사용 시
NamedPoint origin(0.0, 0.0, "origin");
NamedPoint upperRight(1.0, 1.0, "upperRight");

// 자동으로 Point로 업캐스팅됨 - 안전하고 편리!
double dist = origin.distance(upperRight);  // 명시적 캐스팅 불필요
```

#### 객체지향이 제공하는 편의성

1. **타입 안전성**: 컴파일러가 상속 관계를 검증
2. **자동 변환**: 명시적 캐스팅 없이도 부모 타입으로 사용 가능
3. **실수 방지**: 잘못된 타입 캐스팅으로 인한 버그 예방

```cpp
// 다형성과 함께 사용할 때 더욱 강력
vector<Point*> points;
points.push_back(new Point(1.0, 2.0));        // 일반 Point
points.push_back(new NamedPoint(3.0, 4.0, "A")); // 자동 업캐스팅

// 모든 점들을 Point로 다룰 수 있음
for(Point* p : points) {
    // p->someMethod(); // Point의 메서드 호출 가능
}
```

**결론**: 상속은 객체지향이 새로 만든 개념이 아니지만, **자동 업캐스팅과 타입 안전성**을 통해 데이터 구조에 가면을 씌우는 일을 훨씬 안전하고 편리하게 만들어줌

### 3. 다형성 (Polymorphism)

#### 다형성이란?

동일한 인터페이스를 통해 다른 구현체들을 호출할 수 있는 능력

#### C언어에서의 다형성

```c
#include <stdio.h>

void copy() {
    int c;
    while ((c = getchar()) != EOF)
        putchar(c);
}
```

**다형성의 예**:

- `getchar()`와 `putchar()`는 실제로는 다양한 장치(키보드, 파일, 네트워크 등)에서 동작
- STDIN, STDOUT이 어떤 장치인지에 따라 다른 동작 수행

#### Unix에서의 구현 방식

```c
struct FILE {
    int (*read)(void);
    int (*write)(int);
    int (*open)(char*);
    int (*close)(void);
    int (*seek)(int);
};

extern struct FILE* STDIN;

int getchar() {
    return STDIN->read();  // 함수 포인터를 통한 호출
}
```

**핵심**: 함수 포인터를 사용한 다형적 행위 - 이것이 객체지향 다형성의 근간

### 의존성 역전 (Dependency Inversion) - 객체지향의 진짜 힘

#### 전통적인 의존성

```
main → getchar → 키보드 드라이버
```

소스코드 의존성 방향 = 제어 흐름 방향

#### 객체지향에서의 의존성 역전

```
main → InputStream 인터페이스 ← KeyboardDriver
```

제어 흐름: main → KeyboardDriver
소스코드 의존성: KeyboardDriver → InputStream 인터페이스

#### 실제 예시: 플러그인 아키텍처

```
업무규칙 (고수준 정책)
    ↑         ↑
    |         |
   UI    데이터베이스
(저수준)   (저수준)
```

**혜택**:

- **배포 독립성**: 각 컴포넌트를 독립적으로 배포 가능
- **개발 독립성**: 팀별로 독립적 개발 가능
- **테스트 용이성**: 의존성을 쉽게 모킹 가능

### 객체지향의 본질

> **소프트웨어 아키텍처 관점에서 객체지향이란**
> 다형성을 이용해 전체 시스템의 모든 소스코드 의존성에 대한 절대적인 제어 권한을 획득할 수 있는 능력

**핵심 가치**:

- 플러그인 아키텍처 구성 가능
- 고수준 정책과 저수준 세부사항의 분리
- 저수준 모듈을 플러그인으로 만들어 독립적 개발/배포 가능

## 6장: 함수형 프로그래밍

### 배경과 개념

#### 함수형 프로그래밍의 탄생

- **역사**: 프로그래밍 자체보다 앞서 등장 (1930년대 알론조 처치의 λ-계산법)
- **핵심 원칙**: 변수는 변경되지 않는다 (불변성, Immutability)
- **대표 언어**: Clojure, Haskell, Erlang 등

#### 가변성 vs 불변성

**전통적인 언어 (Java, C++ 등)**:

```java
int x = 5;
x = x + 1;  // 변수 x의 값이 변경됨
```

**함수형 언어 (Clojure)**:

```clojure
(def x 5)    ; x는 5로 영원히 고정됨
; x를 "변경"하는 대신 새로운 값을 생성
(def y (+ x 1))  ; x는 그대로, y는 6
```

### 불변성이 아키텍처에 미치는 영향

#### 동시성 문제의 근본 원인

모든 동시성 문제는 **가변변수** 때문에 발생:

- **경합 조건 (Race Condition)**: 두 스레드가 같은 변수를 동시 수정
- **교착상태 (Deadlock)**: 락이 서로를 기다리는 상황
- **동시 업데이트**: 한 스레드가 수정 중인 데이터를 다른 스레드가 읽는 문제

#### 구체적인 동시성 문제 예시

**문제가 있는 코드 (가변 상태)**:

```javascript
// 공유 상태 - 문제의 원인
let counter = 0;

function increment() {
  counter = counter + 1; // 여러 스레드가 동시 접근시 문제 발생
}

// Thread 1과 Thread 2가 동시에 increment() 호출
// 예상: counter = 2
// 실제: counter = 1 (Race Condition 발생)
```

**불변성으로 해결한 코드**:

```javascript
// 불변 접근법 - 새로운 상태 생성
const createCounter = (currentValue) => ({
  value: currentValue,
  increment: () => createCounter(currentValue + 1),
});

const counter1 = createCounter(0);
const counter2 = counter1.increment(); // 새로운 객체 반환
const counter3 = counter2.increment(); // 기존 객체는 변경 안됨

// 각 스레드가 독립적인 상태를 가짐 → 동시성 문제 없음
```

#### 불변성의 한계와 현실적 제약

**이상적 조건**:

- 무한한 저장 공간
- 무한한 처리 속도

**현실적 제약**:

- 메모리 한계로 인한 성능 문제
- 처리 속도의 오버헤드
- 따라서 **타협**이 필요

### 현실적 해결책: 타협 전략

#### 1. 가변성의 분리 (Segregation of Mutability)

애플리케이션을 **불변 컴포넌트**와 **가변 컴포넌트**로 분리:

```
┌─────────────────────────────────────┐
│         불변 컴포넌트                │
│    (순수 함수형 처리)               │
│    - 비즈니스 로직                   │
│    - 데이터 변환                     │
│    - 계산 로직                       │
└─────────────┬───────────────────────┘
              │
              │ 통신
              │
┌─────────────▼───────────────────────┐
│         가변 컴포넌트                │
│    (상태 변경 담당)                 │
│    - 상태 저장                       │
│    - I/O 처리                       │
│    - 외부 시스템 연동                │
└─────────────────────────────────────┘
```

#### React에서의 가변성 분리 예시

**좋은 예시 - 가변성이 분리된 구조**:

```jsx
// 불변 컴포넌트 - 순수 함수형 계산 로직
const calculateTotal = (items) => {
  return items.reduce((sum, item) => sum + item.price * item.quantity, 0);
};

const applyDiscount = (total, discountRate) => {
  return total * (1 - discountRate);
};

const formatPrice = (price) => {
  return new Intl.NumberFormat("ko-KR", {
    style: "currency",
    currency: "KRW",
  }).format(price);
};

// 가변 컴포넌트 - 상태 관리만 담당
function ShoppingCart() {
  const [items, setItems] = useState([]); // 가변 상태
  const [discountRate, setDiscountRate] = useState(0);

  // 순수 함수들을 조합하여 사용
  const total = calculateTotal(items);
  const discountedTotal = applyDiscount(total, discountRate);
  const formattedPrice = formatPrice(discountedTotal);

  const addItem = (newItem) => {
    // 불변성 유지하며 상태 업데이트
    setItems((prevItems) => [...prevItems, newItem]);
  };

  return (
    <div>
      {items.map((item) => (
        <CartItem key={item.id} item={item} />
      ))}
      <div>Total: {formattedPrice}</div>
    </div>
  );
}
```

**문제가 있는 예시 - 가변성이 섞인 구조**:

```jsx
// 안티패턴: 비즈니스 로직과 상태 변경이 섞임
function ShoppingCart() {
  const [items, setItems] = useState([]);
  const [total, setTotal] = useState(0);

  const addItem = (newItem) => {
    // 상태 직접 변경 + 계산 로직이 섞임 (가변성 분리 실패)
    items.push(newItem); // 직접 변경 - 문제!

    let newTotal = 0;
    for (let item of items) {
      // 계산 로직이 상태 변경과 섞임
      newTotal += item.price * item.quantity;
    }
    setTotal(newTotal);
  };

  return (
    <div>
      {items.map((item) => (
        <CartItem key={item.id} item={item} />
      ))}
      <div>Total: {total}</div>
    </div>
  );
}
```

#### 프론트엔드에서 가변성 분리의 실제 적용

**상태 관리 패턴 (Redux)**:

```javascript
// 불변 컴포넌트 - 순수 함수인 리듀서
const todosReducer = (state = [], action) => {
  switch (action.type) {
    case "ADD_TODO":
      return [...state, action.todo]; // 새 배열 생성 (불변)
    case "TOGGLE_TODO":
      return state.map((todo) =>
        todo.id === action.id
          ? { ...todo, completed: !todo.completed } // 새 객체 생성
          : todo
      );
    default:
      return state;
  }
};

// 가변 컴포넌트 - 상태 저장소
const store = createStore(todosReducer);
```

#### 2. 이벤트 소싱 (Event Sourcing)

**기본 아이디어**: 상태 변경 대신 **변경 이벤트**를 저장

**전통적 방식 (상태 저장)**:

```javascript
// 계좌 상태를 직접 저장
const account = {
  id: "account-123",
  balance: 1500, // 현재 잔고만 저장
};

// 입금시 기존 상태 덮어씀
account.balance += 500; // 이전 정보 손실
```

**이벤트 소싱 방식 (이벤트 저장)**:

```javascript
// 이벤트들을 저장 (불변)
const events = [
  { type: "ACCOUNT_OPENED", accountId: "account-123", initialBalance: 1000 },
  { type: "DEPOSIT", accountId: "account-123", amount: 500 },
  { type: "WITHDRAWAL", accountId: "account-123", amount: 200 },
  { type: "DEPOSIT", accountId: "account-123", amount: 200 },
];

// 현재 상태가 필요할 때 이벤트들을 재계산 (순수 함수)
const calculateBalance = (events, accountId) => {
  return events
    .filter((event) => event.accountId === accountId)
    .reduce((balance, event) => {
      switch (event.type) {
        case "ACCOUNT_OPENED":
          return event.initialBalance;
        case "DEPOSIT":
          return balance + event.amount;
        case "WITHDRAWAL":
          return balance - event.amount;
        default:
          return balance;
      }
    }, 0);
};

const currentBalance = calculateBalance(events, "account-123"); // 1500
```

#### 이벤트 소싱의 장점

1. **완전한 불변성**: 이벤트는 한번 생성되면 절대 변경되지 않음
2. **CRUD → CR**: Delete와 Update가 없음, Create와 Read만 존재
3. **동시성 문제 해결**: 이벤트 append만 하므로 Lock 불필요
4. **완전한 이력 추적**: 모든 변경 사항의 완전한 기록

#### 프론트엔드에서의 이벤트 소싱 예시

**상태 관리 with 이벤트 소싱**:

```javascript
// 이벤트 저장
const userEvents = [];

// 이벤트 생성 함수들 (불변)
const createLoginEvent = (userId, timestamp) => ({
  type: "USER_LOGGED_IN",
  userId,
  timestamp,
});

const createProfileUpdateEvent = (userId, updates, timestamp) => ({
  type: "PROFILE_UPDATED",
  userId,
  updates,
  timestamp,
});

// 현재 상태 계산 (순수 함수)
const calculateUserState = (events, userId) => {
  return events
    .filter((event) => event.userId === userId)
    .reduce(
      (state, event) => {
        switch (event.type) {
          case "USER_LOGGED_IN":
            return { ...state, isLoggedIn: true, lastLogin: event.timestamp };
          case "PROFILE_UPDATED":
            return {
              ...state,
              profile: { ...state.profile, ...event.updates },
            };
          default:
            return state;
        }
      },
      { isLoggedIn: false, profile: {}, lastLogin: null }
    );
};

// 사용 예시
userEvents.push(createLoginEvent("user-123", Date.now()));
userEvents.push(
  createProfileUpdateEvent("user-123", { name: "John" }, Date.now())
);

const currentUserState = calculateUserState(userEvents, "user-123");
```

### 함수형 프로그래밍의 의의

#### 아키텍처 관점에서의 가치

1. **동시성 안전성**: 불변 데이터는 여러 스레드가 안전하게 공유 가능
2. **예측 가능성**: 순수 함수는 동일한 입력에 항상 동일한 출력 보장
3. **테스트 용이성**: 부작용이 없는 함수는 테스트하기 쉬움
4. **디버깅 편의성**: 상태 변경 추적이 명확함

#### 현실적 적용 원칙

1. **핵심 비즈니스 로직**: 가능한 한 순수 함수로 작성
2. **상태 변경 최소화**: 가변 상태를 애플리케이션 경계로 밀어내기
3. **이벤트 기반 설계**: 상태 변경을 이벤트로 모델링
4. **불변 데이터 구조 활용**: Immutable.js, Immer 등의 라이브러리 사용

> **💭 개인 생각**:
>
> 함수형 프로그래밍의 핵심은 "데이터/계산/액션" 분리인 것 같네요
>
> - **데이터와 계산의 범위를 늘리고** (불변 컴포넌트)
> - **액션의 범위를 줄이는 것** (가변 컴포넌트)

---

## 🔖 정리 및 다음 학습 방향

### 핵심 포인트

1. 구조적 프로그래밍은 제어흐름의 직접적인 전환에 부과되는 규율
2. 객체지향 프로그래밍은 제어흐름의 간접적인 전환에 부과되는 규율
3. 함수형 프로그래밍은 변수 할당에 부과되는 규율

세 프래러다임은 우리에게서 무엇인가를 뺏어감
그래서 우리가 코드를 작성하는 방식의 형태를 한정시키고, 어떤 패러다임도 우리의 권한이나 능력에 자유로움이나 보탬을 주진 않음,

우린 지난 반세기동안 "해서는 안되는 것"에 대해 배움.
도구는 달라졌고 하드웨어도 변했지만, "소프트웨어"의 핵심은 여전히 순차, 분기, 반복, 참조로 구성된다.
그 이상도 이하도 아니다.
