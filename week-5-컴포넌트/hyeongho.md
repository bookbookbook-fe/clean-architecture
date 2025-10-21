# 12장: 컴포넌트

## 주요 개념

### 컴포넌트란?

> 💡 **컴포넌트는 배포 단위다 (Deployment Unit)**
> 시스템의 구성 요소로 배포할 수 있는 가장 작은 단위

**⚠️ 주의**: 여기서 말하는 "컴포넌트"는 React/Vue의 UI 컴포넌트와 **완전히 다른 개념**입니다!

```
Clean Architecture의 컴포넌트 vs React 컴포넌트

┌─────────────────────────────┐   ┌─────────────────────────────┐
│  Clean Architecture 컴포넌트 │   │     React 컴포넌트           │
├─────────────────────────────┤   ├─────────────────────────────┤
│ 배포 단위                    │   │ UI 렌더링 단위               │
│ jar, dll, npm package 등    │   │ function/class 단위          │
│ 독립적으로 배포 가능         │   │ 화면 요소 단위               │
│ 예: lodash, axios            │   │ 예: Button, Modal            │
└─────────────────────────────┘   └─────────────────────────────┘
```

### 컴포넌트의 특징

**잘 설계된 컴포넌트라면:**

1. **독립적으로 배포 가능** (Independently Deployable)
2. **독립적으로 개발 가능** (Independently Developable)

**예시:**

- Java: `.jar` 파일
- C/C++: `.dll` (Windows), `.so` (Linux)
- JavaScript: npm 패키지
- Python: wheel 패키지

## 컴포넌트의 역사

### 1. 재배치성 (Relocatability)

초기 프로그래밍에서는 메모리 주소를 직접 지정해야 했습니다.

```
과거의 문제:
┌─────────────────────────────┐
│  메모리 주소 0x0000          │
│  ├─ 프로그램 A (0x0000)     │
│  ├─ 프로그램 B (0x1000)     │
│  └─ 라이브러리 (0x2000)     │
└─────────────────────────────┘

문제점:
- 프로그램 A 크기가 커지면 B와 충돌
- 라이브러리 주소 하드코딩 필요
```

**재배치 가능한 바이너리 등장:**

- 로더가 코드를 메모리 어디에든 배치 가능
- 재배치 코드에 플래그 삽입하여 주소 자동 조정

### 2. 링킹 로더 (Linking Loader)

**외부 참조 해결:**

```
프로그램에서 라이브러리 함수 호출

┌──────────────┐
│  main.c      │
│  printf()    │ ─→ 외부 참조 (External Reference)
└──────────────┘

┌──────────────┐
│  libc.so     │
│  printf()    │ ─→ 외부 정의 (External Definition)
└──────────────┘

링킹 로더가 연결해줌!
```

**장점:**

- 프로그램을 개별적으로 컴파일 가능
- 개별적으로 로드 가능한 단위로 분할
- 모듈화의 시작!

### 3. 링커 (Linker)

**링킹 로더의 진화:**

```
컴파일 타임      링크 타임         런타임
     ↓              ↓                ↓
┌─────────┐    ┌─────────┐     ┌─────────┐
│ main.o  │    │         │     │         │
│ util.o  │ →  │ Linker  │  →  │ Loader  │
│ lib.o   │    │         │     │         │
└─────────┘    └─────────┘     └─────────┘
                    ↓
              ┌─────────┐
              │ a.out   │ ← 링크 완료된 실행 파일
              └─────────┘
```

**링커의 역할:**

- 링크가 완료된 재배치 코드 생성
- 로더의 로딩 과정이 매우 빨라짐
- 실행 파일 하나로 통합

### 4. 동적 링크 (Dynamic Linking)

**무어의 법칙 vs 머피의 법칙:**

- 프로그램은 메모리보다 빠르게 커짐
- 모든 라이브러리를 실행 파일에 포함하면 너무 큼

**해결책: 동적 링크 파일**

```
정적 링크 (과거)              동적 링크 (현재)
┌──────────────┐             ┌──────────────┐
│ 실행 파일     │             │ 실행 파일     │
│ - main 코드  │             │ - main 코드  │
│ - libc 전체  │             │ - libc 참조  │ ─┐
│ - libm 전체  │             │ - libm 참조  │ ─┤
│ (파일 크기 큼)│             │ (파일 작음)   │  │
└──────────────┘             └──────────────┘  │
                                                │
                             ┌──────────────────┘
                             ↓
                        런타임에 로드
                        ┌──────────┐
                        │ libc.so  │
                        │ libm.so  │
                        └──────────┘
```

**동적 링크 파일 = 소프트웨어 컴포넌트!**

- 런타임에 플러그인 형태로 결합
- `.dll` (Windows), `.so` (Linux), `.dylib` (macOS)

## Frontend 관점의 컴포넌트

> 💡 **Frontend 관점**: npm 패키지, 번들 청크, 마이크로프론트엔드가 Clean Architecture의 "컴포넌트" 개념에 해당한다.

### 1. npm 패키지 (독립 배포 단위)

**함수형 프로그래밍 예제:**

```typescript
// ✅ 독립적으로 배포 가능한 컴포넌트 (npm 패키지)

// @myapp/auth-component 패키지
export const createAuthService = () => ({
  login: async (email: string, password: string) => {
    // 인증 로직
  },
  logout: async () => {
    // 로그아웃 로직
  },
});

// @myapp/payment-component 패키지
export const createPaymentService = () => ({
  processPayment: async (amount: number) => {
    // 결제 로직
  },
});
```

**패키지 구조:**

```
프로젝트 구조

┌─────────────────────────────────┐
│  메인 애플리케이션               │
│  - package.json                 │
│    dependencies:                │
│      @myapp/auth-component      │ ← 독립 배포
│      @myapp/payment-component   │ ← 독립 배포
│      @myapp/ui-component        │ ← 독립 배포
└─────────────────────────────────┘

각 패키지를 독립적으로 개발/배포 가능!
```

### 2. 코드 스플리팅 (번들 청크)

**동적 import = 런타임 동적 링크**

```typescript
// ✅ 런타임에 동적으로 로드되는 컴포넌트 (청크)

// 정적 import (모든 코드가 main 번들에 포함)
import { HeavyChart } from "./HeavyChart";

// ❌ 문제점: main.bundle.js가 너무 큼

// 동적 import (런타임에 필요할 때 로드)
const loadChart = async () => {
  const { HeavyChart } = await import("./HeavyChart");
  return HeavyChart;
};

// ✅ 장점: chart.chunk.js로 분리, 필요할 때만 로드
```

**번들 구조:**

```
빌드 결과

┌──────────────────────┐
│  main.bundle.js      │  ← 핵심 코드만
│  - App.js            │
│  - Router.js         │
└──────────────────────┘
         │
         │ 필요시 동적 로드
         ↓
┌──────────────────────┐
│  chart.chunk.js      │  ← 차트 컴포넌트
│  admin.chunk.js      │  ← 관리자 모듈
│  reports.chunk.js    │  ← 리포트 모듈
└──────────────────────┘

각 청크 = 독립 배포 단위 (동적 컴포넌트)
```

### 3. 마이크로프론트엔드 (독립 애플리케이션)

**함수형 프로그래밍 예제:**

```typescript
// ✅ 마이크로프론트엔드 = 여러 독립 컴포넌트 조합

// Shell 애플리케이션 (컨테이너)
const ShellApp = () => {
  const [AuthApp, setAuthApp] = useState(null);
  const [DashboardApp, setDashboardApp] = useState(null);

  useEffect(() => {
    // 런타임에 원격 컴포넌트 로드 (동적 링크와 유사)
    loadRemoteModule("auth", "https://auth.example.com/remoteEntry.js").then(
      (module) => setAuthApp(() => module.App)
    );

    loadRemoteModule(
      "dashboard",
      "https://dashboard.example.com/remoteEntry.js"
    ).then((module) => setDashboardApp(() => module.App));
  }, []);

  return (
    <div>
      {AuthApp && <AuthApp />}
      {DashboardApp && <DashboardApp />}
    </div>
  );
};
```

**마이크로프론트엔드 아키텍처:**

```
독립 배포 & 런타임 통합

┌──────────────────────────────────────┐
│         Shell Application            │
│         (https://main.com)           │
└──────────────────────────────────────┘
              ↓ 런타임 로드
    ┌─────────┼─────────┐
    │         │         │
┌───▼───┐ ┌──▼───┐ ┌──▼───┐
│ Auth  │ │ Shop │ │ Admin│ ← 각각 독립 배포
│ App   │ │ App  │ │ App  │
└───────┘ └──────┘ └──────┘
독립 팀    독립 팀   독립 팀

= Clean Architecture의 "컴포넌트" 개념!
```

## 컴포넌트 개념 비교표

| 항목       | Clean Architecture 컴포넌트 | React 컴포넌트        |
| ---------- | --------------------------- | --------------------- |
| **정의**   | 배포 단위                   | UI 렌더링 단위        |
| **범위**   | 시스템 아키텍처 레벨        | 코드 구현 레벨        |
| **예시**   | npm 패키지, .jar, .dll      | Button, Modal, Header |
| **독립성** | 독립 배포/개발 가능         | 재사용 가능           |
| **크기**   | 큼 (수백~수천 파일)         | 작음 (1개 파일)       |
| **역할**   | 시스템 구성 요소            | UI 조각               |

## 핵심 교훈

### 컴포넌트의 본질

> ⚠️ **컴포넌트 = 배포 단위 = 플러그인**

**특징:**

1. **독립적 배포**: 다른 컴포넌트에 영향 없이 업데이트 가능
2. **독립적 개발**: 다른 팀이 동시에 개발 가능
3. **플러그인 구조**: 런타임에 결합 가능

### Frontend에서의 적용

```
                  컴포넌트 설계
                       │
        ┌──────────────┼──────────────┐
        │              │              │
   npm 패키지     코드 스플리팅   마이크로프론트엔드
        │              │              │
  독립 라이브러리   번들 청크        독립 앱
  @myapp/*        *.chunk.js      remoteEntry.js
```

**체크리스트:**

- ✅ 각 컴포넌트는 독립적으로 배포 가능한가?
- ✅ 다른 컴포넌트 변경 시 재배포가 필요한가?
- ✅ 런타임에 동적으로 로드 가능한가?
- ✅ 명확한 인터페이스(계약)가 정의되어 있는가?

## 정리 요약

**Clean Architecture의 컴포넌트**는 배포 단위로, React 컴포넌트와는 전혀 다른 개념이다. 컴포넌트는 시스템을 구성하는 독립적으로 배포 가능한 모듈이며, 역사적으로 링커와 동적 링크의 발전을 통해 진화했다.

Frontend에서는 npm 패키지, 코드 스플리팅 청크, 마이크로프론트엔드, Monorepo 패키지가 이러한 컴포넌트 개념에 해당한다. 각 컴포넌트는 독립적으로 개발하고 배포할 수 있어야 하며, 런타임에 플러그인처럼 결합할 수 있어야 한다.

잘 설계된 컴포넌트 구조는 팀 간 협업을 용이하게 하고, 시스템의 유연성과 확장성을 크게 향상시킨다.

---

# 13장 - 컴포넌트 응집도

## 개요

> 💡 **핵심 질문**: 어떤 클래스들을 하나의 컴포넌트로 묶어야 할까?

컴포넌트 응집도는 **어떤 클래스들이 함께 묶여야 하는가**를 결정하는 원칙입니다. 세 가지 원칙이 있습니다:

1. **REP (Reuse/Release Equivalence Principle)**: 재사용/릴리스 등가 원칙
2. **CCP (Common Closure Principle)**: 공통 폐쇄 원칙
3. **CRP (Common Reuse Principle)**: 공통 재사용 원칙

---

## 1. REP - 재사용/릴리스 등가 원칙

### 원칙 정의

> **재사용 단위는 릴리스 단위와 같다**

- 단일 컴포넌트는 응집성 높은 클래스와 모듈들로 구성되어야 함
- 컴포넌트를 구성하는 모든 모듈은 서로 공유하는 **중요한 테마나 목적**이 있어야 함
- 하나의 컴포넌트로 묶인 클래스와 모듈은 반드시 **함께 릴리스**할 수 있어야 함

### Frontend 관점 설명

> 💡 **Frontend 관점**: npm 패키지처럼 하나의 목적을 가진 기능들을 함께 버전 관리하고 배포해야 합니다.

**구조 시각화:**

```
컴포넌트 (npm 패키지)
↓
┌────────────────────────────────┐
│  @myapp/ui-components v1.2.0  │
├────────────────────────────────┤
│  ✅ Button                     │
│  ✅ Input                      │
│  ✅ Modal                      │  ← 함께 릴리스
│  ✅ Dropdown                   │
└────────────────────────────────┘
```

### TypeScript 예제

```typescript
// ✅ 좋은 예: 하나의 테마로 묶인 컴포넌트
// @myapp/ui-components 패키지

// components/Button.tsx
export const Button = ({ children, onClick }: ButtonProps) => (
  <button onClick={onClick}>{children}</button>
);

// components/Input.tsx
export const Input = ({ value, onChange }: InputProps) => (
  <input value={value} onChange={onChange} />
);

// index.ts - 함께 릴리스
export { Button } from './components/Button';
export { Input } from './components/Input';
export { Modal } from './components/Modal';

// package.json
{
  "name": "@myapp/ui-components",
  "version": "1.2.0"  // 모든 UI 컴포넌트가 함께 버전 관리됨
}
```

```typescript
// ❌ 나쁜 예: 서로 관련 없는 것들을 한 패키지에
// @myapp/random-stuff 패키지

export { Button } from "./ui/Button"; // UI 컴포넌트
export { fetchUser } from "./api/user"; // API 함수
export { Analytics } from "./tracking"; // 분석 도구
// → 테마가 없고, 함께 릴리스할 이유가 없음
```

---

## 2. CCP - 공통 폐쇄 원칙

### 원칙 정의

> **동일한 이유로 동일한 시점에 변경되는 클래스를 같은 컴포넌트로 묶어라**
> **서로 다른 시점에 다른 이유로 변경되는 클래스는 다른 컴포넌트로 분리하라**

- 단일 컴포넌트는 **변경의 이유가 여러 개** 있어서는 안 됨 (SRP의 컴포넌트 버전)
- 변경될 가능성이 있는 클래스는 **모두 한곳에 묶어라**
- 물리적 또는 개념적으로 강하게 결합되어 **항상 함께 변경**되는 클래스들은 하나의 컴포넌트에 속해야 함

**핵심 문장:**

> 동일한 시점에 동일한 이유로 변경되는 것들을 한데 묶어라.
> 서로 다른 시점에 다른 이유로 변경되는 것들은 서로 분리하라.

### Frontend 관점 설명

> 💡 **Frontend 관점**: 함께 변경되는 기능들은 같은 모듈에, 따로 변경되는 것들은 분리하세요.

**Before/After 비교:**

```
❌ Before: 변경 이유가 다른 것들이 섞여 있음
┌─────────────────────────────┐
│  ProductModule              │
│  - ProductCard (UI 변경)    │
│  - fetchProduct (API 변경)  │  ← 서로 다른 이유로 변경
│  - Analytics (분석 변경)    │
└─────────────────────────────┘

✅ After: 변경 이유별로 분리
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│ UI 컴포넌트   │  │ API Layer    │  │ Analytics    │
│ ProductCard  │  │ fetchProduct │  │ trackEvent   │
└──────────────┘  └──────────────┘  └──────────────┘
```

### TypeScript 예제

```typescript
// ✅ 좋은 예: 같은 이유로 변경되는 것들을 묶음

// features/product-display/
// → "상품 표시" 기능이 변경될 때 함께 변경됨

// ProductCard.tsx
export const ProductCard = ({ product }: { product: Product }) => (
  <div>
    <ProductImage src={product.image} />
    <ProductInfo name={product.name} price={product.price} />
    <ProductActions productId={product.id} />
  </div>
);

// ProductImage.tsx
export const ProductImage = ({ src }: { src: string }) => (
  <img src={src} alt="product" />
);

// ProductInfo.tsx
export const ProductInfo = ({ name, price }: InfoProps) => (
  <div>
    <h3>{name}</h3>
    <span>{price}원</span>
  </div>
);

// → UI 디자인이 변경되면 이 파일들이 함께 변경됨
```

```typescript
// ❌ 나쁜 예: 서로 다른 이유로 변경되는 것들이 섞여 있음

// ProductModule.tsx
export const ProductModule = () => {
  // UI 로직 (디자인 변경 시)
  const renderProduct = () => <div>...</div>;

  // API 로직 (백엔드 변경 시)
  const fetchProduct = async (id: string) => {
    return fetch(`/api/products/${id}`);
  };

  // 분석 로직 (분석 요구사항 변경 시)
  const trackView = (productId: string) => {
    analytics.track("product_viewed", { productId });
  };

  // → 변경 이유가 3가지! CCP 위반
};
```

---

## 3. CRP - 공통 재사용 원칙

### 원칙 정의

> **컴포넌트 사용자들을 필요하지 않는 것에 의존하게 강요하지 마라**

- 같이 재사용되는 경향이 있는 클래스와 모듈들은 **같은 컴포넌트**에 포함해야 함
- 강하게 결합되지 않은 클래스들을 동일한 컴포넌트에 위치시켜서는 안 됨
- **컴포넌트에서 단 하나의 클래스만 사용**한다고 해도 의존성이 약해지지 않음

### ISP와의 관계

| 원칙    | 범위     | 조언                                                 |
| ------- | -------- | ---------------------------------------------------- |
| **ISP** | 클래스   | 사용하지 않은 메서드가 있는 클래스에 의존하지 말라   |
| **CRP** | 컴포넌트 | 사용하지 않는 클래스를 가진 컴포넌트에 의존하지 말라 |

**핵심:**

> 필요하지 않은 것에 의존하면 안 됨

### 컨테이너와 이터레이터 사례

- 컨테이너와 이터레이터는 **서로 강하게 결합**되어 있음
- 반드시 **동일한 컴포넌트**에 위치해야 함
- CRP는 강하게 결합되지 않은 클래스들을 동일한 컴포넌트에 위치시켜서는 안 된다고 조언

### Frontend 관점 설명

> 💡 **Frontend 관점**: 함께 사용되는 기능만 묶고, 불필요한 의존성을 강요하지 마세요.

**관계 시각화:**

```
┌─────────────────────────────┐
│  @myapp/data-table v2.0.0   │
├─────────────────────────────┤
│  Table ←┐                   │
│         │ 강하게 결합        │
│  Pagination ←┘              │  ✅ 함께 재사용됨
│  SortHeader ←┐              │
│         └─ 강하게 결합       │
└─────────────────────────────┘

❌ 나쁜 예: 함께 사용되지 않는 것들
┌─────────────────────────────┐
│  @myapp/ui-bundle v1.0.0    │
├─────────────────────────────┤
│  Table                      │
│  Chart        ← 거의 분리됨 │
│  Calendar                   │
│  Map                        │
└─────────────────────────────┘
→ Table만 쓰려는데 Chart, Map도 설치됨!
```

### TypeScript 예제

```typescript
// ✅ 좋은 예: 함께 재사용되는 것들만 묶음

// @myapp/data-table 패키지
// Table.tsx
import { useState } from 'react';

interface TableProps<T> {
  data: T[];
  columns: Array<{ key: keyof T; label: string }>;
}

export const Table = <T>({ data, columns }: TableProps<T>) => {
  const [page, setPage] = useState(1);

  return (
    <div>
      <TableHeader columns={columns} /> {/* 함께 사용됨 */}
      <TableBody data={data} /> {/* 함께 사용됨 */}
      <Pagination page={page} onChange={setPage} /> {/* 함께 사용됨 */}
    </div>
  );
};

// Pagination.tsx - Table과 강하게 결합
export const Pagination = ({ page, onChange }: PaginationProps) => (
  <div>
    <button onClick={() => onChange(page - 1)}>이전</button>
    <span>{page}</span>
    <button onClick={() => onChange(page + 1)}>다음</button>
  </div>
);

// → Table을 쓰면 Pagination도 자연스럽게 함께 사용됨
```

```typescript
// ❌ 나쁜 예: 함께 재사용되지 않는 것들을 억지로 묶음

// @myapp/ui-bundle 패키지
export { Table } from "./Table";
export { Chart } from "./Chart";
export { Calendar } from "./Calendar";
export { Map } from "./Map";

// 사용자 코드
import { Table } from "@myapp/ui-bundle";
// → Table만 쓰는데 Chart, Calendar, Map도 번들에 포함됨!
// → 불필요한 의존성 강요
```

**함수형 예제: 트리쉐이킹 (Tree-shaking) 고려**

```typescript
// ✅ 좋은 예: 독립적인 유틸 함수들
// utils/array.ts
export const map = <T, U>(arr: T[], fn: (item: T) => U): U[] =>
  arr.map(fn);

export const filter = <T>(arr: T[], predicate: (item: T) => boolean): T[] =>
  arr.filter(predicate);

// 사용자는 필요한 것만 import
import { map } from '@myapp/utils/array';  // filter는 번들에 포함 안 됨

// ❌ 나쁜 예: 모든 것이 하나로 결합
// utils/index.ts
export default {
  map: ...,
  filter: ...,
  reduce: ...,
  // 100개의 함수들
};

import utils from '@myapp/utils';  // 모든 함수가 번들에 포함됨!
```

---

## 4. 세 원칙의 균형

### 원칙의 성격

| 원칙    | 성격      | 효과                        |
| ------- | --------- | --------------------------- |
| **REP** | 포함 원칙 | 컴포넌트를 더 **크게** 만듦 |
| **CCP** | 포함 원칙 | 컴포넌트를 더 **크게** 만듦 |
| **CRP** | 배제 원칙 | 컴포넌트를 더 **작게** 만듦 |

### 상호작용 다이어그램

```
  REP                           CCP
재사용성                     유지보수성
   \                           /
    \                         /
     \                       /
      \                     /
       \                   /
        \                 /
         \               /
          \             /
           \           /
            \         /
             \       /
              \     /
               \   /
                \ /
                CRP
            최소 의존성
              (분리)

┌──────────────────────────────────────┐
│  각 변의 의미 (트레이드오프)          │
├──────────────────────────────────────┤
│  REP ↔ CRP: 컴포넌트 변경이 너무 빈번│
│  REP ↔ CCP: 불필요한 릴리스가 너무 빈번│
│  CCP ↔ CRP: 재사용성이 어려움        │
└──────────────────────────────────────┘

⚠️ 핵심 질문:
"이 공간의 어디에 컴포넌트를 배치시켜야 할까?"

→ 프로젝트의 현재 단계와 우선순위에 따라 삼각형 내부의
   적절한 위치를 선택해야 함
```

### 프로젝트 진화 과정

```
프로젝트 초기                프로젝트 성숙
(CCP + CRP 사이)            (REP + CRP 사이)
┌─────────────┐           ┌─────────────┐
│     CCP     │           │ REP + CRP   │
│ 유지보수성   │   ───→    │ 재사용성    │
│ 빠른 개발   │           │ 의존성 최소 │
│ (재사용 희생)│           │ (변경 빈번) │
└─────────────┘           └─────────────┘

⚠️ 일반적 흐름:
1. 초기: CCP 중심 (오른쪽 하단)
   - 함께 변경되는 것들을 묶어 빠른 개발
   - 재사용성은 희생 (REP 무시)

2. 성숙: REP + CRP 고려 (왼쪽 하단)
   - 재사용 가능한 컴포넌트로 분리
   - 불필요한 의존성 제거
   - 대신 변경이 빈번해질 수 있음
```

### Frontend 프로젝트 예시

```typescript
// 🎯 초기 단계: CCP 중심 (빠른 개발)
// features/product-list/
//   - ProductList.tsx
//   - ProductCard.tsx
//   - ProductFilter.tsx
//   → 상품 목록 기능이 변경되면 여기만 수정 (CCP)
//   → 아직 재사용은 고려 안 함

// 🎯 성숙 단계: REP + CRP 고려 (재사용)
// shared/ui/
//   - Card.tsx       ← ProductCard에서 추출
//   - Filter.tsx     ← ProductFilter에서 추출
//
// features/product-list/
//   - ProductList.tsx
//   → shared/ui의 Card, Filter 재사용
```

---

## 5. 응집도 개념의 변화

### 전통적 응집도

> 모듈은 단 하나의 기능만 수행해야 한다

### 현대적 응집도 (이 장의 관점)

> 컴포넌트는 하나의 변경 이유, 하나의 재사용 단위, 하나의 릴리스 단위를 가져야 한다

**변화 비교:**

```
전통적 응집도               현대적 응집도
┌─────────────┐           ┌─────────────┐
│ 하나의 기능  │           │ 변경 이유    │
│             │   ───→    │ 재사용 단위  │
│             │           │ 릴리스 단위  │
└─────────────┘           └─────────────┘
```

---

## 정리 요약

### 세 가지 원칙 한눈에 보기

```
┌────────────────────────────────────────────────────┐
│  REP: 재사용/릴리스 등가 원칙                       │
│  → 함께 릴리스되는 것들을 묶어라                    │
│  → npm 패키지처럼 버전 관리                        │
├────────────────────────────────────────────────────┤
│  CCP: 공통 폐쇄 원칙                               │
│  → 함께 변경되는 것들을 묶어라                      │
│  → 변경 영향을 최소화                              │
├────────────────────────────────────────────────────┤
│  CRP: 공통 재사용 원칙                             │
│  → 함께 사용되는 것들만 묶어라                      │
│  → 불필요한 의존성 제거                            │
└────────────────────────────────────────────────────┘
```

# 14장 - 컴포넌트 결합

## 개요

> 💡 **핵심 질문**: 컴포넌트 간의 의존성을 어떻게 관리해야 할까?

컴포넌트 결합(Component Coupling)은 **컴포넌트 간의 관계**를 다루는 원칙입니다. 세 가지 원칙이 있습니다:

1. **ADP (Acyclic Dependencies Principle)**: 의존성 비순환 원칙
2. **SDP (Stable Dependencies Principle)**: 안정된 의존성 원칙
3. **SAP (Stable Abstractions Principle)**: 안정된 추상화 원칙

---

## 1. ADP - 의존성 비순환 원칙

### 원칙 정의

> **컴포넌트 의존성 그래프에 순환(cycle)이 있어서는 안 된다**

### 숙취 증후군 (Morning After Syndrome)

**문제 상황:**

- 많은 개발자가 **동일한 소스 파일을 수정**하는 환경에서 발생
- 어제까지 잘 돌아가던 코드가 오늘 아침에 **갑자기 동작하지 않음**
- 누군가가 밤새 수정한 코드 때문에 발생하는 현상

**전통적 해결책:**

```
주 단위 빌드 (Weekly Build)
├─ 월 ~ 목: 자유 개발 (통합 신경 안 씀)
└─ 금요일: 변경된 코드를 모두 통합하여 배포

❌ 문제점:
→ 프로젝트 규모가 커질수록 장점 상실
→ 금요일 통합 시간이 점점 길어짐
→ 결국 주 단위 → 월 단위로 변경
```

### 순환 의존성 제거

**해결 방법:**

```
개발 환경을 릴리스 가능한 컴포넌트 단위로 분리
↓
┌──────────────────────────────────────┐
│ 컴포넌트 기반 개발 흐름               │
├──────────────────────────────────────┤
│ 1. 개발자가 컴포넌트를 동작하게 만듦  │
│ 2. 릴리스 번호를 부여                │
│ 3. 다른 팀이 사용할 디렉터리로 이동  │
│ 4. 개발자는 자신만의 공간에서 계속 개발│
│ 5. 나머지 팀은 릴리스된 버전 사용    │
└──────────────────────────────────────┘

✅ 장점:
→ 어떤 팀도 다른 팀에 의해 좌우되지 않음
→ 새 버전 사용 여부를 각 팀이 결정
```

**성공 조건:**

> 컴포넌트 사이의 의존성 구조를 반드시 관리해야 함
> → **비순환 방향 그래프(DAG, Directed Acyclic Graph)**

**비순환 방향 그래프:**

- 컴포넌트는 **정점(vertex)** 에 해당
- 의존성 관계는 **방향이 있는 간선(directed edge)** 에 해당
- 어느 컴포넌트에서 시작하더라도, 의존성 관계를 따라가면서 **최초의 컴포넌트로 되돌아갈 수 없다**

### Frontend 관점 설명

> 💡 **Frontend 관점**: npm 패키지처럼 컴포넌트를 독립적으로 버전 관리하고, 순환 의존성을 제거하세요.

**순환 의존성 문제:**

```
❌ 순환 의존성 발생
┌──────────┐      ┌──────────┐
│  UserUI  │ ───→ │  Auth    │
└──────────┘      └──────────┘
     ↑                  │
     │                  ↓
     │            ┌──────────┐
     └─────────── │  Profile │
                  └──────────┘

문제점:
→ UserUI → Auth → Profile → UserUI (순환!)
→ 어느 컴포넌트를 먼저 빌드해야 할지 모름
→ 변경 영향 범위를 파악하기 어려움
```

**비순환 그래프:**

```
✅ 비순환 의존성 (올바른 구조)
┌──────────┐      ┌──────────┐
│  UserUI  │ ───→ │  Auth    │
└──────────┘      └──────────┘
     │                  │
     ↓                  ↓
┌──────────┐      ┌──────────┐
│  Profile │ ───→ │  Common  │
└──────────┘      └──────────┘

장점:
→ 명확한 빌드 순서: Common → Auth → Profile → UserUI
→ 변경 영향 범위 명확
→ 독립적인 테스트 가능
```

### TypeScript 예제

```typescript
// ❌ 나쁜 예: 순환 의존성
// user/UserList.tsx
import { getCurrentUser } from "../auth/authService"; // Auth에 의존

export const UserList = () => {
  const user = getCurrentUser();
  return <div>{user.name}</div>;
};

// auth/authService.ts
import { UserProfile } from "../user/UserProfile"; // User에 의존

export const getCurrentUser = () => {
  // UserProfile을 사용...
  return new UserProfile();
};

// user/UserProfile.tsx
import { checkAuthStatus } from "../auth/authService"; // 다시 Auth에 의존!

export class UserProfile {
  constructor() {
    checkAuthStatus(); // 순환 의존성 발생!
  }
}

// 순환 발생: UserList → authService → UserProfile → authService (순환!)
```

```typescript
// ✅ 좋은 예: DIP를 활용한 순환 제거

// shared/types/User.ts (공통 인터페이스)
export interface IUser {
  id: string;
  name: string;
  email: string;
}

// auth/authService.ts
import type { IUser } from "../shared/types/User";

export const getCurrentUser = (): IUser => {
  return {
    id: "1",
    name: "John",
    email: "john@example.com",
  };
};

// user/UserList.tsx
import { getCurrentUser } from "../auth/authService";
import type { IUser } from "../shared/types/User";

export const UserList = () => {
  const user: IUser = getCurrentUser();
  return <div>{user.name}</div>;
};

// 의존성 흐름:
// UserList → authService → IUser
// (순환 없음, 모두 IUser 인터페이스에 의존)
```

### 순환 끊기 전략

**1. DIP (의존성 역전 원칙) 적용**

```
Before: A → B → C → A (순환)
After:  A → Interface ← B → C
```

**2. 새로운 컴포넌트 생성**

```
Before: A ↔ B (순환)
After:  A → Common ← B
```

### Jitters (흐트러짐)

> 요구사항이 변경되면 컴포넌트 구조도 변경될 수 있다는 사실

**중요한 인사이트:**

- 의존성 구조는 **하향식 설계(Top-down)로 만들 수 없음**
- 시스템이 성장하고 변경되면서 **진화**함
- 최우선 관심사: **변동성을 격리**하는 일

---

## 2. SDP - 안정된 의존성 원칙

### 원칙 정의

> **안정성의 방향으로 의존하라** > **변경이 쉽지 않은 컴포넌트가 변동이 예상되는 컴포넌트에 의존하게 만들어서는 안 된다**

### 안정성(Stability)의 의미

**안정성 ≠ 변화 빈도**

- 안정성은 변화가 발생하는 **빈도와 직접적인 관련이 없음**
- 안정성 = **변경하기 어려운 정도**

**안정적인 컴포넌트 만들기:**

> 수많은 다른 컴포넌트가 해당 컴포넌트에 의존하게 만들기

```
안정적인 컴포넌트 X
┌──────────┐
│    X     │ ← A
│          │ ← B
│          │ ← C
└──────────┘

특징:
→ 3개의 컴포넌트가 X에 의존
→ X를 변경하지 말아야 할 이유가 3가지
→ 사소한 변경이라도 의존하는 모든 컴포넌트를 만족시켜야 함
→ X는 책임이 크다 (Responsible)
→ 외부의 영향이 없음 (Independent)
→ 안정적 (Stable)
```

```
불안정한 컴포넌트 Y
         ┌──────────┐
    A ←─ │    Y     │
    B ←─ │          │
    C ←─ │          │
         └──────────┘

특징:
→ Y는 3개의 컴포넌트에 의존
→ Y를 변경해야 할 이유가 3가지
→ Y는 책임이 없다 (Irresponsible)
→ 외부의 영향을 받음 (Dependent)
→ 불안정 (Unstable)
```

### 안정성 지표

**Fan-in (들어오는 의존성):**

- 컴포넌트 **내부**의 클래스에 의존하는 **외부** 클래스 개수

**Fan-out (나가는 의존성):**

- 컴포넌트 **외부**의 클래스에 의존하는 **내부** 클래스 개수

**불안정성 지표 (I):**

```
I = Fan-out / (Fan-in + Fan-out)

범위: [0, 1]
- I = 0: 최대 안정 (다른 컴포넌트가 의존, 자신은 의존 안 함)
- I = 1: 최대 불안정 (자신은 의존, 다른 컴포넌트는 의존 안 함)
```

### Frontend 관점 설명

> 💡 **Frontend 관점**: 안정적인 컴포넌트(공통 라이브러리)는 불안정한 컴포넌트(기능 모듈)에 의존하면 안 됩니다.

**안정성 계층 구조:**

```
안정성 계층 구조
┌─────────────────────────────┐
│  Feature (I ≈ 1)            │  ← 불안정 (변경 잦음)
│  - ProductList              │
│  - UserDashboard            │
└─────────────────────────────┘
           ↓ 의존
┌─────────────────────────────┐
│  Shared UI (I ≈ 0.5)        │  ← 중간 안정성
│  - Button, Input            │
└─────────────────────────────┘
           ↓ 의존
┌─────────────────────────────┐
│  Utils (I ≈ 0)              │  ← 안정 (변경 드뭄)
│  - formatDate, validators   │
└─────────────────────────────┘
```

### TypeScript 예제

```typescript
// ✅ 좋은 예: 불안정 → 안정 방향 의존

// shared/utils/formatters.ts (I ≈ 0, 안정적)
export const formatPrice = (price: number): string => {
  return `${price.toLocaleString()}원`;
};

export const formatDate = (date: Date): string => {
  return date.toLocaleDateString("ko-KR");
};

// features/product/ProductCard.tsx (I ≈ 1, 불안정)
import { formatPrice } from "@/shared/utils/formatters"; // ✅ 안정 방향 의존

export const ProductCard = ({ product }: { product: Product }) => (
  <div>
    <h3>{product.name}</h3>
    <p>{formatPrice(product.price)}</p> {/* 안정적인 함수 사용 */}
  </div>
);
```

```typescript
// ❌ 나쁜 예: 안정 → 불안정 방향 의존

// shared/utils/formatters.ts (안정적이어야 함)
import { ProductCard } from "@/features/product/ProductCard"; // ❌ 불안정한 것에 의존

export const formatProductPrice = (product: Product) => {
  // ProductCard에 의존하면 안 됨!
  return <ProductCard product={product} />;
};
```

### React에서 불안정성 계산

```typescript
// 예시: Button 컴포넌트의 불안정성 계산

// shared/ui/Button.tsx
import { useTheme } from "../hooks/useTheme"; // Fan-out: 1

export const Button = ({ children }: ButtonProps) => {
  const theme = useTheme();
  return <button>{children}</button>;
};

// Fan-in을 계산하려면:
// Button을 import하는 모든 파일을 찾아야 함
// 예: ProductList, UserProfile, LoginForm → Fan-in: 3

// 불안정성 I = 1 / (3 + 1) = 0.25 (상대적으로 안정)
```

**자동화 도구:**

- `dependency-cruiser`: 의존성 그래프 분석
- `madge`: 순환 의존성 탐지
- ESLint 플러그인: import 규칙 강제

---

## 3. SAP - 안정된 추상화 원칙

### 원칙 정의

> **컴포넌트는 안정된 정도만큼만 추상화되어야 한다**

**핵심:**

- 안정적인 컴포넌트 → 추상적이어야 함 (확장에 열려 있음)
- 불안정한 컴포넌트 → 구체적이어야 함 (변경하기 쉬움)

### 추상화 정도 측정

**추상화 지표 (A):**

```
A = Na / Nc

- Nc: 컴포넌트의 클래스 개수
- Na: 추상 클래스와 인터페이스 개수
- 범위: [0, 1]
  - A = 0: 구체적 (추상 클래스/인터페이스 없음)
  - A = 1: 완전 추상적 (모두 추상/인터페이스)
```

### 주계열 (Main Sequence)

**I-A 그래프:**

```
A (추상화)
↑
1 │ (0,1)                (1,1)
  │  ┌────────────────────┐
  │  │ 쓸모없는 구역       │
  │  │ (Zone of Useless) │
  │  └────────────────────┘
  │       ╲
  │        ╲  주계열
  │         ╲  (Main Sequence)
  │          ╲
  │           ╲
  │  ┌────────╲───────────┐
  │  │ 고통의 구역╲        │
  │  │ (Zone of Pain)     │
  │  └────────────────────┘
0 │ (0,0)                (1,0)
  └──────────────────────────→ I (불안정성)
  0                           1

┌─────────────────────────────────┐
│  주요 구역                       │
├─────────────────────────────────┤
│ (0,1): 안정적 + 추상적 (이상적) │
│ (1,0): 불안정 + 구체적 (이상적) │
│ (0,0): 안정적 + 구체적 (고통)   │
│ (1,1): 불안정 + 추상적 (쓸모X)  │
└─────────────────────────────────┘
```

**쓸모없는 구역 (Zone of Uselessness) - (1,1) 주변:**

- 최고로 추상적이지만 **아무도 사용하지 않음**
- 불안정 + 추상적 = 죽은 코드

**고통의 구역 (Zone of Pain) - (0,0) 주변:**

- 매우 안정적이지만 **구체적**
- 변경하기 어려운 구체 클래스
- 많은 컴포넌트가 의존하므로 변경 시 고통

**주계열 (Main Sequence):**

- (0,1)과 (1,0)을 잇는 대각선
- 컴포넌트가 위치할 **가장 바람직한 지점**
- 배제 구역으로부터 최대한 멀리 떨어진 위치

### 주계열과의 거리

**거리 지표 (D):**

```
D = |A + I - 1|

범위: [0, 1]
- D = 0: 주계열 바로 위에 위치 (이상적)
- D = 1: 주계열로부터 가장 멀리 위치 (문제)
```

**활용:**

- 평균과 분산 계산
- 분산을 통해 **극히 예외적인 컴포넌트** 식별
- 관리 한계 설정

### Frontend 관점 설명

> 💡 **Frontend 관점**: 안정적인 공통 라이브러리는 인터페이스/타입으로 추상화하고, 불안정한 기능 모듈은 구체적으로 구현하세요.

**이상적인 구조:**

```
안정적 + 추상적 (A=1, I=0)
┌─────────────────────────────┐
│  Core Interfaces            │
│  - IRepository              │
│  - IApiClient               │
│  - IStorage                 │
└─────────────────────────────┘
           ↑
           │ (의존)
           │
┌─────────────────────────────┐
│  Adapters (A≈0.5, I≈0.5)   │
│  - LocalStorageAdapter      │
│  - FetchApiClient           │
└─────────────────────────────┘
           ↑
           │ (의존)
           │
불안정 + 구체적 (A=0, I=1)
┌─────────────────────────────┐
│  Features                   │
│  - ProductListPage          │
│  - UserDashboard            │
└─────────────────────────────┘
```

### TypeScript 예제

```typescript
// ✅ 좋은 예: 안정적 + 추상적 (A=1, I=0)

// core/interfaces/IApiClient.ts
export interface IApiClient {
  get<T>(url: string): Promise<T>;
  post<T>(url: string, data: unknown): Promise<T>;
}

export interface IRepository<T> {
  findAll(): Promise<T[]>;
  findById(id: string): Promise<T | null>;
  create(data: T): Promise<T>;
}

// A = 2/2 = 1 (완전 추상)
// I = 0 (많은 컴포넌트가 이 인터페이스에 의존)
```

```typescript
// 중간: 중간 추상화 (A≈0.5, I≈0.5)

// adapters/FetchApiClient.ts
import type { IApiClient } from "@/core/interfaces/IApiClient";

export class FetchApiClient implements IApiClient {
  async get<T>(url: string): Promise<T> {
    const response = await fetch(url);
    return response.json();
  }

  async post<T>(url: string, data: unknown): Promise<T> {
    const response = await fetch(url, {
      method: "POST",
      body: JSON.stringify(data),
    });
    return response.json();
  }
}

// A = 1/2 = 0.5 (IApiClient는 추상, FetchApiClient는 구체)
// I ≈ 0.5 (일부 컴포넌트가 의존)
```

```typescript
// ✅ 좋은 예: 불안정 + 구체적 (A=0, I=1)

// features/product/ProductRepository.ts
import type { IRepository } from "@/core/interfaces/IRepository";
import type { IApiClient } from "@/core/interfaces/IApiClient";
import type { Product } from "./types";

export class ProductRepository implements IRepository<Product> {
  constructor(private apiClient: IApiClient) {}

  async findAll(): Promise<Product[]> {
    return this.apiClient.get<Product[]>("/api/products");
  }

  async findById(id: string): Promise<Product | null> {
    return this.apiClient.get<Product>(`/api/products/${id}`);
  }

  async create(data: Product): Promise<Product> {
    return this.apiClient.post<Product>("/api/products", data);
  }
}

// Na = 0 (이 파일에는 인터페이스 정의가 없고, 인터페이스를 구현만 함)
// Nc = 1 (ProductRepository 클래스 1개)
// A = 0/1 = 0 (구체 클래스, 추상화 없음)
// I ≈ 1 (외부에 의존하고, 자신을 의존하는 곳 거의 없음)
```

```typescript
// ❌ 나쁜 예: 안정적 + 구체적 (고통의 구역)

// shared/utils/ProductHelper.ts
export class ProductHelper {
  // 구체 클래스
  static formatPrice(price: number): string {
    return `${price.toLocaleString()}원`;
  }

  static calculateDiscount(price: number, rate: number): number {
    return price * (1 - rate);
  }

  // ... 100개의 메서드
}

// 문제점:
// - 많은 컴포넌트가 ProductHelper에 의존 (안정적, I≈0)
// - 구체 클래스라 확장 어려움 (A=0)
// - 변경 시 모든 의존 컴포넌트에 영향
// → 고통의 구역 (0,0)
```

**함수형 예제: 쓸모없는 구역 피하기**

```typescript
// ❌ 나쁜 예: 불안정 + 추상적 (쓸모없는 구역)

// features/unused/AbstractDataProcessor.ts
export interface IDataProcessor<T> {
  // 추상
  process(data: T[]): T[];
  filter(data: T[]): T[];
  transform(data: T[]): T[];
}

// 문제점:
// - 아무도 사용하지 않는 추상 인터페이스 (I=1)
// - 완전히 추상적 (A=1)
// → 쓸모없는 구역 (1,1)
// → 삭제 대상!
```

---

## 4. 세 원칙의 통합

### 컴포넌트 결합 원칙 요약

```
┌────────────────────────────────────────────┐
│  ADP: 의존성 비순환 원칙                    │
│  → 순환 의존성을 제거하라                   │
│  → DIP를 활용하여 순환 끊기                │
├────────────────────────────────────────────┤
│  SDP: 안정된 의존성 원칙                    │
│  → 불안정한 것이 안정적인 것에 의존         │
│  → I = Fan-out / (Fan-in + Fan-out)       │
├────────────────────────────────────────────┤
│  SAP: 안정된 추상화 원칙                    │
│  → 안정적일수록 추상적이어야 함             │
│  → A = Na / Nc                            │
│  → D = |A + I - 1| (주계열 거리)          │
└────────────────────────────────────────────┘
```

### 실전 적용 흐름

```
1. ADP 적용
   ↓
순환 의존성 제거 (DIP 활용)
   ↓
2. SDP 적용
   ↓
안정성 방향으로 의존성 정렬
   ↓
3. SAP 적용
   ↓
안정적인 컴포넌트를 추상화
   ↓
주계열에 가까운 구조 달성
```

### Frontend 아키텍처 예시

```
레이어 구조 (위에서 아래로 의존)

┌─────────────────────────────────────┐
│  Features (I≈1, A≈0)                │  ← 불안정 + 구체적
│  - ProductList, UserDashboard       │
└─────────────────────────────────────┘
              ↓ (ADP, SDP 준수)
┌─────────────────────────────────────┐
│  Application (I≈0.5, A≈0.5)        │  ← 중간
│  - UseCases, Services               │
└─────────────────────────────────────┘
              ↓
┌─────────────────────────────────────┐
│  Domain (I≈0, A≈1)                  │  ← 안정적 + 추상적
│  - Interfaces, Types                │
└─────────────────────────────────────┘
              ↑
┌─────────────────────────────────────┐
│  Infrastructure (I≈0.5, A≈0)       │  ← 외부 구현체
│  - ApiClient, LocalStorage          │
└─────────────────────────────────────┘

✅ ADP: 순환 없음 (단방향 흐름)
✅ SDP: 불안정 → 안정 방향 의존
✅ SAP: Domain은 추상적, Features는 구체적
```

---

## 정리 요약

### 세 가지 원칙 한눈에 보기

| 원칙    | 목적        | 핵심 메트릭                  | Frontend 적용                   |
| ------- | ----------- | ---------------------------- | ------------------------------- |
| **ADP** | 순환 제거   | DAG 구조                     | DIP로 순환 끊기                 |
| **SDP** | 안정성 방향 | I = Fan-out/(Fan-in+Fan-out) | Utils는 안정, Features는 불안정 |
| **SAP** | 추상화 균형 | A = Na/Nc, D = \|A+I-1\|     | 안정적인 것은 인터페이스로      |
