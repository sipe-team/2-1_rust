<!-- TOC start (generated with https://github.com/derlin/bitdowntoc) -->

- [Week 3 Submission](#week-3-submission)
- [TIL](#til)
  * [1. 문자열(string)](#1-string)
    + [1.1. 문자열 생성](#11-)
    + [1.2. 문자열 업데이트](#12-)
    + [1.3. 문자열 인덱싱](#13-)
    + [1.4. 문자열 슬라이싱](#14-)
    + [1.5. 문자열 반복](#15-)
  * [2. modules](#2-modules)
    + [2.1. crate](#21-crate)
    + [2.2. package](#22-package)
    + [2.3. module 관련 규칙](#23-module-)
    + [2.4. 경로](#24-)
    + [2.5. pub 키워드로 경로 노출](#25-pub-)
    + [2.6. super 키워드](#26-super-)
    + [2.7. 구조체와 열거형의 공개](#27-)
    + [2.8. use 키워드를 통해 경로를 스코프 안으로 가져오기](#28-use-)
    + [2.9. as 키워드](#29-as-)
    + [2.10. pub use로 다시 내보내기](#210-pub-use-)
    + [2.11. 중첩 경로](#211-)
  * [3. Hashmap](#3-hashmap)
    + [3.1. 해시맵 선언](#31-)
    + [3.2. 해시맵 값 접근](#32-)
    + [3.3. 해시맵 반복](#33-)
    + [3.4. 해시맵과 소유권](#34-)
    + [3.5. 해시맵 업데이트](#35-)
    + [3.6. 기존 값에 기초하여 업데이트](#36-)
  * [4. Option](#4-option)
    + [4.1. while let](#41-while-let)

<!-- TOC end -->

<!-- TOC --><a name="week-3-submission"></a>
# Week 3 Submission

![result.png](./images/result.png)
- week3 다 풀고 스크린샷을 못찍어서 week4 스샷을 대신 첨부합니다 🤣

<!-- TOC --><a name="til"></a>
# TIL

<!-- TOC --><a name="1-string"></a>
## 1. 문자열(string)
- 러스트에서 **문자열**(string)은 **바이트의 컬렉션**이며, **UTF-8**로 인코딩되어있다.
- 러스트 코어에서 사용되는 문자열은 **문자열 슬라이스 `str`** (&str로 사용) 만 존재하며, `String` 타입은 표준 라이브러리에 존재한다.

<!-- TOC --><a name="11-"></a>
### 1.1. 문자열 생성
- 벡터에서 쓸 수 있는 연산 대부분을 문자열에서 동일하게 사용 가능하다.
  - `String` 타입은 **바이트 벡터**를 Wrapping한 형태이기 때문이다.
- 벡터 생성자를 통해 String 생성
  - `let mut s = String::new();`
- 문자열 리터럴로부터 String 생성
  - `let s = "initial contents".to_string();`
  - `let hello = String::from("안녕하세요");`

<!-- TOC --><a name="12-"></a>
### 1.2. 문자열 업데이트
```rust
let mut s1 = String::from("foo");
let s2 = "bar";
s1.push_str(s2);
println!("s2 is {s2}");

//...

let mut s = String::from("lo");
s.push('l');
```
- `push_str` 및 `push` 메서드는 소유권을 가져오지 않는다.
```rust
let s1 = String::from("Hello, ");
let s2 = String::from("world!");
let s3 = s1 + &s2; // s1은 여기로 이동되어 더 이상 사용할 수 없음을 주의하세요
```
- 다음과 같이 + 연산자를 통해 접합 또한 가능하다.
  - `s2`가 참조자인 이유는, + 연산자의 행위를 구현하는 `add` 메서드의 파라미터가 s2이기 때문이다.
    - `fn add(self, s: &str) -> String {`
  - 그러나 예제의 `s2`는 String 타입인데.. `&String != &str` 아닌가?
    - deref coercion(역참조 강제)라는 기능을 통해 `&s2`를 `&s2[..]`, 즉 문자열 슬라이싱으로 변경한다.
      - 나중에 더 자세히..
  - `self`의 소유권을 가져오고 있으므로, `s1`은 더이상 유효하지 않을 것이다. 주의.
```rust
let s1 = String::from("tic");
let s2 = String::from("tac");
let s3 = String::from("toe");

let s = format!("{s1}-{s2}-{s3}");
```
- `format!` 매크로를 활용하면 + 연산자에 대한 복잡성을 줄일 수 있다.
  - 소유권에 관여하지 않기 때문이다.

<!-- TOC --><a name="13-"></a>
### 1.3. 문자열 인덱싱
```rust
let s1 = String::from("hello");
let h = s1[0];

//...
error[E0277]: the type `String` cannot be indexed by `{integer}`
```
- rust는 문자열 인덱싱을 지원하지 않는다.
  - 인덱싱은 O(1)의 시간복잡도를 고려해줘야 하는데에 반해, 문자열은 이를 보장할 수 없기 때문이다.
- UTF-8 기반 인코딩은 한 글자가 1~4 byte로 가변적이다.
  - 반면 문자열은 byte의 벡터이다.
  - 즉, 문자열의 n 번째 문자를 가져오려 할 때, 해당 문자는 벡터의 n 번째 요소가 아니다.
  - 이를 찾아내기 위해선 탐색의 과정이 필요하므로, O(1)의 시간복잡도를 보장할 수 없다.

<!-- TOC --><a name="14-"></a>
### 1.4. 문자열 슬라이싱
- `let s = &hello[0..4];`
  - 다음과 같이 문자열의 일정 바이트 범위를 지정할 수 있다.
  - 만약 온전하지 않은 문자 바이트의 일부를 얻으려 한다면 에러가 발생하므로, 주의하자.

<!-- TOC --><a name="15-"></a>
### 1.5. 문자열 반복
```rust
for c in "Зд".chars() {
    println!("{c}");
}

for b in "Зд".bytes() {
    println!("{b}");
}
```
- 다음과 같이 문자 단위 반복, byte 단위 반복이 가능하다.

<!-- TOC --><a name="2-modules"></a>
## 2. modules
- rust는 코드 조직화를 위해 **모듈 시스템**을 활용한다.
- 모듈 시스템은 아래와 같이 구성된다.
  - **크레이트**: **라이브러리**나 **실행 가능한 모듈**로 구성된 **트리** 구조
  - **패키지**: **크레이트**를 **빌드**하고, **테스트**하고, **공유**
  - **모듈과 use**: **구조, 스코프**를 **제어**하고, **조직 세부 경로**를 감추는 데 사용
  - **경로**: 구조체, 함수, 모듈 등의 **이름**을 지정

<!-- TOC --><a name="21-crate"></a>
### 2.1. crate
- `crate`는 rust가 컴파일 한 차례에 고려하는 가장 작은 코드 단위이다.
- 크레이트는 아래와 같은 두 종류로 나뉜다.
  - 바이너리 크레이트 (binary crate)
    - 커맨드 라인 프로그램이나 서버처럼 실행 가능한 실행파일로 컴파일할 수 있는 프로그램
    - main 함수를 포함해야 한다.
  - 라이브러리 크레이트 (library crate)
    - main 함수를 가지고 있지 않고 실행파일 형태로 컴파일되지 않는다.
    - 여러 프로젝트에서 공용으로 사용될 기능이 포함된다.
    - **크레이트**라고 말하면 대부분 **라이브러리 크레이트**를 의미하는 것으로 생각하면 된다.
- `crate root`는 러스트가 컴파일을 시작하는 소스 파일이다.

<!-- TOC --><a name="22-package"></a>
### 2.2. package
- 하나 이상의 크레이트로 구성된 번들이다.
  - 크레이트들을 빌드하는 방법이 정의된 `Cargo.toml` 파일이 포함된다.
- 바이너리 크레이트는 여러개 넣을 수 있지만, 라이브러리 크레이트는 단 하나만 포함시킬 수 있다.
- 적어도 하나 이상의 크레이트가 포함되어야 한다.
```bash
$ cargo new my-project
     Created binary (application) `my-project` package
$ ls my-project
Cargo.toml
src
$ ls my-project/src
main.rs
```
- `src/main.rs`가 크레이트 루트가 되는 것은 관례이다.
- `src/lib.rs`가 패키지 내에 존재할 경우, 패키지 명과 똑같은 이름의 라이브러리가 있는 것으로 간주한다.
- 카고는 빌드 시, 이 크레이트 루트를 `rustc`에 전달한다.

<!-- TOC --><a name="23-module-"></a>
### 2.3. module 관련 규칙
- 컴파일 시 컴파일러는 가장 먼저 **크레이트 루트**를 확인
- 크레이트 루트엔, `mod garden;`과 같이 **새로운 모듈**을 선언 가능
  - 선언 시, 컴파일러는 아래의 순서대로 **해당 모듈이 존재**하는지 확인
    - `mod garden` 뒤에 세미콜론 대신 **중괄호**를 써서 안쪽에 코드를 적은 인라인
    - `src/garden.rs` 파일 안
    - `src/garden/mod.rs` 파일 안
- 크레이트 루트가 아닌 다른 파일에서 선언되는 모듈은 **서브모듈**로 칭함
  - `src/garden.rs` 안에 `mod vegetables;` 키워드가 있다면..
    - `mod vegetables` 뒤에 세미콜론 대신 중괄호를 써서 안쪽에 코드를 적은 인라인
    - `src/garden/vegetables.rs` 파일 안
    - `src/garden/vegetables/mod.rs` 파일 안
- 일단 모듈이 크레이트 내에서 공개되면, 크레이트 어디에서든 사용 가능
  - `crate::garden::vegetables::Asparagus`
- 모듈은 기본적으로 부모 모듈에게 비공개이며, `pub mod`와 같이 `pub` 키워드를 붙여 공개로 전환 가능
- `use` 키워드를 통해 단축 경로 사용 가능
  - `use crate::garden::vegetables::Asparagus;`로 단축 후엔 `Asparagus`만 써도 타입 사용 가능

<!-- TOC --><a name="24-"></a>
### 2.4. 경로
- 경로는 두 가지 형태가 존재
  - 절대 경로(absolute path): 외부에선 해당 크레이트 이름으로 경로 시작, 내부에선 `crate` 키워드로 경로 시작
  - 상대 경로 (relative path): `self`, `super` 혹은 현재 모듈 내의 식별자 사용
- `::` 식별자를 통해 경로 구분
```rust
mod front_of_house {
    mod hosting {
        fn add_to_waitlist() {}
    }
}

pub fn eat_at_restaurant() {
    // 절대 경로
    crate::front_of_house::hosting::add_to_waitlist();

    // 상대 경로
    front_of_house::hosting::add_to_waitlist();
}
```

<!-- TOC --><a name="25-pub-"></a>
### 2.5. pub 키워드로 경로 노출
```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

pub fn eat_at_restaurant() {
    // 절대 경로
    crate::front_of_house::hosting::add_to_waitlist();

    // 상대 경로
    front_of_house::hosting::add_to_waitlist();
}
```
- 다음과 같이, 모듈의 내용도 `pub` 키워드를 통해 공개해야 사용 가능하다.

<!-- TOC --><a name="26-super-"></a>
### 2.6. super 키워드
```rust
fn deliver_order() {}

mod back_of_house {
    fn fix_incorrect_order() {
        cook_order();
        super::deliver_order();
    }

    fn cook_order() {}
}
```
- 다음과 같이, `super` 키워드를 통해 부모 모듈에 상대 경로로 접근이 가능하다.

<!-- TOC --><a name="27-"></a>
### 2.7. 구조체와 열거형의 공개
```rust
mod back_of_house {
    pub struct Breakfast {
        pub toast: String,
        seasonal_fruit: String,
    }

    impl Breakfast {
        pub fn summer(toast: &str) -> Breakfast {
            Breakfast {
                toast: String::from(toast),
                seasonal_fruit: String::from("peaches"),
            }
        }
    }
}
```
- 구조체에 `pub` 키워드를 부착하면 구조체 자체는 공개되지만, 구조체에 필드는 공개되지 않는다.
  - 필드도 공개하려면 필드에도 `pub` 키워드를 붙여야 한다.
```rust
mod back_of_house {
    pub enum Appetizer {
        Soup,
        Salad,
    }
}
```
- 반대로, 열거형은 `enum`에 `pub`을 부착하면 모든 배리언트가 공개된다.

<!-- TOC --><a name="28-use-"></a>
### 2.8. use 키워드를 통해 경로를 스코프 안으로 가져오기
```rust
use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
}
```
- 다음과 같이, `use` 키워드를 사용하면 단축 경로를 만들 수 있으며, 스코프 내 어디에서든 사용이 가능하다.
```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

use crate::front_of_house::hosting;

mod customer {
    pub fn eat_at_restaurant() {
        hosting::add_to_waitlist();
    }
}

// ...

warning: unused import: `crate::front_of_house::hosting`
```
- 다음과 같이 `use` 키워드는 사용된 스코프 내에서만 유효한 것을 확인할 수 있다.
  - 사용하려는 모듈 내로 `use`를 옮기거나, `super::hosting`과 같이 부모 모듈의 단축 경로를 호출하면 된다.
  -

<!-- TOC --><a name="29-as-"></a>
### 2.9. as 키워드
```rust
use std::fmt::Result;
use std::io::Result as IoResult;

fn function1() -> Result {
    // --생략--
}

fn function2() -> IoResult<()> {
    // --생략--
}
```
- 다음과 같이, `as` 키워드를 사용하면 타입의 이름을 변경하여 사용할 수 있다.
  - 모듈 간 충돌을 방지할 수 있다.

<!-- TOC --><a name="210-pub-use-"></a>
### 2.10. pub use로 다시 내보내기
```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

pub use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
}
```
- 다음과 같이, `pub use` 키워드를 사용하면 가져온 모듈을 본인 경로를 통해 접근 가능하도록 다시 내보내기 할 수 있다.
  - 코드 구성과, 노출 구성을 다르게 지정할 수 있다.
  - 제작과 사용을 별개로 가져갈 수 있으므로, 굉장히 유용하게 활용된다.

<!-- TOC --><a name="211-"></a>
### 2.11. 중첩 경로
```rust
use std::{cmp::Ordering, io};
use std::io::{self, Write}; // use std::io; use std::io::Write
use std::collections::*;
```
- 위와 같이, 경로를 그룹화 할 수 있다.
  - `use` 구문 사용 횟수를 크게 줄일 수 있다.
- 글롭 연산자 `*` 사용 시, 해당 모듈 안의 모든 공개 아이템을 가져올 수 있다.

<!-- TOC --><a name="3-hashmap"></a>
## 3. Hashmap
- 러스트에서 해시맵은 `HashMap<K, V>` 타입으로 구성되어 있다.
  - `K` 타입의 키와 `V` 타입의 값에 대해 **해시 함수** (hashing function) 를 사용하여 매핑한 것을 저장한다.
  - 해시 함수는 키와 값을 메모리 어디에 저장할지 지정한다.
- 해시맵은 모두 아는 개념이므로, 사용 문법만 익히고 빠르게 넘어가보자.

<!-- TOC --><a name="31-"></a>
### 3.1. 해시맵 선언
```rust
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);
```

<!-- TOC --><a name="32-"></a>
### 3.2. 해시맵 값 접근
```rust
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);

let team_name = String::from("Blue");
let score = scores.get(&team_name).copied().unwrap_or(0);
```

<!-- TOC --><a name="33-"></a>
### 3.3. 해시맵 반복
```rust
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);

for (key, value) in &scores {
    println!("{key}: {value}");
}
```
- key, value 쌍이 **임의의 순서**로 반복된다는 것에 유의해야 한다.
  - 해시맵은 순서가 없다.

<!-- TOC --><a name="34-"></a>
### 3.4. 해시맵과 소유권
```rust
use std::collections::HashMap;

let field_name = String::from("Favorite color");
let field_value = String::from("Blue");

let mut map = HashMap::new();
map.insert(field_name, field_value);
// field_name과 field_value는 이 시점부터 유효하지 않습니다.
```
- 해시맵에 키와 밸류로 사용되는 순간, 소유권이 해시맵으로 넘어간다.
  - 참조자를 사용하여 키와 밸류로 넘겨주면 소유권이 유지되지만, 해시맵이 유효한 동안 해당 키와 밸류의 소유권이 유효함이 보장되어야 한다.

<!-- TOC --><a name="35-"></a>
### 3.5. 해시맵 업데이트
```rust
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Blue"), 25);

println!("{:?}", scores);
```
- 같은 키에 대해 `insert`할 경우, **덮어씌워진다.**
```rust
use std::collections::HashMap;

let mut scores = HashMap::new();
scores.insert(String::from("Blue"), 10);

scores.entry(String::from("Yellow")).or_insert(50);
scores.entry(String::from("Blue")).or_insert(50);

println!("{:?}", scores);
```
- `entry`의 반환값은 열거형 `Entry`로, 해당 키의 존재 여부를 나타낸다.
  - `or_insert` 사용 시, 키가 없을 때에만 업데이트가 진행된다.

<!-- TOC --><a name="36-"></a>
### 3.6. 기존 값에 기초하여 업데이트
```rust
use std::collections::HashMap;

let text = "hello world wonderful world";

let mut map = HashMap::new();

for word in text.split_whitespace() {
    let count = map.entry(word).or_insert(0);
    *count += 1;
}

println!("{:?}", map);
```
- `or_insert` 메서드는 기존 값에 대한 가변 참조자 `&mut v`를 반환한다.
  - 이 말인 즉슨, `v`를 `*` 키워드를 통해 역참조할 시 값을 수정할 수 있게 된다.
- 위 예시에서 모든 가변 참조자는 `for` 문의 끝에서 스코프를 벗어나므로 대여 규칙에 위반되지 않는다.

<!-- TOC --><a name="4-option"></a>
## 4. Option
- `Option` 열거형에 대해선 week2에서 이미 다루어버렸다.
- 그 때 다루지 않은 while let 구문만 확인하고 넘어가자.

<!-- TOC --><a name="41-while-let"></a>
### 4.1. while let
```rust
while let Some(Some(integer)) = optional_integers.pop() {
assert_eq!(integer, cursor);
cursor -= 1;
}
```
- 다음과 같이, 반복 처리와 갈래 처리를 동시에 진행할 수 있다.
  - `optional_integers`는 `Vec<Option<u8>>` 타입이다.
  - 해당 벡터의 요소가 `Some`일 경우에만 반복이 진행될 것이다.
