# 7장 SRP - 단일 책임 원칙

## 개념 정의

### SRP란?

**단일 책임 원칙(Single Responsibility Principle)**은 SOLID 원칙 중 하나로, 모듈이 변경되는 이유는 오직 하나여야 한다는 원칙입니다.

> "하나의 모듈은 하나의, 오직 하나의 액터에 대해서만 책임져야 한다."

### 핵심 개념

- **응집성(Cohesion)**: 단일 액터를 책임지는 코드를 함께 묶어주는 힘
- **액터(Actor)**: 변경을 요구하는 사용자 그룹
- **모듈**: 하나 이상의 함수와 데이터 구조로 구성된 응집된 집합

## SRP 위반 징후

### 1. 우발적 중복 (Accidental Duplication)

서로 다른 액터가 동일한 코드를 사용할 때 발생하는 문제

### 2. 병합 충돌 (Merge Conflicts)

여러 개발자가 서로 다른 이유로 같은 소스 파일을 수정할 때 발생

## 해결책

### 1. 메서드 분리

- 메서드를 각기 다른 클래스로 이동
- 공통 데이터 구조 생성하여 공유

### 2. 퍼사드 패턴 (Facade Pattern)

- 여러 클래스 인스턴스화의 복잡성 해결
- 단순한 인터페이스 제공

## 적용 수준

### 1. 메서드/클래스 수준

- 함수는 단 하나의 일만 수행
- 클래스는 단일 책임을 가짐

### 2. 컴포넌트 수준

- **공통 폐쇄 원칙(Common Closure Principle)**으로 발전

### 3. 아키텍처 수준

- 아키텍처 경계 생성의 기준이 됨

## Frontend 개발자를 위한 실제 예제

### ❌ SRP 위반 사례

```javascript
// SRP를 위반하는 UserProfile 컴포넌트
class UserProfile {
  constructor(userId) {
    this.userId = userId;
  }

  // 사용자 데이터 관리 (Data Layer 책임)
  async fetchUserData() {
    const response = await fetch(`/api/users/${this.userId}`);
    return response.json();
  }

  // 사용자 데이터 검증 (Business Logic 책임)
  validateUserData(userData) {
    return userData.email && userData.name && userData.age > 0;
  }

  // UI 렌더링 (Presentation Layer 책임)
  render(userData) {
    return `
      <div class="user-profile">
        <h2>${userData.name}</h2>
        <p>${userData.email}</p>
        <p>Age: ${userData.age}</p>
      </div>
    `;
  }

  // 사용자 데이터 저장 (Data Layer 책임)
  async saveUserData(userData) {
    await fetch(`/api/users/${this.userId}`, {
      method: "PUT",
      body: JSON.stringify(userData),
    });
  }

  // 이메일 전송 (External Service 책임)
  async sendWelcomeEmail(email) {
    await fetch("/api/email/welcome", {
      method: "POST",
      body: JSON.stringify({ email }),
    });
  }
}
```

**문제점:**

- 하나의 클래스가 데이터 관리, 비즈니스 로직, UI 렌더링, 외부 서비스 등 여러 책임을 가짐
- API 변경, UI 변경, 비즈니스 룰 변경 등 다양한 이유로 수정될 수 있음

### ✅ SRP 준수 사례

```javascript
// 1. 데이터 관리 책임 - API 통신만 담당
class UserApiService {
  async fetchUser(userId) {
    const response = await fetch(`/api/users/${userId}`);
    return response.json();
  }

  async saveUser(userId, userData) {
    return fetch(`/api/users/${userId}`, {
      method: "PUT",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(userData),
    });
  }
}

// 2. 비즈니스 로직 책임 - 데이터 검증과 변환만 담당
class UserValidator {
  validate(userData) {
    const errors = {};

    if (!userData.email || !this.isValidEmail(userData.email)) {
      errors.email = "Valid email is required";
    }

    if (!userData.name || userData.name.trim().length < 2) {
      errors.name = "Name must be at least 2 characters";
    }

    if (!userData.age || userData.age < 0 || userData.age > 120) {
      errors.age = "Valid age is required";
    }

    return {
      isValid: Object.keys(errors).length === 0,
      errors,
    };
  }

  isValidEmail(email) {
    return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
  }
}

// 3. 외부 서비스 책임 - 이메일 서비스만 담당
class EmailService {
  async sendWelcomeEmail(email) {
    return fetch("/api/email/welcome", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ email }),
    });
  }
}

// 4. UI 렌더링 책임 - 화면 표시만 담당
class UserProfileRenderer {
  render(userData, errors = {}) {
    return `
      <div class="user-profile">
        <div class="user-info">
          <h2>${userData.name || ""}</h2>
          ${errors.name ? `<span class="error">${errors.name}</span>` : ""}

          <p>${userData.email || ""}</p>
          ${errors.email ? `<span class="error">${errors.email}</span>` : ""}

          <p>Age: ${userData.age || ""}</p>
          ${errors.age ? `<span class="error">${errors.age}</span>` : ""}
        </div>
      </div>
    `;
  }
}

// 5. 조합 역할 - 퍼사드 패턴 적용
class UserProfileManager {
  constructor(userId) {
    this.userId = userId;
    this.apiService = new UserApiService();
    this.validator = new UserValidator();
    this.emailService = new EmailService();
    this.renderer = new UserProfileRenderer();
  }

  async loadAndDisplayUser() {
    try {
      const userData = await this.apiService.fetchUser(this.userId);
      const validation = this.validator.validate(userData);

      if (validation.isValid) {
        return this.renderer.render(userData);
      } else {
        return this.renderer.render(userData, validation.errors);
      }
    } catch (error) {
      return this.renderer.render({}, { general: "Failed to load user data" });
    }
  }

  async saveUser(userData) {
    const validation = this.validator.validate(userData);

    if (!validation.isValid) {
      return { success: false, errors: validation.errors };
    }

    try {
      await this.apiService.saveUser(this.userId, userData);
      await this.emailService.sendWelcomeEmail(userData.email);
      return { success: true };
    } catch (error) {
      return { success: false, errors: { general: "Save failed" } };
    }
  }
}
```

### React 컴포넌트에서의 SRP 적용

```jsx
// ❌ SRP 위반 - 하나의 컴포넌트가 너무 많은 책임을 가짐
function UserDashboard() {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(false);
  const [filter, setFilter] = useState("");
  const [sortBy, setSortBy] = useState("name");

  // 데이터 페칭 로직
  useEffect(() => {
    fetchUsers();
  }, []);

  const fetchUsers = async () => {
    setLoading(true);
    const response = await fetch("/api/users");
    const data = await response.json();
    setUsers(data);
    setLoading(false);
  };

  // 필터링 로직
  const filteredUsers = users.filter((user) =>
    user.name.toLowerCase().includes(filter.toLowerCase())
  );

  // 정렬 로직
  const sortedUsers = filteredUsers.sort((a, b) => {
    if (sortBy === "name") return a.name.localeCompare(b.name);
    if (sortBy === "email") return a.email.localeCompare(b.email);
    return 0;
  });

  // 복잡한 렌더링 로직
  return (
    <div className="user-dashboard">
      <div className="controls">
        <input
          type="text"
          placeholder="Filter users..."
          value={filter}
          onChange={(e) => setFilter(e.target.value)}
        />
        <select value={sortBy} onChange={(e) => setSortBy(e.target.value)}>
          <option value="name">Sort by Name</option>
          <option value="email">Sort by Email</option>
        </select>
      </div>

      {loading ? (
        <div className="loading">Loading...</div>
      ) : (
        <div className="user-grid">
          {sortedUsers.map((user) => (
            <div key={user.id} className="user-card">
              <h3>{user.name}</h3>
              <p>{user.email}</p>
              <p>{user.department}</p>
            </div>
          ))}
        </div>
      )}
    </div>
  );
}
```

```jsx
// ✅ SRP 준수 - 각 컴포넌트가 단일 책임을 가짐

// 1. 사용자 데이터 관리만 담당하는 커스텀 훅
function useUsers() {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  const fetchUsers = async () => {
    setLoading(true);
    setError(null);
    try {
      const response = await fetch("/api/users");
      const data = await response.json();
      setUsers(data);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  };

  useEffect(() => {
    fetchUsers();
  }, []);

  return { users, loading, error, refetch: fetchUsers };
}

// 2. 필터링 로직만 담당하는 커스텀 훅
function useUserFilter(users) {
  const [filter, setFilter] = useState("");
  const [sortBy, setSortBy] = useState("name");

  const filteredAndSortedUsers = useMemo(() => {
    let filtered = users.filter((user) =>
      user.name.toLowerCase().includes(filter.toLowerCase())
    );

    return filtered.sort((a, b) => {
      if (sortBy === "name") return a.name.localeCompare(b.name);
      if (sortBy === "email") return a.email.localeCompare(b.email);
      return 0;
    });
  }, [users, filter, sortBy]);

  return {
    filter,
    setFilter,
    sortBy,
    setSortBy,
    filteredUsers: filteredAndSortedUsers,
  };
}

// 3. 컨트롤 UI만 담당하는 컴포넌트
function UserControls({ filter, onFilterChange, sortBy, onSortChange }) {
  return (
    <div className="controls">
      <input
        type="text"
        placeholder="Filter users..."
        value={filter}
        onChange={(e) => onFilterChange(e.target.value)}
      />
      <select value={sortBy} onChange={(e) => onSortChange(e.target.value)}>
        <option value="name">Sort by Name</option>
        <option value="email">Sort by Email</option>
      </select>
    </div>
  );
}

// 4. 단일 사용자 카드만 담당하는 컴포넌트
function UserCard({ user }) {
  return (
    <div className="user-card">
      <h3>{user.name}</h3>
      <p>{user.email}</p>
      <p>{user.department}</p>
    </div>
  );
}

// 5. 사용자 목록만 담당하는 컴포넌트
function UserGrid({ users, loading }) {
  if (loading) {
    return <div className="loading">Loading...</div>;
  }

  return (
    <div className="user-grid">
      {users.map((user) => (
        <UserCard key={user.id} user={user} />
      ))}
    </div>
  );
}

// 6. 조합만 담당하는 메인 컴포넌트
function UserDashboard() {
  const { users, loading, error } = useUsers();
  const { filter, setFilter, sortBy, setSortBy, filteredUsers } =
    useUserFilter(users);

  if (error) {
    return <div className="error">Error: {error}</div>;
  }

  return (
    <div className="user-dashboard">
      <UserControls
        filter={filter}
        onFilterChange={setFilter}
        sortBy={sortBy}
        onSortChange={setSortBy}
      />
      <UserGrid users={filteredUsers} loading={loading} />
    </div>
  );
}
```

## 핵심 포인트

### Frontend 개발에서 SRP 적용 시 고려사항

1. **컴포넌트 분리**: UI 로직, 비즈니스 로직, 데이터 로직을 별도 컴포넌트/훅으로 분리
2. **Custom Hook 활용**: 상태 관리와 비즈니스 로직을 재사용 가능한 훅으로 추출
3. **서비스 레이어**: API 통신과 데이터 변환을 별도 서비스로 분리
4. **단일 목적**: 각 컴포넌트와 함수는 하나의 명확한 목적만 가져야 함

# 8장 OCP - 개방-폐쇄 원칙

## 개념 정의

### OCP란?

**개방-폐쇄 원칙(Open-Closed Principle)**은 SOLID 원칙 중 하나로, 소프트웨어 개체는 확장에는 열려 있어야 하고, 변경에는 닫혀 있어야 한다는 원칙입니다.

> "소프트웨어 개체(artifact)는 확장에 열려 있어야 하고, 변경에는 닫혀 있어야 한다."

### 핵심 개념

- **확장에 열려 있음**: 새로운 기능을 추가할 수 있어야 함
- **변경에 닫혀 있음**: 기존 코드를 수정하지 않고 기능을 확장할 수 있어야 함
- **이상적인 변경량**: 새로운 요구사항에 대해 기존 코드 변경량이 0에 가까워야 함

## OCP 구현 전략

### 1. 책임 분리 (SRP와 연계)

서로 다른 목적으로 변경되는 요소를 적절하게 분리

### 2. 의존성 체계화 (DIP와 연계)

요소 사이의 의존성을 체계화하여 변경량을 최소화

### 3. 방향성 제어

고수준 컴포넌트가 저수준 컴포넌트의 변경으로부터 보호되도록 설계

### 4. 정보 은닉

내부 구현을 숨기고 인터페이스를 통해서만 접근 가능하도록 설계

## 실제 적용 예시

### 보고서 생성 시스템 분리

책임을 두 개로 분리:

1. **데이터 계산 책임**: 보고서용 데이터를 계산
2. **표현 책임**: 데이터를 웹이나 프린트에 적합한 형태로 표현

→ 한 책임의 변경이 다른 책임에 영향을 주지 않도록 의존성 조직화

## Frontend 개발자를 위한 실제 예제

### ❌ OCP 위반 사례

```javascript
// OCP를 위반하는 알림 시스템
class NotificationManager {
  sendNotification(type, message, recipient) {
    if (type === "email") {
      // 이메일 전송 로직
      console.log(`Sending email to ${recipient}: ${message}`);
      // SMTP 설정, 이메일 템플릿 등...
    } else if (type === "sms") {
      // SMS 전송 로직
      console.log(`Sending SMS to ${recipient}: ${message}`);
      // SMS API 호출 등...
    } else if (type === "push") {
      // 푸시 알림 전송 로직
      console.log(`Sending push notification to ${recipient}: ${message}`);
      // FCM API 호출 등...
    } else if (type === "slack") {
      // Slack 알림 전송 로직 (새로운 요구사항)
      console.log(`Sending Slack message to ${recipient}: ${message}`);
      // Slack API 호출 등...
    }
  }
}
```

**문제점:**

- 새로운 알림 방식을 추가할 때마다 기존 코드를 수정해야 함
- if-else 체인이 계속 늘어남
- 각 알림 방식의 변경이 다른 방식에도 영향을 줄 수 있음

### ✅ OCP 준수 사례

```javascript
// 1. 알림 인터페이스 정의 (추상화)
class NotificationChannel {
  send(message, recipient) {
    throw new Error("send method must be implemented");
  }
}

// 2. 각 알림 방식을 별도 클래스로 구현 (확장에 열려 있음)
class EmailNotification extends NotificationChannel {
  send(message, recipient) {
    console.log(`Sending email to ${recipient}: ${message}`);
    // 이메일 전송 로직
    this.setupSMTP();
    this.sendEmail(message, recipient);
  }

  setupSMTP() {
    // SMTP 설정 로직
  }

  sendEmail(message, recipient) {
    // 실제 이메일 전송
  }
}

class SMSNotification extends NotificationChannel {
  send(message, recipient) {
    console.log(`Sending SMS to ${recipient}: ${message}`);
    // SMS 전송 로직
    this.callSMSAPI(message, recipient);
  }

  callSMSAPI(message, recipient) {
    // SMS API 호출
  }
}

class PushNotification extends NotificationChannel {
  send(message, recipient) {
    console.log(`Sending push notification to ${recipient}: ${message}`);
    // 푸시 알림 전송 로직
    this.sendToFCM(message, recipient);
  }

  sendToFCM(message, recipient) {
    // FCM API 호출
  }
}

// 3. 새로운 알림 방식 추가 (기존 코드 수정 없이 확장 가능)
class SlackNotification extends NotificationChannel {
  send(message, recipient) {
    console.log(`Sending Slack message to ${recipient}: ${message}`);
    this.sendToSlackAPI(message, recipient);
  }

  sendToSlackAPI(message, recipient) {
    // Slack API 호출
  }
}

// 4. 알림 관리자 (변경에 닫혀 있음)
class NotificationManager {
  constructor() {
    this.channels = new Map();
  }

  // 알림 채널 등록 (런타임에 확장 가능)
  registerChannel(type, channel) {
    this.channels.set(type, channel);
  }

  sendNotification(type, message, recipient) {
    const channel = this.channels.get(type);
    if (!channel) {
      throw new Error(`Unknown notification type: ${type}`);
    }
    channel.send(message, recipient);
  }
}

// 사용 예시
const notificationManager = new NotificationManager();

// 알림 채널 등록
notificationManager.registerChannel("email", new EmailNotification());
notificationManager.registerChannel("sms", new SMSNotification());
notificationManager.registerChannel("push", new PushNotification());
notificationManager.registerChannel("slack", new SlackNotification()); // 새로운 채널 추가

// 사용
notificationManager.sendNotification("email", "Hello!", "user@example.com");
notificationManager.sendNotification("slack", "Deploy completed!", "#dev-team");
```

### React 컴포넌트에서의 OCP 적용

```jsx
// ❌ OCP 위반 - UI 컴포넌트가 다양한 데이터 소스에 직접 의존
function DataDisplay({ dataType, config }) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(false);

  useEffect(() => {
    const fetchData = async () => {
      setLoading(true);

      if (dataType === "users") {
        // 사용자 데이터 가져오기
        const response = await fetch("/api/users");
        const users = await response.json();
        setData(users.map((user) => ({ id: user.id, name: user.name })));
      } else if (dataType === "products") {
        // 상품 데이터 가져오기
        const response = await fetch("/api/products");
        const products = await response.json();
        setData(
          products.map((product) => ({ id: product.id, name: product.title }))
        );
      } else if (dataType === "orders") {
        // 주문 데이터 가져오기 (새로운 요구사항)
        const response = await fetch("/api/orders");
        const orders = await response.json();
        setData(
          orders.map((order) => ({
            id: order.id,
            name: `Order #${order.number}`,
          }))
        );
      }

      setLoading(false);
    };

    fetchData();
  }, [dataType]);

  if (loading) return <div>Loading...</div>;

  return (
    <div>
      {data?.map((item) => (
        <div key={item.id}>{item.name}</div>
      ))}
    </div>
  );
}
```

```jsx
// ✅ OCP 준수 - 데이터 소스 추상화 및 확장 가능한 구조

// 1. 데이터 소스 인터페이스 정의
class DataSource {
  async fetchData() {
    throw new Error("fetchData must be implemented");
  }
}

// 2. 각 데이터 소스 구현 (확장에 열려 있음)
class UserDataSource extends DataSource {
  async fetchData() {
    const response = await fetch("/api/users");
    const users = await response.json();
    return users.map((user) => ({
      id: user.id,
      name: user.name,
      subtitle: user.email,
    }));
  }
}

class ProductDataSource extends DataSource {
  async fetchData() {
    const response = await fetch("/api/products");
    const products = await response.json();
    return products.map((product) => ({
      id: product.id,
      name: product.title,
      subtitle: `$${product.price}`,
    }));
  }
}

// 3. 새로운 데이터 소스 추가 (기존 코드 수정 없이)
class OrderDataSource extends DataSource {
  async fetchData() {
    const response = await fetch("/api/orders");
    const orders = await response.json();
    return orders.map((order) => ({
      id: order.id,
      name: `Order #${order.number}`,
      subtitle: `$${order.total}`,
    }));
  }
}

class AnalyticsDataSource extends DataSource {
  async fetchData() {
    const response = await fetch("/api/analytics");
    const analytics = await response.json();
    return analytics.map((metric) => ({
      id: metric.id,
      name: metric.name,
      subtitle: metric.value,
    }));
  }
}

// 4. 데이터 소스 팩토리 (확장 가능)
class DataSourceFactory {
  static sources = new Map([
    ["users", UserDataSource],
    ["products", ProductDataSource],
    ["orders", OrderDataSource],
    ["analytics", AnalyticsDataSource],
  ]);

  static create(type) {
    const SourceClass = this.sources.get(type);
    if (!SourceClass) {
      throw new Error(`Unknown data source type: ${type}`);
    }
    return new SourceClass();
  }

  // 런타임에 새로운 데이터 소스 등록 가능
  static register(type, sourceClass) {
    this.sources.set(type, sourceClass);
  }
}

// 5. 데이터 fetching 로직 분리 (Custom Hook)
function useDataSource(dataType, config = {}) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  useEffect(() => {
    const fetchData = async () => {
      setLoading(true);
      setError(null);

      try {
        const dataSource = DataSourceFactory.create(dataType);
        const result = await dataSource.fetchData(config);
        setData(result);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    };

    fetchData();
  }, [dataType, JSON.stringify(config)]);

  return { data, loading, error };
}

// 6. UI 컴포넌트 (변경에 닫혀 있음)
function DataDisplay({ dataType, config }) {
  const { data, loading, error } = useDataSource(dataType, config);

  if (loading) return <div className="loading">Loading...</div>;
  if (error) return <div className="error">Error: {error}</div>;

  return (
    <div className="data-display">
      {data?.map((item) => (
        <div key={item.id} className="data-item">
          <h3>{item.name}</h3>
          {item.subtitle && <p>{item.subtitle}</p>}
        </div>
      ))}
    </div>
  );
}

// 7. 컴포넌트 렌더러 확장 예시
class CustomRenderer {
  render(data) {
    throw new Error("render must be implemented");
  }
}

class CardRenderer extends CustomRenderer {
  render(data) {
    return data.map((item) => (
      <div key={item.id} className="card">
        <h3>{item.name}</h3>
        <p>{item.subtitle}</p>
      </div>
    ));
  }
}

class TableRenderer extends CustomRenderer {
  render(data) {
    return (
      <table>
        <tbody>
          {data.map((item) => (
            <tr key={item.id}>
              <td>{item.name}</td>
              <td>{item.subtitle}</td>
            </tr>
          ))}
        </tbody>
      </table>
    );
  }
}

// 렌더러를 사용하는 확장 가능한 컴포넌트
function FlexibleDataDisplay({ dataType, renderType = "card", config }) {
  const { data, loading, error } = useDataSource(dataType, config);

  const renderers = {
    card: CardRenderer,
    table: TableRenderer,
  };

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;

  const RendererClass = renderers[renderType];
  const renderer = new RendererClass();

  return <div>{renderer.render(data)}</div>;
}
```

### State Management에서의 OCP 적용

```javascript
// ✅ Redux에서 OCP 준수 예시

// 1. 액션 타입 정의
const ActionTypes = {
  USER_LOGIN: "USER_LOGIN",
  USER_LOGOUT: "USER_LOGOUT",
  PRODUCT_ADD: "PRODUCT_ADD",
  ORDER_CREATE: "ORDER_CREATE", // 새로운 기능 추가 시
};

// 2. 액션 생성자 (각각 독립적)
const userActions = {
  login: (user) => ({ type: ActionTypes.USER_LOGIN, payload: user }),
  logout: () => ({ type: ActionTypes.USER_LOGOUT }),
};

const productActions = {
  add: (product) => ({ type: ActionTypes.PRODUCT_ADD, payload: product }),
};

// 3. 새로운 기능 추가 시 기존 코드 수정 없이 확장
const orderActions = {
  create: (order) => ({ type: ActionTypes.ORDER_CREATE, payload: order }),
};

// 4. 리듀서도 독립적으로 관리 (combineReducers 활용)
const userReducer = (state = {}, action) => {
  switch (action.type) {
    case ActionTypes.USER_LOGIN:
      return { ...state, user: action.payload, isLoggedIn: true };
    case ActionTypes.USER_LOGOUT:
      return { ...state, user: null, isLoggedIn: false };
    default:
      return state;
  }
};

const productReducer = (state = [], action) => {
  switch (action.type) {
    case ActionTypes.PRODUCT_ADD:
      return [...state, action.payload];
    default:
      return state;
  }
};

// 5. 새로운 리듀서 추가 (기존 코드 영향 없음)
const orderReducer = (state = [], action) => {
  switch (action.type) {
    case ActionTypes.ORDER_CREATE:
      return [...state, action.payload];
    default:
      return state;
  }
};

// 6. 루트 리듀서에서 조합
const rootReducer = combineReducers({
  user: userReducer,
  products: productReducer,
  orders: orderReducer, // 새로운 리듀서 추가
});
```

## 핵심 포인트

### Frontend 개발에서 OCP 적용 시 고려사항

1. **추상화 활용**: 인터페이스나 추상 클래스를 통해 확장 가능한 구조 설계
2. **팩토리 패턴**: 새로운 타입 추가 시 기존 코드 변경 최소화
3. **Custom Hook 분리**: 로직과 UI를 분리하여 독립적 확장 가능
4. **컴포넌트 합성**: props를 통한 확장 가능한 컴포넌트 설계
5. **State 분리**: 각 도메인별 상태 관리를 독립적으로 구성

### OCP의 목표

- **확장성**: 새로운 기능을 쉽게 추가할 수 있는 시스템
- **안정성**: 기존 기능에 영향을 주지 않는 변경
- **유지보수성**: 컴포넌트 단위의 독립적 관리

# 9장 LSP - 리스코프 치환 원칙

## 개념 정의

### LSP란?

**리스코프 치환 원칙(Liskov Substitution Principle)**은 SOLID 원칙 중 하나로, 상위 타입의 객체를 하위 타입의 객체로 치환해도 프로그램의 정확성이 깨지지 않아야 한다는 원칙입니다.

> "상속을 사용할 때, 하위 타입은 상위 타입을 완전히 대체할 수 있어야 한다."

### 핵심 개념

- **치환 가능성**: 하위 클래스는 상위 클래스를 완전히 대체할 수 있어야 함
- **행위 호환성**: 클라이언트는 상위 타입과 하위 타입의 차이를 알 필요가 없어야 함
- **계약 준수**: 하위 타입은 상위 타입이 정의한 계약을 반드시 지켜야 함

## LSP 준수 예제

### 라이선스 시스템 (올바른 예시)

```javascript
// License 기본 클래스
class License {
  calculateFee() {
    throw new Error("calculateFee must be implemented");
  }

  isValid() {
    return true;
  }
}

// 하위 타입들이 상위 타입을 완전히 대체 가능
class PersonalLicense extends License {
  calculateFee() {
    return 100;
  }
}

class BusinessLicense extends License {
  calculateFee() {
    return 500;
  }
}

// Billing 시스템은 어떤 라이선스 타입인지 알 필요 없음
class BillingSystem {
  processLicense(license) {
    // License 타입이면 모두 동일하게 동작
    if (license.isValid()) {
      return license.calculateFee();
    }
    return 0;
  }
}
```

## LSP 위반 사례

### 1. 정사각형/직사각형 문제 (고전적인 위반 사례)

```javascript
// ❌ LSP 위반 사례
class Rectangle {
  constructor(width, height) {
    this.width = width;
    this.height = height;
  }

  setWidth(width) {
    this.width = width;
  }

  setHeight(height) {
    this.height = height;
  }

  getArea() {
    return this.width * this.height;
  }
}

class Square extends Rectangle {
  constructor(side) {
    super(side, side);
  }

  // 문제: 정사각형은 너비와 높이가 항상 같아야 함
  setWidth(width) {
    this.width = width;
    this.height = width; // 높이도 함께 변경됨
  }

  setHeight(height) {
    this.width = height; // 너비도 함께 변경됨
    this.height = height;
  }
}

// 클라이언트 코드에서 문제 발생
function testRectangle(rectangle) {
  rectangle.setWidth(5);
  rectangle.setHeight(4);

  // Rectangle이라면 20이어야 하지만, Square라면 16이 됨
  console.log(`Expected: 20, Actual: ${rectangle.getArea()}`);
}

const rect = new Rectangle(3, 4);
const square = new Square(3);

testRectangle(rect); // Expected: 20, Actual: 20 ✓
testRectangle(square); // Expected: 20, Actual: 16 ❌ LSP 위반!
```

### 2. 택시 파견 시스템 위반 사례

```javascript
// ❌ LSP 위반 - 택시 파견 시스템
class Vehicle {
  startEngine() {
    console.log("Engine started");
  }

  accelerate() {
    console.log("Vehicle accelerating");
  }
}

class ElectricTaxi extends Vehicle {
  startEngine() {
    // 전기차는 엔진이 없음!
    throw new Error("Electric vehicles don't have engines");
  }

  accelerate() {
    console.log("Electric taxi accelerating silently");
  }
}

// 클라이언트는 Vehicle로 동작을 예상하지만...
function dispatchTaxi(vehicle) {
  vehicle.startEngine(); // ElectricTaxi에서 예외 발생!
  vehicle.accelerate();
}
```

## Frontend 개발자를 위한 실제 예제

### ❌ LSP 위반 사례

```jsx
// LSP를 위반하는 폼 필드 컴포넌트
class FormField {
  constructor(props) {
    this.props = props;
    this.value = props.defaultValue || "";
  }

  setValue(value) {
    this.value = value;
  }

  getValue() {
    return this.value;
  }

  validate() {
    return this.props.required ? this.value.length > 0 : true;
  }

  render() {
    return `<input value="${this.value}" />`;
  }
}

class ReadOnlyField extends FormField {
  // 문제: setValue를 호출하면 예외 발생
  setValue(value) {
    throw new Error("Cannot set value on read-only field");
  }

  render() {
    return `<span>${this.value}</span>`;
  }
}

class FileUploadField extends FormField {
  // 문제: 값의 타입이 다름 (문자열이 아닌 File 객체)
  setValue(file) {
    if (!(file instanceof File)) {
      throw new Error("FileUploadField only accepts File objects");
    }
    this.value = file;
  }

  getValue() {
    return this.value; // File 객체 반환 (문자열 예상과 다름)
  }

  validate() {
    return this.props.required ? this.value !== null : true;
  }
}

// 클라이언트 코드에서 문제 발생
function processForm(fields) {
  fields.forEach((field) => {
    // ReadOnlyField에서 예외 발생 가능
    field.setValue(field.getValue().toUpperCase());

    // FileUploadField에서 문자열 메서드 호출 시 오류
    const value = field.getValue();
    if (typeof value === "string") {
      console.log(value.length); // File 객체에는 length 속성이 다름
    }
  });
}
```

### ✅ LSP 준수 사례

```jsx
// LSP를 준수하는 폼 필드 설계

// 1. 기본 인터페이스 정의
class FormField {
  constructor(props) {
    this.props = props;
    this.isReadOnly = false;
  }

  // 모든 하위 클래스에서 동일하게 동작해야 하는 메서드
  setValue(value) {
    if (this.isReadOnly) {
      return false; // 예외 대신 false 반환
    }
    return this.doSetValue(value);
  }

  // 하위 클래스에서 구현해야 하는 추상 메서드
  doSetValue(value) {
    throw new Error("doSetValue must be implemented");
  }

  getValue() {
    throw new Error("getValue must be implemented");
  }

  getDisplayValue() {
    // 화면 표시용 문자열 반환 (모든 필드에서 일관성 유지)
    const value = this.getValue();
    return value ? value.toString() : "";
  }

  validate() {
    const value = this.getValue();
    if (this.props.required && !value) {
      return { isValid: false, message: "This field is required" };
    }
    return { isValid: true, message: "" };
  }

  canEdit() {
    return !this.isReadOnly;
  }
}

// 2. 일반 텍스트 필드
class TextField extends FormField {
  constructor(props) {
    super(props);
    this.value = props.defaultValue || "";
  }

  doSetValue(value) {
    this.value = String(value);
    return true;
  }

  getValue() {
    return this.value;
  }
}

// 3. 읽기 전용 필드
class ReadOnlyField extends FormField {
  constructor(props) {
    super(props);
    this.value = props.value || "";
    this.isReadOnly = true; // 기본 클래스의 플래그 설정
  }

  doSetValue(value) {
    // 읽기 전용이므로 설정하지 않지만 예외 발생하지 않음
    return false;
  }

  getValue() {
    return this.value;
  }
}

// 4. 파일 업로드 필드
class FileUploadField extends FormField {
  constructor(props) {
    super(props);
    this.file = null;
  }

  doSetValue(file) {
    if (file instanceof File || file === null) {
      this.file = file;
      return true;
    }
    return false;
  }

  getValue() {
    return this.file;
  }

  getDisplayValue() {
    return this.file ? this.file.name : "";
  }

  validate() {
    if (this.props.required && !this.file) {
      return { isValid: false, message: "Please select a file" };
    }

    if (
      this.file &&
      this.props.maxSize &&
      this.file.size > this.props.maxSize
    ) {
      return { isValid: false, message: "File size too large" };
    }

    return { isValid: true, message: "" };
  }
}

// 5. 숫자 필드
class NumberField extends FormField {
  constructor(props) {
    super(props);
    this.value = props.defaultValue || 0;
  }

  doSetValue(value) {
    const numValue = Number(value);
    if (!isNaN(numValue)) {
      this.value = numValue;
      return true;
    }
    return false;
  }

  getValue() {
    return this.value;
  }

  validate() {
    const baseValidation = super.validate();
    if (!baseValidation.isValid) {
      return baseValidation;
    }

    if (this.props.min !== undefined && this.value < this.props.min) {
      return {
        isValid: false,
        message: `Value must be at least ${this.props.min}`,
      };
    }

    if (this.props.max !== undefined && this.value > this.props.max) {
      return {
        isValid: false,
        message: `Value must be at most ${this.props.max}`,
      };
    }

    return { isValid: true, message: "" };
  }
}

// 6. 폼 처리 클래스 (LSP 준수로 안전한 사용)
class FormProcessor {
  constructor(fields) {
    this.fields = fields;
  }

  // 모든 필드 타입에서 동일하게 동작
  updateFields(values) {
    const results = [];

    this.fields.forEach((field, index) => {
      const value = values[index];

      if (field.canEdit()) {
        const success = field.setValue(value);
        results.push({
          index,
          success,
          value: field.getDisplayValue(), // 모든 필드에서 일관된 문자열 반환
        });
      }
    });

    return results;
  }

  validateAll() {
    return this.fields.map((field) => ({
      field: field.constructor.name,
      validation: field.validate(),
      displayValue: field.getDisplayValue(),
    }));
  }

  // 타입에 관계없이 동일하게 처리
  exportValues() {
    return this.fields.map((field) => ({
      value: field.getValue(),
      displayValue: field.getDisplayValue(),
      canEdit: field.canEdit(),
    }));
  }
}

// 7. 사용 예시
const fields = [
  new TextField({ defaultValue: "John Doe", required: true }),
  new NumberField({ defaultValue: 25, min: 0, max: 120 }),
  new FileUploadField({ required: true, maxSize: 1024 * 1024 }),
  new ReadOnlyField({ value: "Admin User" }),
];

const processor = new FormProcessor(fields);

// 모든 필드 타입에서 동일하게 동작
const updateResults = processor.updateFields([
  "Jane Doe",
  30,
  null,
  "Should not change",
]);
const validationResults = processor.validateAll();
const exportedValues = processor.exportValues();

console.log("Update Results:", updateResults);
console.log("Validation:", validationResults);
console.log("Export:", exportedValues);
```

### React Hook에서의 LSP 적용

```jsx
// ✅ LSP를 준수하는 데이터 fetching hooks

// 기본 hook 인터페이스
function useDataFetcher(config) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  // 모든 구현체에서 동일한 인터페이스
  const fetch = useCallback(async () => {
    setLoading(true);
    setError(null);

    try {
      const result = await fetchData(config);
      setData(result);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  }, [config]);

  // 하위 구현체에서 구현해야 하는 메서드
  const fetchData = async (config) => {
    throw new Error("fetchData must be implemented");
  };

  return { data, loading, error, refetch: fetch };
}

// REST API hook
function useRestApi(endpoint, options = {}) {
  const config = { endpoint, ...options };

  const fetchData = async ({ endpoint, ...opts }) => {
    const response = await fetch(endpoint, opts);
    if (!response.ok) {
      throw new Error(`HTTP ${response.status}: ${response.statusText}`);
    }
    return response.json();
  };

  return useDataFetcher({ ...config, fetchData });
}

// GraphQL hook
function useGraphQL(query, variables = {}) {
  const config = { query, variables };

  const fetchData = async ({ query, variables }) => {
    const response = await fetch("/graphql", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ query, variables }),
    });

    const result = await response.json();
    if (result.errors) {
      throw new Error(result.errors[0].message);
    }

    return result.data;
  };

  return useDataFetcher({ ...config, fetchData });
}

// WebSocket hook
function useWebSocket(url) {
  const config = { url };

  const fetchData = async ({ url }) => {
    return new Promise((resolve, reject) => {
      const ws = new WebSocket(url);

      ws.onmessage = (event) => {
        resolve(JSON.parse(event.data));
        ws.close();
      };

      ws.onerror = (error) => {
        reject(new Error("WebSocket error"));
      };
    });
  };

  return useDataFetcher({ ...config, fetchData });
}

// 클라이언트 컴포넌트에서는 어떤 hook을 사용해도 동일한 방식으로 처리
function DataDisplayComponent({ dataSource, config }) {
  let hookResult;

  // 모든 hook이 동일한 인터페이스 반환 (LSP 준수)
  switch (dataSource) {
    case "rest":
      hookResult = useRestApi(config.endpoint, config.options);
      break;
    case "graphql":
      hookResult = useGraphQL(config.query, config.variables);
      break;
    case "websocket":
      hookResult = useWebSocket(config.url);
      break;
    default:
      throw new Error(`Unknown data source: ${dataSource}`);
  }

  const { data, loading, error, refetch } = hookResult;

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;

  return (
    <div>
      <button onClick={refetch}>Refresh</button>
      <pre>{JSON.stringify(data, null, 2)}</pre>
    </div>
  );
}
```

## 핵심 포인트

### LSP 위반을 방지하는 방법

1. **계약 준수**: 상위 클래스가 정의한 모든 계약을 하위 클래스에서 지켜야 함
2. **사전 조건 약화**: 하위 클래스는 상위 클래스보다 더 엄격한 입력 조건을 요구하면 안됨
3. **사후 조건 강화**: 하위 클래스는 상위 클래스보다 더 느슨한 결과를 반환하면 안됨
4. **예외 일관성**: 예상치 못한 예외를 발생시키면 안됨

### Frontend 개발에서 LSP 적용 시 고려사항

1. **Hook 인터페이스**: 동일한 반환 타입과 메서드 제공
2. **컴포넌트 Props**: 일관된 props 인터페이스 유지
3. **State 관리**: 동일한 액션과 상태 구조 보장
4. **API 추상화**: 다양한 데이터 소스에 대한 일관된 인터페이스

### LSP의 아키텍처적 중요성

- **치환 가능성**: 시스템의 유연성과 확장성 보장
- **타입 안전성**: 런타임 오류 방지
- **코드 신뢰성**: 예측 가능한 동작 보장
