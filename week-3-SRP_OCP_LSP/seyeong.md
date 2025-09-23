# 클린 아키텍처 7-9장 정리

## 📋 목차

- [7장: 단일 책임 원칙 (SRP)](#7장-단일-책임-원칙-srp-single-responsibility-principle)
- [8장: 개방-폐쇄 원칙 (OCP)](#8장-개방-폐쇄-원칙-ocp-open-closed-principle)
- [9장: 리스코프 치환 원칙 (LSP)](#9장-리스코프-치환-원칙-lsp-liskov-substitution-principle)
- [정리 및 다음 학습 방향](#-정리-및-다음-학습-방향)

---

## 7장 - 단일 책임 원칙 (SRP: Single Responsibility Principle)

### 📖 핵심 개념

> "각 소프트웨어 모듈은 변경의 이유가 하나, 단 하나여야만 한다."

단일 책임 원칙은 SOLID 원칙 중 가장 이해하기 어려운 원칙 중 하나. 많은 개발자들이 "함수는 하나의 일만 해야 한다"고 잘못 이해하지만, 실제로는 **"하나의 모듈은 변경의 이유가 하나여야 한다"**는 의미

### 📖 책임의 정의

- **책임(Responsibility)**: 변경을 요구하는 사용자들을 지칭하는 용어
- **액터(Actor)**: 변경을 요구하는 한 명 이상의 사람들을 가리키는 집단
- **모듈(Module)**: 하나 이상의 함수와 데이터 구조로 구성된 응집된 집합

### 📖 SRP 위반의 증상

#### 1. 우발적 중복 (Accidental Duplication)

Employee 클래스가 세 가지 메서드를 가진다고 가정:

- `calculatePay()`: 회계팀에서 사용 (CFO 보고)
- `reportHours()`: 인사팀에서 사용 (COO 보고)
- `save()`: DBA팀에서 사용 (CTO 보고)

이 경우 SRP를 위반하는 이유:

- 세 메서드가 서로 다른 액터를 책임지고 있음
- 한 액터를 위한 변경이 다른 액터에게 영향을 줄 수 있음

=> SRP는 서로 다른 액터가 의존하는 코드를 서로 분리하라고 말함.

```javascript
// SRP 위반 예시
class Employee {
  calculatePay() {
    // CFO 팀이 명세한 정책
    return this.regularHours() * this.regularRate;
  }

  reportHours() {
    // COO 팀이 명세한 정책
    return this.regularHours();
  }

  save() {
    // CTO 팀이 명세한 정책
    // 데이터베이스에 저장
  }

  regularHours() {
    // 공통 로직 - 위험 요소!
  }
}
```

#### 2. 병합 (Merge)

같은 소스 파일을 서로 다른 이유로 변경하는 두 개발자가 있을 때 병합 충돌이 발생할 수 있음

### 📖 해결방안

#### 1. 메서드 수준에서 분리

```javascript
// 해결방안 1: 각 책임을 별도 클래스로 분리
class PayCalculator {
  calculatePay(employee) {
    return employee.regularHours() * employee.regularRate;
  }
}

class HourReporter {
  reportHours(employee) {
    return employee.regularHours();
  }
}

class EmployeeSaver {
  save(employee) {
    // 데이터베이스 저장 로직
  }
}

class EmployeeData {
  // 공통 데이터만 보관
  regularHours() {
    /* ... */
  }
  regularRate() {
    /* ... */
  }
}
```

#### 2. 퍼사드 패턴 사용

```javascript
// 해결방안 2: 퍼사드로 복잡성 숨기기
class EmployeeFacade {
  constructor(employee) {
    this.employee = employee;
    this.payCalculator = new PayCalculator();
    this.hourReporter = new HourReporter();
    this.employeeSaver = new EmployeeSaver();
  }

  calculatePay() {
    return this.payCalculator.calculatePay(this.employee);
  }

  reportHours() {
    return this.hourReporter.reportHours(this.employee);
  }

  save() {
    this.employeeSaver.save(this.employee);
  }
}
```

### 📖 컴포넌트 수준에서의 SRP

SRP는 다양한 수준에서 적용

- **함수 수준**: 하나의 일만 수행
- **클래스 수준**: 변경의 이유가 하나
- **컴포넌트 수준**: 공통 폐쇄 원칙(CCP)
- **아키텍처 수준**: 아키텍처 경계 생성의 기준

---

## 8장 - 개방-폐쇄 원칙 (OCP: Open-Closed Principle)

### 📖 핵심 개념

> "소프트웨어 개체는 확장에는 열려 있어야 하고, 변경에는 닫혀 있어야 한다."

OCP는 1988년 Bertrand Meyer가 제시한 원칙으로, 기존 코드를 변경하지 않고도 시스템의 행위를 확장할 수 있어야 한다는 것

### 📖 사고 실험: 재무제표 출력

재무제표 데이터를 웹페이지로 출력하는 시스템이 있다고 가정했을 때, 이제 동일한 정보를 보고서 형태로 출력해야 한다는 요구사항이 추가

#### 잘못된 접근 (OCP 위반)

```javascript
class FinancialReportGenerator {
  generateReport(data, format) {
    if (format === "web") {
      return this.generateWebPage(data);
    } else if (format === "print") {
      return this.generatePrintReport(data);
    }
    // 새로운 형식 추가 시 이 코드를 수정해야 함
  }
}
```

#### 올바른 접근 (OCP 준수)

책임을 분리하고 의존성 방향을 제어

1. **재무 데이터 분석** (고수준)
2. **웹 출력** (저수준)
3. **프린터 출력** (저수준)

```javascript
// 고수준 정책
class FinancialDataProcessor {
  constructor(presenter) {
    this.presenter = presenter;
  }

  process(rawData) {
    const processedData = this.calculateFinancialData(rawData);
    this.presenter.present(processedData);
  }

  calculateFinancialData(rawData) {
    // 핵심 비즈니스 로직 (변하지 않음)
    return {
      revenue: rawData.income - rawData.expenses,
      // ... 기타 계산
    };
  }
}

// 추상화
class FinancialDataPresenter {
  present(data) {
    throw new Error("구현 필요");
  }
}

// 저수준 구현체들
class WebPresenter extends FinancialDataPresenter {
  present(data) {
    return `<html><body>${JSON.stringify(data)}</body></html>`;
  }
}

class PrintPresenter extends FinancialDataPresenter {
  present(data) {
    return `재무 보고서\n수익: ${data.revenue}`;
  }
}

// 새로운 출력 형식 추가 시 기존 코드 변경 없음
class PDFPresenter extends FinancialDataPresenter {
  present(data) {
    return this.generatePDF(data);
  }
}
```

### 📖 정보 은닉과 방향성 제어

OCP를 달성하기 위한 핵심 전략

1. **처리 과정을 단계별로 분리**
2. **고수준 단계가 저수준 단계를 보호하도록 의존성 방향 제어**
3. **추상화를 통한 의존성 역전**

### 📖 아키텍처 수준에서의 OCP

가장 중요한 것은 **변경으로부터 시스템을 보호하는 것**

- **Interactor**(업무 규칙)가 가장 높은 수준
- **Controller**, **Presenter**가 중간 수준
- **View**, **Database**가 가장 낮은 수준

의존성은 항상 고수준을 향해야 하며, 저수준의 변경이 고수준에 영향을 주어서는 안 된다.

💭 **개인 생각**

- 버튼 컴포넌트를 만들 때 다양한 스타일(primary, secondary, danger)을 추가할 수 있도록 설계하되, 기본 Button 컴포넌트는 변경하지 않는 것이나, render props나 compound component 패턴도 OCP의 구현 사례가 아닐까

---

## 9장 - 리스코프 치환 원칙 (LSP: Liskov Substitution Principle)

### 📖 핵심 개념

> "상호 대체 가능한 구성요소를 이용해 소프트웨어 시스템을 만들 수 있으려면, 구성요소는 반드시 서로 치환 가능해야 한다."

간단히 말해, **하위 타입은 언제나 상위 타입으로 대체할 수 있어야 함.**

### 📖 상속을 사용하도록 가이드하기

LSP는 상속을 올바르게 사용하는 방법을 알려준다.
하위 클래스는 상위 클래스가 사용되는 곳에서 언제나 상위 클래스로 교체할 수 있어야 한다.

### 📖 정사각형/직사각형 문제

LSP 위반의 전형적인 예시:

```javascript
// LSP 위반 사례
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

  setWidth(width) {
    this.width = width;
    this.height = width; // 정사각형은 너비와 높이가 같아야 함
  }

  setHeight(height) {
    this.width = height; // 정사각형은 너비와 높이가 같아야 함
    this.height = height;
  }
}

// 문제가 되는 클라이언트 코드
function User(rectangle) {
  rectangle.setWidth(5);
  rectangle.setHeight(4);

  // Rectangle이면 20이어야 하지만, Square면 16이 됨
  assert(rectangle.getArea() === 20); // Square일 때 실패!
}
```

### 📖 해결방안

```javascript
// LSP 준수 해결방안
class Shape {
  getArea() {
    throw new Error("구현 필요");
  }

  isSquare() {
    return false;
  }
}

class Rectangle extends Shape {
  constructor(width, height) {
    super();
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

class Square extends Shape {
  constructor(side) {
    super();
    this.side = side;
  }

  setSide(side) {
    this.side = side;
  }

  getArea() {
    return this.side * this.side;
  }

  isSquare() {
    return true;
  }
}
```

### 📖 라이선스 예제

올바른 LSP 적용 사례:

```javascript
class License {
  calculateFee() {
    throw new Error("구현 필요");
  }
}

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

class Billing {
  static calculateLicenseFee(license) {
    return license.calculateFee(); // 어떤 라이선스든 동일하게 작동
  }
}
```

### 📖 아키텍처 수준에서의 LSP

LSP는 인터페이스와 구현체에 관한 것이며, 아키텍처 수준에서는 치환 가능한 컴포넌트들이 동일한 인터페이스를 준수해야 함을 의미

예를 들어, REST 서비스들이 서로 치환 가능하려면

- 동일한 인터페이스 준수
- 서로의 위치를 대체할 수 있어야 함
- 클라이언트가 구별할 수 없어야 함

### 📖 결론: 세 원칙의 상호작용

- **SRP**: 변경의 이유에 따라 코드를 분리
- **OCP**: 분리된 컴포넌트를 단방향 의존성 계층구조로 조직화
- **LSP**: 계층구조에서 치환 가능성을 보장

이 세 원칙이 함께 작용하여 개방-폐쇄 원칙을 달성할 수 있는 인터페이스와 구현체를 생성하게 된다.

💭 **개인 생각**

- LSP는 TypeScript를 사용할 때 특히 중요함을 느낌
- 인터페이스나 제네릭을 잘못 설계하면 런타임에 예상치 못한 동작이 발생
- React에서 props의 타입 정의 시에도 LSP를 고려해야 함 (optional props, default props 등)
- 실제로 상속보다는 composition을 선호하는 React의 철학이 LSP 위반을 줄이는 방법 중 하나

---

## 🔖 정리 및 다음 학습 방향

### 핵심 포인트

1. **SRP**: 변경의 이유에 따라 코드를 분리하여 응집성 높이기
2. **OCP**: 확장에는 열려있고 변경에는 닫힌 설계로 유연성 확보
3. **LSP**: 상위 타입과 하위 타입의 치환 가능성으로 안정성 보장

### 세 원칙의 시너지

- SRP로 역할을 분리하고, OCP로 확장 가능한 구조를 만들며, LSP로 안전한 대체를 보장
- 함께 적용될 때 플러그인 아키텍처의 기반이 됨
- 프론트엔드에서는 컴포넌트 설계, 상태 관리, 비즈니스 로직 분리에 직접적으로 적용 가능

### 다음 장 학습 포인트

- ISP(Interface Segregation Principle)와 DIP(Dependency Inversion Principle)
- SOLID 원칙들이 실제 아키텍처 설계에 미치는 영향
- 컴포넌트 원칙과의 연관성

---

_📝 7~9장 학습 완료일: 2025-01-20_
