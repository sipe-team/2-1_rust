# (늦은) Week 3 Submission

![week3_result.png](./images/week3_result.png)

## strings

- 소유권 규칙을 설명하기에 적합한 타입임(그렇다고 쉬운건 아님)
- `&str` 타입은 스택에 할당됨
  - 문자열 슬라이스라고도 함
  - 컴파일 타임에 이미 알고 있음
  - 그래서 불변성을 띔
- `String` 타입은 heap에 할당됨

  - 런타임 시점에 메모리 할당자에 의해 크기가 결정됨
  - 사용이 끝났을 때 메모리를 해제할 방법이 필요(러스트에서는 스코프를 벗어나면 자동으로 해제됨)
  - UTF-8로 인코딩된 문자열 타입

- `String`은 바이트의 collection으로 구현되있음
  - Vec<u8>과 유사함
  - push_str, push 등의 메소드를 사용할 수 있음
- 문자열 조합에는 `+` 연산자를 사용할 수 있음

```rust
let s1 = String::from("Hello, ");
let s2 = String::from("world!");
let s3 = s1 + &s2;
```

> `+` 연산자는 `add` 메소드를 호출함
> `add` 메소드는 `self`를 소유하게 됨
> `s1`은 더이상 사용할 수 없음
> `&s2`를 사용하는 이유는 `add` 메소드가 `&str`을 받기 때문
> `add` 메소드는 `+` 연산자를 사용할 때마다 호출되기 때문에 비효율적임

- 여러 문자열을 조합하고 싶을 때는 `format!` 매크로를 사용하는 것이 좋음

```rust
let s1 = String::from("tic");
let s2 = String::from("tac");
let s3 = String::from("toe");

let s = format!("{}-{}-{}", s1, s2, s3);
```

- 문자열 내부 인덱싱은 지원하지 않음
  - 인덱스 연산이 언제나 O(1) 시간복잡도를 가지도록 하기 위함
  - 숫자 하나를 사용하는 것보다 범위(슬라이스)를 사용하는 것이 좋음
  - 개별적인 유니코드 스칼라 값에 접근하려면 `chars` 메소드를 사용해야 함

```rust
let s = String::from("hello");
let h = &s[0..1]; // "h"


for c in "नमस्ते".chars() {
    println!("{}", c);
}
```

## modules

- 모듈은 라이브러리를 구성하는 단위
- 모듈은 파일 시스템의 디렉토리 구조와 일치함
- 키워드
  - `mod` : 모듈을 정의함
  - `use` : 모듈을 가져옴
  - `pub` : 모듈을 외부로 공개함
  - `super` : 상위 모듈을 참조함

```rust
mod sound { // 모듈 정의
    pub mod instrument { // 공개 + 모듈
        pub fn clarinet() { // 공개
            // 함수 내용
        }
    }
}

use crate::sound::instrument; // 모듈 가져오기
use self::sound::instrument; // 모듈 가져오기
use super::sound::instrument; // 상위 모듈 가져오기
```

## hashmaps

- `HashMap<K, V>` 타입은 키와 값의 쌍을 저장함
- 키와 값은 모두 제네릭 타입이어야 함
- 벡터처럼 인덱스가 아니라 임의의 키를 이용해서 데이터(값)을 가져오는데 유용함

```rust
use std::collections::HashMap;

let mut values: HashMap<String, i32> = HashMap::new();

values.insert(String::from("blue"), 10);
values.insert(String::from("yellow"), 20);

let blue = values.get("blue");
```

- `HashMap`은 데이터를 소유하고 있지 않음

  - 데이터를 참조하는 방식으로 사용함
  - 데이터를 소유하고 싶다면 `HashMap<K, V>` 대신 `HashMap<K, Box<V>>`를 사용해야 함
  - `Box`는 heap에 데이터를 저장하도록 해줌
  - `HashMap`은 데이터를 소유하지 않기 때문에 데이터를 변경하려면 `mut` 키워드를 사용해야 함

- `get`메소드를 사용해 데이터를 가져올 수 있다.
- `entry` 메소드를 사용해 데이터를 업데이트할 수 있다.

```rust
let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);

let team_name = String::from("Blue");
let score = scores.get(&team_name); // 10

for (key, value) in &scores {
    println!("{}: {}", key, value);
}

scores.entry(String::from("Blue")).or_insert(50); // 값이 없을 때만 추가
```

## options

- `Option`는 `Some`과 `None` 두 가지 값을 가지고 `null`을 대체함
- `Option`는 `match` 표현식을 사용해 값을 추출할 수 있음

```rust
let some_number = Some(5);
let some_string = Some("a string");

let absent_number: Option<i32> = None;

let x: i8 = 5;
let y: Option<i8> = Some(5);

let sum = x + y; // error
```

- `Option`은 `is_some`, `is_none`, `unwrap`, `expect` 등의 메소드를 제공함
- `unwrap`은 `Some`의 값을 반환하고 `None`일 때 패닉을 발생시킴
- `expect`는 `unwrap`과 동일하지만 패닉 메시지를 지정할 수 있음

```rust
let x: Option<i8> = Some(5);
let y: Option<i8> = None;

x.unwrap(); // 5
y.unwrap(); // panic
y.expect("Failed to unwrap"); // panic with message
```

- `ref` 키워드를 사용해 `Option`의 값을 해당 case scope에서 참조할 수 있음

```rust
let x: Option<i8> = Some(5);

match x {
    Some(ref value) => println!("Got a value: {}", value),
    None => println!("No value"),
}
```

## 후기

- String 타입과 &str 타입의 차이점을 이해하는데 시간이 걸렸다.
  - 추가로 &str을 String으로 변환하는 여러 방법도 알게 되었다. (`to_string`, `to_owned`, `into`)
- 그 외 모듈, 해시맵, 옵션 등의 개념은 다른 언어에서도 비슷한 개념이라 쉽게 이해할 수 있었다.
