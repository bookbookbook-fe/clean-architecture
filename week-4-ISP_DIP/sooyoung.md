# 🧹 클린 아키텍처 스터디 4주차 (10~11장) 요약

---

### 10. 인터페이스 분리 원칙 (ISP: Interface Segregation Principle)

> "클라이언트는 자신이 사용하지 않는 메서드에 의존하면 안 된다."

- **핵심**

  - 인터페이스(계약)는 작고 특정 클라이언트를 위한 **맞춤형**이어야 한다.
  - 비대한 범용 인터페이스는 일부 클라이언트에게 쓸모없는 의존을 강요한다.
  - 변화의 축이 다른 기능 묶음은 **분리된 인터페이스**로 나눠야 한다.

- **냄새**

  - "거대한 인터페이스" 하나에 너무 많은 역할이 몰려 있음.
  - 구현체가 `throw new UnsupportedOperationException()` 같은 미구현 메서드를 많이 품고 있음.
  - 변경 시, 쓰지 않는 메서드 시그니처 변경에도 불필요하게 재컴파일/배포가 전파됨.

- **좋은 분리 기준**

  - **클라이언트 중심**: 누가 어떤 기능 세트를 실제로 쓰는가에 따라 인터페이스를 쪼갠다.
  - **변경 이유 중심**: 함께 바뀌는 메서드 묶음은 함께, 다르게 바뀌는 메서드는 분리.
  - **동작 일관성**: 한 인터페이스에 담긴 메서드는 같은 추상화 레벨과 시맨틱을 가진다.

- **실무 팁**

  - UI, 배치, 외부 API 등 서로 다른 클라이언트 별로 **포트(Port) 인터페이스**를 쪼갠다.
  - 출력 전용/입력 전용 인터페이스를 분리해 쓰기-읽기 모델을 단순화한다.
  - 도메인 서비스의 퍼사드 인터페이스를 얇게 유지하고, 세부는 내부로 숨긴다.

---

### 11. 의존성 역전 원칙 (DIP: Dependency Inversion Principle)

> "상위 정책(고수준 모듈)은 하위 구현(저수준 모듈)에 의존하지 않는다. 둘 다 추상화에 의존해야 한다."

- **핵심**

  - 비즈니스 규칙(정책)은 세부사항(프레임워크, DB, UI)에서 **독립**적이어야 한다.
  - 의존의 방향을 **안정된 것(정책) → 불안정한 것(세부)**이 아니라, **정책 ← 추상화 → 세부**로 재배치한다.
  - 추상화(인터페이스, 포트)는 정책이 소유하고, 구현은 바깥에서 **플러그인**처럼 끼운다.

- **아키텍처 관점**

  - **플러그인 아키텍처**: 핵심 정책은 고정하고, DB, 메시징, UI는 교체 가능한 어댑터로 둔다.
  - **의존성 규칙**: 안쪽(도메인, 유스케이스)으로만 의존하도록 계층을 구성한다.
  - **경계(Ports & Adapters)**: 입력/출력 경계를 인터페이스로 정의하고, 어댑터가 외부 세계와 연결한다.

- **구현 패턴**

  - 생성 시 **의존성 주입(DI)**: 정책은 인터페이스로 의존, 구체 구현은 컴포지션/컨테이너에서 주입.
  - **팩토리/추상 팩토리**로 객체 생성 책임을 정책 바깥으로 밀어낸다.
  - **테스트 더블**(Fake/Stub/Mock)로 정책을 인프라 없이 검증 가능하게 한다.

- **효과**

  - 프레임워크 교체, 인프라 변경의 파급을 최소화.
  - 순수 도메인/유스케이스 레벨의 빠른 단위 테스트 가능.
  - 병렬 개발과 배포 단위 분리가 쉬워진다.

---

### ISP · DIP의 연결과 실전 적용

- **분리 후 역전**

  - ISP로 클라이언트 맞춤 인터페이스(포트)를 쪼갠 뒤, DIP로 그 인터페이스의 **소유권을 정책**에 둔다.
  - 구현(어댑터)은 바깥으로 밀어내 **독립 배포/교체**가 가능해진다.

- **변경 시나리오 예시**

  - DB 교체: `UserRepository` 포트를 유지한 채 `JpaUserRepositoryAdapter → DynamoUserRepositoryAdapter`로 교체.
  - 통신 방식 변경: `NotificationSender` 포트 유지, `EmailAdapter ↔ SlackAdapter` 전환.
  - UI 개편: 유스케이스 인터페이스 고정, `ReactPresenter ↔ CLIView` 교체.

- **주의점**
  - 추상화 남용 금지: 인터페이스는 실제 **변화의 필요**가 있을 때 도입한다.
  - 과도한 계층화는 복잡도/간접비를 키운다. 도메인 모델의 단순함을 우선한다.

---

### 마무리: 경계는 좁고, 정책은 깨끗하게

- **ISP**로 경계를 좁혀 의존을 가볍게 하고,
- **DIP**로 의존의 방향을 정책 중심으로 뒤집어,
- 프레임워크/인프라를 **플러그인**처럼 다루자. 이렇게 하면 핵심 규칙은 오래가고, 주변은 자유롭게 바뀐다.

---

### 리액트 예시: ISP / DIP 빠른 샘플

- **ISP 예시: 읽기/쓰기 인터페이스 분리**

```ts
// ports/user.ts
export interface UserReader {
  getUsers(): Promise<Array<{ id: string; name: string }>>;
}

export interface UserWriter {
  updateUserName(id: string, name: string): Promise<void>;
}
```

```tsx
// components/UserList.tsx — 읽기만 필요
import { useEffect, useState } from 'react';
import type { UserReader } from '../ports/user';

type Props = { reader: UserReader };

export function UserList({ reader }: Props) {
  const [users, setUsers] = useState<Array<{ id: string; name: string }>>([]);
  useEffect(() => {
    reader.getUsers().then(setUsers);
  }, [reader]);
  return (
    <ul>
      {users.map((u) => (
        <li key={u.id}>{u.name}</li>
      ))}
    </ul>
  );
}
```

```tsx
// components/UserEditor.tsx — 쓰기 기능도 필요
import { useState } from 'react';
import type { UserReader, UserWriter } from '../ports/user';

type Props = { reader: UserReader; writer: UserWriter };

export function UserEditor({ reader, writer }: Props) {
  const [id, setId] = useState('');
  const [name, setName] = useState('');
  return (
    <form
      onSubmit={async (e) => {
        e.preventDefault();
        await writer.updateUserName(id, name);
        setName('');
      }}
    >
      <input
        placeholder="id"
        value={id}
        onChange={(e) => setId(e.target.value)}
      />
      <input
        placeholder="name"
        value={name}
        onChange={(e) => setName(e.target.value)}
      />
      <button type="submit">Save</button>
    </form>
  );
}
```

- **DIP 예시: 포트 소유는 정책, 어댑터는 외부**

```ts
// adapters/httpUserRepository.ts — 어댑터(세부 구현)
import type { UserReader, UserWriter } from '../ports/user';

export class HttpUserRepository implements UserReader, UserWriter {
  constructor(private readonly baseUrl: string) {}

  async getUsers() {
    const res = await fetch(`${this.baseUrl}/users`);
    return (await res.json()) as Array<{ id: string; name: string }>;
  }

  async updateUserName(id: string, name: string) {
    await fetch(`${this.baseUrl}/users/${id}`, {
      method: 'PATCH',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ name }),
    });
  }
}
```

```tsx
// app/Providers.tsx — 컴포지션 루트에서 주입
import { createContext, useContext } from 'react';
import type { UserReader, UserWriter } from '../ports/user';
import { HttpUserRepository } from '../adapters/httpUserRepository';

const repo = new HttpUserRepository('/api');
const UserReaderCtx = createContext<UserReader>(repo);
const UserWriterCtx = createContext<UserWriter>(repo);

export const useUserReader = () => useContext(UserReaderCtx);
export const useUserWriter = () => useContext(UserWriterCtx);

export function AppProviders({ children }: { children: React.ReactNode }) {
  return (
    <UserReaderCtx.Provider value={repo}>
      <UserWriterCtx.Provider value={repo}>{children}</UserWriterCtx.Provider>
    </UserReaderCtx.Provider>
  );
}
```

```tsx
// usage 예: 컴포넌트는 추상(포트)만 안다
import { useUserReader, useUserWriter } from './app/Providers';

export function Dashboard() {
  const reader = useUserReader();
  const writer = useUserWriter();
  // 위에서 만든 UserList/UserEditor 재사용
  return (
    <div>
      <h2>Users</h2>
      <UserList reader={reader} />
      <UserEditor reader={reader} writer={writer} />
    </div>
  );
}
```

```ts
// 테스트 대체 구현 — 인프라 없이 빠른 테스트
import type { UserReader, UserWriter } from '../ports/user';

export class InMemoryUserRepository implements UserReader, UserWriter {
  constructor(private data: Array<{ id: string; name: string }>) {}
  async getUsers() {
    return this.data;
  }
  async updateUserName(id: string, name: string) {
    const u = this.data.find((d) => d.id === id);
    if (u) u.name = name;
  }
}
```
