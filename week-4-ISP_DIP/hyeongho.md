# 10장: ISP - 인터페이스 분리 원칙

## 주요 개념

### ISP란?

> 💡 **인터페이스 분리 원칙 (Interface Segregation Principle)**
> 클라이언트는 자신이 사용하지 않는 메서드에 의존하지 않아야 한다.

### 핵심 문제: 불필요한 의존성

**문제 상황 시각화:**

```
┌─────────────────┐
│   OPS 클래스     │
├─────────────────┤
│ op1() method    │ ← User1 사용
│ op2() method    │ ← User2 사용 (User1도 의존!)
│ op3() method    │ ← User3 사용 (User1도 의존!)
└─────────────────┘

문제점:
- User1은 op1만 사용하지만, op2/op3에도 의존
- op2가 변경되면 User1도 재컴파일/재배포 필요
```

**문제점:**

- **User1**은 `op2`와 `op3`을 전혀 사용하지 않음
- 그러나 User1의 소스 코드는 이 두 메서드에 **의존하게 됨**
- **결과**: OPS 클래스에서 `op2`의 소스 코드가 변경되면 User1도 다시 **컴파일하고 재배포**해야 함

### 언어 타입과 ISP

ISP는 **언어의 타입 시스템**에 따라 영향이 달라진다.

| 구분                 | 정적 타입 언어        | 동적 타입 언어     |
| -------------------- | --------------------- | ------------------ |
| **언어 예시**        | TypeScript, Java, C++ | JavaScript, Python |
| **타입 체크**        | 컴파일 타임           | 런타임             |
| **소스 코드 의존성** | ✅ 명확히 존재        | ❌ 거의 없음       |
| **재컴파일**         | ✅ 필요               | ❌ 불필요          |
| **재배포**           | ✅ 필요               | ❌ 불필요          |
| **유연성**           | 낮음                  | 높음               |
| **결합도**           | 높음                  | 낮음               |

**핵심:**

- 동적 타입 언어를 사용하면 정적 타입 언어보다 **유연하며 결합도가 낮은** 시스템을 만들 수 있는 이유
- 하지만 TypeScript는 정적 타입이지만, 명확한 인터페이스 분리로 유연성 확보 가능

## Frontend 관점의 ISP

> 💡 **Frontend 관점**: React 컴포넌트와 훅(Hook) 설계 시 필요한 인터페이스만 제공하여 불필요한 리렌더링과 의존성을 줄일 수 있다.

### 문제가 있는 예제: 비대한 Props 인터페이스

**함수형 프로그래밍 예제 (문제):**

```typescript
// ❌ 나쁜 예: 모든 기능이 하나의 인터페이스에 몰려있음
interface UserOperations {
  // User1만 사용
  getUserProfile: (id: string) => Promise<User>;

  // User2만 사용
  updateUserEmail: (id: string, email: string) => Promise<void>;

  // User3만 사용
  deleteUser: (id: string) => Promise<void>;
}

// User1 컴포넌트는 getUserProfile만 필요한데 모든 메서드에 의존
const UserProfile = ({ operations }: { operations: UserOperations }) => {
  const [user, setUser] = useState<User | null>(null);

  useEffect(() => {
    // getUserProfile만 사용하지만 다른 메서드가 변경되어도 리렌더링 발생 가능
    operations.getUserProfile("123").then(setUser);
  }, [operations]);

  return <div>{user?.name}</div>;
};
```

### ISP를 적용한 개선 예제

**개선 전략 시각화:**

```
❌ 개선 전 (ISP 위반)
┌──────────────────────────┐
│   UserOperations         │
│  - getUserProfile()      │
│  - updateUserEmail()     │
│  - deleteUser()          │
└──────────────────────────┘
         ↑    ↑    ↑
         │    │    │
    ┌────┘    │    └────┐
    │         │         │
UserProfile  UserEmail  UserDelete
(op1만 필요)  (op1,2필요) (op3만 필요)

모든 컴포넌트가 모든 메서드에 의존!

✅ 개선 후 (ISP 적용)
GetUserProfile()  UpdateUserEmail()  DeleteUser()
      ↑                  ↑                ↑
      │                  │                │
UserProfile      UserEmailEditor    UserDeleteButton
(필요한 것만)      (필요한 것만)       (필요한 것만)

각 컴포넌트가 필요한 인터페이스만 의존!
```

**함수형 프로그래밍 예제 (개선):**

```typescript
// ✅ 좋은 예: 인터페이스를 역할별로 분리
type GetUserProfile = (id: string) => Promise<User>;
type UpdateUserEmail = (id: string, email: string) => Promise<void>;
type DeleteUser = (id: string) => Promise<void>;

// User1 컴포넌트는 필요한 함수만 의존
const UserProfile = ({
  getUserProfile,
}: {
  getUserProfile: GetUserProfile;
}) => {
  const [user, setUser] = useState<User | null>(null);

  useEffect(() => {
    // 오직 getUserProfile 함수에만 의존
    getUserProfile("123").then(setUser);
  }, [getUserProfile]);

  return <div>{user?.name}</div>;
};

// User2 컴포넌트
const UserEmailEditor = ({
  getUserProfile,
  updateUserEmail,
}: {
  getUserProfile: GetUserProfile;
  updateUserEmail: UpdateUserEmail;
}) => {
  const [user, setUser] = useState<User | null>(null);
  const [email, setEmail] = useState("");

  const handleUpdate = async () => {
    await updateUserEmail(user!.id, email);
  };

  return (
    <div>
      <input value={email} onChange={(e) => setEmail(e.target.value)} />
      <button onClick={handleUpdate}>Update</button>
    </div>
  );
};

// User3 컴포넌트
const UserDeleteButton = ({ deleteUser }: { deleteUser: DeleteUser }) => {
  return <button onClick={() => deleteUser("123")}>Delete</button>;
};
```

### Custom Hook으로 ISP 적용

**Hook 분리 시각화:**

```
컴포넌트 계층              Hook 계층 (ISP 적용)
──────────────────        ──────────────────────

UserProfile           →   useUserProfile()
                             └─ getUserProfile()

UserEmailEditor       →   useUserProfile()
                      →   useUserEmail()
                             └─ updateUserEmail()

UserDeleteButton      →   useUserDeletion()
                             └─ deleteUser()

✅ 각 컴포넌트는 필요한 Hook만 의존
✅ Hook은 단일 책임만 수행
```

**함수형 프로그래밍 예제 (Hook 분리):**

```typescript
// ✅ 각 기능을 독립적인 Hook으로 분리
const useUserProfile = () => {
  const getUserProfile: GetUserProfile = async (id) => {
    const response = await fetch(`/api/users/${id}`);
    return response.json();
  };

  return { getUserProfile };
};

const useUserEmail = () => {
  const updateUserEmail: UpdateUserEmail = async (id, email) => {
    await fetch(`/api/users/${id}/email`, {
      method: "PUT",
      body: JSON.stringify({ email }),
    });
  };

  return { updateUserEmail };
};

const useUserDeletion = () => {
  const deleteUser: DeleteUser = async (id) => {
    await fetch(`/api/users/${id}`, { method: "DELETE" });
  };

  return { deleteUser };
};

// 사용: 각 컴포넌트는 필요한 Hook만 사용
const UserProfile = () => {
  const { getUserProfile } = useUserProfile(); // 필요한 것만 의존
  // ...
};

const UserEmailEditor = () => {
  const { getUserProfile } = useUserProfile();
  const { updateUserEmail } = useUserEmail(); // 필요한 것만 의존
  // ...
};
```

### 상태 관리에서의 ISP

**스토어 분리 시각화:**

```
❌ ISP 위반: 하나의 거대한 스토어
┌────────────────────────────┐
│       AppStore             │
│  - user: User              │
│  - products: Product[]     │
│  - cart: CartItem[]        │
│  - setUser()               │
│  - addProduct()            │
│  - addToCart()             │
│  - removeFromCart()        │
└────────────────────────────┘
      ↑         ↑         ↑
      │         │         │
UserProfile  ProductList  CartList
(user만 필요) (products만) (cart만 필요)

모든 컴포넌트가 전체 스토어에 의존!

✅ ISP 적용: 도메인별 스토어 분리
┌─────────────┐  ┌──────────────┐  ┌──────────────┐
│  UserStore  │  │ ProductStore │  │  CartStore   │
│  - user     │  │  - products  │  │  - cart      │
│  - setUser()│  │  - addProd() │  │  - addCart() │
└─────────────┘  └──────────────┘  └──────────────┘
      ↑                 ↑                  ↑
      │                 │                  │
UserProfile        ProductList         CartList

각 컴포넌트가 필요한 스토어만 의존!
```

**함수형 프로그래밍 예제 (Zustand):**

```typescript
// ❌ 나쁜 예: 하나의 거대한 스토어
interface AppStore {
  user: User | null;
  products: Product[];
  cart: CartItem[];
  setUser: (user: User) => void;
  addProduct: (product: Product) => void;
  addToCart: (item: CartItem) => void;
  removeFromCart: (id: string) => void;
}

// ✅ 좋은 예: 도메인별로 스토어 분리
type UserStore = {
  user: User | null;
  setUser: (user: User) => void;
};

type ProductStore = {
  products: Product[];
  addProduct: (product: Product) => void;
};

type CartStore = {
  cart: CartItem[];
  addToCart: (item: CartItem) => void;
  removeFromCart: (id: string) => void;
};

// 컴포넌트는 필요한 스토어만 선택
const UserProfile = () => {
  const user = useUserStore((state) => state.user); // UserStore만 의존
  return <div>{user?.name}</div>;
};

const CartList = () => {
  const { cart, removeFromCart } = useCartStore(); // CartStore만 의존
  return (
    <ul>
      {cart.map((item) => (
        <li key={item.id}>
          {item.name}
          <button onClick={() => removeFromCart(item.id)}>Remove</button>
        </li>
      ))}
    </ul>
  );
};
```

## 핵심 교훈

> ⚠️ **불필요한 짐을 실은 무언가에 의존하면 예상치도 못한 문제에 빠질 수 있다.**

### ISP의 이점

| 이점                       | 설명                                       | Frontend 적용            |
| -------------------------- | ------------------------------------------ | ------------------------ |
| **재컴파일/재배포 최소화** | 불필요한 의존성 제거로 변경 영향 범위 감소 | 컴포넌트 리렌더링 최소화 |
| **명확한 책임 분리**       | 각 클라이언트는 자신이 필요한 것만 의존    | 컴포넌트 역할 명확화     |
| **테스트 용이성**          | 작은 인터페이스는 모킹과 테스트가 쉬움     | Hook 단위 테스트 용이    |
| **유연성 향상**            | 변경에 유연하게 대응                       | 스토어 독립적 확장       |

### Frontend에서의 실천 방법

```
                    ISP 실천 방법
                         │
        ┌────────────────┼────────────────┐
        │                │                │
    컴포넌트            Hook            상태관리
        │                │                │
  ┌─────┴─────┐    ┌─────┴─────┐    ┌─────┴─────┐
Props 최소화   기능별 분리   도메인별     타입
단일 책임      조합 가능     스토어      역할별 세분화
                          선택적 구독   함수 타입 우선
```

**체크리스트:**

- ✅ 컴포넌트 Props는 최소한으로 유지
- ✅ Custom Hook은 단일 책임을 가지도록 분리
- ✅ 상태 관리는 도메인별로 스토어 분리
- ✅ 타입 정의는 역할별로 세분화
- ✅ 함수형 접근으로 작은 단위의 순수 함수 조합

## 정리 요약

ISP는 **"필요한 것만 의존하라"**는 원칙이다. 특히 정적 타입 언어에서는 불필요한 의존성이 재컴파일과 재배포를 유발하므로 인터페이스를 작고 명확하게 분리해야 한다. 동적 타입 언어는 이러한 제약이 적지만, 설계의 명확성과 유지보수성을 위해 ISP를 적용하는 것이 바람직하다.

Frontend 개발에서는 컴포넌트, Hook, 상태 관리를 설계할 때 ISP를 적용하여 불필요한 리렌더링과 의존성을 줄이고, 테스트 가능하고 유지보수하기 쉬운 코드를 작성할 수 있다.

---

# 11장: DIP - 의존성 역전 원칙

## 주요 개념

### DIP란?

> 💡 **의존성 역전 원칙 (Dependency Inversion Principle)**
> 소스 코드 의존성이 추상(abstraction)에 의존하며, 구체(concretion)에는 의존하지 않는 시스템

**핵심:**

- 유연성이 극대화된 시스템
- 추상적인 선언(인터페이스)만을 참조
- 구체적인 구현에 의존하면 안 됨

### 의존성의 방향

```
❌ 잘못된 의존성 방향 (DIP 위반)
┌──────────────┐      ┌──────────────────┐
│  상위 모듈   │ ───→ │  구체 클래스      │
│ (비즈니스)   │      │  (변동성 큼)      │
└──────────────┘      └──────────────────┘
직접 의존 → 구체 클래스 변경 시 상위 모듈도 영향

✅ DIP 적용 (의존성 역전)
┌──────────────┐      ┌──────────────────┐
│  상위 모듈   │ ───→ │   인터페이스      │
│ (비즈니스)   │      │   (추상)          │
└──────────────┘      └──────────────────┘
                             ↑
                             │ 구현
                      ┌──────┴───────┐
                      │  구체 클래스  │
                      │  (변동성 큼)  │
                      └──────────────┘
추상에 의존 → 구체 클래스 변경해도 상위 모듈 안전
```

### 안정된 추상화

**원칙:**

- **인터페이스는 구현체보다 변동성이 낮다**
- 안정된 소프트웨어 아키텍처 = 변동성이 큰 구현체 지양 + 안정된 추상 인터페이스 선호

**변동성이란?**

- 구체적인 요소 = 개발 중 자주 변경될 수밖에 없는 모듈
- 안정성이 보장된 환경(OS, 플랫폼)은 예외 → 변경되지 않는다면 의존 가능

## DIP 실천 규칙

### 1. 변동성이 큰 구체 클래스를 참조하지 마라

```
❌ 나쁜 예
import ConcreteService from './ConcreteService';

✅ 좋은 예
import { ServiceInterface } from './ServiceInterface';
```

- 대신 **추상 인터페이스를 참조**하라
- 객체 생성 방식을 강하게 제약 → **추상 팩토리 사용**을 강제

### 2. 변동성이 큰 구체 클래스로부터 파생하지 마라

```
❌ 나쁜 예
class MyService extends ConcreteService { }

✅ 좋은 예
class MyService implements ServiceInterface { }
```

- 상속은 의존성을 제거할 수 없게 만듦
- 실제로는 그 의존성을 상속하게 됨

### 3. 구체 함수를 오버라이드 하지 마라

```
❌ 나쁜 예
class Child extends Parent {
  // 구체 함수 오버라이드
  doSomething() { ... }
}

✅ 좋은 예
interface Strategy {
  execute(): void;
}

class ConcreteStrategy implements Strategy {
  execute() { ... }
}
```

- 의존성을 상속하게 됨
- 대신 **추상 함수**를 선언하고 구현하라

### 4. 구체적이며 변동성이 크다면 절대로 그 이름을 언급하지 마라

- DIP 원칙을 다른 방식으로 표현한 것
- 구체 클래스의 이름 자체를 코드에서 제거

## Frontend 관점의 DIP

> 💡 **Frontend 관점**: API 클라이언트, 외부 라이브러리, 상태 관리 등 구체적인 구현에 직접 의존하지 않고, 추상 인터페이스를 통해 의존성을 역전시켜 테스트와 변경에 유연한 구조를 만든다.

### 문제: API 클라이언트 직접 의존

**함수형 프로그래밍 예제 (문제):**

```typescript
// ❌ 나쁜 예: 구체 클래스(axios)에 직접 의존
import axios from "axios";

const UserProfile = () => {
  const [user, setUser] = useState<User | null>(null);

  useEffect(() => {
    // axios에 직접 의존 → axios 변경 시 모든 컴포넌트 수정 필요
    axios.get("/api/users/123").then((res) => setUser(res.data));
  }, []);

  return <div>{user?.name}</div>;
};
```

### 해결: 추상 인터페이스로 의존성 역전

**함수형 프로그래밍 예제 (개선):**

```typescript
// ✅ 좋은 예: 추상 인터페이스에 의존

// 1. 추상 인터페이스 정의
type UserRepository = {
  getUser: (id: string) => Promise<User>;
};

// 2. 구체 구현 (axios 사용)
const createHttpUserRepository = (): UserRepository => ({
  getUser: async (id) => {
    const response = await axios.get(`/api/users/${id}`);
    return response.data;
  },
});

// 3. 컴포넌트는 추상 인터페이스에만 의존
const UserProfile = ({ userRepo }: { userRepo: UserRepository }) => {
  const [user, setUser] = useState<User | null>(null);

  useEffect(() => {
    // UserRepository 인터페이스에만 의존
    userRepo.getUser("123").then(setUser);
  }, [userRepo]);

  return <div>{user?.name}</div>;
};

// 4. 사용 (의존성 주입)
const App = () => {
  const userRepo = createHttpUserRepository();
  return <UserProfile userRepo={userRepo} />;
};
```

**의존성 역전 시각화:**

```
❌ Before (DIP 위반)
UserProfile 컴포넌트
       ↓ (직접 의존)
    axios 라이브러리
(axios 변경 시 컴포넌트도 변경)

✅ After (DIP 적용)
UserProfile 컴포넌트
       ↓
UserRepository 인터페이스 (추상)
       ↑ (구현)
HttpUserRepository (구체)
       ↓
    axios 라이브러리
(axios 변경 시 HttpUserRepository만 변경)
```

### 추상 팩토리 패턴 적용

**함수형 프로그래밍 예제:**

```typescript
// ✅ 팩토리 함수로 구체 구현 생성 캡슐화
type RepositoryFactory = {
  createUserRepository: () => UserRepository;
  createProductRepository: () => ProductRepository;
};

const createHttpRepositoryFactory = (): RepositoryFactory => ({
  createUserRepository: () => ({
    getUser: async (id) => {
      const response = await axios.get(`/api/users/${id}`);
      return response.data;
    },
  }),

  createProductRepository: () => ({
    getProducts: async () => {
      const response = await axios.get("/api/products");
      return response.data;
    },
  }),
});

// Mock 팩토리 (테스트용)
const createMockRepositoryFactory = (): RepositoryFactory => ({
  createUserRepository: () => ({
    getUser: async (id) => ({ id, name: "Test User" }),
  }),

  createProductRepository: () => ({
    getProducts: async () => [{ id: "1", name: "Test Product" }],
  }),
});

// 사용
const App = () => {
  const factory =
    process.env.NODE_ENV === "test"
      ? createMockRepositoryFactory()
      : createHttpRepositoryFactory();

  const userRepo = factory.createUserRepository();

  return <UserProfile userRepo={userRepo} />;
};
```

**팩토리 패턴 시각화:**

```
┌─────────────────────────┐
│  Application (상위)      │
└─────────────────────────┘
           ↓
┌─────────────────────────┐
│  RepositoryFactory      │ ← 추상 인터페이스
└─────────────────────────┘
           ↑
    ┌──────┴──────┐
    │             │
HttpFactory   MockFactory  ← 구체 구현
    │             │
    ↓             ↓
  axios      Mock Data
```

### 상태 관리에서의 DIP

**함수형 프로그래밍 예제 (Zustand):**

```typescript
// ✅ 상태 관리 인터페이스 정의 (추상)
type StateStore<T> = {
  getState: () => T;
  setState: (updater: (state: T) => T) => void;
  subscribe: (listener: (state: T) => void) => () => void;
};

// Zustand 구현 (구체)
const createZustandStore = <T>(initialState: T): StateStore<T> => {
  const store = create<T>(() => initialState);

  return {
    getState: store.getState,
    setState: (updater) => store.setState(updater),
    subscribe: store.subscribe
  };
};

// 컴포넌트는 StateStore 인터페이스에만 의존
const useStore = <T>(store: StateStore<T>) => {
  const [state, setState] = useState(store.getState());

  useEffect(() => {
    return store.subscribe(setState);
  }, [store]);

  return state;
};

// 사용 (Zustand → Redux로 변경해도 컴포넌트는 영향 없음)
const userStore = createZustandStore({ user: null });

const UserProfile = () => {
  const { user } = useStore(userStore);
  return <div>{user?.name}</div>;
};
```

### 외부 라이브러리 의존성 역전

**함수형 프로그래밍 예제 (날짜 라이브러리):**

```typescript
// ✅ 날짜 처리 인터페이스 (추상)
type DateFormatter = {
  format: (date: Date, format: string) => string;
  parse: (dateString: string) => Date;
};

// date-fns 구현 (구체)
const createDateFnsFormatter = (): DateFormatter => ({
  format: (date, formatStr) => dateFnsFormat(date, formatStr),
  parse: (dateString) => dateFnsParse(dateString, "yyyy-MM-dd", new Date()),
});

// moment.js 구현 (구체 - 쉽게 교체 가능)
const createMomentFormatter = (): DateFormatter => ({
  format: (date, formatStr) => moment(date).format(formatStr),
  parse: (dateString) => moment(dateString).toDate(),
});

// 컴포넌트는 DateFormatter 인터페이스에만 의존
const DateDisplay = ({
  date,
  formatter,
}: {
  date: Date;
  formatter: DateFormatter;
}) => {
  return <div>{formatter.format(date, "yyyy-MM-dd")}</div>;
};

// 사용 (date-fns → moment로 변경해도 컴포넌트는 불변)
const App = () => {
  const formatter = createDateFnsFormatter();
  return <DateDisplay date={new Date()} formatter={formatter} />;
};
```

## 제어 흐름과 의존성 역전

> ⚠️ **핵심**: 제어 흐름은 소스 코드 의존성과는 **정반대 방향**으로 흐른다.

```
제어 흐름 (런타임)
Application → ServiceInterface 구현체 → Database
     ↓              ↓                      ↓
   실행          메서드 호출              저장

소스 코드 의존성 (컴파일타임)
Application → ServiceInterface ← ServiceImpl
     ↓              ↑                 ↓
  import       implements         import

의존성이 역전됨! (DIP)
```

**이유:**

- 소스 코드 의존성은 제어 흐름과 반대 방향으로 역전
- 이러한 이유로 **"의존성 역전"** 이라고 부름

## 구체 컴포넌트 (Main)

**현실:**

- DIP 위배를 모두 없앨 수는 없음
- 누군가는 구체 클래스의 인스턴스를 생성해야 함

**해결책:**

- DIP를 위배하는 클래스들을 **소수의 구체 컴포넌트**로 모음
- 시스템의 나머지 부분과 **분리**
- 이러한 컴포넌트를 **"Main"** 이라고 함 (main 함수 포함)

```
시스템 아키텍처

┌─────────────────────────────────────┐
│         비즈니스 로직 (추상)          │
│  - 인터페이스에만 의존               │
│  - DIP 준수                         │
└─────────────────────────────────────┘
              ↑
              │ 의존성 주입
┌─────────────────────────────────────┐
│         Main 컴포넌트 (구체)         │
│  - 구체 클래스 인스턴스 생성          │
│  - 의존성 주입 설정                  │
│  - DIP 위배 허용 (격리된 영역)        │
└─────────────────────────────────────┘
              ↓
         구체 구현체들
```

**Frontend에서의 Main:**

```typescript
// ✅ main.tsx - 구체 구현을 생성하고 주입
import ReactDOM from "react-dom/client";
import App from "./App";

// Main에서만 구체 클래스 생성 (DIP 위배 허용)
const httpClient = axios.create({ baseURL: "/api" });
const userRepo = createHttpUserRepository(httpClient);
const productRepo = createHttpProductRepository(httpClient);

// 의존성 주입
ReactDOM.createRoot(document.getElementById("root")!).render(
  <App userRepo={userRepo} productRepo={productRepo} />
);
```

## 핵심 교훈

### DIP의 이점

| 이점              | 설명                                | Frontend 적용           |
| ----------------- | ----------------------------------- | ----------------------- |
| **유연성**        | 구체 구현 교체 용이                 | axios → fetch 쉽게 변경 |
| **테스트 용이성** | Mock 객체로 대체 가능               | 컴포넌트 단위 테스트    |
| **변경 격리**     | 구체 클래스 변경이 상위에 영향 없음 | 라이브러리 변경 안전    |
| **의존성 명확화** | 인터페이스로 계약 명시              | API 계약 명확           |

### Frontend에서의 실천 방법

```
                DIP 실천 방법
                      │
       ┌──────────────┼──────────────┐
       │              │              │
   API 계층       상태 관리      외부 라이브러리
       │              │              │
인터페이스 정의   Store 추상화   Adapter 패턴
의존성 주입      Factory 패턴    인터페이스화
```

**체크리스트:**

- ✅ API 클라이언트는 인터페이스로 추상화
- ✅ 상태 관리 라이브러리는 직접 노출하지 않음
- ✅ 외부 라이브러리는 Adapter 패턴으로 감싸기
- ✅ 의존성 주입은 최상위(Main)에서만
- ✅ 테스트용 Mock 구현 준비

## 정리 요약

DIP는 **"추상에 의존하고, 구체에 의존하지 마라"**는 원칙이다. 소스 코드 의존성을 제어 흐름과 반대 방향으로 역전시켜, 변동성이 큰 구체 클래스의 변경이 상위 모듈에 영향을 주지 않도록 한다.

Frontend 개발에서는 API 클라이언트, 상태 관리, 외부 라이브러리 등을 추상 인터페이스로 감싸고, 의존성 주입 패턴과 팩토리 패턴을 활용하여 테스트 가능하고 유연한 아키텍처를 구축할 수 있다.

구체 클래스의 인스턴스 생성은 Main 컴포넌트로 격리하여, 시스템의 나머지 부분은 순수하게 추상에만 의존하도록 설계한다.

---

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
