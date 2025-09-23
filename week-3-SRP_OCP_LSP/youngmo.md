## SRP 단일 책임원칙

> 서로다른 액터가 의존하는 코드

```js
export class Employee {
  constructor(person){
    this.person = person
  }
  // CTO를 위한 메서드
  calculatePay(){
    ...
  }
  // CFO를 위한 메서드
  reportHours(){
    ...
  }
  // COO를 위한 메서드
  save(){
    ...
  }
}
```

각 이해관계자의 기능들이 수정될때 Employee 클래스는 자꾸 바뀌게 되어 테스트도 서로 영향을 주게 된다.

> 해결방법

책임분리 + 퍼사드 (메서드와 클래스 수준의 원칙)

- Entity
- Service
- Facade

```js
<!-- Entity -->
export class EmployeeRecord {
  constructor(person){
    this.person = person
  }
}
// Service
export class PayCalculator {
  static caculator(){
    ...
  }
}
// Service
export class HourReporter {
  static report(){
    ...
  }
}
// CFO를 위한 메서드
export class Save {
  static save(){
    ...
  }
}
// Facade
export class Employee {
  constructor(person){
    this.record = person
  }
  calculatePay(){
    return PayCalculator.caculator(this.record)
  }
  reportHours(){
    return HourReporter.report(this.record)
  }
  save(){
    return Save.save(this.record)
  }
}
```

## OCP 개방-폐쇄 원칙

> 처리과정을 클래스단위로 분리하고 클래스는 컴포넌트 단위로 분리한다.
> 확장에는 열려있고 변경에는 닫혀 있어야한다.


예로는 컴파운드 컴포넌트 패턴


```tsx
<Card title="foo" content="bar" comments={comments} user={user}  />
```

props으로 추가되는 요구사항들을 구현하게되는 경우 Card 컴포넌트 전체 변경사항이 자주 일어나게 된다.

```tsx
<Card>
	<Card.Title>foo</Card.Title>
	<Card.Content>bar</Card.Content>
	{user
		? <Card.CommentsWithAuth comments={comments} />
		: <Card.CommentsWithAnonymous comments={comments} />
	}
</Card>
```

Card 이하 서브컴포넌트들은 각각 명확한 책임을 가지게되고, 책임이 분리되었으나, Card 컴포넌트에 종속되어있어 응집도가 높습니다. (SRP)

Card 기능이 추가될때 기존코드를 수정하지않고 확장할수가 있습니다. (OCP)

## LSP 리스코프 치환 원칙

> 하위타입은 상위타입을 대체 할수 있어야한다.

```tsx
type UsePermissionForm = (initialValues: any, onSubmit: (values: any) => void) => {
  values: any,
  handleChange: (event: React.ChangeEvent<HTMLInputElement>) => void,
  handleSubmit: (event: React.FormEvent<HTMLFormElement>) => void
}
```

UsePermissionForm이라는 인터페이스 (상위타입)

```tsx
import { useFormik } from 'formik';

const useCustomFormWithFormik = (initialValues, onSubmit) => {
  const formik = useFormik({
    initialValues: initialValues,
    onSubmit: onSubmit,
  });

  return {
    values: formik.values,
    handleChange: formik.handleChange,
    handleSubmit: formik.handleSubmit,
  };
};

export default useCustomFormWithFormik;
```

formik 라이브러리를 사용한 하위타입으로 대체 가능해야한다.

```tsx
import { useForm } from 'react-hook-form';

const useCustomFormWithReactHookForm = (initialValues, onSubmit) => {
  const { register, handleSubmit, setValue } = useForm({
    defaultValues: initialValues
  });

  const handleChange = (event) => {
    setValue(event.target.name, event.target.value);
  };

  return {
    values: register,
    handleChange: handleChange,
    handleSubmit: () => handleSubmit(data => onSubmit(data)),
  };
};

export default useCustomFormWithReactHookForm;
```

react-hook-form 라이브러리를 사용한 하위타입으로 대체 가능해야한다.
