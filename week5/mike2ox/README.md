# Week 5 Submission

## String

### &str vs String

- &str은 문자열을 표현하는 primitive한 타입이다.

  - 문자열 리터럴을 변수로 갖거나, 다른 변수가 소유한 문자열의 일부를 슬라이스하여 가져올 때 사용(그래서 string slice라 불리는거 같음)

- String은 힙에 할당된 문자열을 소유하는 타입이다.
- Rust 표준 라이브러리에서 제공해줌 -> 별도 설치 필요없음
- UTF-8로 인코딩된 텍스트를 저장 -> 다양한 언어의 문자를 저장할 수 있음
- 가변, 소유권을 가지고 있음 -> 러스트의 소유권 규칙에 따라 메모리 관리

```rust
fn main() {
    let hello = String::from("Hello, world!"); // String 인스턴스 생성
    let mut s = String::new(); // 빈 String 인스턴스 생성
    let s = "initial contents".to_string();
    let hello = String::from("안녕하세요");
}
```

### String을 만드는 방법들

- `to_owned()`
  - 새로운 인스턴스를 생성하는 메서드
- `to_string()`
  - `to_owned()`와 동일한 역할을 한다. but, `to_string()`은 임의의 타입을 `String`으로 변환하는데 사용되는데 여기서 `to_owned()`보다 더 많은 메모리를 사용할 수 있다.
- `into()`
  - `From` 트레이트를 구현하고 있어서 원하는 타입의 `from()`을 사용할 수 있다.

=> Rust 커뮤니티에서는 `to_owned` 사용을 권장한다.

- 안정성을 위해 `to_owned` 권장
- `to_string`이 Display를 구현한 모든 타입에 사용할 수 있기에 str이 아닌 타입이 의도치 않게 String으로 변환될 수 있다.
- 의미상 &str와 String은 모두 문자열을 의미하기에 단순 `to_string`보다는 소유권 이전을 명시적으로 표현하는 `to_owned`를 사용하는 것이 적합

### 문자열 결합

- `+` 연산자를 사용하면 두 문자열을 결합할 수 있다.
  - 단, `+` 연산자는 `String` 타입의 소유권을 가져가기 때문에 결합에 사용한 이전 String 중 `+` 앞 String은 이후에 사용할 수 없다.

```rust
fn main() {
    let s1 = String::from("Hello, ");
    let s2 = String::from("world!");
    let s3 = s1 + &s2; // 이후 s1은 사용 불가, s2는 사용 가능
}
```

- 만약, 여러 String을 결합해야 한다면 `format!` 매크로를 사용하는 것이 좋다.

```rust
fn main() {
    let s1 = String::from("tic");
    let s2 = String::from("tac");
    let s3 = String::from("toe");

    // format! 안에 원하는 문자열 형태를 넣어주면 됨
    let s = format!("{}-{}-{}", s1, s2, s3);
}
```

## 참고

- [to_owned vs to_string vs into](https://dev-seonghun.medium.com/rust-%EB%AC%B8%EC%9E%90%EC%97%B4-%EB%A6%AC%ED%84%B0%EB%9F%B4%EC%97%90%EC%84%9C-to-string-vs-to-owned-f40adc2b7ab5)
