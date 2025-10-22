# 📚 Week 4: ISP, DIP

## 📅 일정

**기간**: 9/17 ~ 9/23  
**모임**: 9/24 (수)

## 📖 읽은 챕터

- 10장 - ISP: 인터페이스 분리 원칙
- 11장 - DIP: 의존성 역전 원칙

## 💭 주요 인사이트

### ISP (Interface Segregation Principle)

- 클라이언트는 자신이 사용하지 않는 메서드에 의존하지 않아야 함
- **정적 타입 언어(Java, C#)**: 인터페이스 변경 시 사용하지 않는 메서드라도 재컴파일 필요 → ISP 중요
- **JavaScript/TypeScript**: 런타임에 실제 사용하는 메서드만 호출, 구조적 타이핑(structural typing) 특성상 ISP 위반의 실질적 영향이 적음

### DIP (Dependency Inversion Principle)

- 구체적인 구현체가 아닌 추상화에 의존해야 함
- 프론트엔드 예시: 컴포넌트에서 axios 직접 의존 → 인터페이스를 통한 의존으로 개선
