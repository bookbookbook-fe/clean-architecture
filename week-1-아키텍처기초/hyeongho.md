# 클린 아키텍처 스터디 - Week 1

| 장  | 제목                                                          |
| --- | ------------------------------------------------------------- |
| 1장 | [설계와 아키텍처란?](#1장-설계와-아키텍처란)                  |
| 2장 | [두 가지 가치에 대한 이야기](#2장-두-가지-가치에-대한-이야기) |
| 3장 | [패러다임 개요](#3장-패러다임-개요)                           |

## 1장: 설계와 아키텍처란?

### 설계와 아키텍처의 정의

**설계(Design)**와 **아키텍처(Architecture)**는 본질적으로 같은 개념이다.

- **저수준 세부사항**: 모듈, 컴포넌트 내부의 구체적인 구현 방식
- **고수준 구조**: 시스템 전반의 구조와 컴포넌트 간의 관계
- 둘은 연속된 직물의 일부이며, 고수준에서 저수준까지 매끄럽게 연결된다

### 소프트웨어 아키텍처의 목표

**"필요한 시스템을 만들고 유지보수하는 데 투입되는 인력을 최소화하는 것"**

- 단순히 동작하는 소프트웨어를 만드는 것이 아님
- **개발 생산성 유지**: 시간이 지나도 개발 속도가 느려지지 않도록
- **유지보수 비용 최소화**: 기능 추가나 버그 수정이 용이하도록

### 흔한 착각: "시장 출시가 먼저"

#### 🚫 잘못된 접근 방식

- "일단 빨리 만들고 나중에 정리하자"
- "코드 품질보다는 출시 일정이 우선"
- "기술 부채는 나중에 해결하면 돼"

#### 📉 이러한 접근의 결과

```
개발자 증가 → 생산성 감소
    ↓
더 많은 개발자 투입 → 더 큰 생산성 감소
    ↓
악순환 지속
```

#### ✅ 올바른 접근 방식

- **초기 투자**: 처음에는 시간이 더 걸릴 수 있음
- **장기적 이익**: 지속적인 개발 속도 유지
- **품질 우선**: TDD, 코드 리뷰, 리팩토링을 통한 코드 품질 관리

### 핵심 원칙: "빠른 유일한 방법은 제대로 가는 것"

이는 개발에서 가장 중요한 역설 중 하나이다:

- 단기적으로는 느려 보일 수 있음
- 중장기적으로는 압도적으로 빠름
- 기술 부채의 복리 효과를 피할 수 있음

### 결론

좋은 소프트웨어 아키텍처를 만들기 위해서는:

1. **비용 최소화**와 **생산성 최대화**를 동시에 달성
2. 시스템 아키텍처가 지닌 **근본 속성**에 대한 이해
3. 단순히 동작하는 것을 넘어서 **지속 가능한 시스템** 구축

## 2장: 두 가지 가치에 대한 이야기

### 소프트웨어의 두 가지 가치

모든 소프트웨어는 이해관계자에게 두 가지 서로 다른 가치를 제공한다:

#### 1️⃣ 행위(Behavior) - 첫 번째 가치

- **정의**: 요구사항에 명시된 기능을 구현하고 버그를 수정하는 것
- **특징**:
  - 사용자가 기능 명세를 정의하면, 개발자는 해당 기능이 동작하도록 코드를 작성
  - 기능이 잘못 동작하면 디버깅을 통해 수정
  - **긴급하지만 항상 중요한 것은 아님**

#### 2️⃣ 구조(Architecture) - 두 번째 가치

- **정의**: 소프트웨어를 "부드럽게(soft)" 유지하는 것
- **소프트웨어의 존재 이유**: **"기계를 쉽게 변경할 수 있게 하기 위해"**
- **핵심 원칙**: 소프트웨어는 변경하기 쉬워야 함

### 변경의 어려움과 아키텍처의 관계

**이상적인 아키텍처의 조건**:

```
변경의 어려움 = f(변경 범위) ≠ f(변경 형태)
개발 비용 ∝ 요청된 변경사항의 크기
```

#### 🔍 이 공식의 의미 자세히 분석

**좋은 아키텍처에서는**:

- **변경의 어려움**이 **변경 범위(scope)**에만 비례해야 함
- **변경 형태(shape)**와는 무관해야 함

**📝 구체적 예시로 이해하기**:

**변경 범위 (Scope)** - 영향받는 코드의 양:

- **작은 범위**: 한 개 함수 수정 → 쉬워야 함
- **중간 범위**: 한 개 클래스 수정 → 보통 어려움
- **큰 범위**: 여러 모듈 수정 → 어려워야 함 (당연함)

**변경 형태 (Shape)** - 변경사항의 종류:

- UI 색상 변경
- 새로운 결제 방식 추가
- 데이터베이스 테이블 구조 변경
- 새로운 비즈니스 규칙 추가

**❌ 나쁜 아키텍처의 예시**:

```javascript
// 강결합된 전자상거래 시스템
class OrderService {
  async createOrder(items, userInfo, paymentType) {
    // 주문 검증
    if (!this.validateItems(items)) {
      throw new Error("Invalid items");
    }

    // 결제 처리 (직접 구현)
    if (paymentType === "card") {
      // 신용카드 결제 로직 200줄
      const cardResponse = await fetch("/payment/card", {
        method: "POST",
        body: JSON.stringify({ amount: this.calculateTotal(items) }),
      });
      if (!cardResponse.ok) throw new Error("Card payment failed");
    } else if (paymentType === "bank") {
      // 계좌이체 로직 150줄
      const bankResponse = await fetch("/payment/bank", {
        method: "POST",
        body: JSON.stringify({ amount: this.calculateTotal(items) }),
      });
      if (!bankResponse.ok) throw new Error("Bank transfer failed");
    }

    // 재고 관리 (직접 구현)
    for (const item of items) {
      const inventory = await this.getInventory(item.id);
      if (inventory.stock < item.quantity) {
        throw new Error(`Insufficient stock for ${item.name}`);
      }
      await this.updateInventory(item.id, inventory.stock - item.quantity);
    }

    // 이메일 발송 (직접 구현)
    const emailHTML = `
      <h1>주문 확인</h1>
      <p>안녕하세요 ${userInfo.name}님,</p>
      <p>주문이 완료되었습니다.</p>
    `;
    await fetch("/email/send", {
      method: "POST",
      body: JSON.stringify({
        to: userInfo.email,
        subject: "주문 확인",
        html: emailHTML,
      }),
    });

    // 데이터베이스 저장 (직접 구현)
    const query = `
      INSERT INTO orders (user_id, items, total_amount, status, created_at) 
      VALUES (?, ?, ?, 'completed', NOW())
    `;
    await this.db.execute(query, [
      userInfo.id,
      JSON.stringify(items),
      this.calculateTotal(items),
    ]);
  }
}
```

**이 경우의 변경 어려움**:

- 새 결제 방식 추가 → OrderService 전체를 수정해야 함 (😱 어려움)
- 이메일 템플릿 변경 → OrderService를 수정해야 함 (😱 어려움)
- UI 색상 변경 → 다행히 영향 없음 (😌 쉬움)

**변경 형태에 따라 어려움이 천차만별!**

**✅ 좋은 아키텍처의 예시**:

```javascript
// 느슨하게 결합된 시스템
class OrderService {
  constructor(
    paymentProcessor,
    inventoryService,
    notificationService,
    orderRepository
  ) {
    this.paymentProcessor = paymentProcessor;
    this.inventoryService = inventoryService;
    this.notificationService = notificationService;
    this.orderRepository = orderRepository;
  }

  async createOrder(items, userInfo, paymentInfo) {
    // 주문 검증
    if (!this.validateItems(items)) {
      throw new Error("Invalid items");
    }

    // 각 서비스에 위임
    await this.paymentProcessor.process(paymentInfo);
    await this.inventoryService.reserveItems(items);
    await this.notificationService.sendOrderConfirmation(userInfo, items);

    const order = {
      userId: userInfo.id,
      items,
      totalAmount: this.calculateTotal(items),
      status: "completed",
    };

    return await this.orderRepository.save(order);
  }
}

// 각 서비스는 인터페이스를 통해 구현
class CardPaymentProcessor {
  async process(paymentInfo) {
    // 신용카드 결제만 처리
    return await fetch("/payment/card", {
      method: "POST",
      body: JSON.stringify(paymentInfo),
    });
  }
}

class EmailNotificationService {
  async sendOrderConfirmation(userInfo, items) {
    // 이메일 알림만 처리
    const emailHTML = this.generateOrderEmail(userInfo, items);
    return await this.sendEmail(userInfo.email, "주문 확인", emailHTML);
  }
}
```

**이 경우의 변경 어려움**:

- 새 결제 방식 추가 → PaymentProcessor 구현체만 추가 (😌 쉬움)
- 이메일 템플릿 변경 → NotificationService만 수정 (😌 쉬움)
- 새 알림 채널 추가 → NotificationService만 수정 (😌 쉬움)

**변경 형태와 무관하게 해당 영역만 수정하면 됨!**

**📊 비교표**:

| 변경 사항         | 나쁜 아키텍처 어려움 | 좋은 아키텍처 어려움 |
| ----------------- | -------------------- | -------------------- |
| 새 결제 방식 추가 | 😱 매우 어려움       | 😌 쉬움              |
| 이메일 → SMS 변경 | 😱 매우 어려움       | 😌 쉬움              |
| 새 재고 정책 추가 | 😡 어려움            | 😌 쉬움              |
| UI 색상 변경      | 😌 쉬움              | 😌 쉬움              |

**🎯 결론**:
좋은 아키텍처는 **"무엇을 변경하느냐(형태)"**가 아니라 **"얼마나 많이 변경하느냐(범위)"**에만 어려움이 비례한다.

#### "형태에 독립적인 아키텍처"란? 🤔

**📝 예제로 이해하기**:

**❌ 형태에 의존적인 설계**:

```javascript
// 결제 시스템 예시 - 잘못된 설계
class PaymentService {
  async processCreditCardPayment(amount, cardInfo) {
    // 신용카드 결제 로직
    return await fetch("/payment/card", {
      method: "POST",
      body: JSON.stringify({ amount, cardInfo }),
    });
  }

  async processBankTransfer(amount, bankInfo) {
    // 계좌이체 로직
    return await fetch("/payment/bank", {
      method: "POST",
      body: JSON.stringify({ amount, bankInfo }),
    });
  }

  async processPayPal(amount, paypalInfo) {
    // PayPal 결제 로직
    return await fetch("/payment/paypal", {
      method: "POST",
      body: JSON.stringify({ amount, paypalInfo }),
    });
  }
}
```

새로운 결제 방식(카카오페이, 네이버페이)을 추가할 때마다 PaymentService를 직접 수정해야 함

**✅ 형태에 독립적인 설계**:

```javascript
// 좋은 설계 - 인터페이스 기반
class PaymentService {
  constructor(paymentProcessor) {
    this.paymentProcessor = paymentProcessor;
  }

  async makePayment(amount, paymentInfo) {
    return await this.paymentProcessor.process(amount, paymentInfo);
  }
}

// 각 결제 방식별 구현체
class CreditCardProcessor {
  async process(amount, paymentInfo) {
    return await fetch("/payment/card", {
      method: "POST",
      body: JSON.stringify({ amount, cardInfo: paymentInfo }),
    });
  }
}

class KakaoPayProcessor {
  async process(amount, paymentInfo) {
    return await fetch("/payment/kakaopay", {
      method: "POST",
      body: JSON.stringify({ amount, kakaoInfo: paymentInfo }),
    });
  }
}
```

새로운 결제 방식을 추가해도 기존 코드는 전혀 수정할 필요 없음

### 완벽함 vs 유연성의 트레이드오프

**가치있는 통찰** 💡:

- **완벽하지만 수정 불가한 프로그램** vs **동작하지 않지만 수정 가능한 프로그램**
- **수정 가능한 프로그램이 더 좋은 프로그램**

**이유**:

- 완벽한 프로그램도 요구사항이 변하면 무용지물
- 수정 가능한 프로그램은 요구사항에 맞게 발전 가능

### 아이젠하워 매트릭스와 소프트웨어 가치

#### 📊 4가지 우선순위 분류

| 구분      | 긴급함 | 중요함 | 예시                                       |
| --------- | ------ | ------ | ------------------------------------------ |
| **1순위** | ✅     | ✅     | 긴급하고 중요한 기능 (보안 이슈 수정)      |
| **2순위** | ❌     | ✅     | 중요하지만 긴급하지 않은 것 (**아키텍처**) |
| **3순위** | ✅     | ❌     | 긴급하지만 중요하지 않은 것 (사소한 버그)  |
| **4순위** | ❌     | ❌     | 긴급하지도 중요하지도 않은 것              |

#### 🎯 핵심 통찰들

**Uncle Bob의 관찰**:

- **긴급한 문제가 아주 중요한 문제일 경우는 드물다**
- **중요한 문제가 몹시 긴급한 경우는 거의 없다**

**각 가치의 특성**:

- **행위(Behavior)**: 긴급하지만, 매번 높은 중요도를 가지는 것은 아님 → 1순위, 3순위
- **아키텍처(Architecture)**: 중요하지만, 즉각적인 긴급성은 없음 → **1순위, 2순위**

#### ⭐ 우선순위 분석

**아키텍처가 더 중요한 이유**:

```
아키텍처 = 1순위 + 2순위 (긴급 + 중요, 중요)
행위 = 1순위 + 3순위 (긴급 + 중요, 긴급)
```

**아키텍처가 행위보다 우선순위가 높은 두 영역을 차지**하므로, 장기적으로는 아키텍처에 더 집중해야 한다.

### 업무 관리자와 개발자의 흔한 실수

#### 🚨 문제 상황

**"3순위를 1순위로 착각하는 현상"**

- 긴급하지만 중요하지 않은 기능과 긴급하면서 중요한 기능을 구분하지 못함
- 결과: 중요도가 높은 아키텍처를 무시한 채 중요도가 떨어지는 기능을 우선시

#### 💡 해결책: 개발자의 역할

**소프트웨어 개발자를 채용하는 진짜 이유**:

- 기능의 중요성이 아닌 **아키텍처의 중요성을 업무 관리자에게 설득**하기 위해
- 소프트웨어를 안전하게 보호해야 할 책임

### 아키텍처 무시의 결과

**아키텍처가 후순위가 되면**:

- 시스템 개발 비용이 기하급수적으로 증가
- 일부 또는 전체 시스템 변경이 현실적으로 불가능해짐
- **기술 부채의 이자가 복리로 누적**

**개발자의 사명** 🎯:
업무 관리자와 협력하여 기능과 아키텍처의 균형을 유지하되, **아키텍처의 중요성을 끊임없이 옹호**해야 한다.

## 3장: 패러다임 개요

### 세 가지 프로그래밍 패러다임

프로그래밍 역사상 단 세 가지 패러다임이 등장했다.

#### 1️⃣ 구조적 프로그래밍 (Structured Programming)

- **정의**: 제어흐름의 직접적인 전환에 대해 규칙을 부과
- **핵심**: **goto문 사용을 금지**
- **대안**: if/then/else, do/while 등 구조화된 제어문 사용
- **목적**: 코드의 흐름을 예측 가능하고 이해하기 쉽게 만듦

**📝 JavaScript 예시**:

```javascript
// ❌ goto 방식 (구조적 프로그래밍 이전)
function processData(data) {
  let i = 0;
  // JavaScript는 goto를 지원하지 않지만 개념적 예시
  while (true) {
    if (i >= data.length) break; // goto end 역할
    if (data[i].isValid) {
      processItem(data[i]);
    }
    i++;
  }
  return "완료";
}

// ✅ 구조적 프로그래밍 방식
function processData(data) {
  for (let i = 0; i < data.length; i++) {
    if (data[i].isValid) {
      processItem(data[i]);
    }
  }
  return "완료";
}
```

#### 2️⃣ 객체지향 프로그래밍 (Object-Oriented Programming)

- **정의**: 제어흐름의 간접적인 전환에 대해 규칙을 부과
- **핵심**: **함수 포인터 사용을 제한**하고 다형성으로 대체
- **특징**: 캡슐화, 상속, 다형성을 통한 코드 구조화
- **목적**: 의존성 방향을 제어하여 플러그인 아키텍처 구현

**📝 JavaScript 예시**:

```javascript
// ❌ 함수 포인터 방식 (위험한 접근)
function processPayment(type, amount, processor) {
  // 함수를 직접 전달받아 호출 - 위험함
  return processor(amount);
}

// ✅ 객체지향 방식 (다형성 활용)
class PaymentService {
  constructor(paymentProcessor) {
    this.processor = paymentProcessor; // 의존성 주입
  }

  async processPayment(amount) {
    return await this.processor.process(amount); // 다형성
  }
}

class CreditCardProcessor {
  async process(amount) {
    return await this.processCreditCard(amount);
  }
}

class BankTransferProcessor {
  async process(amount) {
    return await this.processBankTransfer(amount);
  }
}
```

#### 3️⃣ 함수형 프로그래밍 (Functional Programming)

- **정의**: 할당문에 대해 규칙을 부과
- **핵심**: **변수 할당을 제한** (불변성 원칙)
- **특징**: 순수 함수, 불변 데이터, 함수 조합
- **목적**: 동시성 문제 해결 및 코드 예측 가능성 향상

**📝 JavaScript 예시**:

```javascript
// ❌ 명령형 방식 (할당문 남발)
function calculateTotal(items) {
  let total = 0;
  let taxRate = 0.1;

  for (let i = 0; i < items.length; i++) {
    total += items[i].price; // 변수 재할당
    if (items[i].category === "luxury") {
      taxRate = 0.2; // 변수 재할당
    }
  }

  total = total * (1 + taxRate); // 변수 재할당
  return total;
}

// ✅ 함수형 방식 (불변성 원칙)
const calculateTotal = (items) => {
  const baseTotal = items.reduce((sum, item) => sum + item.price, 0);
  const hasLuxuryItem = items.some((item) => item.category === "luxury");
  const taxRate = hasLuxuryItem ? 0.2 : 0.1;

  return baseTotal * (1 + taxRate);
};
```

### 🎯 패러다임의 본질: "제약을 통한 자유"

**핵심 통찰** 💡:
패러다임은 프로그래머에게 **권한을 박탈**한다. 무엇을 해야 할지 말하기보다는, **무엇을 하면 안 되는지**를 알려준다.

#### 각 패러다임이 박탈하는 것들:

| 패러다임     | 박탈하는 것 | 제공하는 대안     | 얻는 이익        |
| ------------ | ----------- | ----------------- | ---------------- |
| **구조적**   | goto문      | if/while/for      | 코드 흐름 명확화 |
| **객체지향** | 함수 포인터 | 다형성/인터페이스 | 의존성 제어      |
| **함수형**   | 할당문      | 불변성/순수함수   | 동시성 안전성    |

### 아키텍처와 패러다임의 연관성

#### 🏗️ 아키텍처의 세 가지 큰 관심사

**Uncle Bob의 핵심 관찰**:
세 패러다임은 아키텍처의 세 가지 큰 관심사와 직접적으로 연관된다.

1. **함수 (Function)** ← **구조적 프로그래밍**

   - 모듈의 내부 알고리즘 구조화
   - 복잡한 로직을 이해 가능한 구조로 분해

2. **컴포넌트 분리 (Component Separation)** ← **객체지향 프로그래밍**

   - 아키텍처 경계를 넘나들기 위한 메커니즘으로 **다형성** 사용
   - 플러그인 아키텍처 구현
   - 의존성 방향 제어

3. **데이터 관리 (Data Management)** ← **함수형 프로그래밍**
   - 동시성 문제 해결
   - 상태 변경으로 인한 버그 방지

#### 🔗 실제 아키텍처에서의 활용

**모던 웹 애플리케이션 예시**:

```javascript
// 1. 구조적 프로그래밍: 비즈니스 로직 구조화
class OrderValidator {
  validate(order) {
    if (!order) return false;
    if (!order.items || order.items.length === 0) return false;
    if (!order.customer) return false;

    // goto 없는 명확한 제어 흐름
    for (const item of order.items) {
      if (!this.validateItem(item)) return false;
    }

    return true;
  }
}

// 2. 객체지향: 컴포넌트 분리와 다형성
class OrderService {
  constructor(validator, paymentService, notificationService) {
    this.validator = validator;
    this.paymentService = paymentService; // 다형성으로 구현체 교체 가능
    this.notificationService = notificationService;
  }

  async processOrder(order) {
    if (!this.validator.validate(order)) {
      throw new Error("Invalid order");
    }

    // 아키텍처 경계를 넘나드는 다형성 활용
    await this.paymentService.process(order.payment);
    await this.notificationService.notify(order.customer);
  }
}

// 3. 함수형: 불변 데이터 관리
const orderReducer = (state = initialState, action) => {
  switch (action.type) {
    case "ADD_ITEM":
      return {
        ...state, // 기존 상태를 변경하지 않고 새 객체 생성
        items: [...state.items, action.payload],
      };
    case "REMOVE_ITEM":
      return {
        ...state,
        items: state.items.filter((item) => item.id !== action.payload.id),
      };
    default:
      return state; // 상태 변경 없음
  }
};
```

### 🚀 패러다임의 미래

**더 이상 새로운 패러다임은 등장하지 않을 것**이라는 Uncle Bob의 예측:

**이유**:

1. **goto 제거** → 구조적 프로그래밍
2. **함수 포인터 제한** → 객체지향 프로그래밍
3. **할당문 제한** → 함수형 프로그래밍

이 세 가지가 프로그래밍에서 제거할 수 있는 **유일한 부정적 요소들**이며, 더 이상 제거할 것이 없다.

### 결론

**패러다임의 진정한 가치**:

- 기술적 혁신이 아닌 **제약을 통한 질서 확립**
- 각각이 아키텍처의 핵심 관심사와 1:1 대응
- **함수, 컴포넌트 분리, 데이터 관리**라는 아키텍처의 영원한 과제 해결

**개발자의 숙제** 📚:
이 세 패러다임을 **조화롭게 결합**하여 견고하고 유연한 소프트웨어 아키텍처를 구축하는 것이다.
