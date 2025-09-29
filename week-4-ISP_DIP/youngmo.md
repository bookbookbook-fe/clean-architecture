## ISP 인터페이스 분리원칙

> 클라이언트는 사용하지않는것에 의존하지않아야한다.

## DIP 의존성 역전원칙

> 추상에 의존하며, 구체에는 의존하지않는 원칙

```jsx
    // 1. 인증 서비스 인터페이스
    interface IAuthService {
      login(credentials: any): Promise<User>;
      logout(): Promise<void>;
      getCurrentUser(): User | null;
    }
    
    // 2. 인증 컨텍스트 및 프로바이더 구현
    const AuthContext = createContext<IAuthService | null>(null);
    
    const AuthProvider = ({ authService, children }) => {
      return (
        <AuthContext.Provider value={authService}>
          {children}
        </AuthContext.Provider>
      );
    };
```

```jsx
    // 3. 구체적인 인증 서비스 구현
    // 추후 다른 인증 방법 추가 시 IAuthService를 기반으로 새로운 클래스 생성
    class BasicAuthService implements IAuthService {
      // BasicAuthService의 구체적인 구현
      login(credentials: any): Promise<User> {
        // 로그인 로직
      }
    
      logout(): Promise<void> {
        // 로그아웃 로직
      }
    
      getCurrentUser(): User | null {
        // 현재 사용자 정보 반환
      }
    }
```

```jsx
  // 4. 컨텍스트를 사용하는 로그인 컴포넌트
    const Login = () => {
      const authService = useContext(AuthContext);
    
      const handleLogin = async () => {
        try {
          const user = await authService.login({ username: 'user', password: 'pass' });
          console.log('Logged in user:', user);
        } catch (error) {
          console.error('Login failed:', error);
        }
      };
    
      return <button onClick={handleLogin}>Login</button>;
    };
```

```jsx
    // 5. 최종 구현
    const App = () => {
      const authService = new BasicAuthService();
    
      return (
        <AuthProvider authService={authService}>
          <Login />
        </AuthProvider>
      );
    };
```

