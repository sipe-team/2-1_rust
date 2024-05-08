## Week 3 Result

![week3_result.png](images/week3_result.png)

## What I learned


### String
* String 생성
  * 대부분 Vec에서 사용하는 연산들은 String에서도 사용한다.
  * String은 UTF-8로 인코딩 되어있음.
```rust
// 비어있는 String 생성
let mut s = String::new();

// String에 담아두고 시작할 초기값 지정
// [1] to_string() 메소드 사용
let s2 = s.to_string();
let s3 = "hello".to_string();

// [2] String::from 함수 사용
let s4 = String::from("hi!");
```

* String 갱신
```rust
// String 추가하기
// [1] push_str 메소드 사용 (String Slice을 추가하기 위한)
let mut s1 = String::from("hello! ");
s1.push_str("how are you?");

let mut s2 = String::from("hi! ");
let s3 = "nice!";
s2.push_str(s3);

// [2] push 메소드 사용 (String 한 글자를 추가하기 위한)
let mut s4 = String::from("lo");
s4.push("l");

// [3] '+' 연산자 사용하기
let s5 = String::from("Hello, ");
let s6 = String::from("world!");
let s7 = s5 + &s6;

// [4] format! 사용하기
let s8 = String::from("holy");
let s9 = String::from("moly");
let s10 = format!("{}-{}", s8, s9);
```
* String 인덱싱
  * Rust에서는 String을 UTF-8로 저장한다.
    * 따라서 Rust에서 `char`는 항상 `4bytes`다.
    * `4bytes`는 모든 유니코드 스칼라 값(UTF-8)의 정수 값을 담을 수 있는 `2bytes` 수의 가장 작은 거듭제곱이기 때문
  * 인덱스 연산 시간이 항상 상수 시간(`O(1)`)으로 기대를 받지만 Rust의 위 구조상 String에서 그러한 퍼포먼스를 기대하기 어려움
    * String 내에 어떤 유효문자가 존재하는지 처음부터 지정된 인덱스까지 모든 곳을 훑어야하기 때문
```rust
let s1 = String::from("hello");
let h = s1[0];

// [RESULT] 'rust는 String 인덱싱을 지원하지 않는다.' 라는 컴파일 에러
// error: the trait bound `std::string::String: std::ops::Index<_>` is not
// satisfied [--explain E0277]
// |>
// |>     let h = s1[0];
// |>             ^^^^^
// note: the type `std::string::String` cannot be indexed by `_`
```

### Module
* Module이란?
  * Rust에서 코드를 쪼갬으로서 조직화된 방식으로 코드를 재사용 할 수 있게끔 Module 시스템을 제공한다.
  * 파일 시스템과 같은 구성을 보인다.
```rust
// 모듈 생성
mod network {
  fn connect() {
  }
}

// client.rs의 내용을 다른 위치에서 찾아야 함을 선언
mod client;
```
```rust
// 'src/client.rs'
fn connect() {
}
```
* 가시성 제어
  * 기본적으로 Module은 Private 성질을 가지고 있다.
  * 해당 Module을 만든 이유는 나만이 아닌 외부에서도 사용하기 위함을 기억해야 한다.
  * `Private 규칙`
    1. 만일 어떤 아이템이 공개라면, 이는 부모 모듈의 어디에서건 접근 가능합니다.
    2. 만일 어떤 아이템이 비공개라면, 같은 파일 내에 있는 부모 모듈 및 이 부모의 자식 모듈에서만 접근 가능합니다.
```rust
// 모듈 생성
mod network {
  // connect 함수 Public 전환
  pub fn connect() {
  }
}

// network 모듈의 connect 기능 사용
fn main() {
  network::connect();
}
```

### HashMap
* HashMap이란?
  * `HashMap<K, V>`
    * `K` 타입의 Key와 `V` 타입의 Value를 맵핑한 것
  * 해쉬 함수(hashing function)을 통해 동작
    * 해당 Key와 Value를 메모리 어디에 저장할지를 결정
```rust
// HashMap 생성    
let mut basket = HashMap::new();

// HashMap 값 넣기
basket.insert(String::from("banana"), 2);

// 키에 할당된 값이 없을 경우에만 삽입하기
basket.entry(String::from("banana")).or_insert(5); // HashMap 변경 X
scores.entry(String::from("apple")).or_insert(8); // HashMap의 값 추가
```