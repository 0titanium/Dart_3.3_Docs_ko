# 비동기 프로그래밍: future, async, await Asynchronous programming: futures, async, await

이 튜토리얼에서는 future와 async 및 await 키워드를 사용하여 비동기 코드를 작성하는 방법을 설명합니다. 내장된 DartPad 편집기를 사용하면 예제 코드를 실행하고 연습을 완료하여 지식을 테스트할 수 있습니다.

이 튜토리얼을 최대한 활용하려면 다음이 필요합니다.

- 기본 Dart 문법에 대한 지식

- 다른 언어로 비동기 코드를 작성해본 경험

이 튜토리얼에서는 다음 자료를 다룹니다.

- 어떻게 언제 async 및 await 키워드를 사용해야하는지

- async 및 await 사용이 실행 순서에 미치는 영향

- 비동기 함수에서 try-catch 표현식을 사용하여 비동기 호출의 오류를 처리하는 방법

이 튜토리얼을 완료하는 데 예상되는 시간: 40~60분.

```
노트

이 페이지는 내장된 DartPad를 사용하여 예제와 연습을 표시합니다. DartPad 대신 빈 상자가 표시되면 DartPad 문제 해결 페이지로 이동하세요.
```

이 튜토리얼의 연습에는 부분적으로 완성된 코드 스니펫이 있습니다. DartPad를 사용하여 코드를 완성하고 실행 버튼을 클릭하여 지식을 테스트할 수 있습니다. 메인 함수 아래의 테스트 코드를 편집하지 마세요.

도움이 필요하면 각 연습 후에 힌트 또는 솔루션 드롭다운을 확장하세요.

## 비동기 코드가 중요한 이유 Why asynchronous code matters

비동기 작업을 통해 프로그램은 다른 작업이 완료되기를 기다리는 동안 작업을 완료할 수 있습니다. 다음은 몇 가지 일반적인 비동기 작업입니다.

- 네트워크를 통해 데이터를 가져오기.

- 데이터베이스에 쓰기.

- 파일에서 데이터를 읽기.

이러한 비동기 연산은 일반적으로 결과를 Future로 제공하거나 결과에 여러 부분이 있는 경우 Stream으로 제공합니다. 이러한 연산은 프로그램에 비동기성을 도입합니다. 초기 비동기성을 적용하려면 다른 일반 Dart 함수도 비동기화되어야 합니다.

이러한 비동기 결과와 상호 작용하려면 async 및 aWait 키워드를 사용할 수 있습니다. 대부분의 비동기 함수는 본질적으로 비동기 연산에 의존하는 비동기 Dart 함수입니다.

## 예: 올바르지 않게 비동기 함수를 사용

다음 예에서는 비동기 함수(fetchUserOrder())를 사용하는 잘못된 방법을 보여줍니다. 나중에 async 및 await를 사용하여 예제를 수정합니다. 이 예제를 실행하기 전에 문제를 찾아보십시오. 결과가 어떻게 나올 것이라고 생각하시나요?

```dart
// 이 예는 비동기 Dart 코드를 작성하지 *않는* 방법을 보여줍니다.

String createOrderMessage() {
  var order = fetchUserOrder();
  return 'Your order is: $order';
}

Future<String> fetchUserOrder() =>
    // 이 함수가 더 복잡하고 느리다고 상상해 보세요.
    Future.delayed(
      const Duration(seconds: 2),
      () => 'Large Latte',
    );

void main() {
  print(createOrderMessage());
}
```

예제에서 fetchUserOrder()가 결국 생성하는 값을 인쇄하지 못하는 이유는 다음과 같습니다.

- fetchUserOrder()는 지연 후 사용자의 주문을 설명하는 문자열인 "Large Latte"를 제공하는 비동기 함수입니다.

- 사용자의 주문을 얻으려면 createOrderMessage()가 fetchUserOrder()를 호출하고 완료될 때까지 기다려야 합니다. createOrderMessage()는 fetchUserOrder()가 완료될 때까지 기다리지 않기 때문에 createOrderMessage()는 fetchUserOrder()가 결국 제공하는 문자열 값을 가져오는 데 실패합니다.

- 대신, createOrderMessage()는 완료해야 할 보류 중인 작업, 즉 완료되지 않은 미래에 대한 표현을 가져옵니다. 다음 섹션에서 future에 대해 자세히 알아봅니다.

- createOrderMessage()가 사용자 주문을 설명하는 값을 가져오지 못하기 때문에 이 예제에서는 "Large Latte"를 콘솔에 인쇄하지 못하고 대신 "Your order is: Instance of '_Future<String>'"을 출력합니다.

다음 섹션에서는 fetchUserOrder()가 원하는 값("Large Latte")을 콘솔에 출력하도록 만드는 데 필요한 코드를 작성할 수 있도록 future와 future 작업(async 및 await 사용)에 대해 알아봅니다.


```
주요 용어

- 동기 작업: 동기 작업은 완료될 때까지 다른 작업의 실행을 차단합니다.
- 동기 함수: 동기 함수은 동기 작업만 수행합니다.
- 비동기 작업: 일단 시작되면 비동기 작업을 통해 완료되기 전에 다른 작업을 실행할 수 있습니다.
- 비동기 함수: 비동기 함수는 하나 이상의 비동기 작업을 수행하며 동기 작업도 수행할 수 있습니다.
```

## future란? What is a future?

future(소문자 "f")는 Future(대문자 "F") 클래스의 인스턴스입니다. future는 비동기 작업의 결과를 나타내며 완료되지 않음 또는 완료라는 두 가지 상태를 가질 수 있습니다.

```
노트

미완료(Uncompleted)은 future가 값을 생산하기 전의 상태를 가리키는 Dart 용어입니다.
```

## 미완료 Uncompleted

비동기 함수를 호출하면 완료되지 않은 future가 반환됩니다. 해당 future는 함수의 비동기 작업이 완료되거나 오류가 발생하기를 기다리고 있습니다.

## 완료 Completed

비동기 작업이 성공하면 future는 값으로 완료됩니다. 그렇지 않으면 오류와 함께 완료됩니다.

## 값으로 완료 Completing with a value

Future<T> 타입의 future는 T 타입의 값으로 완료됩니다. 예를 들어 Future<String> 타입의 future는 문자열 값을 생성합니다. future가 사용 가능한 값을 생성하지 않는 경우 future의 타입은 Future<void>입니다.


## 오류로 완료 Completing with an error

어떤 이유로든 함수에 의해 수행된 비동기 작업이 실패하면 future는 오류와 함께 완료됩니다.

## 예: future 소개

다음 예제에서 fetchUserOrder()는 콘솔에 출력한 후 완료되는 future를 반환합니다. 사용 가능한 값을 반환하지 않기 때문에 fetchUserOrder()는 Future<void> 타입을 갖습니다. 예제를 실행하기 전에 "Large Latte" 또는 "Fetching user order..." 중 어느 것이 먼저 출력될지 예측해 보세요.

```dart
Future<void> fetchUserOrder() {
  // 이 함수가 다른 서비스나 데이터베이스에서 사용자 정보를 가져오고 있다고 상상해 보세요.
  return Future.delayed(const Duration(seconds: 2), () => print('Large Latte'));
}

void main() {
  fetchUserOrder();
  print('Fetching user order...');
}
```

앞의 예에서 fetchUserOrder()는 8행의 print() 호출 이전에 실행되었지만, 콘솔에는 fetchUserOrder()("Large Latte")의 출력 이전에 8행("Fetching user order...")의 출력이 표시됩니다. 이는 fetchUserOrder()가 "Large Latte"를 인쇄하기 전에 지연되기 때문입니다.

## 예: 오류로 완료

다음 예제를 실행하여 어떻게 future가 오류와 함께 완료되는지 확인하세요. 잠시 후에 오류를 처리하는 방법을 배우게 됩니다.

```dart
Future<void> fetchUserOrder() {
  // 이 함수가 사용자 정보를 가져오다가 버그가 발생했다고 상상해 보세요.
  return Future.delayed(
    const Duration(seconds: 2),
    () => throw Exception('Logout failed: user ID is invalid'),
  );
}

void main() {
  fetchUserOrder();
  print('Fetching user order...');
}
```

이 예에서 fetchUserOrder()는 사용자 ID가 유효하지 않다는 오류와 함께 완료됩니다.

future와 future가 어떻게 완료되는지 배웠습니다. 그런데 비동기 함수의 결과를 어떻게 사용할까요? 다음 섹션에서는 async 및 wait 키워드를 사용하여 결과를 얻는 방법을 알아봅니다.

```
빠른 리뷰

- Future<T> 인스턴스는 T 타입의 값을 생성합니다.

- future가 사용 가능한 값을 생성하지 않으면 future의 타입은 Future<void>입니다.

- future는 미완료 또는 완료라는 두 가지 상태 중 하나일 수 있습니다.

- future를 반환하는 함수를 호출하면 함수는 수행할 작업을 대기열에 추가하고 완료되지 않은 future를 반환합니다.

- future의 작업이 완료되면 future는 값이나 오류와 함께 완료됩니다.


주요 용어

- Future: Dart Future 클래스.

- future: Dart Future 클래스의 인스턴스입니다.
```

## future 작업: 비동기 및 대기 Working with futures: async and await

async 및 await 키워드는 비동기 함수를 정의하고 해당 결과를 사용하는 선언적 방법을 제공합니다. async 및 await를 사용할 때 다음 두 가지 기본 지침을 기억하십시오.

- 비동기 함수를 정의하려면 함수 본문 앞에 async를 추가하세요.

- await 키워드는 비동기 함수에서만 작동합니다.

다음은 main()을 동기 함수에서 비동기 함수로 변환하는 예입니다.

먼저 함수 본문 앞에 async 키워드를 추가합니다.

```dart
void main() async { ··· }
```

함수에 선언된 반환 타입이 있는 경우 타입을 Future<T>로 업데이트합니다. 여기서 T는 함수가 반환하는 값의 타입입니다. 함수가 명시적으로 값을 반환하지 않으면 반환 타입은 Future<void>입니다.

```dart
Future<void> main() async { ··· }
```

이제 비동기 함수가 있으므로 await 키워드를 사용하여 미래가 완료될 때까지 기다릴 수 있습니다.

```dart
print(await createOrderMessage());
```

다음 두 예제에서 볼 수 있듯이 async 및 await 키워드는 동기 코드와 매우 유사한 비동기 코드를 생성합니다.

유일한 차이점은 비동기 예제에서 강조 표시되며, 창이 충분히 넓은 경우 동기 예제의 오른쪽에 있습니다.

## 예: 동기 함수 Example: synchronous functions

```dart
String createOrderMessage() {
  var order = fetchUserOrder();
  return 'Your order is: $order';
}

Future<String> fetchUserOrder() =>
    // 이 함수가 더 복잡하고 느리다고 상상해보세요.
    Future.delayed(
      const Duration(seconds: 2),
      () => 'Large Latte',
    );

void main() {
  print('Fetching user order...');
  print(createOrderMessage());
}
```

```
Fetching user order...
Your order is: Instance of '_Future<String>'
```

## 예: 비동기 함수 Example: asynchronous functions

```dart
Future<String> createOrderMessage() async {
  var order = await fetchUserOrder();
  return 'Your order is: $order';
}

Future<String> fetchUserOrder() =>
    // 이 함수가 더 복잡하고 느리다고 상상해보세요.
    Future.delayed(
      const Duration(seconds: 2),
      () => 'Large Latte',
    );

Future<void> main() async {
  print('Fetching user order...');
  print(await createOrderMessage());
}
```

```
Fetching user order...
Your order is: Large Latte
```

비동기 예제는 세 가지 측면에서 다릅니다.

- createOrderMessage()의 반환 타입이 String에서 Future<String>으로 변경됩니다.

- async 키워드는 createOrderMessage() 및 main()의 함수 본문 앞에 나타납니다.

- await 키워드는 비동기 함수 fetchUserOrder() 및 createOrderMessage()를 호출하기 전에 나타납니다.

```
주요 용어

- async: 함수 본문 앞에 async 키워드를 사용하여 함수를 비동기식으로 표시할 수 있습니다.

- 비동기 함수: 비동기 함수는 async 키워드가 붙은 함수입니다.

- await: 비동기 표현식의 완성된 결과를 얻기 위해 await 키워드를 사용할 수 있습니다. await 키워드는 비동기 함수 내에서만 작동합니다.
```

## async 와 await를 사용한 실행 흐름 Execution flow with async and await

비동기 함수는 첫 번째 await 키워드가 나올 때까지 동기적으로 실행됩니다. 이는 비동기 함수 본문 내에서 첫 번째 await 키워드 이전의 모든 동기 코드가 즉시 실행된다는 것을 의미합니다.

## 예: async 함수 내 실행 Example: Execution within async functions

다음 예제를 실행하여 async 함수 본문 내에서 실행이 어떻게 진행되는지 확인하세요. 결과는 어떻게 될 것이라고 생각하시나요?

```dart
Future<void> printOrderMessage() async {
  print('Awaiting user order...');
  var order = await fetchUserOrder();
  print('Your order is: $order');
}

Future<String> fetchUserOrder() {
  // 이 함수가 더 복잡하고 느리다고 상상해보세요.
  return Future.delayed(const Duration(seconds: 4), () => 'Large Latte');
}

void main() async {
  countSeconds(4);
  await printOrderMessage();
}

// 이 함수는 무시해도 됩니다. 이 예에서는 지연 시간을 시각화하기 위해 여기에 있습니다.
void countSeconds(int s) {
  for (var i = 1; i <= s; i++) {
    Future.delayed(Duration(seconds: i), () => print(i));
  }
}
```

이전 예제의 코드를 실행한 후 2행과 3행을 반대로 바꿔보세요.

```dart
var order = await fetchUserOrder();
print('Awaiting user order...');
```

이제 printOrderMessage()의 첫 번째 await 키워드 뒤에 print('Awaiting user order')가 나타나므로 출력 타이밍이 변경됩니다.

## 연습: async와 await 사용 연습 Exercise: Practice using async and await

다음 연습은 부분적으로 완료된 코드 스니펫이 포함된 실패한 단위 테스트입니다. 작업할 것은 테스트를 통과하도록 코드를 작성하여 연습을 완료하는 것입니다. main()을 구현할 필요는 없습니다.

비동기 작업을 시뮬레이션하려면 제공된 다음 함수를 호출하세요.

| 함수 | 타입 시그니처 | 설명 |
|---|---|---|
| fetchRole() | Future<String> fetchRole() | 사용자 역할에 대한 간단한 설명을 가져옵니다. |
| fetchLoginAmount() | Future<int> fetchLoginAmount() |	사용자가 로그인한 횟수를 가져옵니다. |

### Part 1: reportUserRole()

다음을 수행하도록 reportUserRole() 함수에 코드를 추가합니다.

- 다음 문자열로 완성되는 future를 반환합니다: "User role: <user role>"
  - 참고: fetchRole()에서 반환된 실제 값을 사용해야 합니다; 예제 반환 값을 복사하여 붙여넣어도 테스트가 통과되지 않습니다.
  - 반환 값 예시: "User role: tester"

- 제공된 함수 fetchRole()를 호출하여 사용자 역할을 가져옵니다.

### Part 2: reportLogins()

다음을 수행하도록 비동기 함수 reportLogins()를 구현합니다:

- "Total number of logins: <# of logins>"라는 문자열을 반환합니다.
  - 참고: fetchLoginAmount()에서 반환된 실제 값을 사용해야 합니다. 예제 반환 값을 복사하여 붙여넣어도 테스트가 통과되지 않습니다.
  - ReportLogins()의 반환 값 예: "Total number of logins: 57"

- 제공된 함수 fetchLoginAmount()를 호출하여 로그인 수를 가져옵니다.

## 오류 처리 Handling errors

비동기 함수의 오류를 처리하려면 try-catch를 사용하세요.

```dart
try {
  print('Awaiting user order...');
  var order = await fetchUserOrder();
} catch (err) {
  print('Caught error: $err');
}
```

비동기 함수 내에서는 동기 코드에서와 동일한 방식으로 try-catch 절을 작성할 수 있습니다.

## 예: try-catch를 사용한 비동기 및 대기 Example: async and await with try-catch

다음 예제를 실행하여 비동기 함수의 오류를 처리하는 방법을 확인하세요. 결과는 어떻게 될 것이라고 생각하시나요?

```dart
Future<void> printOrderMessage() async {
  try {
    print('Awaiting user order...');
    var order = await fetchUserOrder();
    print(order);
  } catch (err) {
    print('Caught error: $err');
  }
}

Future<String> fetchUserOrder() {
  // 이 함수가 더 복잡하다고 상상해보세요.
  var str = Future.delayed(
      const Duration(seconds: 4),
      () => throw 'Cannot locate user order');
  return str;
}

void main() async {
  await printOrderMessage();
}
```

## 연습: 오류 처리 연습 Exercise: Practice handling errors

다음 연습에서는 이전 섹션에서 설명한 접근 방식을 사용하여 비동기 코드로 오류를 처리하는 연습을 제공합니다. 비동기 작업을 시뮬레이션하기 위해 코드는 제공된 다음 함수를 호출합니다.

| 함수 | 타입 시그니처 | 설명 |
|---|---|---|
| fetchNewUsername() | Future<String> fetchNewUsername() | 이전 사용자 이름을 대체하는 데 사용할 수 있는 새 사용자 이름을 반환합니다. |

async 및 wait를 사용하여 다음을 수행하는 비동기changeUsername() 함수를 구현합니다.

- 제공된 비동기 함수 fetchNewUsername()을 호출하고 그 결과를 반환합니다.
  - ChangeUsername()의 반환 값 예: "jane_smith_92"
- 발생하는 모든 오류를 포착하고 오류의 문자열 값을 반환합니다.
  - toString() 메서드를 사용하여 예외와 오류를 모두 문자열화할 수 있습니다.

## 연습: 모든 것을 하나로 합치기 Exercise: Putting it all together

이제 마지막 연습에서 배운 내용을 연습해 볼 시간입니다. 비동기 작업을 시뮬레이션하기 위해 이 연습에서는 비동기 함수 fetchUsername() 및 logoutUser()를 제공합니다.

| 함수 | 타입 시그니처 | 설명 |
|---|---|---|
| fetchUsername() | Future<String> fetchUsername() | 현재 사용자와 연관된 이름을 반환합니다. |
| logoutUser() | Future<String> logoutUser() | 현재 사용자의 로그아웃을 수행하고 로그아웃된 사용자 이름을 반환합니다. |

다음을 작성하세요.

### Part 1: addHello()

- 단일 문자열 인자를 갖는 함수 addHello()를 작성하세요.
- addHello()는 'Hello' 앞에 문자열 인자를 반환합니다.
  예: addHello('Jon')은 'Hello Jon'을 반환합니다.

### Part 2: greetUser()

- 인자가 없는 GreetingUser() 함수를 작성하세요.
- 사용자 이름을 얻기 위해 GreetingUser()는 제공된 비동기 함수 fetchUsername()을 호출합니다.
- GreetingUser()는 addHello()를 호출하고 사용자 이름을 전달하고 결과를 반환하여 사용자에 대한 인사말을 만듭니다.
  예: fetchUsername()이 'Jenny'를 반환하면 GreetingUser()는 'Hello Jenny'를 반환합니다.

### Part 3: sayGoodbye()

- 다음을 수행하는 sayGoodbye() 함수를 작성하세요.
  - 인자를 받지 않습니다.
  - 오류를 포착합니다.
  - 제공된 비동기 함수 logoutUser()를 호출합니다.
- logoutUser()가 실패하면 sayGoodbye()는 원하는 문자열을 반환합니다.
- logoutUser()가 성공하면 sayGoodbye()는 '<result> Thanks, see you next time'라는 문자열을 반환합니다. 여기서 <result>는 logoutUser()를 호출하여 반환된 문자열 값입니다.

### 다음은? What's next?

축하합니다. 튜토리얼을 완료했습니다! 더 자세히 알아보고 싶다면 다음 단계에 대한 몇 가지 제안을 참조하세요.

- DartPad로 플레이하세요.
- 다른 튜토리얼을 시도해 보세요.
- Dart의 future와 비동기 코드에 대해 자세히 알아보세요.
  - Streams 튜토리얼: 일련의 비동기 이벤트로 작업하는 방법을 알아보세요.
  - Dart의 동시성: Dart에서 동시성을 구현하는 방법을 이해하고 알아보세요.
  - 비동기 지원: 비동기 코딩을 위한 Dart의 언어 및 라이브러리 지원을 살펴보세요.
  - Google의 Dart 동영상: 비동기 코딩에 관한 동영상을 하나 이상 시청하세요.
- 다트 SDK를 다운로드하세요!