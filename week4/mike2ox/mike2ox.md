# (늦은) Week 4 Submission

![week4_result.png](./images/week4_result.png)

## error_handling (여기는 심화단계 공부하면서 여러번 볼 필요가 있음)

- 러스트에서는 복구 가능한 에러와 복구 불가능한 에러 두가지로 나눔
  - 복구 가능한 에러는 `Result` 열거형을 사용하여 처리
  - 복구 불가능한 에러는 `panic!` 매크로를 사용하여 처리
- `panic!` 매크로는 프로그램을 즉시 종료하고 에러 메시지를 출력함

```rust
fn main() {
    panic!("crash and burn");
}
```

- `Result`와 `Option`을 사용하여 에러를 처리함
  - `Result`는 에러를 반환할 때 사용(enum에서 `Ok`와 `Err`으로 구성됨)
  - `Option`은 값이 없을 때 사용(enum에서 `Some`과 `None`으로 구성됨)
  - `unwrap` 메소드를 사용하여 값을 꺼낼 수 있음
    - `unwrap`은 에러가 발생하면 프로그램이 패닉 상태가 됨(ok -> ok값, err -> panic! 호출)
    - `expect` 메소드를 사용하여 에러 메시지를 출력할 수 있음(`unwrap`과 동일하지만 에러 메시지를 출력할 수 있음)
    - `unwrap_or` 메소드를 사용하여 기본값을 반환할 수 있음
    - `unwrap_or_else` 메소드를 사용하여 클로저를 사용하여 기본값을 반환할 수 있음

```rust
use std::fs::File;

fn main() {
    let f = File::open("hello.txt");

    let f = match f {
        Ok(file) => file,
        Err(error) => {
            panic!("Problem opening the file: {:?}", error)
        },
    };
}
```

- `Result`를 반환하는 함수를 사용할 때는 `match` 표현식을 사용하여 에러를 처리함
  - `?` 연산자를 사용하여 에러를 반환할 수 있음
    - `Result`를 반환하는 함수에서만 사용할 수 있다

```rust
use std::fs::File;
use std::io::{self, Read};

fn read_username_from_file() -> Result<String, io::Error> {
    let mut username_file = File::open("hello.txt")?; // ? 연산자를 사용하여 에러를 반환할 수 있음
    let mut username = String::new();
    username_file.read_to_string(&mut username)?;
    Ok(username)
}
```

## generics

- 제네릭은 함수나 구조체를 정의할 때 타입을 파라미터로 받아서 사용할 수 있게 해줌
  - 기존 타 언어의 템플릿과 유사함
  - 특히, 타입스크립트와 유사함
- 러스트의 제네릭은 컴파일 타임에 타입을 체크하기 때문에 런타임 오버헤드가 없음
- 중복 코드가 줄어들고 코드 재사용성이 높아짐
- 막상 제네릭 함수나 구조체를 사용할 때는 타입을 명시해줘야 함

```rust
fn largest<T>(list: &[T]) -> T { // &[T]는 제네릭 타입의 슬라이스를
    let mut largest = list[0]; // list의 첫번째 원소를 largest로 초기화

    for &item in list.iter() {
        if item > largest {
            largest = item;
        }
    }

    largest
}
```

## traits

- traits는 러스트의 타입 시스템을 확장하는 방법 중 하나로 다른 언어에서의 인터페이스와 유사
  - 구조체에 traits를 구현하면 해당 구조체에 traits의 기능을 사용할 수 있음
- traits는 다른 타입에 대해 공통적인 기능을 정의할 수 있음

```rust
pub trait Summary {
    fn summarize(&self) -> String;
}

pub struct NewsArticle {
    pub headline: String,
    pub location: String,
    pub author: String,
    pub content: String,
}

impl Summary for NewsArticle { // NewsArticle 구조체에 Summary traits를 구현
    fn summarize(&self) -> String {
        format!("{}, by {} ({})", self.headline, self.author, self.location)
    }
}
```

- 매개변수로 traits를 받을 수 있음

```rust
fn notify(item: &impl Summary) { // Summary traits를 구현한 타입을 받음(trait bound의 문법 설탕임)
    println!("Breaking news! {}", item.summarize());
}

fn notify<T: Summary>(item: &T) { // Summary traits를 구현한 타입을 받음
    println!("Breaking news! {}", item.summarize());
}
```

- `+` 연산자를 사용하여 traits를 구현할 수 있음

```rust
pub trait Summary {
    fn summarize(&self) -> String;
}

pub trait Display {
    fn display(&self) -> String;
}

pub struct NewsArticle {
    pub headline: String,
    pub location: String,
    pub author: String,
    pub content: String,
}

impl Summary for NewsArticle {
    fn summarize(&self) -> String {
        format!("{}, by {} ({})", self.headline, self.author, self.location)
    }
}

impl Display for NewsArticle {
    fn display(&self) -> String {
        format!("{}, by {} ({})", self.headline, self.author, self.location)
    }
}

fn notify(item: &(impl Summary + Display)) { // Summary와 Display traits를 구현한 타입을 받음
    println!("Breaking news! {}", item.summarize());
    println!("Display: {}", item.display());
}
```

- trait bound를 사용하여 여러 trait를 구현한 타입을 받을 수 있음

```rust
fn notify<T: Summary + Display>(item: &T) {
    println!("Breaking news! {}", item.summarize());
    println!("Display: {}", item.display());
}
```

## lifetimes

- 러스트의 라이프타임은 `어떤 참조자가 필요한 기간동안 유효함을 보장`하기 위한 것
  - 결국 `댕글링 참조(dangling reference)`를 방지위한 것
  - 함수의 파라미터나 반환값에 사용됨
  - `'` 기호로 표현됨

```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str { // 'a는 라이프타임을 의미
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

- `'` 명시를 한다고 참조자의 lifetime이 바뀌진 않음

  - 여러 참조자에 대한 lifetime에 영향 주지 않으면서 관계를 정의할 수 있음

- 함수의 시그니처에 lifetime을 명시하면 함수의 파라미터와 반환값의 lifetime이 같아야 함
  - lifetime elision이라는 문법 설탕이 있어서 명시하지 않아도 컴파일러가 자동으로 lifetime을 추론해줌

```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str { // 명시하지 않아도 컴파일러가 자동으로 lifetime을 추론해줌
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

## 후기

> error_handling가 가장 어려웠다. error_handling을 지나고 나니 다른 것들은 의외로 잘 풀린(?) 느낌스
> 2차 미션 시작전에 정리하고 있는데도 여전히 error_handling 파트의 내용이 머리속에 잘 안들어오는 것 같다. grep개발하면서 다시 한번 복습해야겠다.
