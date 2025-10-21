# 🧹 클린 아키텍처 스터디 5주차 (컴포넌트) 요약

---

## 컴포넌트 응집도 원칙 (Component Cohesion Principles)

### 1. 재사용/릴리스 등가 원칙 (REP: Reuse/Release Equivalence Principle)

> "재사용 단위는 릴리스 단위와 같다."

- **핵심**

  - 컴포넌트는 **재사용 가능한 단위**이자 **릴리스 가능한 단위**여야 한다.
  - 버전 관리, 문서화, 의존성 관리가 가능한 수준으로 응집되어야 한다.
  - 재사용자들이 **의존할 수 있는 안정된 인터페이스**를 제공해야 한다.

- **실무 적용**
  - npm 패키지, 라이브러리, 모듈 단위로 배포 가능한 크기로 설계.
  - 시맨틱 버저닝을 통한 호환성 관리.
  - API 문서화와 변경 로그 유지.

### 2. 공통 폐쇄 원칙 (CCP: Common Closure Principle)

> "같은 이유로 변경되는 클래스들은 같은 컴포넌트에 속해야 한다."

- **핵심**

  - **변경의 축**이 같은 것들끼리 묶어서 변경의 파급효과를 최소화.
  - 단일 책임 원칙의 컴포넌트 버전.
  - 하나의 변경 요구사항이 **하나의 컴포넌트**만 수정되도록 설계.

- **냄새**
  - 하나의 변경으로 여러 컴포넌트를 동시에 수정해야 함.
  - 관련 없는 기능들이 한 컴포넌트에 섞여 있음.
  - 변경 시 예상치 못한 사이드 이펙트 발생.

### 3. 공통 재사용 원칙 (CRP: Common Reuse Principle)

> "함께 재사용되는 클래스들은 같은 컴포넌트에 속해야 한다."

- **핵심**

  - **재사용 단위**를 기준으로 컴포넌트를 구성.
  - 불필요한 의존성을 피하기 위해 함께 쓰이지 않는 것들은 분리.
  - 클라이언트가 **꼭 필요한 것만** 의존하도록 설계.

- **실무 팁**
  - UI 컴포넌트 라이브러리에서 독립적으로 사용 가능한 컴포넌트들 분리.
  - 유틸리티 함수들을 기능별로 그룹화.
  - 도메인별 서비스 레이어 분리.

---

## 컴포넌트 결합도 원칙 (Component Coupling Principles)

### 1. 비순환 의존성 원칙 (ADP: Acyclic Dependencies Principle)

> "컴포넌트 의존성 그래프에 순환이 있어서는 안 된다."

- **핵심**

  - **순환 의존성**은 빌드, 테스트, 배포를 복잡하게 만든다.
  - 변경의 파급효과가 순환적으로 전파되어 예측하기 어려워진다.
  - 의존성 그래프는 **DAG(방향성 비순환 그래프)** 형태여야 한다.

- **해결 방법**
  - **의존성 역전**: 공통 인터페이스를 통해 순환을 끊는다.
  - **이벤트 기반**: 직접 의존 대신 이벤트/메시지로 통신.
  - **팩토리 패턴**: 생성 책임을 중앙으로 집중.

### 2. 안정된 의존성 원칙 (SDP: Stable Dependencies Principle)

> "안정된 컴포넌트는 불안정한 컴포넌트에 의존하면 안 된다."

- **핵심**

  - **안정성 = 변경 빈도가 낮음**
  - 안정된 컴포넌트는 불안정한 컴포넌트의 변경에 영향받지 않아야 한다.
  - 의존성 방향은 **안정된 것 → 불안정한 것**으로 흘러야 한다.

- **안정성 측정**
  - **Fan-in**: 이 컴포넌트를 의존하는 컴포넌트 수 (높을수록 안정)
  - **Fan-out**: 이 컴포넌트가 의존하는 컴포넌트 수 (낮을수록 안정)
  - **불안정도 = Fan-out / (Fan-in + Fan-out)**

### 3. 안정된 추상화 원칙 (SAP: Stable Abstractions Principle)

> "안정된 컴포넌트는 추상화되어야 한다."

- **핵심**

  - 안정된 컴포넌트는 **인터페이스/추상 클래스**로 구성되어야 한다.
  - 구체적인 구현은 불안정한 컴포넌트에 두고, 안정된 것은 추상화.
  - **의존성 역전**과 **인터페이스 분리**의 조합.

- **추상화 정도 측정**
  - **추상성 = 추상 클래스와 인터페이스 수 / 전체 클래스 수**
  - **주요 시퀀스**: 안정성과 추상성이 비례하는 선상에 위치해야 함.

---

## 실전 적용: 컴포넌트 설계 전략

### 응집도 vs 결합도의 균형

- **초기 단계**: CCP 우선 (변경의 축에 따라 묶기)
- **성장 단계**: REP 도입 (재사용 단위 고려)
- **성숙 단계**: CRP 적용 (불필요한 의존성 제거)

### 마이크로서비스와의 연관성

- **서비스 경계 = 컴포넌트 경계**
- **API 버전 관리 = 릴리스 등가**
- **이벤트 기반 통신 = 비순환 의존성**

### 모노레포에서의 적용

- **패키지 구조**: 도메인별/기능별 패키지 분리
- **의존성 관리**: 내부 패키지 간 의존성 그래프 관리
- **공유 라이브러리**: 안정된 추상화를 통한 공통 기능 제공

---

## React/TypeScript 예시: 컴포넌트 설계

### 1. 응집도 원칙 적용

```ts
// ❌ 나쁜 예: 여러 책임이 섞인 컴포넌트
export class UserService {
  async getUser(id: string) {
    /* ... */
  }
  async updateUser(user: User) {
    /* ... */
  }
  async sendEmail(to: string, subject: string) {
    /* ... */
  } // 다른 책임
  async logActivity(action: string) {
    /* ... */
  } // 다른 책임
}

// ✅ 좋은 예: 단일 책임으로 분리
export class UserRepository {
  async getUser(id: string): Promise<User> {
    /* ... */
  }
  async updateUser(user: User): Promise<void> {
    /* ... */
  }
}

export class EmailService {
  async sendEmail(to: string, subject: string): Promise<void> {
    /* ... */
  }
}

export class ActivityLogger {
  async logActivity(action: string): Promise<void> {
    /* ... */
  }
}
```

### 2. 결합도 원칙 적용

```ts
// ❌ 나쁜 예: 순환 의존성
// UserService → OrderService → UserService

// ✅ 좋은 예: 의존성 역전으로 순환 제거
export interface UserRepository {
  getUser(id: string): Promise<User>;
}

export interface OrderRepository {
  getOrdersByUserId(userId: string): Promise<Order[]>;
}

export class UserService {
  constructor(
    private userRepo: UserRepository,
    private orderRepo: OrderRepository
  ) {}

  async getUserWithOrders(id: string) {
    const [user, orders] = await Promise.all([
      this.userRepo.getUser(id),
      this.orderRepo.getOrdersByUserId(id),
    ]);
    return { user, orders };
  }
}
```

### 3. 안정된 추상화 원칙

```ts
// 안정된 추상화 (인터페이스)
export interface PaymentProcessor {
  processPayment(amount: number, currency: string): Promise<PaymentResult>;
}

// 불안정한 구현들
export class StripePaymentProcessor implements PaymentProcessor {
  async processPayment(
    amount: number,
    currency: string
  ): Promise<PaymentResult> {
    // Stripe API 호출
  }
}

export class PayPalPaymentProcessor implements PaymentProcessor {
  async processPayment(
    amount: number,
    currency: string
  ): Promise<PaymentResult> {
    // PayPal API 호출
  }
}

// 안정된 서비스가 불안정한 구현에 의존하지 않음
export class OrderService {
  constructor(private paymentProcessor: PaymentProcessor) {}

  async createOrder(orderData: OrderData) {
    // 비즈니스 로직
    const result = await this.paymentProcessor.processPayment(
      orderData.amount,
      orderData.currency
    );
    // ...
  }
}
```

### 4. 컴포넌트 경계 설정

```ts
// 도메인별 컴포넌트 분리
// packages/user/
export interface UserRepository {
  /* ... */
}
export class UserService {
  /* ... */
}

// packages/order/
export interface OrderRepository {
  /* ... */
}
export class OrderService {
  /* ... */
}

// packages/payment/
export interface PaymentProcessor {
  /* ... */
}
export class PaymentService {
  /* ... */
}

// packages/shared/ (안정된 추상화)
export interface DomainEvent {
  /* ... */
}
export interface ValueObject {
  /* ... */
}
```

---

## 마무리: 컴포넌트는 시스템의 DNA

- **응집도 원칙**으로 **변경의 축**에 따라 컴포넌트를 묶고,
- **결합도 원칙**으로 **의존성 방향**을 관리하여,
- 시스템의 **진화 가능성**과 **안정성**을 동시에 확보하자.

컴포넌트 설계는 단순한 코드 조직화가 아니라, **시스템의 미래**를 결정하는 핵심 설계 활동이다.
