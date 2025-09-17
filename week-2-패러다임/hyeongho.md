# 클린 아키텍처 스터디 - Week 2: 패러다임

## 4장: 구조적 프로그래밍

### 핵심 개념

**구조적 프로그래밍의 본질은 "제어 흐름의 직접적 전환을 제거"하는 것입니다.**

- **구조적 프로그래밍의 정의**: 순차(sequence), 선택(selection), 반복(iteration)이라는 세 가지 구조만을 사용하여 프로그램을 작성하는 방법
- **무분별한 점프의 해로움**: goto문과 같은 직접적인 제어 전송은 프로그램의 구조를 해치고 이해를 어렵게 함

### 주요 원칙

1. **분해(Decomposition)**: 프로그램을 증명 가능한 세부 기능 집합으로 재귀적으로 분해
2. **기능적 분해**: 큰 문제를 작은 문제들로 나누어 해결
3. **모듈화**: 각 모듈은 입증 가능하고 독립적으로 테스트 가능해야 함

### 테스트와 증명

- **다익스트라의 명제**: "테스트는 버그가 있음을 보여줄 뿐, 버그가 없음을 보장할 수는 없다"
- **반증 가능성**: 테스트를 통해 증명 가능한 세부 기능들이 거짓인지를 증명하려고 시도
- **과학적 방법**: 올바름을 증명할 수는 없지만, 틀렸음을 증명할 수는 있음

### Frontend 적용

```typescript
// ❌ 복잡한 제어 흐름 (안티패턴)
function processUserDataBad(users: User[]) {
  // goto와 같은 복잡한 제어 흐름은 피해야 함
}

// ✅ 구조적 프로그래밍 원칙 적용
function processUserData(users: User[]): CategorizedUser[] {
  return users
    .filter(isValidUser) // 선택(Selection)
    .map(categorizeUser); // 반복(Iteration)
}

// React에서의 기능적 분해
function UserDashboard() {
  return (
    <div className="user-dashboard">
      <UserSearchBar onSearch={handleSearch} />
      <UserList users={users} loading={loading} />
      <UserStatistics users={users} />
    </div>
  );
}
```

### 현대적 의미

- **구조적 프로그래밍이 오늘날까지 가치 있는 이유**: 반증 가능한 단위를 만들어 낼 수 있는 능력
- **아키텍처 관점**: 기능적 분해를 통해 모듈과 컴포넌트의 기초가 됨

## 5장: 객체 지향 프로그래밍

### 핵심 개념

**객체 지향 프로그래밍의 본질은 "제어 흐름의 간접적 전환(함수 포인터)을 제거"하여 안전한 다형성을 제공하는 것입니다.**

- **세 가지 특성**: 캡슐화(encapsulation), 상속(inheritance), 다형성(polymorphism)
- **가장 중요한 것**: 다형성을 통한 의존성 제어

### 1. 캡슐화 (Encapsulation)

**"데이터와 기능을 하나로 묶어서 숨기기"**

```typescript
// ✅ 캡슐화 적용
class User {
  private _age: number; // private: 외부에서 접근 불가
  private _name: string;

  constructor(name: string, age: number) {
    this._name = name;
    this._age = age;
  }

  // 검증이 포함된 안전한 업데이트
  setAge(newAge: number) {
    if (newAge < 0 || newAge > 120) {
      throw new Error("올바르지 않은 나이입니다!");
    }
    this._age = newAge;
  }

  get age(): number {
    return this._age;
  }
  get name(): string {
    return this._name;
  }
}
```

### 2. 상속 (Inheritance)

**"공통된 것을 물려받기"**

```typescript
// 부모 클래스
abstract class Animal {
  protected name: string;

  constructor(name: string) {
    this.name = name;
  }

  abstract makeSound(): string;
}

// 자식 클래스들
class Dog extends Animal {
  makeSound(): string {
    return `${this.name}: 멍멍!`;
  }
}

class Cat extends Animal {
  makeSound(): string {
    return `${this.name}: 야옹~`;
  }
}
```

### 3. 다형성 (Polymorphism)

**"같은 이름, 다른 행동" - 객체 지향의 핵심!**

```typescript
// 인터페이스로 추상화
interface SoundMaker {
  makeSound(): string;
}

// 다형성 활용
class Zoo {
  makeAllSounds(animals: SoundMaker[]): string[] {
    return animals.map((animal) => animal.makeSound());
  }
}

// React에서 다형성 활용
function AnimalSelector() {
  const [selectedType, setSelectedType] = useState<"dog" | "cat">("dog");
  const [sound, setSound] = useState<string>("");

  const makeSound = () => {
    let animal: SoundMaker;

    // 다형성: 런타임에 구체적인 타입 결정
    switch (selectedType) {
      case "dog":
        animal = new Dog("바둑이");
        break;
      case "cat":
        animal = new Cat("나비");
        break;
    }

    setSound(animal.makeSound());
  };

  return (
    <div>
      <select
        onChange={(e) => setSelectedType(e.target.value as "dog" | "cat")}
      >
        <option value="dog">강아지</option>
        <option value="cat">고양이</option>
      </select>
      <button onClick={makeSound}>소리내기</button>
      <p>{sound}</p>
    </div>
  );
}
```

### 의존성 역전 (Dependency Inversion)

**객체 지향의 진정한 힘은 의존성 제어에 있습니다.**

```typescript
// ❌ 의존성 역전 없이
class MusicPlayer {
  private speaker = new BluetoothSpeaker(); // 구체적인 클래스에 의존

  play(song: string) {
    this.speaker.playSound(song); // 변경이 어려움
  }
}

// ✅ 의존성 역전 적용
interface Speaker {
  playSound(sound: string): void;
}

class MusicPlayer {
  constructor(private speaker: Speaker) {} // 추상화에 의존

  play(song: string) {
    this.speaker.playSound(song); // 어떤 스피커든 사용 가능
  }
}

// React에서 의존성 주입
function MusicApp() {
  const [speakerType, setSpeakerType] = useState<"bluetooth" | "wired">(
    "bluetooth"
  );

  const playMusic = (song: string) => {
    let speaker: Speaker;

    // 의존성 주입: 외부에서 구체적인 구현 결정
    switch (speakerType) {
      case "bluetooth":
        speaker = new BluetoothSpeaker();
        break;
      case "wired":
        speaker = new WiredSpeaker();
        break;
    }

    const player = new MusicPlayer(speaker);
    player.play(song);
  };

  return (
    <div>
      <select
        onChange={(e) =>
          setSpeakerType(e.target.value as "bluetooth" | "wired")
        }
      >
        <option value="bluetooth">블루투스</option>
        <option value="wired">유선</option>
      </select>
      <button onClick={() => playMusic("좋은 노래")}>재생</button>
    </div>
  );
}
```

### 의존성 역전의 장점

```
기존 방식:
UI → Business Rules → Database
(소스코드 의존성이 제어 흐름을 따름)

의존성 역전 후:
UI → Business Rules ← Database
(Database가 Business Rules의 인터페이스에 의존)
```

**결과:**

- **배포 독립성**: 각 컴포넌트를 독립적으로 배포 가능
- **개발 독립성**: 서로 다른 팀이 독립적으로 개발 가능
- **테스트 용이성**: Mock 객체를 이용한 단위 테스트 가능
- **유연성**: 구현체를 런타임에 교체 가능

### 현대적 의미

- **플러그인 아키텍처**: 다형성을 통해 확장 가능한 시스템 구축
- **Clean Architecture**: 의존성 방향을 제어하여 안정적인 아키텍처 설계
- **Frontend 적용**: React의 Props, Context API, 의존성 주입 패턴

## 6장: 함수형 프로그래밍

### 핵심 개념

**함수형 프로그래밍의 본질은 "가변성(mutability)을 제거"하여 동시성 문제를 해결하는 것입니다.**

- **람다 계산법**: 함수형 프로그래밍의 수학적 기초 (1930년대 알론조 처치)
- **불변성**: 함수형 언어에서 변수는 한번 할당되면 절대 변경되지 않는다
- **클로저**: 함수가 정의된 환경의 변수들을 기억하고 접근할 수 있는 기능

### 가변 변수의 문제점

**핵심 원리:**

> **"가변 변수가 없다면 동시성 애플리케이션에서 마주치는 모든 문제가 절대로 생기지 않는다"**

**가변 변수로 인한 주요 문제들:**

1. **경합 조건(Race Condition)**: 여러 스레드가 동시에 같은 변수에 접근할 때 발생
2. **교착 상태(Deadlock)**: 두 개 이상의 스레드가 서로를 기다리며 무한 대기하는 상태
3. **동시 업데이트(Concurrent Update)**: 동시에 같은 데이터를 수정하려 할 때 발생하는 문제

```typescript
// ❌ 가변 상태로 인한 문제 예시
class MutableCounter {
  private count = 0;

  increment() {
    this.count++; // 여러 스레드에서 동시 호출 시 Race Condition 발생 가능
  }

  getCount() {
    return this.count;
  }
}

// ✅ 불변성을 활용한 해결책
interface CounterState {
  readonly count: number;
}

const incrementCounter = (state: CounterState): CounterState => ({
  count: state.count + 1,
});

// React에서의 활용
const useCounter = () => {
  const [state, setState] = useState<CounterState>({ count: 0 });

  const increment = useCallback(() => {
    setState((prevState) => incrementCounter(prevState));
  }, []);

  return { count: state.count, increment };
};
```

### 함수형 프로그래밍의 해결책

#### 1. 불변성 (Immutability)

**기본 원칙:**

- 한 번 할당된 변수는 절대 변경되지 않음
- 새로운 값이 필요하면 새로운 객체를 생성
- 원본 데이터를 보존하면서 새로운 데이터 생성

```typescript
// ✅ 불변성을 활용한 객체 업데이트
interface User {
  readonly id: number;
  readonly name: string;
  readonly email: string;
  readonly preferences: readonly string[];
}

// 새로운 객체를 생성하여 업데이트
const updateUserEmail = (user: User, newEmail: string): User => ({
  ...user,
  email: newEmail,
});

// 배열에 아이템 추가 (원본 배열 보존)
const addPreference = (user: User, preference: string): User => ({
  ...user,
  preferences: [...user.preferences, preference],
});

// React에서의 활용 예시
const UserProfile: React.FC<{ user: User }> = ({ user }) => {
  const [currentUser, setCurrentUser] = useState(user);

  const handleEmailUpdate = (newEmail: string) => {
    // 새로운 객체 생성으로 state 업데이트
    setCurrentUser((prevUser) => updateUserEmail(prevUser, newEmail));
  };

  return (
    <div>
      <p>Email: {currentUser.email}</p>
      <button onClick={() => handleEmailUpdate("new@email.com")}>
        Update Email
      </button>
    </div>
  );
};
```

#### 2. 이벤트 소싱 (Event Sourcing)

**핵심 전략:**

- 상태가 아닌 트랜잭션(이벤트)을 저장하는 전략
- 현재 상태 대신 상태를 변경한 이벤트들을 저장
- 필요 시 이벤트들을 재실행하여 현재 상태를 복원

```typescript
// 이벤트 소싱 패턴 구현
interface BankAccount {
  readonly id: string;
  readonly balance: number;
  readonly isActive: boolean;
}

// 이벤트 타입 정의
type AccountEvent =
  | { type: "ACCOUNT_CREATED"; payload: { id: string; initialBalance: number } }
  | { type: "MONEY_DEPOSITED"; payload: { amount: number } }
  | { type: "MONEY_WITHDRAWN"; payload: { amount: number } }
  | { type: "ACCOUNT_CLOSED"; payload: {} };

// 상태 복원 함수 (Projection)
const projectBankAccount = (events: AccountEvent[]): BankAccount | null => {
  return events.reduce<BankAccount | null>((account, event) => {
    switch (event.type) {
      case "ACCOUNT_CREATED":
        return {
          id: event.payload.id,
          balance: event.payload.initialBalance,
          isActive: true,
        };

      case "MONEY_DEPOSITED":
        return account
          ? {
              ...account,
              balance: account.balance + event.payload.amount,
            }
          : null;

      case "MONEY_WITHDRAWN":
        return account
          ? {
              ...account,
              balance: account.balance - event.payload.amount,
            }
          : null;

      case "ACCOUNT_CLOSED":
        return account
          ? {
              ...account,
              isActive: false,
            }
          : null;

      default:
        return account;
    }
  }, null);
};

// React에서의 활용
const useBankAccount = (accountId: string) => {
  const [events, setEvents] = useState<AccountEvent[]>([]);
  const account = useMemo(() => projectBankAccount(events), [events]);

  const executeCommand = useCallback((event: AccountEvent) => {
    setEvents((prevEvents) => [
      ...prevEvents,
      {
        ...event,
        timestamp: Date.now(),
      },
    ]);
  }, []);

  return { account, executeCommand, history: events };
};
```

### 세 패러다임의 본질

각 패러다임은 프로그래밍에서 특정 능력을 **제거**함으로써 규율을 부과합니다:

| 패러다임                            | 제거 요소                      | 목표                        | 현대적 활용              |
| ----------------------------------- | ------------------------------ | --------------------------- | ------------------------ |
| **구조적 프로그래밍** (1960년대)    | goto문 (직접적 제어 전환)      | 예측 가능한 제어 흐름       | if/else, for/while       |
| **객체 지향 프로그래밍** (1970년대) | 함수 포인터 (간접적 제어 전환) | 안전한 다형성과 의존성 제어 | 인터페이스, 상속, 캡슐화 |
| **함수형 프로그래밍** (1980년대~)   | 가변성 (변수 할당)             | 동시성 문제 해결            | 불변 데이터, 순수 함수   |

### Frontend에서 세 패러다임의 조화

```typescript
// 세 패러다임의 조화: React 컴포넌트에서
interface TodoItem {
  readonly id: string;
  readonly text: string;
  readonly completed: boolean;
}

// 함수형: 순수 함수로 상태 변경 로직
const toggleTodo = (todo: TodoItem): TodoItem => ({
  ...todo,
  completed: !todo.completed,
});

// 객체 지향: 인터페이스를 통한 추상화
interface TodoRepository {
  save(todo: TodoItem): Promise<void>;
  findById(id: string): Promise<TodoItem | null>;
}

// 구조적: 명확한 제어 흐름
const TodoComponent: React.FC<{ repository: TodoRepository }> = ({
  repository,
}) => {
  const [todos, setTodos] = useState<TodoItem[]>([]);

  const handleToggle = useCallback(
    async (id: string) => {
      // 구조적: if/else를 통한 명확한 흐름
      const todo = todos.find((t) => t.id === id);
      if (!todo) return;

      // 함수형: 불변 업데이트
      const updatedTodo = toggleTodo(todo);
      setTodos((prevTodos) =>
        prevTodos.map((t) => (t.id === id ? updatedTodo : t))
      );

      // 객체 지향: 인터페이스를 통한 추상화된 호출
      await repository.save(updatedTodo);
    },
    [todos, repository]
  );

  return (
    <ul>
      {todos.map((todo) => (
        <li key={todo.id}>
          <input
            type="checkbox"
            checked={todo.completed}
            onChange={() => handleToggle(todo.id)}
          />
          {todo.text}
        </li>
      ))}
    </ul>
  );
};
```

### 소프트웨어의 본질

> **"소프트웨어는 순차(sequence), 분기(selection), 반복(iteration), 참조(indirection)으로 구성된다. 그 이상도 그 이하도 아니다."**

**Frontend 개발에서의 네 가지 요소:**

1. **순차(Sequence)**: 컴포넌트 라이프사이클, Hook 호출 순서
2. **분기(Selection)**: 조건부 렌더링, 사용자 인터랙션 분기
3. **반복(Iteration)**: 리스트 렌더링, 배열 메서드 활용
4. **참조(Indirection)**: Props 전달, Context API, 함수 참조

### 정리 요약

함수형 프로그래밍은 **가변성을 제거**하여 동시성 문제를 해결하는 패러다임입니다. Frontend 개발에서는:

- **React Hooks**를 통한 함수형 컴포넌트 구현
- **불변 상태 관리**로 예측 가능한 UI 구축
- **순수 함수**로 테스트하기 쉬운 비즈니스 로직 작성
- **이벤트 소싱** 패턴으로 상태 변경 추적

세 패러다임(구조적, 객체 지향, 함수형)은 각각 특정 문제를 해결하기 위해 특정 기능을 **제거**하는 규율을 제공하며, 현대 Frontend 개발에서는 이 모든 패러다임을 적절히 조합하여 사용합니다.

---

## 🎯 Week 2 핵심 정리

### 패러다임별 핵심 메시지

1. **구조적 프로그래밍**: "분해와 증명"

   - 큰 문제를 작은 단위로 나누어 테스트 가능하게 만들기

2. **객체 지향 프로그래밍**: "의존성 제어"

   - 다형성을 통해 플러그인 아키텍처 구현하기

3. **함수형 프로그래밍**: "불변성과 동시성"
   - 가변성을 제거하여 예측 가능한 프로그램 만들기

### Frontend 개발자가 기억해야 할 것

**"Clean Architecture는 이 세 패러다임을 조합하여 변경에 유연하고, 테스트하기 쉬우며, 이해하기 쉬운 시스템을 만드는 것입니다."**
