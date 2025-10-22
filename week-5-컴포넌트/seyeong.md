# 클린 아키텍처 12-14장 정리

> 📖 = 책 내용 | 💭 = 개인 생각

## 📋 목차

- [12장: 컴포넌트](#12장-컴포넌트)
- [13장: 컴포넌트 응집도](#13장-컴포넌트-응집도)
- [14장: 컴포넌트 결합](#14장-컴포넌트-결합)
- [정리 및 다음 학습 방향](#-정리-및-다음-학습-방향)

---

## 12장: 컴포넌트

### 📖 컴포넌트란?

**컴포넌트는 배포 단위**다. 시스템의 구성요소로 배포할 수 있는 가장 작은 단위.

- **Java**: jar 파일
- **.NET**: DLL
- **Ruby**: gem
- **JavaScript/Node**: npm 패키지

컴포넌트는 개별적으로 배포 가능하고, 독립적으로 개발할 수 있는 단위.

### 📖 컴포넌트의 역사

#### 초창기의 문제

프로그래머가 라이브러리 소스코드를 애플리케이션 코드에 직접 포함시켜 컴파일하던 시절

```
애플리케이션 (메모리 시작 위치 00000)
├── 메인 프로그램
└── 라이브러리 (메모리 직접 할당)

문제점:
- 메모리가 부족하면 라이브러리와 애플리케이션 충돌
- 라이브러리가 커지면 컴파일 시간 증가
- 메모리 주소를 수동으로 관리해야 함
```

#### 재배치 가능성의 등장

컴파일러가 재배치 가능한 바이너리를 생성하게 되면서,

- 재배치 가능 바이너리 내 함수 이름을 메타데이터로 생성
- 링커(Linker)가 여러 바이너리를 적절한 메모리 주소로 연결
- 하지만 여전히 링킹 시간이 오래 걸림 (수시간)

#### 링킹 로더와 동적 링킹

- **링킹 로더**: 링킹과 로드를 한번에 수행, 외부 참조만 링킹 (컴파일 시간 단축)
- **동적 링킹**: 실행 시점에 필요한 라이브러리만 메모리에 로드 (.so, .dll)

### 📖 현대의 컴포넌트 플러그인 아키텍처

- 링킹과 로딩이 찰나에 이루어짐
- 플러그인 아키텍처가 가능해짐
- jar, DLL, npm 패키지 등을 쉽게 교체 가능
- 컴포넌트 간 결합이 느슨해짐

💭 **개인 생각**

- npm 패키지가 컴포넌트의 좋은 예시. 필요한 라이브러리를 독립적으로 설치/제거 가능
- React 컴포넌트와는 다른 개념: 배포 단위로서의 컴포넌트
- MSA(Micro Service Architecture)도 컴포넌트 개념의 연장선

---

## 13장: 컴포넌트 응집도

### 📖 컴포넌트 응집도 원칙

컴포넌트에 어떤 클래스를 포함시킬지 결정하는 세 가지 원칙

1. **REP (재사용/릴리스 등가 원칙)**
2. **CCP (공통 폐쇄 원칙)**
3. **CRP (공통 재사용 원칙)**

### 📖 REP: 재사용/릴리스 등가 원칙 (Reuse/Release Equivalence Principle)

> **"재사용 단위는 릴리스 단위와 같다"**

- 재사용 가능한 컴포넌트는 릴리스 과정을 통해 추적 관리되어야 함
- 버전 번호가 있어야 함
- 릴리스 노트와 변경 사항이 문서화되어야 함
- 컴포넌트 내부의 클래스와 모듈은 **응집성**이 있어야 함

**프론트엔드 예시:**

```json
// package.json
{
  "name": "@myapp/button-component",
  "version": "2.1.0",
  "description": "재사용 가능한 버튼 컴포넌트"
}
```

### 📖 CCP: 공통 폐쇄 원칙 (Common Closure Principle)

> **"동일한 이유로 동일한 시점에 변경되는 클래스를 같은 컴포넌트로 묶어라"**

- SRP의 컴포넌트 버전
- 변경이 여러 컴포넌트에 분산되면 배포 비용 증가
- 변경이 한 컴포넌트에 집중되도록 설계

**잘못된 예시:**

```
📦 auth-component
├── LoginButton.tsx        // UI 변경 발생
├── LoginForm.tsx         // UI 변경 발생
└── AuthService.ts        // 인증 로직 변경 (다른 이유)

📦 user-component
├── UserProfile.tsx       // UI 변경 발생
└── UserAPI.ts           // API 스펙 변경 (다른 이유)
```

**올바른 예시:**

```
📦 ui-components (UI 변경 시 함께 변경)
├── LoginButton.tsx
├── LoginForm.tsx
└── UserProfile.tsx

📦 auth-logic (인증 로직 변경 시 함께 변경)
├── AuthService.ts
└── TokenManager.ts

📦 api-client (API 스펙 변경 시 함께 변경)
├── UserAPI.ts
└── AuthAPI.ts
```

### 📖 CRP: 공통 재사용 원칙 (Common Reuse Principle)

> **"컴포넌트 사용자들을 필요하지 않는 것에 의존하게 강요하지 말라"**

- ISP의 컴포넌트 버전
- 함께 재사용되는 클래스와 모듈들을 같은 컴포넌트에 포함
- 함께 재사용되지 않는 것은 분리

**잘못된 예시:**

```typescript
// 하나의 큰 유틸리티 패키지
📦 @myapp/utils
├── formatDate.ts
├── validateEmail.ts
├── debounce.ts
├── deepClone.ts
└── ... (수십개의 유틸)

// 문제: formatDate만 필요한데 전체 패키지를 설치해야 함
import { formatDate } from '@myapp/utils';
```

**올바른 예시:**

```typescript
// 기능별로 분리된 패키지
📦 @myapp/date-utils
└── formatDate.ts

📦 @myapp/validation-utils
└── validateEmail.ts

📦 @myapp/function-utils
└── debounce.ts

// 필요한 것만 설치
import { formatDate } from '@myapp/date-utils';
```

### 📖 응집도 원칙들 사이의 균형

세 원칙은 서로 상충되는 면이 있어 균형을 찾아야 함:

```
        REP
        /\
       /  \
      /    \
     /      \
    /   좋은  \
   /   균형점  \
  /            \
CCP ----------- CRP

- REP + CCP 집중: 너무 많은 컴포넌트 변경
- REP + CRP 집중: 불필요한 릴리스가 너무 많아짐
- CCP + CRP 집중: 재사용성 떨어짐
```

프로젝트 초기에는 **CCP**에 집중 (개발 용이성), 성숙기에는 **CRP**와 **REP**로 이동 (재사용성)

💭 **개인 생각**

- 모노레포 구조가 이 원칙들을 적용하기 좋은 환경
- Turborepo, Nx 같은 도구들이 컴포넌트(패키지) 관리를 도와줌
- 프론트엔드에서 디자인 시스템 패키지 구조도 이 원칙 적용 사례

---

## 14장: 컴포넌트 결합

### 📖 컴포넌트 결합 원칙

컴포넌트 사이의 관계를 설명하는 세 가지 원칙:

1. **ADP (의존성 비순환 원칙)**
2. **SDP (안정된 의존성 원칙)**
3. **SAP (안정된 추상화 원칙)**

### 📖 ADP: 의존성 비순환 원칙 (Acyclic Dependencies Principle)

> **"컴포넌트 의존성 그래프에 순환이 있어서는 안 된다"**

#### 순환 의존성의 문제

```
A → B → C → A (순환!)

문제점:
- A를 수정하면 B 영향
- B를 수정하면 C 영향
- C를 수정하면 A 영향
→ 컴포넌트를 독립적으로 릴리스할 수 없음
```

#### 순환 끊기 방법 1: 의존성 역전 원칙(DIP) 적용

```typescript
// ❌ 순환 의존성
// UserService.ts
import { NotificationService } from "./NotificationService";
class UserService {
  constructor(private notification: NotificationService) {}
}

// NotificationService.ts
import { UserService } from "./UserService";
class NotificationService {
  constructor(private userService: UserService) {}
}

// ✅ 인터페이스로 순환 끊기
// UserService.ts
interface INotificationService {
  send(message: string): void;
}

class UserService {
  constructor(private notification: INotificationService) {}
}

// NotificationService.ts
import { INotificationService } from "./interfaces";
class NotificationService implements INotificationService {
  send(message: string): void {
    // 구현
  }
}
```

#### 순환 끊기 방법 2: 새로운 컴포넌트 생성

```
기존: A ⇄ B (순환)

변경: A → C ← B
     (C는 공통 의존성을 포함하는 새 컴포넌트)
```

### 📖 SDP: 안정된 의존성 원칙 (Stable Dependencies Principle)

> **"안정성의 방향으로 의존하라"**

**안정성이란?**

- 변경하기 어려운 정도
- 많은 컴포넌트가 의존할수록 안정적 (변경 시 영향 큼)
- 아무것도 의존하지 않을수록 안정적 (외부 변경에 영향 없음)

**안정성 측정: 불안정성 지표 (I)**

```
I = Fan-out / (Fan-in + Fan-out)

Fan-in: 안으로 들어오는 의존성 (내게 의존하는 컴포넌트)
Fan-out: 밖으로 나가는 의존성 (내가 의존하는 컴포넌트)

I = 0: 최고로 안정 (다른 컴포넌트에 의존하지 않음)
I = 1: 최고로 불안정 (아무도 의존하지 않음)
```

**프론트엔드 예시**

```
📦 @myapp/core (I ≈ 0, 매우 안정)
↑ 많은 패키지가 의존
├── @myapp/ui-components (I ≈ 0.3)
├── @myapp/utils (I ≈ 0.3)
└── @myapp/api-client (I ≈ 0.3)
    ↑
    └── @myapp/feature-user (I ≈ 0.7, 불안정)
        ↑
        └── @myapp/page-dashboard (I ≈ 1, 가장 불안정)
```

**원칙 위반 사례**

```typescript
// ❌ 불안정한 컴포넌트가 안정된 컴포넌트에 영향
📦 @myapp/core (안정)
└── TemporaryFeature.ts (실험적 기능)
    // 이 파일 변경이 전체 시스템에 영향

// ✅ 안정된 것이 불안정한 것에 의존하지 않도록
📦 @myapp/core (안정)
└── CoreTypes.ts

📦 @myapp/experimental (불안정)
├── TemporaryFeature.ts
└── (core에 의존 가능)
```

### 📖 SAP: 안정된 추상화 원칙 (Stable Abstractions Principle)

> **"컴포넌트는 안정된 정도만큼 추상화되어야 한다"**

- 안정된 컴포넌트는 추상 컴포넌트여야 함 (확장 가능)
- 불안정한 컴포넌트는 구체 컴포넌트여야 함 (쉽게 변경 가능)

**추상화 정도 측정 (A)**

```
A = 추상 클래스와 인터페이스 수 / 전체 클래스 수

A = 0: 완전히 구체적
A = 1: 완전히 추상적
```

**주계열(Main Sequence)**

```
A (추상화)
│
1│    (0,1)     이상적 영역
 │    •
 │      ╲
 │       ╲ 주계열
 │        ╲
 │         ╲
 │          •
0│           (1,0)
 └─────────────────→ I (불안정성)
 0                  1
```

**구역별 의미**

1. **(0, 1) - 고통의 구역 (Zone of Pain)**

   - 매우 안정적이면서 구체적
   - 변경하기 어렵고 확장 불가능
   - 예: 데이터베이스 스키마, 유틸리티 라이브러리

2. **(1, 0) - 쓸모없는 구역 (Zone of Uselessness)**
   - 매우 추상적이면서 불안정
   - 아무도 사용하지 않는 추상 클래스
   - 예: 사용되지 않는 인터페이스, 추상 클래스

**프론트엔드 적용**

```typescript
// ✅ 안정적이고 추상적 (주계열 근처)
📦 @myapp/core-interfaces (I=0, A=1)
export interface DataRepository<T> {
  get(id: string): Promise<T>;
  save(data: T): Promise<void>;
}

export interface Presenter<T> {
  present(data: T): void;
}

// ✅ 불안정하고 구체적 (주계열 근처)
📦 @myapp/feature-dashboard (I=0.8, A=0.1)
import { DataRepository, Presenter } from '@myapp/core-interfaces';

class DashboardComponent {
  constructor(
    private dataRepo: DataRepository<DashboardData>,
    private presenter: Presenter<DashboardView>
  ) {}

  async load() {
    const data = await this.dataRepo.get('dashboard');
    this.presenter.present(this.transform(data));
  }
}
```

**거리 측정 (D)**

주계열로부터 얼마나 떨어져 있는지

```
D = |A + I - 1|

D = 0: 주계열 위 (이상적)
D = 1: 최대로 벗어남
```

💭 **개인 생각**

- 추상화/안정성 그래프는 아키텍처 건강도를 측정하는 좋은 도구
- 프론트엔드에서 코어 타입 정의가 가장 안정적이고 추상적이어야 함
- 페이지 컴포넌트는 가장 불안정하고 구체적 → 자주 변경 가능
- 의존성 그래프를 주기적으로 분석하면 아키텍처 부채 발견 가능

---

## 🔖 정리

### 핵심 포인트

#### 컴포넌트 응집도

1. **REP**: 재사용 단위 = 릴리스 단위 (버전 관리)
2. **CCP**: 같이 변경되는 것은 같은 컴포넌트에 (SRP의 컴포넌트 버전)
3. **CRP**: 같이 사용되는 것만 같은 컴포넌트에 (ISP의 컴포넌트 버전)

#### 컴포넌트 결합

1. **ADP**: 순환 의존성 금지 (DIP나 새 컴포넌트로 해결)
2. **SDP**: 안정성 방향으로 의존 (불안정 → 안정)
3. **SAP**: 안정적일수록 추상적이어야 함
