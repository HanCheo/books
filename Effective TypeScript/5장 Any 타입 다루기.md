## 아이템 38. Any 타입은 가능한 한 좁은 범위에서 사용하기

any는 어떻게 사용해야 할까?

```typescript
function processBar(b: Bar) {
  /*...*/
}

function f() {
  const x = expressionReturningFoo(); // x : Foo;
  processBar(x);
  // ~ 'Foo' is not `Bar`
}
```

위와 같은 상황이 있다 return 받은 값을 대입해야하는데 타입이 다른경우이다. any를 사용하지 않는 것이 베스트긴 하나 any를 추가해서 해결할 수 있는 방안이 2가지가 있는데 다음과 같다.

```typescript
function f() {
  const x: any = expressionReturningFoo(); // x : any;
  prcessBar(x);

  return x; // x: any
}

function f() {
  const x = expressionReturningFoo(); // x : Foo;
  processBar(x as any);

  return x; // x: Foo
}
```

2가지 함수 모두 동일한 동작을하고 processBar가 동작을 한다. 다만 첫번째 함수처럼 `const x: any`를 하게되면 해당 함수 내부에서 x를 다른곳에서 사용하거나 x를 return 하면 any 타입으로 반환되어 이후작업이 타입체킹이 안되는 경우가 발생한다. 그나마 아래쪽으로 사용하는 방법이 낫다.

또한 다음과 같이 해결할 수도있다.

```typescript
function f() {
  const x = expressionReturningFoo(); // x : Foo;
  // @ts-ignore
  processBar(x);

  return x; // x: Foo
}
```

다음라인은 ts 검사를 안하게 해주는 코드로 any를 사용하지는 않지만 근본적인 해결은 아니다.

원인을 찾아 바로잡도록하자

함수와 동일하게 객체또한 부분 any로 검증을 피할 수 있다.

```typescript
// 타입에러가 나는 예시
const config: Config = {
  x: 1,
  y: 2,
  c: {
    key: value, // if type error is here
  },
};

// 안좋은 해결책
const config: Config = {
  x: 1,
  y: 2,
  c: {
    key: value,
  },
} as any;

// 그나마 나은 해결책
const config: Config = {
  x: 1, // 타입 체킹 됨
  y: 2, // 타입 체킹 됨
  c: {
    key: value as any,
  },
};
```

위와같이 any를 쓰게된다면 최소한으로 사용 할 수 있도록 하자. 물론 가장 좋은 것은 근본적인 원인을 찾아 any를 사용하는 것을 막는 것이 가장 좋다.

> any는 의도치 않은 타입안정성을 해치므로 최소화 해야한다.
>
> 함수의 반환타입이 any인 경우를 만들지 말자
>
> 강제로 타입 오류를 제거하려면 any 대신 @ts-ignore를 사용하는 것이 좋다.

<br />

## 아이템 39. Any를 구체적으로 변형해서 사용하기

any는 모든 값의 형태를 아우르므로 분명 좀더 나은 타입으로 변경할수 있다는 것을 명심해두고 있어야한다. 예를 든다면 다음과같다.

```typescript
// 안좋음.
function getArrayLength(a: any) {
  return a.length;
} // a는 모든값의 형태이므로 length가 있을수도 있고 없을 수도 있다.

// 그나마 나은 방식
function getArrayLength(a: any[]) {
  return a.length;
}
```

무심결에보면 타입이 더 많아진것 같지만 오히려 좀더 좁아진 형태이다.
위와 같이 함으로 얻는 점은 다음과같다.

1. Array.length 타입이 체킹이된다.
2. 반환타입이 any가아닌 number이다.
3. 함수가 호출될때 매개변수가 array인지 체킹된다.

배열로 사용할 때는 위처럼 `any[]`를 사용하면되고 객체로 사용할때는 `[key:string]: any` 형태로 사용하면 된다.

함수에서도 any를 좀더 좁은 형태로 가능하다

```typescript
type f1 = () => any; // 매개변수 없이 호출이 가능한 **함수**
type f2 = (arg: any) => any; // 한개의 매개변수를 가지는 함수
type f3 = (...arg: any[]) => any; //여러개의 매개변수를 가지는 함수배열
```

위 세가지 타입은 단순 any로 타입은 선언한것보다 좀더 구체적이다.

> any를 사용할 때 정말 모든 타입이 허용되어야 하는지 고민할 필요가 있다.
>
> any를 보다 정확하게 모델링할 수 있도록 `any[]`, `{[key: string]: any}`, `() => any` 와 같이 좀더 좁은 형태로 사용할 수 있도록 하게 하자.

<br />

## 아이템 40. 함수 안으로 타입 단언문 감추기

안전한 타입로직을 작성하는것은 가장 바람직하지만 함수 내부 로직이 복잡하여 함수 안전한 타입로직을 만드는데 어려움이 있다면 return 타입을 명확하게 하고 내부로직에서는 타입단언을 사용하는 것도 고민할 필요가 있다.

다시말하면 함수 내부에서는 타입단언으로 유연하게하고 return 타입은 명확하게 하는 방법이다.

2개의 배열을 받아서 같은지 비교하는 swallowEqual 함수를 확인해보자

```typescript
declare function swallowEqual(a: any, b: any): boolean;
function swallowEqual<T extends Object>(a: T, b: T) {
  for (const [k, aVal] of Object.entries(a)) {
    if (!(k in b) || aVal !== b[k]) {
      //error b[k] : Element implicitly has an 'any' type because expression of type 'string' can't be used to index type 'Object'.
      return false;
    }
  }
  return Object.keys(a).length === Object.keys(b).length;
}
```

if 구문의 `k in b`로 한번 체킹을 했으나 b[k]에서 에러가 발생햇다. 하지만 우리는 실제 오류가 아닌 것을 알고 있기 때문에 as any로 단언 하여 사용할 수 있다.

```typescript
declare function swallowEqual(a: any, b: any): boolean;
function swallowEqual<T extends Object>(a: T, b: T) {
  for (const [k, aVal] of Object.entries(a)) {
    if (!(k in b) || aVal !== (b[k] as any)) {
      return false;
    }
  }
  return Object.keys(a).length === Object.keys(b).length;
}
```

이렇게 하면 정확한 타입으로 구현되며 제대로된 함수로 동작하게된다. 객체가 같은지 비교하기 위해 객체 순회와 단언문이 코드에 들어가는 것보다 별도로 동작을 분리하여 설계하는 것이 좋은 방법이다.

> 타입 단언문은 일반적으로 타입을 위험하게 만들지만 상황에 따라 필요하기도 하고 현실적인 해결책이 되기도한다. 불가피하게 사용해야한다면 정확한 정의를 가지는 함수 안으로 숨기자.

<br />

## 아이템 41. any의 진화를 이해하기

이번 아이템에서는 명시적 타입 선언이 일어나지 않은 변수에 대해서 나타나는 any 타입의 변화에 대해서 말하고 있다.

```typescript
function range(start: number, limit: number) {
  const out = []; // any[]
  for (let i = start; i < limit; i++) {
    out.push(i); // any[]
  }
  return out; // number[]
}

let value; //any

if (Math.random() < 0.5) {
  value = /Hello/;
  value; // RegExp;
} else {
  value = 12;
  value; // number;
}
val; // RegExp | number;
```

위와 같이 암시적 any타입은 해당 변수에 어떤 값이 들어가냐에 따라 타입이 진화된다.
하지만 명시적으로 any 타입을 지정할 경우에는 타입이 그대로 유지된다.

```typescript
let value; //any

if (Math.random() < 0.5) {
  value = /Hello/;
  value; // any;
} else {
  value = 12;
  value; // any;
}
val; // any;
```

any의 타입진화는 암시적 any타입에 값을 할당할 때만 발생한다. 변수가 추론되는 원리와 동일하고 진화를 이용하여 최종적으로 return 되는 타입을 검증할수 있다 그렇지만 이렇게 타입을 진화시키는 방법보다 명시적으로 타입을 미리 선언하는 것이 더 좋은 설계이다.

> 일반적인 타입은 정제되기만 하지만 암시적 any와 any[] 타입은 진화가 가능하다. 이번 아이템은 이러한 동작이 발생하는 코드를 인지하는 것이 목표이다.
>
> 물론 any를 진화시키는 것보다 명시적 타입 구문을 사용하는 것이 안전하다.

<br />

## 아이템 42. 모르는 타입의 값에는 any 대신 unknown을 사용하기

any타입은 타입체킹을 무시하는 타입으로 강력하지만 typescript의 기능을 충분히 활용하지 못하는 타입과 같다.

이번 아이템에서는 any, unknown, never의 차이점을 알고간다.

any:

1. 어떠한 타입이든 any 타입에 할당이 가능하다.
2. Any 타입은 어떠한 타입으로도 할당 가능하다. (never 타입 예외)

unknown:

1. 어떠한 타입이든 unknown 타입에 할당이 가능하다.
2. unknown타입은 unknown과 any에 타입으로만 할당이 가능하다

never:

1. 어떠한 타입도 never에 할당할 수 없다.
2. never 타입은 어떠한 타입으로도 할당이 가능하다

```typescript
//any
function sum(a: number, b: number) {
  return a + b;
}

let a: any = 1;
let b: any = 2;

sum(a, b);
//여기서 a,b의 타입은 any이지만 sum의 매개변수에 할당이 가능해진다.

//unknown
let a: unknown = 1;
let b: unknown = 2;

sum(a, b); //ERROR
//Argument of type 'unknown' is not assignable to parameter of type 'number'.
//unknown은 unknown, any를 제외한 타입에는 할당이 불가능하다.

//never
let a: never = 1; //Error
// Type 'number' is not assignable to type 'never'.
let b: never = 2; //Error
// Type 'number' is not assignable to type 'never'.
//어떠한 타입도 never에 할당할 수 없다.
sum(a, b);

//unknown never
let a: unknown = 1;
let b: unknown = 2;

sum(a as never, b as never); //pass
//never은 어떠한 타입으로도 할당이 가능하다. 위와같이 사용하면 안되지만 any와 동일한 동작을한다.
```

<br />

### 이중단언문에서 undefined 사용하기

```typescript
declare const foo: Foo;
let barAny = foo as any as Bar;
let barUnk = foo as unknown as Bar;
```

> `barAny`와 `barUnk`는 기능적으로 동일하지만 나중에 두개의 단언문을 분리하는 리팩터링을 하게되는 경우 unknown 형태가 더 안전하다. any는 분리되는 순간 영향력이 퍼진다

<br />

### unknown 처럼 사용되었던 object, {} 차이 파악하기

- {} 타입은 null과 undefined를 제외한 모든 값을 포함한다.
- object 타입은 모든 비기본형(non-primitive) 타입으로 이루어진다. 여기에는 `true` 또는 `12` 또는 "foo"가 포함되지 않지만 객체와 배열은 포함된다.

`{}`타입은 `unknown`에서 `null`과 `undefined`를 제외한 타입이라 생각하면된다. 따라서 `null`과 `undefined`가 불가능하다고 판단되는 경우만 `unknown 대신 `{}`를 사용하면된다.

<br />

## 아이템 43. 몽키 패치보다는 안전한 타입을 사용하기

> 몽키 패치
>
> 프로그램을 확장하거나, 로컬 시스템 소프트웨어를 지원하고 수정하는 방법이다.(그냥 런타임 중 코드를 수정한다는 의미) - 오직 실행중인 프로그램의 인스턴스에 영향을 미친다.

자바스크립트의 특징중 하나는 선언된 객체와 클래스에 임의의 속성을 추가할 수 있을만큼 유연하다. 예를 들어 브라우저 환경에서는 document, window와 같은 객체에도 값을 할당하여 전역변수를 만들기도 한다.

```typescript
window.monkey = 'Tamarin';
document.monkey = 'Howler';
```

또는 domElement에 직접 접근하여 데이터를 추가하기 위해서도 사용한다.

```typescript
const el = document.getElementById('colobus');
el.home = 'tree';
```

심지어 내장 프로토타입에도 속성을 추가할 수 있다.

```typescript
RegExp.prototype.monkey = 'Capuchin';
/123/.monkey; // Capuchin;
```

다만 자바스크립트에서 해당되는 것이며 타입스크립트에서는 문제가 발생한다. 타입 체커는 Document와 HTMLElement의 속성에 대해서는 알고는 있지만 임의로 추가한 속성에 대해서는 모른다.

<br />

### 타입 확장하기

```typescript
document.monkey = 'Tarmarin';
//error `monkey` not in document;
```

any 사용하기

```typescript
(document as any).monkey = 'Tamarin'; // 정상
```

하지만 타입안정성을 상실하고 타입스크립트 언어 서비스를 사용할 수 없다.

타입 보강하기

```typescript
interface Document {
  monkey: string;
}
document.monkey = 'Tamarin'; //pass
```

interface 타입의 특성을 이용하여 document 타입을 보강하여 사용한다. any보다 좋은 이유는 다음과같다.

1.  타입이 더 안전하다. 타입오류를 표시해준다.
2.  TSDoc을 이용해 속성에 주석을 붙일 수 있다.
3.  속성 자동완성 사용이 가능하다.
4.  몽키 패치가 어떤 부분에 적용되었는지 기록이 남는다.

Global 타입 보강하기.

모듈의 관점에서 타입을 보강하는 방법이다.

```typescript
export {};
declare global {
  interface Document {
    /**몽키 패치 속(genus) 또는 종(species) */
    monkey: string;
  }
}
document.monkey = 'Tamarin'; //pass
```

타입 단언문 사용하기

```typescript
interface MonkeyDocument extends Document {
  /**몽키 패치 속(genus) 또는 종(species) */
  monkey: string;
}
(document as MonkeyDocument).monkey = 'Macaque';
```

`Document`를 `extends`하여 사용하기 떄문에 타입은 안전하고 `Document` 타입을 그대로 사용하는 것이 아닌 별도로 빼내어서 사용했기 때문에 모듈영역에서도 안전하다. 따라서 몽키패치된 속성을 사용하는 곳에서만 단언을 사용하면 된다.

<br />

> 그래도 몽키패치는 되도록 사용하면 좋지 않다.
>
> - 전역 변수나 DOM에 데이터를 저장하지 말고, 데이터를 분리하여 사용해야한다.
> - 내장 타입에 데이터를 저장하는 경우, 안전한 타입 접근법 중 하나(보강, 사용자정의 인터페이스로 단언)을 사용해야한다.
> - 보강의 모듈 영역 문제를 이해해야한다.

<br />

## 아이템 44. 타입 커버리지를 추적하여 타입안정성 유지하기

현재 프로젝트의 any타입이 얼마나 사용되었는지 추적할수 있는 npm 패키지이다.

```bash
npx type-coverage
9985 / 10117 98.69%
```

### 파일단위로 추적하기

```bash
npx type-coverage --detail
path/to/code.ts:1:10 getColumnInfo
path/to/module.ts:7:1 pt2
...
```

이를 이용하여 any의 근원지를 찾아 any타입 사용처를 해결할 수 있다.

> noImplicitAny가 설정되어 있어도, 명시적 any 또는 서드파티 타입 선언 (@types)을 통해 any 타입은 코드 내에 여전히 존재할수 있다.
> 작성된 프로그램 타입이 얼마나 잘 선언되어 있는지 추적할 수 있어야한다.
