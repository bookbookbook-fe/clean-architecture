# 클린 아키텍처 10-11장 정리

## 📋 목차

- [10장: 인터페이스 분리 원칙 (ISP)](#10장-인터페이스-분리-원칙-isp)
- [11장: 의존성 역전 원칙 (DIP)](#11장-의존성-역전-원칙-dip)

---

## 10장 - 인터페이스 분리 원칙 (ISP)

### 📖 로버트 마틴의 핵심 정의

**"클라이언트는 자신이 사용하지 않는 메서드에 의존하면 안 된다"**

객체는 자신이 호출하지 않는 메소드에 의존하지 않아야 한다는 원칙.

### 🎯 책에서 제시하는 문제 상황

```
[Ops 클래스]
├── op1() - User1만 사용
├── op2() - User2만 사용
└── op3() - User3만 사용

[문제점]
User1이 op1만 필요한데, op2나 op3의 변경에도 영향을 받게 됨
```

이 상황에서 한 사용자가 자신이 사용하지 않는 메서드의 변경에도 영향을 받게 되는 **불필요한 결합**이 발생

### 🔄 언어별 차이점 (책의 중요한 통찰)

로버트 마틴은 언어의 특성에 따른 ISP의 영향도 차이를 설명

#### 정적 타입 언어 (Java, C#, TypeScript)

- 사용자가 **타입 선언문을 사용하도록 강제**
- 선언문으로 인해 **소스 코드 의존성 발생**
- **재컴파일 또는 재배포가 강제됨**

#### 동적 타입 언어 (JavaScript, Python, Ruby)

- 선언문이 존재하지 않아 **소스 코드 의존성이 없음**
- **재컴파일과 재배포가 불필요**

### ⚠️ ISP의 근본적인 동기

> **"일반적으로 필요 이상으로 많은 걸 포함하는 모듈에 의존하는 것은 해로운 일이다. 불필요한 짐을 실은 무언가에 의존하면 예상치 못한 문제에 빠질 수 있다."**

### 🏗️ 아키텍처 수준에서의 ISP

책에서는 더 고수준인 **아키텍처 레벨**에서도 동일한 상황이 발생

- 선형적인 의존성을 가진 경우
- 불필요한 기능과 변경이 최종 시스템에 영향을 주는 상황 초래
- **시스템 A → 프레임워크 F → 데이터베이스 D** 같은 의존성에서 D의 변경이 A에까지 영향

---

### 💻 프론트엔드에서의 ISP 적용

#### 🚫 잘못된 예시 (ISP 위반)

```typescript
// ❌ 거대한 사용자 서비스 인터페이스
interface UserService {
  // 인증 관련
  login(email: string, password: string): Promise<User>;
  logout(): Promise<void>;
  refreshToken(): Promise<string>;

  // 프로필 관리
  updateProfile(userId: string, data: UserProfile): Promise<void>;
  uploadAvatar(userId: string, file: File): Promise<string>;
  getProfile(userId: string): Promise<UserProfile>;

  // 관리자 기능
  deleteUser(userId: string): Promise<void>;
  getAllUsers(): Promise<User[]>;
  banUser(userId: string): Promise<void>;

  // 결제 관련
  updatePaymentMethod(userId: string, payment: PaymentInfo): Promise<void>;
  getPaymentHistory(userId: string): Promise<Payment[]>;
  processRefund(paymentId: string): Promise<void>;
}

// 로그인 폼이 결제, 관리자 기능까지 알게 됨
class LoginForm {
  constructor(private userService: UserService) {} // 불필요한 의존성

  async handleLogin(email: string, password: string) {
    return this.userService.login(email, password); // login만 필요한데!
  }
}
```

#### ✅ 올바른 예시 (ISP 준수)

```typescript
// ✅ 역할별로 분리된 인터페이스
interface AuthService {
  login(email: string, password: string): Promise<User>;
  logout(): Promise<void>;
  refreshToken(): Promise<string>;
}

interface ProfileService {
  updateProfile(userId: string, data: UserProfile): Promise<void>;
  uploadAvatar(userId: string, file: File): Promise<string>;
  getProfile(userId: string): Promise<UserProfile>;
}

interface AdminService {
  deleteUser(userId: string): Promise<void>;
  getAllUsers(): Promise<User[]>;
  banUser(userId: string): Promise<void>;
}

interface PaymentService {
  updatePaymentMethod(userId: string, payment: PaymentInfo): Promise<void>;
  getPaymentHistory(userId: string): Promise<Payment[]>;
  processRefund(paymentId: string): Promise<void>;
}

// 각 컴포넌트는 필요한 기능만 의존
class LoginForm {
  constructor(private authService: AuthService) {} // 인증만 의존

  async handleLogin(email: string, password: string) {
    return this.authService.login(email, password);
  }
}

class UserProfilePage {
  constructor(
    private profileService: ProfileService,
    private paymentService: PaymentService
  ) {} // 필요한 것만 의존

  async updateProfile(data: UserProfile) {
    return this.profileService.updateProfile("user-id", data);
  }

  async getPayments() {
    return this.paymentService.getPaymentHistory("user-id");
  }
}
```

#### 🎯 React Props에서의 ISP

```typescript
// ❌ 거대한 Props 인터페이스
interface ButtonProps {
  text: string;
  onClick: () => void;
  // 모든 버튼이 필요하지 않은 속성들
  isLoading?: boolean;
  loadingText?: string;
  icon?: React.ReactNode;
  iconPosition?: "left" | "right";
  confirmMessage?: string;
  onConfirm?: () => void;
  tooltipText?: string;
  tooltipPosition?: "top" | "bottom";
}

// ✅ 역할별로 분리된 Props
interface BaseButtonProps {
  text: string;
  onClick: () => void;
  variant?: "primary" | "secondary";
  size?: "small" | "medium" | "large";
}

interface LoadingButtonProps extends BaseButtonProps {
  isLoading: boolean;
  loadingText?: string;
}

interface IconButtonProps extends BaseButtonProps {
  icon: React.ReactNode;
  iconPosition?: "left" | "right";
}

interface ConfirmButtonProps extends BaseButtonProps {
  confirmMessage: string;
  onConfirm: () => void;
}

// 용도에 맞는 컴포넌트 사용
const SaveButton = ({ onSave }: { onSave: () => void }) => (
  <LoadingButton
    text="저장"
    onClick={onSave}
    isLoading={isSaving}
    loadingText="저장 중..."
  />
);
```

---

## 11장 - 의존성 역전 원칙 (DIP)

### 📖 로버트 마틴의 핵심 정의

**"상위 수준의 모듈은 하위 수준의 모듈에 의존해서는 안 된다. 둘 다 추상화에 의존해야 한다."**

의존성 역전 원칙에서 말하는 **"유연성이 극대화된 시스템"**이란 소스코드 의존성이 추상에 의존하며 구체에는 의존하지 않는 시스템

### 🎯 3가지 핵심 실천 사항

1. **변동성이 큰 구체 클래스를 참조하지 말라**

   - 대신 추상 인터페이스를 참조하라

2. **변동성이 큰 구체 클래스로부터 파생하지 말라**

   - 상속은 가장 강력한 의존성 관계이므로 신중하게 사용

3. **구체 함수를 오버라이드 하지 말라**
   - 구체 함수는 소스코드 의존성을 필요로 하므로, 의존성을 상속하게 됨

### 🔄 안정성과 변동성

> **"추상 인터페이스에 변경이 생기면 이를 구체화한 구현체도 따라서 수정해야 한다. 반대로 구현체에 변경이 생기더라도 그 구현체가 구현하는 인터페이스는 대부분 변경될 필요가 없다."**

**뛰어난 아키텍트의 특징:**

- 인터페이스의 변동성을 낮추기 위해 애씀
- 인터페이스를 변경하지 않고도 구현체에 기능을 추가할 수 있는 방법을 찾음

### 🏗️ 아키텍처 경계와 의존성 방향

```
[아키텍처 경계]
구체적인 것들 ← 곡선 → 추상적인 것들
     ↑                    ↑
   제어흐름              소스코드 의존성
   (런타임)              (컴파일타임)
```

**중요한 원리:**

- 곡선은 구체적인 것들로부터 추상적인 것들을 분리
- **소스 코드 의존성**은 곡선과 교차할 때 모두 **추상적인 쪽으로 향함**
- **제어흐름**은 소스코드 의존성과는 **정반대 방향**으로 곡선을 가로지름
- 소스 코드 의존성이 제어흐름과 반대로 역전되므로 **"의존성 역전"**

---

### 💻 프론트엔드에서의 DIP 적용

#### 🚫 잘못된 예시 (DIP 위반)

```typescript
// ❌ 구체적인 구현체에 직접 의존
class ApiClient {
  async fetchUsers(): Promise<User[]> {
    const response = await fetch("/api/users");
    return response.json();
  }
}

class UserListComponent {
  private apiClient: ApiClient; // 구체 클래스에 직접 의존

  constructor() {
    this.apiClient = new ApiClient(); // 강한 결합
  }

  async loadUsers() {
    return this.apiClient.fetchUsers(); // fetch에 종속
  }
}

// 문제점:
// 1. GraphQL로 변경하려면 UserListComponent 수정 필요
// 2. 테스트 시 실제 API 호출 발생
// 3. ApiClient 변경 시 UserListComponent도 영향
```

#### ✅ 올바른 예시 (DIP 준수)

```typescript
// ✅ 추상화 정의 (상위 수준 모듈이 정의)
interface UserRepository {
  getUsers(): Promise<User[]>;
  getUserById(id: string): Promise<User>;
  createUser(user: CreateUserDto): Promise<User>;
}

// 구체적인 구현체들 (하위 수준 모듈)
class RestApiUserRepository implements UserRepository {
  async getUsers(): Promise<User[]> {
    const response = await fetch("/api/users");
    return response.json();
  }

  async getUserById(id: string): Promise<User> {
    const response = await fetch(`/api/users/${id}`);
    return response.json();
  }

  async createUser(user: CreateUserDto): Promise<User> {
    const response = await fetch("/api/users", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(user),
    });
    return response.json();
  }
}

class GraphQLUserRepository implements UserRepository {
  async getUsers(): Promise<User[]> {
    const query = `
      query GetUsers {
        users { id name email }
      }
    `;
    const response = await this.graphqlClient.query(query);
    return response.data.users;
  }

  async getUserById(id: string): Promise<User> {
    const query = `
      query GetUser($id: ID!) {
        user(id: $id) { id name email }
      }
    `;
    const response = await this.graphqlClient.query(query, { id });
    return response.data.user;
  }

  async createUser(user: CreateUserDto): Promise<User> {
    const mutation = `
      mutation CreateUser($input: CreateUserInput!) {
        createUser(input: $input) { id name email }
      }
    `;
    const response = await this.graphqlClient.mutate(mutation, { input: user });
    return response.data.createUser;
  }
}

class MockUserRepository implements UserRepository {
  private users: User[] = [
    { id: "1", name: "John Doe", email: "john@example.com" },
    { id: "2", name: "Jane Smith", email: "jane@example.com" },
  ];

  async getUsers(): Promise<User[]> {
    return Promise.resolve([...this.users]);
  }

  async getUserById(id: string): Promise<User> {
    const user = this.users.find((u) => u.id === id);
    if (!user) throw new Error("User not found");
    return Promise.resolve(user);
  }

  async createUser(user: CreateUserDto): Promise<User> {
    const newUser = { ...user, id: Date.now().toString() };
    this.users.push(newUser);
    return Promise.resolve(newUser);
  }
}

// 상위 수준 모듈 - 추상화에만 의존
class UserListComponent {
  constructor(private userRepository: UserRepository) {} // 추상화에 의존

  async loadUsers() {
    return this.userRepository.getUsers();
  }

  async createUser(userData: CreateUserDto) {
    return this.userRepository.createUser(userData);
  }
}

// 사용 시점에 구체적인 구현체 주입
const restApiRepo = new RestApiUserRepository();
const graphqlRepo = new GraphQLUserRepository();
const mockRepo = new MockUserRepository();

const userListWithRest = new UserListComponent(restApiRepo);
const userListWithGraphQL = new UserListComponent(graphqlRepo);
const userListForTest = new UserListComponent(mockRepo);
```

#### 🎯 React에서의 DIP 활용

```typescript
// React Context를 통한 의존성 주입
interface ServiceContainer {
  userRepository: UserRepository;
  notificationService: NotificationService;
  logger: Logger;
}

const ServiceContext = React.createContext<ServiceContainer | null>(null);

const useUserRepository = (): UserRepository => {
  const services = useContext(ServiceContext);
  if (!services) throw new Error("ServiceContext not provided");
  return services.userRepository;
};

// 컴포넌트는 추상화에만 의존
const UserProfile: React.FC<{ userId: string }> = ({ userId }) => {
  const userRepository = useUserRepository(); // 추상화에 의존
  const [user, setUser] = useState<User | null>(null);

  useEffect(() => {
    userRepository.getUserById(userId).then(setUser);
  }, [userId, userRepository]);

  if (!user) return <div>Loading...</div>;

  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
};

// 앱 전체에 서비스 제공
const App = () => {
  const services: ServiceContainer = {
    userRepository: new RestApiUserRepository(),
    notificationService: new EmailNotificationService(),
    logger: new ConsoleLogger(),
  };

  return (
    <ServiceContext.Provider value={services}>
      <UserProfile userId="123" />
    </ServiceContext.Provider>
  );
};
```

### 🔧 팩토리 패턴과 DIP

```typescript
// 추상 팩토리 정의
interface RepositoryFactory {
  createUserRepository(): UserRepository;
  createPostRepository(): PostRepository;
}

// 구체적인 팩토리들
class RestApiRepositoryFactory implements RepositoryFactory {
  createUserRepository(): UserRepository {
    return new RestApiUserRepository();
  }

  createPostRepository(): PostRepository {
    return new RestApiPostRepository();
  }
}

class GraphQLRepositoryFactory implements RepositoryFactory {
  createUserRepository(): UserRepository {
    return new GraphQLUserRepository();
  }

  createPostRepository(): PostRepository {
    return new GraphQLPostRepository();
  }
}

// 상위 수준 모듈
class Application {
  constructor(private repositoryFactory: RepositoryFactory) {}

  initializeServices() {
    const userRepo = this.repositoryFactory.createUserRepository();
    const postRepo = this.repositoryFactory.createPostRepository();

    return {
      userService: new UserService(userRepo),
      postService: new PostService(postRepo),
    };
  }
}

// 환경에 따른 팩토리 선택
const factory =
  process.env.API_TYPE === "graphql"
    ? new GraphQLRepositoryFactory()
    : new RestApiRepositoryFactory();

const app = new Application(factory);
```

---

## 🚀 프론트엔드 실무에서의 통합 적용

### 1. 상태 관리에서의 DIP + ISP

```typescript
// ISP: 상태 관리 인터페이스 분리
interface ReadonlyState<T> {
  getState(): T;
  subscribe(listener: (state: T) => void): () => void;
}

interface WritableState<T> extends ReadonlyState<T> {
  setState(state: Partial<T>): void;
}

interface AsyncState<T> extends WritableState<T> {
  setLoading(loading: boolean): void;
  setError(error: string | null): void;
}

// DIP: 구체적인 상태 관리 라이브러리에 의존하지 않음
class ZustandStateManager<T> implements AsyncState<T> {
  // Zustand 구현
}

class ReduxStateManager<T> implements AsyncState<T> {
  // Redux 구현
}

// 컴포넌트는 필요한 인터페이스에만 의존
interface UserListProps {
  userState: ReadonlyState<User[]>; // 읽기만 필요
}

interface UserFormProps {
  userState: WritableState<User[]>; // 읽기/쓰기 필요
}
```

### 2. API 클라이언트에서의 적용

```typescript
// ISP: API 기능별 인터페이스 분리
interface AuthAPI {
  login(credentials: LoginDto): Promise<AuthResult>;
  logout(): Promise<void>;
}

interface UserAPI {
  getUsers(): Promise<User[]>;
  updateUser(id: string, data: UpdateUserDto): Promise<User>;
}

interface FileAPI {
  uploadFile(file: File): Promise<string>;
  downloadFile(url: string): Promise<Blob>;
}

// DIP: HTTP 클라이언트 추상화
interface HttpClient {
  get<T>(url: string): Promise<T>;
  post<T>(url: string, data: any): Promise<T>;
  put<T>(url: string, data: any): Promise<T>;
  delete<T>(url: string): Promise<T>;
}

class AxiosHttpClient implements HttpClient {
  // axios 구현
}

class FetchHttpClient implements HttpClient {
  // fetch 구현
}

// 구체적인 API 구현체들
class RestUserAPI implements UserAPI {
  constructor(private httpClient: HttpClient) {}

  async getUsers(): Promise<User[]> {
    return this.httpClient.get("/api/users");
  }

  async updateUser(id: string, data: UpdateUserDto): Promise<User> {
    return this.httpClient.put(`/api/users/${id}`, data);
  }
}
```

### 3. 테스팅에서의 장점

```typescript
describe("UserProfile Component", () => {
  it("사용자 정보를 올바르게 표시한다", async () => {
    // 모킹이 쉬워짐
    const mockUserRepository: UserRepository = {
      getUserById: jest.fn().mockResolvedValue({
        id: "1",
        name: "John Doe",
        email: "john@example.com",
      }),
      getUsers: jest.fn(),
      createUser: jest.fn(),
    };

    render(
      <ServiceContext.Provider value={{ userRepository: mockUserRepository }}>
        <UserProfile userId="1" />
      </ServiceContext.Provider>
    );

    expect(await screen.findByText("John Doe")).toBeInTheDocument();
    expect(mockUserRepository.getUserById).toHaveBeenCalledWith("1");
  });
});
```

---

## 💡 핵심 요약

### ISP (인터페이스 분리 원칙)

- **책의 핵심**: "불필요한 짐을 실은 무언가에 의존하지 말라"
- **FE 적용**: 컴포넌트 Props, 서비스 인터페이스를 역할별로 분리
- **장점**: 재사용성 향상, 번들 최적화, 테스트 용이성

### DIP (의존성 역전 원칙)

- **책의 핵심**: "소스코드 의존성이 제어흐름과 반대로 향하게 하라"
- **FE 적용**: API 클라이언트, 상태 관리를 추상화로 분리
- **장점**: 유연성 확보, 테스트 용이성, 구현체 교체 가능

### 실무 적용 포인트

1. **의존성 주입 컨테이너** 활용으로 DIP 구현
2. **인터페이스 기반 설계**로 TypeScript의 타입 안정성 활용
