## 아이템 45. devDependencies에 typescript와 @types 추가하기

- dependencies
  현재 프로젝트를 실행하는 데 필수적인 라이브러리가 포함된다. 프로젝트 런타임에 lodash가 사용된다면 `dependencies`에 포함되어야 한다. 프로젝트를 npm에 공개하여 다른 사용자가 해당 프로젝트를 설지한다면, `dependencies`에 들어있는 라이브러리도 함께 설치된다. 이러한 현상을 전이 의존성이라고 한다.


- devDependencies
  현재 프로젝트를 개발하고 테스트하는데 사용되지만 런타임에는 필요없는 라이브러리가 포함된다. 예를들어 테스트프레임워크 (jest, mocha, ...), @types등이 해당된다. 프로젝트를 npm에 공개하여 다른 사용자가 해당 프로젝트를 설치한다면, devDependencies 포함된 라이브러리는 제외된다.


- peerDependencies
  런타임에 필요하긴 하지만, 의존성을 직접 관리하지 않는 라이브러리들이 포함된다. 단적인 예로 플러그인을 들 수 있다. 제이쿼리의 플러그인은 다양한 버전의 제이쿼리와 호환되므로 제이쿼리의 버전을 플러그인에서 직접 선택하지 않고, 플러그인이 사용되는 실제 프로젝트에서 선택하도록 만들 때 사용한다.

<br />

### PeerDependencies

`peerDependencies` 에 대해서 여기서 처음 들었는데 내용이 이해가 잘 안되어 좀더 찾아보았다.

위의 제이쿼리의 예를 들어본다면 `dependencies`에 제이쿼리 플러그인을 사용했는데 해당 플러그인이 제이쿼리 1.5 버전과 맞춰 동작한다면 해당 정보를 표시해야하는데 이때 사용하는것이다.

```json
{
  "name" : "JQuery-Project"
  "version" : "1.3.5"
  "peerDependencies": {
  	"jquery": "1.5"
	}
}
```

즉, 내 프로젝트가 어떤 호환성을 가지고 있는지 표시하는 것과 동일하다.

> 추가설명
>
> 보통 `package.json`에서 `dependencies`나 `devDependencies`를 사용하지만 `peerDependencies`는 gulp, grunt 같은 도구의 플러그인을 제작할 때 사용한다.  
> 예를 들어 플러그인이 gulp v3 이상에서만 동작할 때 gulp v3가 설치되어 있다는 전제가 필요하고 이는 소스에서 사용하는 `dependencies`과는 다른데 이를 `peerDependencies`라고 부른다.  
> 또다른 예는 [Chai Assertions for Promises](https://github.com/domenic/chai-as-promised)의 `peerDependencies`는 `"chai": ">= 2.1.2 < 4"`라고 정의되어 있는데 `npm install chai-as-promised`로 설치하면 npm v2에서는 필요한 `peerDependencies`를 함께 설치한다
>
> ```bash
> $ npm ls
> /Users/outsider/peer
> ├─┬ chai@3.5.0
> │ ├── assertion-error@1.0.2
> │ ├─┬ deep-eql@0.1.3
> │ │ └── type-detect@0.1.1
> │ └── type-detect@1.0.0
> └── chai-as-promised@5.3.0
> ```
>
> npm v3에서는 이전처럼 자동으로 설치하지 않고(`peerDependencies`가 꼬이면 피곤하다.) `peerDependencies`가 충족되지 않으면 다음과 같이 경고가 나타난다.
>
> ```
> $ npm ls
> /Users/outsider/peer
> ├── UNMET PEER DEPENDENCY chai@>= 2.1.2 < 4
> └── chai-as-promised@5.3.0
>
> npm ERR! peer dep missing: chai@>= 2.1.2 < 4, required by chai-as-promised@5.3.0
> ```
>
> [npm v3에서 달라진 점 ](https://blog.outsider.ne.kr/1230)

<br />

### 타입스크립트릴 시스템 레벨로 설치하지말자

타입스크립트 프로젝트의 devDependencies에 타입스크립트를 포함시키고 팀원 모두가 동일한 버전을 사용하도록 하자. 만약 버전이 다르게 프로젝트가 진행된다면 코드가 꼬일 수 있다.

> @types 의존성은 dependencies가 아니라 devDependencies에 포함시켜야한다. 런타임에 @types가 필요한 경우라면 별도의 작업이 필요하다.

<br />

## 아이템 46. 타입 선언과 관련된 세 가지 버전 이해하기

타입스크립트를 사용하게된다면 추가적으로 살펴봐야할 버전이 있다.

1. 라이브러리의 버전
2. 라이브러리 @types의 버전
3. 타입스크립트의 버전

위 세가지 버전 중 하나라도 맞지 않으면 의존성과는 상관없어 보이는 곳에서 엉뚱한 오류가 발생할 수 있다.
이번 아이템 내용을 모르고 지나쳤다면 단순 보강이나 타입을 확장하여 사용하여 해결했겠지만 근본적인 원인파악을 하지는 못했을 것이다.

<br />

### 라이브러리와 @types를 별도로 관리하는 방식

기존 자바스크립트 프로젝트에 `definitelytyped`에 별도로 타입 정보를 추가하여 관리하는 방법이다. 이와 같은 방법에는 4가지 문제점이 있다.

1. 라이브러리를 업데이트 했지만 타입 선언은 업데이트 하지 않는 경우  
   -> 라이브러리의 신규 기능을 사용할때마다 타입 오류가 발생한다. 특히나 큰 버전 업데이틀 하위 호환성이 깨지는 경우는 타입체커를 통과하더라도 런타임 오류가 발생한다.  
   -> 보강기법을 사용하거나 타입선언을 직접 업데이트 하여 커뮤니티에 기여하는 방법이 있다.

<br />

2. 라이버리보다 타입 선언 버전이 최신인 경우  
   -> 타입 정보 없이 라이브러리를 사용해오다가 뒤늦게 타입 선언을 설치하려고 할 때 발생한다. 이런 경우는 드물지만 설치하기 전에 라이브러리가 업데이트 되고 타입이 업데이트 되었다면 문제가 발생한다.  
   -> 라이브러리 버전을 올리거나 타입 선언 버전을 내린다.

<br />

3. 프로젝트에서 사용하는 타입스크립트 버전보다 라이브러리에서 필요로하는 타입스크립트 버전이 더 최신인 경우. 일반적으로 로대시, 리액트, 람다 같은 유명 자바스크립트 라이브러리는 타입정보를 좀 더 정확하게 표현하기 위해 타입스크립트가 개선 되면 버전이 올라간다.
   현재 프로젝트보다 라이브러리 타입스크립트 버전이 높다면 @types 선언에서 문제가 발생한다.
   -> 타입스크립트 버전을 올리거나, 해당 라이브러리의 타입선언을 낮춘다.

<br />

4. `@types`의존성이 중복 되는 경우. 런타임에 사용되는 경우라면 타입이 제거되므로 큰 문제는 없지만 `global namespace`에 사용되는 경우라면 문제가 발생한다. 이런 경우에는 `npm ls @types/moduleName`을 실행하여 타입선언 중복 발생을 추적하고 중복된 @types의 버전을 서로 호환되도록 맞추는 방법이 있다.

<br />

### 타입선언을 포함하는 라이브러리

타입스크립트로 작성된 라이브러리들은 자체적으로 타입 선언을 포함(번들링)하게 됩니다. 자체적인 타입 선언은 보통 `package.json`의 "types"필드에서 .d.ts 파일을 가리키도록 되어 있다. 별도로 @types를 만들어주지 않아도 되어 버전 불일치를 해결할 수 있는 장점이 있지만 다음과 같은 4가지 문제점이 있다.

1. 번들된 타입 선언에 보강 기법으로 해결할 수 없는 오류가 있는 경우, 또는 공개 시점에는 잘 동작했지만 타입스크립트 버전이 올라가면서 오류가 발생하는 경우.
   -> @types가 별도로 사용된다면 라이브러리 자체의 버전에 맞추어 선택하지만 번들된 타입은 @types의 버전 선택이 불가능하다. `DefinitelyTyped`는 타입스크립트 버전이 올라갈 때마다 모든 타입 선언을 점검하며, 문제가 발생한 곳은 빠른 시간내에 해결하고 있다.
   <br />

2. 프로젝트 내의 타입 선언이 다른 라이브러리의 타입 선언에 의존한다면 문제가 된다. 타입은 보통 `devDependencies`에 들어가고 프로젝트를 공개하고 `install` 할경우 `dependencies`에 있는 라이브러리만 설치되므로 dev에 설치된 `@types`는 설치되지 않아 타입오류가 발생한다. 그렇다고 dependencies에 포함하기에는 `javascript`만 사용하는 사람은 별로 좋아하지 않을 소식이다.
   -> `DefinitelyTyped`에 타입선언을 공개하는 경우라면 `javascript`를 사용하는 경우는 걱정거리가 없고 타입 선언은 @types에 있으므로 타입스크립트 사용자만이 타입정보를 사용하게 된다. 첫번째 문제에 대해서는 추후 타입스크립트 선언 방법으로 다룬다.
   <br />

3. 프로젝트의 과거 버전에 있는 타입 선언에 문제가 있는 경우 과거 버전으로 돌아가서 패치 업데이트를 해야한다.
   -> `DefinitelyTyped`에서는 동일 라이브러리의 여러 버전의 타입 선언을 동시에 유지보수 할 수 있다.
   <br />

4. 타입 선언의 패치 업데이트를 자주 하기 어렵다는 문제점이 있다. react의 경우 라이브러리보다 타입선언에 대한 패치 업데이트가 훨씬 많다.
   -> `DefinitelyTyped`의 경우 커뮤니티에서 관리 되기 때문에 이러한 작업량을 감당할 수 있지만 개별 프로젝트의 경우에는 개별적으로 관리되기 때문에 처리시간이 보장 받지 못한다.
   <br />

공식적인 권장사항은 타입스크립트로 작성된경우 타입 선언을 라이브러리에 포함하는 것이 좋다.  
타입스크립트 컴파일러가 타입선언을 대신 생성해 주기 때문에, 타입스크립트로 작성된 라이브러리에 타입선언을 포함하는 방식은 잘 동작한다.  
반면 자바스크립트의 경우는 타입선언을 손수 작성해주기 때문에 오류가 있을 가능성이 높고 잦은 업데이트가 필요하다. 이런경우 `DefinitelyTyped`에 공개하여 유지보수 하는 것이 좋다.

<br />

> @types 의존성과 관련된 세가지 버전이 있다. 라이브러리, @types, 타입스크립트 버전.
>
> 라이브러리를 업데이트하는경우 해당 @Types도 업데이트 해야한다.
>
> 타입 선언을 라이브러리에 포함하는 것, DefinitelyTyped에 공개하는 것 사이의 장단점을 파악해야한다. 타입스크립트로 작성된 라이브러리라면 타입선언을 자체적으로 포함하고, 자바스크립트로 작성된 라이브러리라면 타입 선언을 DefinitelyTyped에 공개하는것이 좋다.

<br />

## 아이템 47. 공개 API에 등장하는 모든 타입을 익스포트하기

```typescript
interface SecretName {
  first: string;
  last: string;
}
interface SecretSanta {
  name: SecretName;
  gift: string;
}

export function getGift(name: SecretName, gift: SecretSanta): SecretSanta {
  /.../;
}

/**이렇게 해도 외부에선 타입을 추출할수 있다.*/

type MySanta = ReturnType<typeof getGift>;
type MyName = Parameters<typeof getGift>[0];
```

위에서 타입코드를 추출한 것만 봐도 굉장히 불편하다.

공개 메서드에 등장한 어떤 형태의 타입이든 익스포트해야한다. 어차피 라이브러리 단에서 사용자가 추출할 수 있으므로, 익스포트 하기 쉽게 만드는 것이 좋다.

<br />

## 아이템 48. API 주석에 TSDoc 사용하기

- // : 인라인 주석이라고 한다. 이는 별도로 툴팁표시가 되지 않는다.

- /\*_ ... _/ : Doc형태의 주석이다 함수에 정보나 툴팁의 표시해준다. Markdown을 지원한다.

> 익스포트된 함수, 클래스, 타입 주석은 JSDoc/TSDoc 형태를 사용하자
>
> @param, @returns 구문과 문서서식을 위해 마크다운 사용이 가능하다.
>
> 주의 : 주석에 타입 정보를 포함하지 말자.

<br />

## 아이템 49. 콜백에서 this에 대한 타입제공하기

let과 const가 `렉시컬 스코프`인 반면 자바스크립트의 this는 `다이나믹 스코프`로 호출하는 객체에 따라 그 의미가 변경된다. 따라서 this는 계속 변경될 수 있는 것으로 타입 을 미리 설정을 해두지 않는다면 추후에 혼란이 올 수 있다.

<br />

### 클래스에서 this 바인딩하기

예를 들어 아래 `onClick`의 코드는 this를 계속해서 변경시킬수 있는 코드이다. 따라서 바인딩을 해주어야하는데 그 방법을 알아보자

```typescript
class ResetButton {
  render() {
    return makeButton({text: 'Reset', onClick: this.onClick})
  }
  onClick() { //onClick를 호출하는 객체마다 다른 값을 받아오므로 바인딩을 시켜주어야 한다.
    alert(`Reset ${this}`)
  }
}

//constructor에 바인딩 하기
class ResetButton {
  consturctor() {
    this.onClick = this.onClick.bind(this);
  }
  render() {
    return makeButton({text: 'Reset', onClick: this.onClick})
  }
  onClick() {
    alert(`Reset ${this}`)
  }
}


//Arrow function으로 변경하여 바인딩 하기
class ResetButton {
  render() {
    return makeButton({text: 'Reset', onClick: this.onClick})
  }
  onClick = () => {
    alert(`Reset ${this}`)
  }
}

//실재 동작 코드
class ResetButton {
  constructor() {
    var _this = this;
    this.onClick = function () {
      alert("Reset " + _this);
    };
  }
  render( {
    return makeButton({text: 'Reset', onClick: this.onClick});
  })
}
```

<br />

### 함수사용에서 매개변수에 this를 추가하여 this 체크하기

굉장히 간단한 방법으로 this의 타입체크 문제를 해결하고 있다.

```typescript
function addKeyListener(el: HTMLElement, fn: (this: HTMLElement, e: KeyboardEvent) => void) {
  el.addEventListener('keydown', (e) => {
    fn(el, e); //Expected 1 arguments, but got 2.
  });
}

function addKeyListener(el: HTMLElement, fn: (this: HTMLElement, e: KeyboardEvent) => void) {
  el.addEventListener('keydown', fn); // 자동적으로 fn 함수에 this와, keyevent가 인자로 추가된다.
}

declare let el: HTMLElement;
addKeyListener(el, function (e) {
  this.innerHTML; // this is HTMLElement
}); // pass
```

타입스크립트에서 `this`를 매개변수로 추가하는 경우 자동적으로 인자 호출하는 부분에서 자동적으로 만들어지기 때문에 새롭게 추가해 줄 필요가 없다.

> 콜백 함수에서 this를 사용하게된다면 this에 대한 타입정보를 꼭 명시하자

<br />

## 아이템 50. 오버로딩 타입보다는 조건부 타입 사용하기

반환타입이 유연한 경우 오버로딩, 제네릭보다는 조건부 타입설정을 사용하자

```typescript
function double(x) {
  return x + x;
}

/**함수 오버로딩*/
function double(x: number | string): number | string;
function double(x: any) {return x + x};

//Warring 반환타입이 고정으로 number나 string임에도 불구하고 유니온타입으로 도출됨
const num = double(12); // string | number;
const str = double('x'); // string | number;

/**제네릭 사용*/
function double<T extends number | string>(x: T): T;
function double(x: any) {return x + x};

//Warring 반환타입이 너무 자세해지고 틀려짐
const num = double(12); // 12 => 24로 나와야함
const str = double('x'); // x => xx로 나와야함

/** 여러가지 타입 선언 분리 */
function double(x:number): number;
function double(x:string): string;
function double(x: any) {return x + x};

//pass
const num = double(12); // number
const string = double('x') // string

//하지만 유니온 타입으로 호풀하는 경우 타입에러 발생
function f(x: number | string) {
  return double(x);
  // Argument of type 'string | number' is not assignable to parameter of type 'number'.
  // Argument of type 'string | number' is not assignable to parameter of type 'string'.
}

/** 조건부 타입으로 설정하기 */
function double<T extends number | string> {x : T}: T extends string ? string : number;
function double(x: any){ return x * x };

//pass
const num = double(12); // number
const string = double('x') // string
//pass
function f(x: number | string) {
  return double(x);
}
```

> 오버로딩이나 단순 제네릭을 사용하는것보다 조건부 타입을 이용하는 것이 휠씬 간단하고 추가적인 오버로딩 없이 유니온 타입을 지원 가능하다
>
> 조건부 타입 알아두자

<br />

## 아이템 51. 의존성 분리를 위해 미러 타입 사용하기

csv를 파싱하는 함수가 있다고 한다면 다음과 같이 코드를 짤 수 있다

```typescript
function parseCSV(contents: string | Buffer): { [column: string]: string }[] {
  if (typeof contents === 'object') {
    //버퍼인 경우
    return parseCSV(contents.toString('utf8'));
  }
  // ...
}
```

여기서 Buffer의 타입은 NodeJS의 타입으로  
`npm install --save-dev @types/node` 를 이용하여 얻을수 있다.  
하지만 `@types/node`가 필요하지 않는 집단에게는 Buffer 하나 때문에 `@types/node`를 설치하기에는 혼란스러움이 있다.

이런 경우에는 실제 필요한 선언부만 추출하여 라이브러리에 넣는 것을 고려해 볼 수 있다.

```typescript
interface CsvBuffer {
  toString(encoding: string): string; //parseCSV에서는 toString만 사용하고 있음.
}
function parseCSV(contents: string | CsvBuffer): { [column: string]: string }[];
```

> 위와 같이 별도로 선언하여 구조적 타이핑을 사용하도록 한다.
>
> **공개한** 라이브러리를 사용하는 자바스크립트 사용자가 **@type 의존성**을 가지지 않게 해야한다. 웹 개발자가 NodeJS 관련된 의존성을 가지지 않게 해야한다.

<br />

## 아이템 52. 테스팅 타입의 함정에 주의하기

map 함수의 타입선언을 작성한다고 가정한다면

```typescript
declare function map<U, V>(array: U[], fn: (u: U) => V): V[];
```

위처럼 짤수 있고 이를 확인하기 위해서는 테스트 코드르 확인해야한다.

```typescript
map(['2017', '2018', '2019'], (v) => Number(v));
```

위 테스트 코드에서 문제점을 찾는다면 매개변수에 대한 타입은 체킹을 할수 있다. 그렇지만 매개변수에 단일값 또는 배열이 아닌 것이 온다면 매개변수의 오류는 잡을수 있지만 **반환 값에 대한 검증**은 따로 하지 않기 때문에 완전한 테스트가 아니다.  
다음과 같은 코드라고 생각하면된다.

```typescript
test('square a number', () => {
  square(1);
  square(2);
});
// expect와 같이 검증 로직을 사용하지 않았기 때문에 square함수의 테스트는 통과하게된다.
```

그렇다면 반환값을 특정 타입의 변수에 할당하여 간단히 반환타입을 체크해보자

```typescript
const lengths: number[] = map(['jone', 'paul'], (name) => name.length);
```

위 코드에서 `number[]` 는 일반적으로 `불필요한 타입선언`에 해당된다. 그러나 `테스트 코드 관점` 으로는 매우 중요한 역할을 하고 있다. 그러나 테스팅을 위해 할당을 사용하는 것은 2가지 문제가 있다.

첫 번째로 불필요한 변수를 만들어야 한다.

-> 변수를 도입하는 대신 헬퍼 함수를 정의하자

```typescript
function assertType<T>(x: T) {}
assertType<number[]>(map(['jone', 'paul'], (name) => name.length)); //pass
assertType<number[]>(map(['jone', 'paul'], (name) => name)); //error : Argument of type 'string[]' is not assignable to parameter of type 'number[]'.
```

불필요한 변수를 해결할 수 있지만 다른 문제점이 있다.

두 번째로는 타입이 동일한지 체크를 하는 것이아닌 할당 가능을 체크하는 것이다.

-> 예를들어 n의 경우는 12를 검증해야하는데 number를 검증하여 할당 가능한지 보고 통과시켜버린다.

```typescript
const n = 12; // type: 12
assertType < number > n; // pass
```

또 객체의 타입을 체크하는 경우를 살펴보면 문제를 확인 할 수 있다.

```typescript
const beatles = ['john', 'paul', 'georage', 'ringo'];
assertType<{ name: string }[]>(map(beatles, (name) => ({ name, inYellowSubmarine: name === 'ringo' }))); // pass
```

바로 보이겠지만 name이 있다면 할당가능하여 통과되고 `inYellowSubmarine` 은 체킹 하지 않는다. 또한 함수의 경우도 동일하다.

```typescript
const add = (a: number, b: number) => a + b;
assertType<(a: number, b: number) => number>(add);

const double = (x: number) => 2 * x;
assertType<(a: number, b: number) => number>(double); //pass
```

`double`에도 `a` 라는 값 하나만 있어도 동작이 되는 함수이기 때문에 통과되는것을 볼 수 있다.

<br />

### 그렇다면 올바른 타입 체킹하는 방법은 무엇일까?

`Parameters`와 `ReturnType`을 제너릭 타입을 이용해 분리하여 테스트 할 수 있다.

```typescript
const double = (x: number) => 2 * x;
let p: Parameters<typeof double> = null;

assertType < [number, number] > p;
// Argument of type '[x: number]' is not assignable to parameter of type '[number, number]'.

let r: ReturnType<typeof double> = null;

const double = (x: number) => 2 * x;
let p: Parameters<typeof double> = null;

assertType < [number, number] > p;
// Argument of type '[x: number]' is not assignable to parameter of type '[number, number]'.

let r: ReturnType<typeof double> = null;
assertType < number > r; // pass
```

> `this`를 사용 하는 콜백함수에도 다른 문제를 발견할 수 있다. this 또한 타입 선언으로 모델링 할수 있으므로 타입 선언에 반영해야하며 테스트도 해야한다.

<br />

### 함세 세부사항 테스트 하기

```typescript
assertType<number[]>(map(
	beatles,
  function(name, i, array) {
    // Parameter 'name, i, array' implicitly has an 'any' type.
    assertType<string>(name);
    assertType<number>(i);
    assertType<string[]>(array);
    assertType<string[]>(this);
    //'this' implicitly has type 'any' because it does not have a type annotation.
    // ...
  }
))

//함수 타입 선언을 변경하여 해결
declare function map<U, V>(
  array: U[],
  fn: (this: U[], u: U, i: number, array: U[])) => V
): V[];

```

<br />

### dtslint 사용하기

DefinitelyTyped 선언을 위한도구로 주석틀 통해 동작한다 dtslint를 사용하면 위 예제 테스트를 다음처럼 작성이 가능하다

```typescript
const beatles = ['john', 'paul', 'georage', 'ringo'];
map(
  beatles,
  function (
    name, // $ExpectType string
    i, // $ExpectType number
    array // $ExpectType string[]
  ) {
    this; // $ExpectType string[]
    return name.length;
  }
); // $ExpectType number[]
```

dtslint는 할당 가능성 대신 각 심벌의 타입을 추출하여 글자가 같은지 비교한다. 이 비교과정은 편집기에서 타입 선언을 눈으로 보고 확인하는 것과 같은데 dtslint는 이를 자동화한다. 하지만 `number|string` 과 `string|number`를 다르게 인식함으로 주의해야한다.

> 타입을 테스트 할 때는 특히 함수 타입의 동일성과 할당 가능성의 차이점을 알고 있어야한다.
>
> 콜백이 있는 함수를 테스트할 때, 콜백 매개변수의 추론된 타입을 체크해야한다. 또한 this가 API의 일부분이라면 테스트해야한다.
>
> 타입 관련된 테스트에서 any를 주의해야한다. 엄격한 테스트를 위해 dtslint같은 도구를 사용하자.
