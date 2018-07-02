# 자료형(Types)

솔리디티(Solidity)는 정적 언어 - 변수의 자료형이 컴파일 시간에 결정되는 - 다. 보다 자세한 사항은 아래의 [자료형 추정](#자료형-추정)(Type Deduction)을 참조하라. 솔리디티는 복합 자료형을 만들 수 있는 몇가지 기본 자료형을 제공한다.

추가로 연산자(Operators)를 포함한 표현식(Expression)으로 자료형간의 상호작용이 가능하다. 다양한 연산자에 대한 정보는 [연산자 우선순위](#연산자-우선순위)(Order of Precedence of Operators)를 참조하라.

## 값 자료형(Value Types)

아래의 자료형은 언제나 함수에 복사된 값으로 넘겨지기 때문에(Passed by Value) 값 자료형이라 불린다.

### 부울형(Boolean)

`bool`: 가능한 값은 상수(Constants) `true`와 `false`다.

연산자:

- `!`: 논리 반전(negation)
- `&&`: 논리 곱(conjuction), "이면서"
- `||`: 논리 합(disjunction), "또는"
- `==`: 같음(equality)
- `!=`: 다름(inequality)

`||`와 `&&`은 일반적인 단순 처리(Short circuit)기법을 사용한다.

### 정수형(Integers)

`int`/`uint`: 다양한 크기의 부호 있는 / 없는 정수. 각각 `int8`에서 `int256`까지, `uint8`에서 `uint256`까지 사용할 수 있다. 기본적으로 `int`와 `uint`는 `int256`, `uint256`과 동등한 표현이다.

연산자:

- 비교(Comparisons): `<=`, `<`, `==`, `!=`, `>=`, `>` (반환값은 부울형이다.)
- 비트(Bit) 연산: `&`, `|`, `^`(배타적 논리합), `~`(비트 반전)
- 산술 연산: `+`, `-`, 단항 `+`, 단항 `-`, `*`, `/`, `%`(나머지), `**`(지수), `<<`(왼쪽 이동), `>>`(오른쪽 이동)

나눗셈에서 소수점은 버려진다(Truncation). 해당 내용은 이더리움 가상기계(EVM)의 연산 부호(opcode) `DIV`에 컴파일되어 있다. 그러나 피연산자가 모두 상수 표현(Literal)인 경우에는 버리지 않는다.

0으로 나누거나 나머지연산을 하면 런타임 예외가 반환된다.

이동 연산의 결과는 왼쪽의 피연산자를 따른다. 표현식 `x << y`는 `x * 2**y`와 같고 양의 정수에서 `x >> y`는 `x / 2**y`와 같다. `x`가 음수일 경우, `x >> y` 는 2의 승수로 나누되 결과값을 내림한다. 이동 연산 오른쪽의 피연산자를 음수로 하면 런타임 예외를 반환한다.

> 경고: v0.5.0 전에는 음수 `x`에 대한 `x >> y`가 `x / 2**y`, 즉 결과값을 올림했다.

### 고정 소수점 숫자형(Fixed Point Numbers)

> 경고: 고정 소수점은 아직 솔리디티에서 완벽하게 지원되지 않는다. 선언은 할 수 있지만, 값을 받거나 값으로 넣을 수 없다.

`fixed`/`ufixed`: 다양한 크기의 부호 있는 / 없는 고정 소수점 수. `fixedMxN`과 `ufixedMxN`로 사용하며 `M`은 자료가 사용할 비트 수를, `N`은 소수 몇번째 자리까지 사용할 지를 정한다. `M`은 반드시 8의 배수여야 하며 최소 8에서 최대 256 비트까지 사용할 수 있다. `N`은 0부터 80까지 (0 및 80 포함) 사용할 수 있다. 기본적으로 `fixed`와 `ufixed`는 `fixed128x18`, `ufixed128x18`과 동등한 표현이다.

연산자:

- 비교: `<=`, `<`, `==`, `!=`, `>=`, `>`
- 산술 연산: `+`, `-`, 단항 `+`, 단항 `-`, `*`, `/`, `%`, `**`

> 참고: 부동 소수점(`float`, `double`로 다른 언어에서 쓰이는, 정확하게는 IEEE-754)과 고정 소수점의 주요한 차이는 정수부와 소수부에 사용되는 비트다. 부동 소수점에서 각각에 할당된 비트는 유동적이고, 고정 소수점에서는 명확하게 정의한다. 일반적으로 부동 소수점에서는 대부분의 비트가 숫자를 표현하는데 사용되고 일부분이 소수점의 위치를 정의하는데 사용된다.

### 주소형(Address)

`address`: 이더리움(Ethereum) 주소 크기 20바이트의 값을 담는다. 주소형은 멤버를 가지고 있고, 모든 계약(Contracts)의 기저로 사용된다.

연산자:

- 비교: `<=`, `<`, `==`, `!=`, `>=`, `>`

> 참고: v0.5.0부터 계약은 주소형으로부터 파생되지 않는다. 물론 여전히 주소형으로 명시적 변환이 가능하다.

### 주소형의 멤버(Members of Addresses)

- `balance`와 `transfer`

빠른 참조를 원한다면 [주소 관련](#주소-관련)을 보라.

`balance`를 통해 주소의 잔고를 열람하고 `transfer`함수를 통해 웨이(Wei)의 단위로 이더(Ether)를 주소로 보내는 것이 가능하다.

```JS
address x = 0x123;
address myAddress = this;
if (x.balance < 10 && myAddress.balance >=10) x.transfer(10);
```

> 참고: 만약 `x`가 계약 주소라면, 그 코드 - 폴백(Fallback) 함수, 만약 있다면 - 는 `transfer` 호출과 함께 실행된다. 이것은 이더리움 가상기계의 기능이며 변경할 수 없다. 만약 실행이 가스(Gas) 부족 혹은 다른 이유로 실패한다면 이더 송금은 반려되고 현재 계약은 예외 반환과 함께 멈춘다.

- `send`

`send`는 `transfer`의 하위호환이다. `send`의 실행이 실패했을 때 계약은 **멈추지 않는다**. 단지 `send`가 `false`값을 반환할 뿐이다.

> 경고: `send`를 쓰는데는 많은 위험이 따른다. 만약 스택 깊이(stack depth)가 1024 - 언제든 호출자에 의해 강제될 수 있다 - 를 넘거나 수신자가 가스가 없다면 송금은 실패한다. 따라서 안전한 이더 송금을 위해 언제나 `send`의 반환값을 확인해야 한다. `transfer`를 사용하라. 아니면 더 나은 방법도 있다 - 수신자가 돈을 출금하는 방식으로 코드를 작성하라.

- `call`, `callcode`, 그리고 `delegatecall`

추가적으로 응용 프로그램 이진 인터페이스(Application Binary Interface, ABI)에 종속되지 않은 계약의 인터페이스나, 보다 직접적인 인코딩 전반의 제어를 위해 1바이트 배열을 입력으로 받는 `call`함수가 제공된다. `abi.encode`, `abi.encodePacked`, `abi.encodeWithSelector`, 그리고 `abi.encodeWithSignature` 같은 함수를 구조화된 데이터를 인코딩하는데 사용할 수 있다.

> 경고: 이 모든 함수는 하위 계층(low-level) 함수로 각별한 주의가 필요하다. 좀 더 구체적으로 설명하자면, 알려지지 않은 모든 계약은 위험할 수 있다. 계약을 호출한다면 그 계약에 제어권을 넘기는 것이고, 그 계약이 당신의 계약을 호출해 상태 변수를 바꿀 수 있다. 다른 계약과 상호작용하는 가장 일반적인 방법은 계약 객체의 함수를 호출하는 것이다(`x.f()`).  
> 참고: 솔리디티의 이전 버전들은 이 함수들이 임의의 매개변수나 `bytes4`형의 첫번째 매개변수를 다르게 관리하도록 허용했다. 이런 극단적인 경우들은 v0.5.0에선 제거되었다.

`call`은 호출된 함수가 종료되었으면 `true`를, 이더리움 가상기계 예외를 반환했으면 `false`를 반환한다. 솔리디티만으로 반환된 실제 데이터에 접근하는 것은 불가능하다. 그러나 인라인 어셈블리를 사용한다면 처리하지 않은(Raw) `call`을 만들 수 있고 `returndatacopy` 명령을 통해 실제 데이터에 접근할 수 있다.

조건자 `.gas()`를 통해 공급된 가스를 조정할 수 있다.

```JS
namReg.call.gas(1000000)(abi.encodeWithSignature("register(string)", "MyName"));
```

비슷한 방식으로 공급된 이더도 조정할 수 있다.

```JS
nameReg.call.value(1 ether)(abi.encodeWithSignature("register(string)", "MyName"));
```

이 조건자들을 합칠 수도 있다. 순서는 상관없다.

```JS
nameReg.call.gas(1000000).value(1 ether)(abi.encodeWithSignature("register(string)", "MyName"));
```

> 참고: 오버로드된 함수에 가스나 값(Value) 조건자를 사용하는 것은 아직 불가능하다.  차선책은 가스와 값에 예외를 도입해 오버로드 해소 순간에 해당 사항이 있는지 다시 확인하는 것이다.

비슷한 방식으로 `delegatecall`함수를 사용할 수 있다. 이 함수는 다른 요소는 현재 계약에서 받아오고, 주어진 주소에서 코드만 사용한다. `delegatecall`의 목적은 다른 계약에 저장된 라이브러리 코드를 사용하는 것이다. 물론 사용자는 양쪽 계약의 저장소(Storage) 형태가 위임호출을 사용하는 데 적절한 지 확인해야한다. 홈스테드 버전 이전에는 `callcode`라는 한정된 방식만 가능했다. `callcode`는 원본의 `msg.sender` 및 `msg.value`값에 접근이 불가능하다.

`call`, `delegatecall`, 그리고 `callcode` 모두 매우 하위 계층의 함수로, 도저히 대책이 없을 때만 사용해야 한다. 이 세 함수는 솔리디티의 자료형 안정성(Type Safety)을 해친다.

세 메서드 전부에서 `.gas` 옵션을 사용할 수 있다. 반면 `.value()`의 경우 `delegatecall`에서 사용할 수 없다.

> 참고: 모든 계약은 `address`형으로 변환될 수 있다. 따라서, 현재 계약의 잔고를 `address(this).balance`를 사용해 열람할 수 있다.  
> 참고: `callcode` 함수를 사용하는 것은 권장되지 않으며, 이 함수는 추후 삭제될 것이다.

### 정적 바이트 배열(Fixed-size byte arrays)

`bytes1`, `bytes2`, `bytes3`, ..., `bytes32`. `byte`는 `byte1`과 동등하다.

연산자:

- 비교: `<=`, `<`, `==`, `!=`, `>=`, `>`
- 비트 연산: `&`, `|`, `^`, `~`, `<<`(왼쪽 이동), `>>`(오른쪽 이동)
- 색인 접근(Index Access): 만약 `x`가 `bytesI`형이면, `0 <= k < I`인 `x[k]`가 `k`번째 바이트를 반환한다.(읽기만 가능)

이동 연산자는 어떤 정수형도 오른쪽의 피연산자로 받을 수 있다(물론 반환값은 왼쪽의 피연산자다). 오른쪽 피연산자에 음수를 넣으면 정수형과 마찬가지로 런타임 예외를 반환한다.

멤버:

- `.length`: 바이트 배열의 고정 길이를 반환한다.(읽기만 가능)

> 참고: 바이트 배열을 `byte[]`로 사용할 수도 있다. 그러나 공간의 낭비인데, 왜냐하면 호출을 넘길 때 모든 원소마다 31 바이트를 사용하기 때문이다. 따라서 `bytes`를 사용하는게 낫다.

### 동적 크기 바이트 배열(Dynamically-sized byte array)

`bytes`:

동적으로 만들어지는 바이트 배열이다(값 자료형 아님!). [배열](#배열(Arrays))을 보라.

`string`:

동적으로 만들어지는 유니코드(UTF-8) 문자열이다(값 자료형 아님!). [배열](#배열(Arrays))을 보라.

### 주소 상수 표현(Address Literals)

주소 검사합 확인(Checksum Test)을 통과한 16진 상수 표현 - 예를 들면, `0xdCad3a6d3569DF655070DEd06cb7A1b2Ccd1D3AF`은 `address`형이다. 39 ~ 41자리 사이면서 검사합 확인을 통과하지 못한 16진 상수 표현은 경고를 반환하고 일반 실수 상수 표현으로 처리된다.

> 참고: 혼합형 주소 검사합 형식은 [EIP-55](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-55.md)에 정의되어 있다.

### 실수 및 정수 상수 표현(Rational and Integer Literals)

정수 상수 표현은 0 ~ 9 사이 숫자의 수열(Sequence)로 만들어진다. 수열은 10진수로 해석된다. 예를 들어, `69`는 예순 아홉 내지는 육십 구다. 8진 상수 표현은 솔리디티에 존재하지 않고, 표현 앞의 0은 처리되지 않는다.

10진 소수 표현은 `.`으로 만들어지고, 최소 한 쪽에는 숫자가 있어야 한다. 예를 들어, `1.`, `.1`, 그리고 `1.3`과 같은 식이다.

과학적 표기법(Scientific Notation)도 지원한다. 밑에는 소수를 쓸 수 있으나, 지수에는 사용할 수 없다. 예를 들어, `2e10`, `-2e10`, `2e-10`, `2.5e1`과 같은 식이다.

수의 상수 표현은 비상수 표현으로 변환되기 전까지 임의의 정밀도를 유지한다. 다른 말로 하자면 수의 상수 표현에서는 계산 오버플로우나 나눗셈에서의 버림이 일어나지 않는다는 뜻이다.

예를 들어, `(2**800 + 1) - 2**800`의 결과값은 `uint8`형 상수 `1`이다. 계산 과정의 중간값이 기계의 단위 크기(Word Size)에 맞지 않음에 유의하라. 더 나아가, `.5 * 8`의 결과값은 비정수 표현을 사용했음에도 정수 `4`다.

피연산자가 정수인 한, 정수에 적용할 수 있는 모든 연산자는 수의 상수 표현에도 적용할 수 있다. 그러나 만약 둘 중 하나가 소수라면 비트 연산은 허용되지 않고, 결과값이 비실수(역: 허수)로 나올 수 있기 때문에 소수 지수 또한 허용되지 않는다.

> 참고: 솔리디티는 각 실수에 대해 수의 상수 표현을 갖고 있다. 정수 상수 표현과 실수 상수 표현은 모두 수의 상수 표현에 속한다. 또, 모든 수의 상수 표현은 수의 상수 자료형에 속한다. 따라서 `1+2`와 `2+1`은 모두 같은 수의 상수 자료형 실수 셋 내지는 삼에 속한다.  
> 경고: 초기의 버전에서는 정수 나눗셈이 소수부를 버렸다. 하지만 이제는 실수로 바꾼다. `5 / 2`는 `2`가 아닌 `2.5`다.  
> 참고: 수의 상수 표현은 비상수 표현과 만나자마자 비상수 표현으로 바뀐다. 우리는 다음 예시의 `b`가 정수인 것을 알지만, 표현식의 일부 `2.5+a`의 자료형이 맞지 않기 때문에 코드가 컴파일 되지 않는다.

```JS
uint128 a = 1;
uint128 b = 2.5 + a + 0.5;
```

### 문자열 상수 표현(String Literals)

문자열 상수 표현은 쌍따옴표나 따옴표로 쓰인다(`"Skkrypto"` 혹은 `'제성'`과 같은 식). 원래 C에서는 문자열 상수 표현 뒤에 0이 붙었지만(역: `'\0'`을 의미) 솔리디티에선 그렇지 않다. `"Skkrypto"`는 9바이트가 아닌 8바이트다. 정수 상수 표현과 만날때 자료형이 바뀔 수도 있지만, 보통 암시적으로 `bytes1`, ..., `bytes32`, 그리고 만약 맞는다면 `bytes`와 `string`으로 변환될 수 있다.

문자열 상수 표현은 `\n`, `\xNN`, 그리고 `\uNNNN`과 같은 이스케이프 문자를 지원한다. `\xNN`은 16진 값을 받아 적당한 바이트에 삽입하고, `\uNNNN`은 유니코드 코드포인트를 받아 UTF-8 문자열(Sequence)을 삽입한다.

### 16진 상수 표현(Hexadecimal Literals)

16진 상수 표현은 예약어 `hex`로 설정되고, 따옴표나 쌍따옴표로 감싸져 있다(`hex"001122FF"`). 따옴표 안은 반드시 16진수 문자열이어야 하며, 상수 표현의 값은 해당 문자열의 이진 표현값이다.

16진 상수 표현은 문자열 상수 표현과 같은 방식으로 작동하며, 똑같은 변환 제약을 갖고 있다.

### 열거형(Enums)

열거형은 솔리디티에서 사용자 정의 자료형을 만드는 방법 중 하나다. 열거형은 모든 정수형으로 명시적 변환이 가능하고 또 정수형에서 열거형으로 변환도 가능하지만, 암시적 변환은 허용되지 않는다. 명시적 형변환은 실행시간에 값의 범위를 확인하고, 만일 실패할 시 예외를 반환한다. 열거형은 최소 하나의 멤버를 필요로 한다.

```JS
pragma solidity ^0.4.16;

contract test {
    enum ActionChoices { GoLeft, GoRight, GoStraight, SitStill }
    ActionChoices choice;
    ActionChoices constant defaultChoice = ActionChoices.GoStraight;

    function setGoStraight() public {
        choice = ActionChoices.GoStraight;
    }

    // 열거형은 응용 프로그램 이진 인터페이스의 일부가 아니기 때문에,
    // "getChoice"의 명시적 자료형(Signature)은 솔리디티 외부의 모든 일에 대해
    // 자동으로 "getChoice() returns (uint8)"로 바뀐다.
    // 정수형은 모든 열거 값을 포함할 수 있는 범위로 정해진다.
    // 만약 값이 더 많다면 `uint16`이 사용되는 식이다.
    function getChoice() public view returns (ActionChoices) {
        return choice;
    }

    function getDefaultChoice() public pure returns (uint) {
        return uint(defaultChoice);
    }
}
```

### 함수형(Function Types)

함수형은 함수의 자료형이다. 함수형 변수는 함수에서 할당되고, 함수 호출에서 함수를 반환하거나, 혹은 매개변수로 함수를 넘기기 위해 함수형 매개변수가 사용된다. 함수형은 크게 두가지 종류로 나뉜다 - 내부(Internal) 함수와 외부(External) 함수다.

내부 함수는 현재 계약 안에서만, 더 정확히는 현재의 코드 단위 - 내부 라이브러리 함수와 상속된 함수를 포함하는 - 에서만 호출될 수 있는데, 이는 내부 함수가 현재 계약 밖의 맥락에서 실행되면 안되기 때문이다. 내부 함수의 호출은 현재 계약에서 내부적으로 함수를 호출할 때와 똑같이 진입 지점(Entry Label)으로 이동해 실현된다.

외부 함수는 주소와 명시적 자료형으로 구성되어 있고, 외부 함수 호출에 매개변수나 반환값으로 쓰일 수 있다.

함수형은 다음과 같이 표기한다:

```JS
function (<parameter types>) {internal|external} [pure|constant|view|payable] [returns (<return types>)]
```

매개변수 자료형과 다르게 반환 자료형은 빼놓을 수 없다. 만약 함수가 아무 값도 반환하지 않는다면, `[returns (<return types>)]` 전체를 넣지 말아야 한다.

기본적으로 함수형은 내부형이기 때문에 `internal` 예약어는 생략할 수 있다. 그러나, 계약의 함수들은 기본적으로 공용 - `public` - 이다. 즉, 자료형의 이름으로 쓰일 때만 내부형을 기본값으로 갖는다.

현재 계약의 함수에 접근하는 방법은 두가지가 있다. 바로 함수 이름(예: `f`) 그대로 호출하거나, `this.f`로 호출하는 방식이다. 전자는 내부 함수로 작동하고 후자는 외부 함수로 작동한다.

만약 함수형 변수가 초기화되지 않았다면, 해당 변수를 호출할 경우 예외가 반환된다. 만약 어떤 함수에 `delete`를 사용 후 다시 호출한다면, 마찬가지로 예외가 반환된다.

외부 함수형이 솔리디티 바깥의 맥락에서 사용된다면, `function` 형으로 처리되며 함수 식별자가 제공하는 주소를 `bytes24`형으로 인코딩한다.

현재 계약의 공용함수가 내부와 외부 모두에 쓰일 수 있음을 주목하라. 내부 함수로 쓰고 싶으면 `f`를, 외부 함수로 쓰고 싶으면 `this.f`를 사용하면 된다.

추가적으로 공용(혹은 외부) 함수는 이진 인터페이스 함수 선택기(ABI Function Selector)를 반환하는 `selector`라는 특별한 멤버를 갖는다.

```JS
pragma solidity ^0.4.16;

contract Selector {
  function f() public view returns (bytes4) {
    return this.f.selector;
  }
}
```

내부 함수형을 어떻게 사용하는 지 보여주는 예시다.

```JS
pragma solidity ^0.4.16;

library ArrayUtils {
  //같은 코드 맥락에 있기 때문에,
  //내부 함수를 내부 라이브러리 함수에서 사용할 수 있다.
  function map(uint[] memory self, function (uint) pure returns (uint) f)
    internal
    pure
    returns (uint[] memory r)
  {
    r = new uint[](self.length);
    for (uint i = 0; i < self.length; i++) {
      r[i] = f(self[i]);
    }
  }
  function reduce(
    uint[] memory self,
    function (uint, uint) pure returns (uint) f
  )
    internal
    pure
    returns (uint r)
  {
    r = self[0];
    for (uint i = 1; i < self.length; i++) {
      r = f(r, self[i]);
    }
  }
  function range(uint length) internal pure returns (uint[] memory r) {
    r = new uint[](length);
    for (uint i = 0; i < r.length; i++) {
      r[i] = i;
    }
  }
}

contract Pyramid {
  using ArrayUtils for *;
  function pyramid(uint l) public pure returns (uint) {
    return ArrayUtils.range(l).map(square).reduce(sum);
  }
  function square(uint x) internal pure returns (uint) {
    return x * x;
  }
  function sum(uint x, uint y) internal pure returns (uint) {
    return x + y;
  }
}
```

외부 함수형은 이렇게 사용한다.

```JS
pragma solidity ^0.4.22;

contract Oracle {
  struct Request {
    bytes data;
    function(bytes memory) external callback;
  }
  Request[] requests;
  event NewRequest(uint);
  function query(bytes data, function(bytes memory) external callback) public {
    requests.push(Request(data, callback));
    emit NewRequest(requests.length - 1);
  }
  function reply(uint requestID, bytes response) public {
    // 믿을 수 있는 출처에서 온 답변인지 확인
    requests[requestID].callback(response);
  }
}

contract OracleUser {
  Oracle constant oracle = Oracle(0x1234567); // 이미 알고 있는 계약
  function buySomething() {
    oracle.query("USD", this.oracleResponse);
  }
  function oracleResponse(bytes response) public {
    require(
        msg.sender == address(oracle),
        "Only oracle can call this."
    );
    // 데이터를 사용
  }
}
```

> 참고: 람다식과 인라인 함수는 계획되었지만 아직 지원되지 않는다.

## 참조형(Reference Types)

크기가 256비트를 넘길 수도 있는 복합 자료형(Complex types)은 우리가 지금까지 본 값 자료형보다 더 세심히 다루어야 한다. 복합 자료형을 복사하는 것은 매우 비싸기 때문에, 그 값을 휘발성 메모리에 저장할지 아니면 상태 변수를 담는 저장소에 저장할지 신중히 생각해야 한다.

### 데이터 위치(Data Location)

모든 복합 자료형 - 배열이나 구조체 같은 - 은 메모리에 담을지 저장소에 담을지 정하는 추가 표기가 있다. 맥락에 따라 늘 기본값이 있지만, `storage`나 `memory`를 자료형에 추가함으로써 덮어쓸 수 있다. 함수의 매개변수와 반환값의 기본값은 `memory`, 지역변수의 기본값은 `storage`다. 당연한 이야기지만, 상태 변수는 무조건 `storage`에 담긴다.

세번째 위치도 있는데, 바로 `calldata`다. `calldata`는 휘발성의 수정할 수 없는 영역으로, 함수 인자가 저장된다. 외부 함수의 매개변수(반환값은 제외)는 강제로 `calldata`에 저장되고, `memory`와 비슷하게 작동한다.

데이터 위치는 매우 중요한데, 왜냐하면 이 위치가 할당이 어떻게 실행되는지를 결정하기 때문이다. 저장소, 메모리, 그리고 상태 변수(심지어 다른 상태 변수에서 할당해도)간 할당이 일어날 때마다 언제나 독립된 사본이 생긴다. 지역 저장소 변수에 할당을 하면 무조건 참조를 할당하고, 이 참조는 상태 변수가 중간에 바뀌어도 계속 상태 변수를 가리킨다. 반면 메모리에 저장된 참조형에서 다른 메모리에 저장된 참조형에 할당하는 것은 사본을 만들지 않는다.

```JS
pragma solidity ^0.4.0;

contract C {
    uint[] x; // x의 데이터 위치는 저장소

    // memoryArray의 데이터 위치는 메모리
    function f(uint[] memoryArray) public {
        x = memoryArray; // 배열 전체를 저장소로 복사
        var y = x; // 포인터를 할당. y의 데이터 위치는 저장소
        y[7]; // 8번째 값을 반환
        y.length = 2; // y를 통해 x값을 바꿈
        delete x; // 배열을 지움. 추가적으로 y도 바뀜
        // 다음은 작동하지 않음. 임시 배열을 저장소 안에 만들어야하는데, 저장소는 정적으로 할당되어있음
        // y = memoryArray;
        // 또 이것도 작동하지 않음. 포인터를 리셋하지만, 포인터가 가리키는 위치가 없음.
        // delete y;
        g(x); // g를 호출함. x의 참조를 넘김
        h(x); // h를 호출하고 메모리에 독립된 임시 사본을 만듬
    }

    function g(uint[] storage storageArray) internal {}
    function h(uint[] memoryArray) public {}
}
```

#### 요약

**강제 데이터 위치**:

- 반환값을 제외한 외부 함수의 매개변수: `calldata`
- 상태 변수: `storage`

**기본 데이터 위치**:

- 함수의 매개변수(반환값 포함): `memory`
- 다른 모든 지역변수: `storage`

### 배열(Arrays)

배열은 컴파일 시간에 정적으로 설정할 수도 있고 동적으로 설정할 수도 있다. 저장소 배열은 임의의 자료형 - 다른 배열이나, 매핑, 구조체도 - 을 원소로 가질 수 있다. 그러나 메모리 배열은 매핑을 원소로 가질 수 없으며 만약 모두가 볼 수 있는 함수라면 원소는 응용 프로그램 이진 인터페이스형이어야만 한다.

원소 자료형 `T`와 정적 `k`를 가진 배열은 `T[k]`로 쓴다. 동적 배열은 `T[]`다. 예를 들어, 5개의 `uint` 동적 배열의 배열은 `uint[][5]`다(다른 언어와 비교했을 때 표기가 반대임에 유의하라). 세번째 동적 배열의 두번째 uint에 접근하기 위해서는 `x[2][1]`을 사용한다. 색인값은 0에서 시작하고, 선언과 반대 방향으로 접근한다. 예를 들어 `x[2]`는 자료형의 오른쪽에서 한단계 들어간 것이다.

`bytes`형과 `string`형 변수는 특별한 배열이다. `bytes`는 `byte[]`와 비슷하지만, `calldata`에 압축적으로 저장된다. `string`은 `bytes`와 같지만 길이나 색인 접근을 허용하지 않는다(지금은). `bytes`는 `byte[]`보다 저렴하기 때문에 더 자주 쓰인다. 경험적으로 임의의 바이트 데이터엔 `bytes`를, 임의의 문자열 데이터엔 `string`을 사용하는게 좋다. 만약 일정 길이로 바이트를 한정 지을 수 있다면, 무조건 `bytes1` 에서 `bytes32` 사이의 값을 쓰는게 더 저렴하다.

> 참고: 만약 바이트 단위로 문자열 `s`에 접근하고 싶다면 `bytes(s).length`/`bytes(s)[7] = 'x';`를 사용하자. 각각의 문자가 아닌 UTF-8 표현의 하위 계층 바이트에 접근하고 있음을 유의하라!

배열을 `public`으로 표기하고 솔리디티가 접근자(Getter)를 만들게 하는 것도 가능하다. 접근자의 매개변수로 숫자 색인이 필요하다.

#### 메모리 배열 할당하기(Allocating Memory Arrays)

`new` 예약어를 사용해 메모리에 변수를 길이로 받는 배열을 만들 수 있다. 저장소 배열과는 다르게, `.length` 멤버를 통해 배열의 크기를 재설정 하는 것이 **불가능하다**.

```JS
pragma solidity ^0.4.16;

contract C {
    function f(uint len) public pure {
        uint[] memory a = new uint[](7);
        bytes memory b = new bytes(len);
        // a.length == 7 이고 b.length == len 이다
        a[6] = 8;
    }
}
```

#### 배열 상수 표현 / 인라인 배열(Array Literals / Inline Arrays)

배열 상수 표현은 바로 변수에 할당되지 않고 표현식으로 쓰인 배열이다.

```JS
pragma solidity ^0.4.16;

contract C {
    function f() public pure {
        g([uint(1), 2, 3]);
    }
    function g(uint[3] _data) public pure {
        // ...
    }
}
```

배열 상수 표현의 자료형은 정적 메모리 배열이다. 그 기저 자료형은 주어진 원소들의 공통된 자료형이다. `[1, 2, 3]`의 자료형은 `uint8[3] memory`인데, 이는 상수 각각의 자료형이 `uint8`이기 때문이다. 이 때문에 위 예시의 첫번째 원소를 `uint`로 변환해야만 했다. 현재 정적 메모리 배열은 동적 크기 메모리 배열에 할당될 수 없음에 유의하라. 즉, 다음은 불가능하다.

```JS
// 이것은 컴파일되지 않는다

pragma solidity ^0.4.0;

contract C {
    function f() public {
        // 다음 줄은 자료형 에러를 반환한다
        // 왜냐하면 uint[3] memory는 uint[] memory로 변환될 수 없기 때문이다
        uint[] memory x = [uint(1), 3, 4];
    }
}
```

추후 이 제약을 없애는 것으로 계획되어 있지만, 현재는 응용 프로그램 이진 인터페이스에서 배열이 어떻게 전달되는가와 관해 약간의 문제를 갖고 있다.

#### 멤버(Members)

**길이(length)**:

배열은 원소의 개수를 저장하는 `length`멤버를 갖고 있다. 저장소의 동적 배열은 `.length`멤버를 수정하는 것으로 크기를 재설정할 수 있다. 현재 길이 바깥의 원소에 접근한다고 해서 자동으로 재설정되지는 않는다. 메모리 배열의 크기 - 물론 동적 배열에서는 런타임 매개변수에 따라 바뀔 수 있다 - 는 생성과 함께 고정된다.

**밀어넣기(Push)**:

저장소의 동적 배열과 `bytes`(`string`은 아님)는 배열 끝에 원소를 밀어넣을 수 있는 멤버 함수 `push`를 갖고 있다. 함수는 새 길이를 반환한다.

**내보내기(Pop)**:

저장소의 동적 배열과 `bytes`(`string`은 아님)는 배열 끝의 원소를 내보낼 수 있는 멤버 함수 `pop`을 갖고 있다.

> 경고: 아직 외부 함수에서 배열의 배열(역: 2차원 배열)을 사용할 수는 없다.  
> 경고: 이더리움 가상기계의 한계상, 외부 함수 호출에서 동적 내용물을 반환할 수는 없다. `contract C { function f() returns (uint[]) { ... } }`의 함수 `f`는 웹3(web3.js)에서 호출될 경우 무언가를 반환하겠지만, 솔리디티에서 호출될 경우 아무것도 반환하지 않는다.  현재 가능한 방법은 매우 큰 정적 배열을 사용하는 것 뿐이다.

```JS
pragma solidity ^0.4.16;

contract ArrayContract {
    uint[2**20] m_aLotOfIntegers;
    // 다음 배열이 한 쌍의 동적 배열이 아니라 쌍들의 동적 배열임에 유의하라
    // 다르게 말하자면, 길이 2의 정적 배열 여러개다.
    bool[2][] m_pairsOfFlags;
    // newPairs 는 메모리에 저장된다 - 함수 인자의 기본값이다

    function setAllFlagPairs(bool[2][] newPairs) public {
        // 저장소 배열로의 할당이 전체 배열을 대체한다
        m_pairsOfFlags = newPairs;
    }

    function setFlagPair(uint index, bool flagA, bool flagB) public {
        // 존재하지 않는 색인에 대한 접근은 예외를 반환한다
        m_pairsOfFlags[index][0] = flagA;
        m_pairsOfFlags[index][1] = flagB;
    }

    function changeFlagArraySize(uint newSize) public {
        // 만약 새 크기가 더 작다면, 제거된 배열 원소는 메모리에서 정리된다
        m_pairsOfFlags.length = newSize;
    }

    function clear() public {
        // 다음은 배열을 완전히 정리한다
        delete m_pairsOfFlags;
        delete m_aLotOfIntegers;
        // 같은 효과
        m_pairsOfFlags.length = 0;
    }

    bytes m_byteData;

    function byteArrays(bytes data) public {
        // 바이트 배열("bytes")은 채우기(Padding)없이 저장되기 때문에 다르다
        // 그러나 "uint8[]"과 동일하게 취급할 수 있다
        m_byteData = data;
        m_byteData.length += 7;
        m_byteData[3] = byte(8);
        delete m_byteData[2];
    }

    function addFlag(bool[2] flag) public returns (uint) {
        return m_pairsOfFlags.push(flag);
    }

    function createMemoryArray(uint size) public pure returns (bytes) {
        // 동적 메모리 배열은 `new`를 사용해 만들어진다
        uint[2][] memory arrayOfPairs = new uint[2][](size);
        // 동적 바이트 배열 생성
        bytes memory b = new bytes(200);
        for (uint i = 0; i < b.length; i++)
            b[i] = byte(uint8(i));
        return b;
    }
}
```

### 구조체(Structs)

솔리디티는 다음과 같이 새로운 자료형을 구조체 형태로 정의하는 방식을 제공한다.

```JS
pragma solidity ^0.4.11;

contract CrowdFunding {
    // 두가지 속성(Field)을 가진 새 자료형을 정의
    struct Funder {
        address addr;
        uint amount;
    }

    struct Campaign {
        address beneficiary;
        uint fundingGoal;
        uint numFunders;
        uint amount;
        mapping (uint => Funder) funders;
    }

    uint numCampaigns;
    mapping (uint => Campaign) campaigns;

    function newCampaign(address beneficiary, uint goal) public returns (uint campaignID) {
        campaignID = numCampaigns++; // campaignID는 반환값 변수
        // 새 구조체를 만들어 저장소에 저장. 매핑 자료형은 제외했음
        campaigns[campaignID] = Campaign(beneficiary, goal, 0, 0);
    }

    function contribute(uint campaignID) public payable {
        Campaign storage c = campaigns[campaignID];
        // 주어진 값으로 초기화되는 새 임시 메모리 구조체를 생성
        // 구조체를 저장소로 복사
        // Funder(msg.sender, msg.value)로 초기화 할 수 있음에 유의하라
        c.funders[c.numFunders++] = Funder({addr: msg.sender, amount: msg.value});
        c.amount += msg.value;
    }

    function checkGoalReached(uint campaignID) public returns (bool reached) {
        Campaign storage c = campaigns[campaignID];
        if (c.amount < c.fundingGoal)
            return false;
        uint amount = c.amount;
        c.amount = 0;
        c.beneficiary.transfer(amount);
        return true;
    }
}
```

계약은 크라우드펀딩 계약의 모든 기능을 제공하지는 않지만, 구조체를 이해하기 위해 필요한 기본 개념은 포함하고 있다. 구조체는 매핑과 배열 안에서 사용될 수 있고 또 그 자체가 매핑이나 배열을 속성으로 가질 수 있다.

구조체는 매핑 멤버의 값 자료형으로 사용할 수 있지만, 그 자체 자료형의 멤버로 사용할 수는 없다. 이 제약은 구조체의 크기를 유한하게 만들기 위해 반드시 필요하다.

모든 함수에서 구조체가 지역변수(기본 저장소 데이터 위치)로 할당된 점을 유의하라. 이 방식은 구조체를 복사하지 않고 참조만 저장해 지역 변수의 멤버에 대한 할당으로 상태를 변경할 수 있게 만든다.

물론 당연히 지역 변수에 할당하지 않고도 `campaigns[campaignID].amount = 0` 처럼 구조체의 멤버에 접근할 수 있다.

## 매핑(Mappings)

매핑은 `mapping(_KeyType => _ValueType)`과 같이 선언된다. Here `_KeyType`은 매핑, 동적 배열, 계약, 열거형 그리고 구조체를 제외한 거의 모든 자료형이 될 수 있다. `_ValueType` 은 매핑을 포함한 모든 자료형이 될 수 있다.

매핑은 모든 가능한 키가 존재하고, 자료형의 기본값에 매핑된 상태로 초기화된 일종의 해시 테이블이라 볼 수 있다. 다만 키값은 매핑에 저장되지 않고 `keccak256` 해시값(값을 찾기 위해)만이 저장된다.

이 때문에 매핑은 길이나 설정된 키 혹은 값의 개념이 없다.

매핑은 상태 변수에만 쓸 수 있다(혹은 내부 함수의 저장소 참조형).

매핑을 `public`으로 표기하고 솔리디티가 접근자를 만들게 하는 것도 가능하다. 접근자의 매개변수로 `_KeyType`이 필요하고, 접근자의 반환값은 `_ValueType`이다.

`ValueType` 또한 매핑일 수 있다. 접근자는 재귀적으로 각각의 `_KeyType`에 대해 하나의 매개변수를 갖는다.

```JS
pragma solidity ^0.4.0;

contract MappingExample {
    mapping(address => uint) public balances;

    function update(uint newBalance) public {
        balances[msg.sender] = newBalance;
    }
}

contract MappingUser {
    function f() public returns (uint) {
        MappingExample m = new MappingExample();
        m.update(100);
        return m.balances(this);
    }
}
```

> 참고: 매핑은 반복이 불가능하지만, 위에 자료구조를 구현할 수는 있다. 예제가 필요하다면 반복 가능한 매핑을 참고하라.

## L값을 포함한 연산자(Operators Involving LValues)

만약 `a`가 L값(할당의 대상이 될 수 있는 변수나 다른 무언가)이라면 다음 연산자를 사용할 수 있다.

`a += e`는 `a = a + e`와 같다. `-=`, `*=`, `/=`, `%=`, `|=`, `&=` 그리고 `^=`도 마찬가지로 정의된다. `a++`와 `a--`는 `a += 1` / `a -= 1`과 같지만 표현식 자체는 여전히 `a`의 이전 값을 갖는다. 반면 `--a`와 `++a`는 같은 효과를 갖지만 변경 이후의 값을 반환한다.

### 삭제(Delete)

`delete a`는 `a`에 자료형의 초기값을 할당한다. 예를 들어 정수형에서 `delete a`는 `a=0`과 같다. 배열에서도 사용할 수 있는데, 길이 0의 동적 배열을 할당하거나 같은 길이의 정적 배열의 모든 값을 리셋할 수 있다. 구조체에선 모든 멤버가 리셋된 구조체를 할당한다.

`delete`는 매핑에는 효과가 없다(매핑의 키는 임의값이고 일반적으로 알 수 없기 때문에). 따라서 만약 구조체를 삭제한다면 매핑이 아닌 모든 멤버를 초기화 하고 매핑이 아닌 한 각 멤버에 재귀적으로 진입할 것이다. 그러나, 각각의 키와 그 키들이 매핑하고 있던 자료는 삭제할 수 있다.

`delete a`가 정말 `a`에 대한 할당과 비슷하게 작동한다는 점을 주의할 필요가 있다. `a`에 새로운 객체를 저장하는 것이다.

```JS
pragma solidity ^0.4.0;

contract DeleteExample {
    uint data;
    uint[] dataArray;

    function f() public {
        uint x = data;
        delete x; // x를 0으로 설정한다. data엔 영향이 없다
        delete data; // data를 0으로 설정한다. 사본을 쥐고 있는 x에는 영향이 없다
        uint[] storage y = dataArray;
        delete dataArray; // dataArray.length를 0으로 설정하지만, uint[]이 복합 객체기 때문에 저장소 객체와 동등한 y도 영향을 받는다.
        // 다른 한편, "delete y"는 저장소 객체를 참조하는 지역 변수에 대한 할당이
        // 존재하는 저장소 객체에서만 가능하기 때문에 유효하지 않다.
    }
}
```

## 기본형 간의 형변환(Conversions between Elementary Types)

### 암시적 형변환(Implicit Conversions)

만약 다른 자료형끼리 연산을 할 경우, 컴파일러는 자동적으로 피연산자 하나를 다른 피연산자의 자료형으로 바꾸고자 한다(할당에서도 똑같다). 일반적으로, 의미적으로 옳고 아무 정보도 잃어버리지 않는다면 값 자료형간의 암시적 형변환이 가능하다. 예를 들어, `uint8`은 `uint16`으로, `int128`은 `int256`으로 바꾸는 것이 가능하다. 그러나 `int8`은 `uint256`으로 바꿀 수 없다(`uint256`이 `-1`을 담을 수 없기 때문에). 더 나아가 부호 없는 정수형은 같거나 큰 바이트로 변환될 수 있지만, 역은 성립하지 않는다. `uint160`으로 변환될 수 있는 모든 자료형은 `address`로도 바꿀 수 있다.

### 명시적 형변환(Explicit Conversions)

만약 컴파일러가 암시적 형변환을 허용하지 않지만, 내가 무얼 하는지 정확히 알고 있다면 명시적 형변환도 가능하다. 예상치 못한 결과를 낳을 수 있으니 반드시 원하는 결과가 나오는지 확인해보길 바란다. 음수 `int8`을 `uint`로 바꾸는 다음 예시를 확인해보라.

```JS
int8 y = -3;
uint x = uint(y);
```

x는 `0xfffff..fd`(64개의 16진 문자)의 값을 갖는데, 이건 256비트에서 -3을 2의 보수로 표현한 것이다.

만약 자료형을 작은 자료형으로 명시적 형변환한다면, 높은 자리의 비트가 잘려나간다.

```JS
uint32 a = 0x12345678;
uint16 b = uint16(a); // b will be 0x5678 now
```

v0.5.0부터 정수와 정적 바이트 배열간의 명시적 형변환은 양쪽이 같은 크기일 때만 허용된다. 서로 다른 크기의 정수와 정적 바이트 배열간 변환을 원한다면, 먼저 맞는 크기로 명시적 형변환이 필요하다. 다음은 명시적 정렬(Alignment) 및 채우기다.

```JS
uint16 x = 0xffff;
bytes32(uint256(x)); // 왼쪽 채움
bytes32(bytes2(x)); // 오른쪽 채움
```

## 자료형 추정

편의를 위해, 언제나 변수의 자료형을 명시적으로 적을 필요는 없다. 컴파일러는 자동적으로 변수에 할당된 첫번째 표현식에서 자료형을 추론한다.

```JS
uint24 x = 0x123;
var y = x;
```

여기서 `y`의 자료형은 `uint24`다. `var`를 함수의 매개변수나 반환값으로 사용하는 것은 불가능하다.

> 경고: 자료형은 첫번째 할당에서만 추론되기 때문에, 다음 코드의 반복(Loop)은 무한하다. `i`는 `uint8`값을 갖고 이 자료형의 최대값은 `2000`보다 작기 때문이다.  `for (var i = 0; i < 2000; i++) { ... }`

## 옮긴이의 말

본 문서는 [ReadTheDocs - types](https://solidity.readthedocs.io/en/develop/types.html)를 번역한 것입니다.  
최대한 우리말로 쓰고자 노력했으나 여러 한계상 불가피하게 단어를 혼용하게 되었습니다.  
오역이나 중의적 표현, 이해가 안되는 문장, 혹은 기타 번역에 대한 제안은 resetarsi@gmail.com으로 부탁 드립니다.  
읽어주셔서 감사합니다.