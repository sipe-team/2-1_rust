<!-- TOC --><a name="week-4-submission"></a>
# Week 4 Submission

![result.png](./images/result.png)

<!-- TOC --><a name="til"></a>
# TIL

<!-- TOC start (generated with https://github.com/derlin/bitdowntoc) -->

- [Week 4 Submission](#week-4-submission)
- [TIL](#til)
    * [1. error handling](#1-error-handling)
        + [1.1. panic!으로 복구 불가능한 에러 처리](#11-panic-)
        + [1.2. panic! 백트레이스](#12-panic-)
        + [1.3. `Result`로 복구 가능한 에러 처리](#13-result-)
        + [1.4. 서로 다른 에러 매칭](#14-)
        + [1.5. unwrap, expect](#15-unwrap-expect)
        + [1.6. 에러 전파](#16-)
        + [1.7. `?` 사용처](#17-)
    * [2. Generic Types](#2-generic-types)
        + [2.1. 제네릭 함수 정의](#21-)
        + [2.2. 제네릭 구조체 정의](#22-)
        + [2.3. 제네릭 열거형 정의](#23-)
        + [2.4. 제네릭 메서드 정의](#24-)
        + [2.5. 제네릭 코드의 성능](#25-)
    * [3. Traits](#3-traits)
        + [3.1. 트레이트 정의](#31-)
        + [3.2. 트레이트 구현](#32-)
        + [3.3. 트레이트 사용](#33-)
        + [3.4. 트레이트 구현 제약](#34-)
        + [3.5. 기본 구현](#35-)
        + [3.6. 트레이트 바운드 (Trait Bounds)](#36-trait-bounds)
        + [3.7. 트레이트 반환](#37-)
        + [3.7. 포괄 구현](#37--1)
    * [4. Lifetimes](#4-lifetimes)
        + [4.1. 라이프타임으로 댕글링 참조 방지](#41-)
        + [4.2. 대여 검사기](#42-)
        + [4.3. 라이프타임 명시 문법](#43-)
        + [4.4. 구조체 정의에서 라이프타임 명시](#44-)
        + [4.4. 라이프타임 생략](#44--1)
        + [4.5. 메서드 정의에 라이프타임 명시](#45-)
        + [4.6. 정적 라이프타임](#46-)
        + [4.7. 제네릭 타입 매개변수, 트레이트 바운드, 라이프타임을 한 곳에 사용해 보기](#47-)

<!-- TOC end -->


<!-- TOC --><a name="1-error-handling"></a>
## 1. error handling
- 러스트에서 에러는 다음과 같은 두 가지 범주로 나뉜다.
    - 복구 가능한 에러 (recoverable error)
    - 복구 불가능한 에러 (unrecoverable error)
- 대부분의 언어는 위 두 가지 범주를 예외 처리(exception handling)란 개념을 통해 같은 방식으로 처리
    - 러스트는 예외 처리가 없다. (!)
    - 복구 가능한 에러를 처리하기 위한 `Result<T, E>` 타입과, 복구 불가능한 에러를 처리하는 `panic!` 매크로만 존재

<!-- TOC --><a name="11-panic-"></a>
### 1.1. panic!으로 복구 불가능한 에러 처리
- panic!은 복구 불가능한 에러가 발생 시 프로그램을 종료시킨다.
    - 메시지를 출력하고, **unwinding**(되감기)하여 스택을 청소하고, 종료한다.
- unwinding은 스택을 거꾸로 훑어보며 청소하는 작업으로, 생각보다 간단한 작업이 아니다.
    - aborting 적용 시 이 과정을 진행하지 않으며, 운영체제에 청소를 맡긴다.
      ```toml
      [profile.release]
      panic = 'abort'    
      ```

<!-- TOC --><a name="12-panic-"></a>
### 1.2. panic! 백트레이스
```bash
$ RUST_BACKTRACE=1 cargo run
thread 'main' panicked at 'index out of bounds: the len is 3 but the index is 99', src/main.rs:4:5
stack backtrace:
   0: rust_begin_unwind
             at /rustc/e092d0b6b43f2de967af0887873151bb1c0b18d3/library/std/src/panicking.rs:584:5
   1: core::panicking::panic_fmt
             at /rustc/e092d0b6b43f2de967af0887873151bb1c0b18d3/library/core/src/panicking.rs:142:14
   2: core::panicking::panic_bounds_check
             at /rustc/e092d0b6b43f2de967af0887873151bb1c0b18d3/library/core/src/panicking.rs:84:5
   3: <usize as core::slice::index::SliceIndex<[T]>>::index
             at /rustc/e092d0b6b43f2de967af0887873151bb1c0b18d3/library/core/src/slice/index.rs:242:10
   4: core::slice::index::<impl core::ops::index::Index<I> for [T]>::index
             at /rustc/e092d0b6b43f2de967af0887873151bb1c0b18d3/library/core/src/slice/index.rs:18:9
   5: <alloc::vec::Vec<T,A> as core::ops::index::Index<I>>::index
             at /rustc/e092d0b6b43f2de967af0887873151bb1c0b18d3/library/alloc/src/vec/mod.rs:2591:9
   6: panic::main
             at ./src/main.rs:4:5
   7: core::ops::function::FnOnce::call_once
             at /rustc/e092d0b6b43f2de967af0887873151bb1c0b18d3/library/core/src/ops/function.rs:248:5
note: Some details are omitted, run with `RUST_BACKTRACE=full` for a verbose backtrace.
```
- 다음과 같이, `RUST_BACKTRACE` 환경 변수를 1로 활성화해주면 에러에 대한 스택 트레이스를 확인할 수 있다.
    - 디버그 심볼이 활성화되어 있어야 하며, `cargo build` 혹은 `run` 커맨드 실행 시 `--release` 옵션이 없으면 기본적으로 적용된다.

<!-- TOC --><a name="13-result-"></a>
### 1.3. `Result`로 복구 가능한 에러 처리
```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```
- `Result` 타입은 복구 가능한 에러에 대한 처리를 지원한다.
    - `Ok` 배리언트는 반환한 값을 가지고 있다.
    - `Err` 배리언트는 에러 타입을 가지고 있다.
```rust
fn main() {
    let greeting_file_result = File::open("hello.txt");

    let greeting_file = match greeting_file_result {
        Ok(file) => file,
        Err(error) => panic!("Problem opening the file: {:?}", error),
    };
}
```
- 이렇게 특정 메서드가 `Result` 타입을 반환한다면, 사용하는 측에서 에러 처리 방식을 판별하면 된다.
    - 열거형이므로, `match`를 통한 처리가 가능하다.
- `Result`의 배리언트들도 `Option`과 마찬가지로 프렐루드로부터 가져와진다.
    - 즉, 따로 `use`할 필요가 없다.

<!-- TOC --><a name="14-"></a>
### 1.4. 서로 다른 에러 매칭
```rust
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let greeting_file_result = File::open("hello.txt");

    let greeting_file = match greeting_file_result {
        Ok(file) => file,
        Err(error) => match error.kind() {
            ErrorKind::NotFound => match File::create("hello.txt") {
                Ok(fc) => fc,
                Err(e) => panic!("Problem creating the file: {:?}", e),
            },
            other_error => {
                panic!("Problem opening the file: {:?}", other_error);
            }
        },
    };
}
```
- 위와 같이 반환되는 에러의 종류가 여러개라면, 그에 맞는 처리를 수행할 수 있다.
```rust
fn main() {
    let greeting_file = File::open("hello.txt").unwrap_or_else(|error| {
        if error.kind() == ErrorKind::NotFound {
            File::create("hello.txt").unwrap_or_else(|error| {
                panic!("Problem creating the file: {:?}", error);
            })
        } else {
            panic!("Problem opening the file: {:?}", error);
        }
    });
}
```
- 위와 같이, `unwrap_or_else` 메서드를 활용하면 `match`를 통한 거대한 표현식을 단순화할 수 있음을 알아두자.

<!-- TOC --><a name="15-unwrap-expect"></a>
### 1.5. unwrap, expect
```rust
let greeting_file = File::open("hello.txt").unwrap();
```
- 위와 같이 `unwrap()`을 사용 시, 값이 없다면 알아서 `panic!`을 일으킨다.
```rust
let greeting_file = File::open("hello.txt")
        .expect("hello.txt should be included in this project");
```
- 위와 같이 `expect()`를 사용 시, `panic!`의 에러 메시지를 직접 지정할 수 있다.
    - 더 많은 맥락을 제공할 수 있으므로, 권장된다.

<!-- TOC --><a name="16-"></a>
### 1.6. 에러 전파
```rust
fn read_username_from_file() -> Result<String, io::Error> {
    let username_file_result = File::open("hello.txt");

    let mut username_file = match username_file_result {
        Ok(file) => file,
        Err(e) => return Err(e),
    };

    let mut username = String::new();

    match username_file.read_to_string(&mut username) {
        Ok(_) => Ok(username),
        Err(e) => Err(e),
    }
}
```
- 다음과 같이 `Result`를 반환함으로써, 에러 처리를 호출하는 쪽에 맡길 수 있다.
```rust
fn read_username_from_file() -> Result<String, io::Error> {
    let mut username_file = File::open("hello.txt")?;
    let mut username = String::new();
    username_file.read_to_string(&mut username)?;
    Ok(username)
}
```
- 다음과 같이 `?` 숏컷을 사용하면 에러 전파를 축약하여 사용할 수 있다.
    - 특정 메서드의 결과가 `Err`라면, 그 즉시 `Err`를 반환한다.
- `?`와 `match`의 차이점은, `from` 메서드 호출 여부에 있다.
    - `?`는 에러 반환 시 `From` 트레이트에 구현된 `from` 메서드를 호출하며, 반환할 에러 타입으로 변환하는 작업을 거친다.
    - 이는 다양한 종류의 에러를 일반화하여 반환하고 싶을 때 유용하다.
```rust
fn read_username_from_file() -> Result<String, io::Error> {
    let mut username = String::new();

    File::open("hello.txt")?.read_to_string(&mut username)?;

    Ok(username)
}
```
- `?` 숏컷은 메서드 체이닝 또한 가능하다.
    - 체이닝 된 메서드 모두가 성공해야 정상 동작이 이루어질 것이다.
```rust
fn read_username_from_file() -> Result<String, io::Error> {
    fs::read_to_string("hello.txt")
}
```
- 참고로, 위 내용은 일반적인 동작이므로 `read_to_string` 메서드를 통해 정의되어 있으니 참고하자.

<!-- TOC --><a name="17-"></a>
### 1.7. `?` 사용처
- `?` 숏컷은 `?`이 사용된 값과 호환되는 값을 반환하는 함수에서만 사용 가능하다.
```rust
fn main() {
    let greeting_file = File::open("hello.txt")?;
}
```
- 위 예시에서, `open` 메서드는 `Result`를 반환하지만 `main`은 `unit`을 반환하므로 `?` 사용이 불가능하다.
```rust
fn main() -> Result<(), Box<dyn Error>> {
    let greeting_file = File::open("hello.txt")?;

    Ok(())
}
```
- 위 코드와 같이 수정함으로써 main 함수에서도 `?` 연산자 사용이 가능해진다.
    - unit 타입인 `()`을 반환하도록 한다. (아무것도 반환하지 않는다는 뜻이다.)
    - `Box<dyn Error>`은 트레이트 객체(trait object)라 칭한다.
        - 나중에 배울 내용으로, 지금 당장은 **Error 트레이트를 구현한 모든 객체를 허용**한다 정도로만 알아두면 되겠다.
- 참고로 `?` 숏컷은 `Result` 처리를 위해서만 작동하는건 아니다.
```rust
fn last_char_of_first_line(text: &str) -> Option<char> {
    text.lines().next()?.chars().last()
}
```
- 위와 같이, match를 통한 반환이 이루어지는 경우 모두에 활용 가능하므로 참고하자.

<!-- TOC --><a name="2-generic-types"></a>
## 2. Generic Types
- 러스트에서의 제네릭은 자바와 거의 동일한 개념이다.
    - 함수 시그니처나 구조체에 다양한 데이터 타입을 사용할 수 있도록 허락한다.

<!-- TOC --><a name="21-"></a>
### 2.1. 제네릭 함수 정의
```rust
fn largest<T>(list: &[T]) -> &T {
```
- 다음과 같이, <> 사이에 **타입 배개변수**를 지정해주어 제네릭 함수 정의가 가능하다.
    - ‘largest 함수는 어떤 타입 T에 대한 제네릭 함수’라고 읽을 수 있다.

<!-- TOC --><a name="22-"></a>
### 2.2. 제네릭 구조체 정의
```rust
struct Point<T, U> {
    x: T,
    y: U,
}

fn main() {
    let both_integer = Point { x: 5, y: 10 };
    let both_float = Point { x: 1.0, y: 4.0 };
    let integer_and_float = Point { x: 5, y: 4.0 };
}
```
- 다음과 같이, 여러 타입의 필드 각각에 각기 다른 제네릭 타입을 지정해줄 수 있다.

<!-- TOC --><a name="23-"></a>
### 2.3. 제네릭 열거형 정의
```rust
enum Option<T> {
    Some(T),
    None,
}

enum Result<T, E> {
    Ok(T),
    Err(E),
}
```
- 지금까지 배웠던 `Option`과 `Result`가 제네릭 타입을 포함한 열거형의 대표적인 예시이다.

<!-- TOC --><a name="24-"></a>
### 2.4. 제네릭 메서드 정의
```rust
struct Point<T> {
    x: T,
    y: T,
}

impl<T> Point<T> {
    fn x(&self) -> &T {
        &self.x
    }
}
```
- 다음과 같이, 제네릭 구조체에 대한 제네릭 메서드를 선언할 수 있다.
```rust
impl Point<f32> {
    fn distance_from_origin(&self) -> f32 {
        (self.x.powi(2) + self.y.powi(2)).sqrt()
    }
}
```
- 다음과 같이 제네릭 구조체에 대해, 특정 타입에 대한 메서드만 구현할 수도 있으므로 인지해두자.

<!-- TOC --><a name="25-"></a>
### 2.5. 제네릭 코드의 성능
- 러스트는 컴파일 시에, 제네릭 코드를 단형성화 (monomorphization) 한다고 한다.
    - 즉, 제네릭 타입을 실제 구체 타입으로 변환하여 컴파일한다.
        - 제네릭 코드가 호출된 곳을 전부 찾아서, 사용된 타입을 확인한 후 변환한다.
- 이 말인 즉슨, 제네릭 코드가 우리의 코드 성능을 저하시킬 일은 전혀 없다.

<!-- TOC --><a name="3-traits"></a>
## 3. Traits
- **트레이트**(trait)은 특정한 타입의 **동작 자체**를 정의한다.
    - 여러 타입이 공통된 동작을 수행할 수 있도록 만들어준다.
- 이는 `java`의 `interface`와 매우 유사한 개념이다.

<!-- TOC --><a name="31-"></a>
### 3.1. 트레이트 정의
```rust
pub trait Summary {
    fn summarize(&self) -> String;
}
```
- 다음과 같이 `trait` 키워드를 통해 트레이트 선언이 가능하다.
- 해당 트레이트를 구현하는 모든 타입은, 트레이트 본문에 선언된 메서드 시그니처를 구현하도록 강제된다.
    - 이 때, 트레이트와 구현체의 시그니처가 동일해야 한다는 부분 또한 강제된다.

<!-- TOC --><a name="32-"></a>
### 3.2. 트레이트 구현
```rust
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
```
- `impl {trait} for {type}`과 같은 방식으로 특정 트레이트를 구현할 수 있다.

<!-- TOC --><a name="33-"></a>
### 3.3. 트레이트 사용
```rust
use aggregator::{Summary, Tweet};

fn main() {
    let tweet = Tweet {
        username: String::from("horse_ebooks"),
        content: String::from(
            "of course, as you probably already know, people",
        ),
        reply: false,
        retweet: false,
    };

    println!("1 new tweet: {}", tweet.summarize());
}
```
- 자기 자신의 메서드를 활용하듯이, 자연스럽게 트레이트의 메서드를 호출할 수 있다.
    - 주의할 점은, `trait` 또한 `use` 키워드를 통해 스코프로 가져와야 호출이 가능하다.

<!-- TOC --><a name="34-"></a>
### 3.4. 트레이트 구현 제약
- 외부 타입에 외부 트레이트를 구현하는 것은 불가능하다.
    - 예를 들어, 직접 만든 크레이트에서는 `Vec<T>`에 대한 `Display` 트레이트를 구현할 수 없다.
        - Vec<T>, Display 둘 다 직접 만든 크레이트가 아닌 표준 라이브러리에 정의되어 있기 때문이다.
- 이 규칙이 없다면 두 크레이트가 동일한 타입에 동일한 트레이트를 구현할 수 있게 되고, 러스트는 어떤 구현체를 이용해야 할지 알 수 없게 된다.

<!-- TOC --><a name="35-"></a>
### 3.5. 기본 구현
```rust
pub trait Summary {
    fn summarize(&self) -> String {
        String::from("(Read more...)")
    }
}
```
- 위와 같이, 트레이트 메서드에 기본 동작을 부여할 수 있다.
    - 트레이트를 구현하는 타입은 기본 동작을 그대로 사용할지, `override`하여 동작을 새로 구현할지 선택할 수 있다.
- `impl Summary for NewsArticle {}`와 같이 기본 구현 메서드를 따로 재구현하지 않을 시, 기본 구현을 그대로 사용한다.
- 오버라이딩 시, 기존 기본 구현은 사용할 수 없음에 유의하자.

<!-- TOC --><a name="36-trait-bounds"></a>
### 3.6. 트레이트 바운드 (Trait Bounds)
```rust
pub fn notify(item: &impl Summary) {
    println!("Breaking news! {}", item.summarize());
}
```
- 위와 같이, `impl {trait}`을 매개변수의 타입으로 지정 시, 해당 트레이트를 구현한 모든 타입을 사용할 수 있게 된다.
```rust
pub fn notify<T: Summary>(item: &T) {
    println!("Breaking news! {}", item.summarize());
}
```
- `impl {trait}`는 위와 같은 **트레이트 바운드**로 칭해지는, 트레이트의 제네릭 타입 문법의 축약 버전이다.
    - 제네릭 타입 매개변수 뒤에 콜론과 트레이트 타입을 부착함으로써 트레이트 바운드 선언이 가능하다.
```rust
pub fn notify(item: &(impl Summary + Display)) {
```
- 위와 같이, 여러 트레이트를 모두 허용하는 트레이트 바운드를 만들 수 있다.
```rust
fn some_function<T, U>(t: &T, u: &U) -> i32
where
    T: Display + Clone,
    U: Clone + Debug,
{
```
- 위와 같이, `where` 키워드를 통해 트레이트 바운드를 더 가독성 좋은 형태로 정리할 수 있다.

<!-- TOC --><a name="37-"></a>
### 3.7. 트레이트 반환
```rust
fn returns_summarizable() -> impl Summary {
    Tweet {
        username: String::from("horse_ebooks"),
        content: String::from(
            "of course, as you probably already know, people",
        ),
        reply: false,
        retweet: false,
    }
}
```
- 위와 같이, 트레이트를 구현하는 타입이 반환되도록 지정할 수 있다.
```rust
fn returns_summarizable(switch: bool) -> impl Summary {
    if switch {
        NewsArticle {
            headline: String::from(
                "Penguins win the Stanley Cup Championship!",
            ),
            location: String::from("Pittsburgh, PA, USA"),
            author: String::from("Iceburgh"),
            content: String::from(
                "The Pittsburgh Penguins once again are the best \
                 hockey team in the NHL.",
            ),
        }
    } else {
        Tweet {
            username: String::from("horse_ebooks"),
            content: String::from(
                "of course, as you probably already know, people",
            ),
            reply: false,
            retweet: false,
        }
    }
}
```
- 이 때 주의할 점은, 트레이트의 구현 방식으로 인해 위와 같이 여러 타입이 반환되도록 구성할 수는 없다.
    - 추후 이것을 가능하게 하는 방법을 알아본다고 한다.

<!-- TOC --><a name="37--1"></a>
### 3.7. 포괄 구현
```rust
impl<T: Display> ToString for T {
    // --생략--
}
```
- 위 문법은 **포괄 구현** (blanket implementations) 이라 칭하는 트레이트 구현 방식이다.
    - `Display` 트레이트를 구현하는 모든 타입에 `ToString` 트레이트를 구현한다는 뜻이다.
- 표준 라이브러리 내에서 광범위하게 사용되는 문법이라고 하니, 잘 알아두자.

<!-- TOC --><a name="4-lifetimes"></a>
## 4. Lifetimes
- 라이프타임(lifetimes)은 참조자가 필요한 기간동 유효함을 보장해주는 또 다른 종류의 제네릭이다.
    - 보통 라이프타임은 암묵적으로 추론되지만, 참조자의 수명이 여러 방식으로 연관될 가능성이 있는 경우 명시해줘야 한다.

<!-- TOC --><a name="41-"></a>
### 4.1. 라이프타임으로 댕글링 참조 방지
- 댕글링 참조 개념은 소유권을 배울 때 배웠으니, 넘어가도록 하자.
```rust
fn main() {
    let r;

    {
        let x = 5;
        r = &x;
    }

    println!("r: {}", r);
}
// ...
error[E0597]: `x` does not live long enough
```
- 댕글링 참조의 예시다. 이 경우, `does not live long enough` 라는 에러가 발생한다.
    - r이 참조하는 x를 사용하려는 시점에 x가 이미 스코프를 벗어났기 때문이다.
    - 각 변수의 라이프타임을 **스코프** 개념으로 관리하며, x보다 r의 **스코프**가 더 크기 때문에 해당하는 에러가 발생한 것을 알 수 있다.
- 즉, 러스트에서는 스코프가 더 클수록 **더 오래 산다**(lives longer)고 표현한다.

<!-- TOC --><a name="42-"></a>
### 4.2. 대여 검사기
- 러스트 컴파일러는 대여 검사기 (borrow checker) 로 스코프를 비교하여 대여의 유효성을 판단한다.
```rust
fn main() {
    let r;                // ---------+-- 'a
                          //          |
    {                     //          |
        let x = 5;        // -+-- 'b  |
        r = &x;           //  |       |
    }                     // -+       |
                          //          |
    println!("r: {}", r); //          |
}                         // ---------+
```
- 각 변수의 라이프타임을 `'a`, `'b`로 표현한 모습니다.
- 컴파일러는 두 라이프타임의 크기와, 두 라이프타임 간의 참조 관계를 인지한다.
    - 참조 대상의 라이프타임인 `'b`가 참조자 라이프타임인 `'a`보다 더 길게 살지 못했으므로, 컴파일에 실패한다.
```rust
fn main() {
    let x = 5;            // ----------+-- 'b
                          //           |
    let r = &x;           // --+-- 'a  |
                          //   |       |
    println!("r: {}", r); //   |       |
                          // --+       |
}                         // ----------+
```
- 위와 같이, 참조 대상의 라이프타임을 참조자의 라이프타임보다 더 길게 만들어줌으로써 정상적으로 컴파일 가능하도록 할 수 있다.

<!-- TOC --><a name="43-"></a>
### 4.3. 라이프타임 명시 문법
```rust
&i32        // 참조자
&'a i32     // 명시적인 라이프타임이 있는 참조자
&'a mut i32 // 명시적인 라이프타임이 있는 가변 참조자
```
- 참조자 키워드 `&` 뒤에 `'`(어퍼스트로피)를 부착하면 라이프타임이다.
    - 매우 짧은 소문자를 사용하는 것이 관례이며, `'a``'b``'c`와 같이 사용한다.
- 라이프타임을 명시해준다고 해서 참조자의 수명이 바뀌진 않는다.
    - 단지, 참조자 간 수명의 연관 관계를 정의해주기 위해 사용된다.
```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```
- 위와 같이 `x`와 `y`의 라이프타임이 동일함을 명시할 수 있다.
    - 또한, 반환하려는 문자열 슬라이스 또한 `x`와 `y`의 라이프타임 동한 유효함을 보장해준다는 뜻이다.
```rust
fn main() {
    let string1 = String::from("long string is long");

    {
        let string2 = String::from("xyz");
        let result = longest(string1.as_str(), string2.as_str());
        println!("The longest string is {}", result);
    }
}
```
- 위와 같은 경우, `longest` 함수의 매개변수 중 `string2`의 라이프타임이 가장 짧으므로, `'a`는 `string2`의 라이프타임이 된다.
    - `result`가 유효할 동안 `string2`가 유효함이 보장되므로 성공적으로 컴파일된다.
```rust
fn main() {
    let string1 = String::from("long string is long");
    let result;
    {
        let string2 = String::from("xyz");
        result = longest(string1.as_str(), string2.as_str());
    }
    println!("The longest string is {}", result);
}
// ...
error[E0597]: `string2` does not live long enough
```
- 위 경우에도 `string2`의 라이프타임이 가장 짧으므로 `string2`의 라이프타임이 `'a`가 된다.
    - 그러나 이 경우엔, 반환값인 `result`가 유호할 동안 `string2`가 유효함이 보장되지 않는다.
    - 즉, `'a`의 스코프가 `result`보다 작다.
- 이 경우, 컴파일 에러가 발생한다.

<!-- TOC --><a name="44-"></a>
### 4.4. 구조체 정의에서 라이프타임 명시
- 구조체가 **참조자 필드**를 보유할 수 있도록 하려면, **라이프타임**이 **무조건 명시**되어야 한다.
```rust
struct ImportantExcerpt<'a> {
    part: &'a str,
}
```
- 구조체의 라이프타임 매개변수 선언 방법은 제네릭 데이터 타입과 동일하다.
    - 구조체 이름 뒤 꺾쇠괄호 내에 선언, 구조체 정의 본문에서 라이프타임 매개변수를 이용한다.
- 해당 라이프타임 명시는 ‘ImportantExcerpt 인스턴스는 part 필드가 보관하는 참조자의 라이프타임보다 오래 살 수 없다’라는 의미이다.

<!-- TOC --><a name="44--1"></a>
### 4.4. 라이프타임 생략
- 러스트 초기엔 이와 같이 라이프타임을 항상 명시해줘야 헀다.
    - 하지만 대부분의 상황에서 동일한 라이프타임이 명시되는 것을 확인하였으며, 모두 예측 가능한 상황임을 알게 되었다.
- 이에, 러스트는 **라이프타임 생략 규칙** (lifetime elision rules)을 지정하여 참조자 분석 기능에 포함시켰다.
    - 세 가지 규칙을 모두 적용하여도 특정 참조자의 라이프타임을 알아내지 못할 경우, 라이프타임을 명시해줘야 한다.
- 먼저 아래와 같은 정의를 인지해야 한다.
    - 함수나 메서드 매개변수의 라이프타임은 **입력 라이프타임** (input lifetime)
    - 반환 값의 라이프타임은 **출력 라이프타임** (output lifetime)
- 규칙은 아래와 같다.
    1. 컴파일러가 참조자인 매개변수 각각에게 라이프타임 매개변수를 할당한다.
        - `fn foo<'a>(x: &'a i32)`처럼 매개변수가 하나인 함수는 하나의 라이프타임 매개변수를 갖게 된다.
        - `fn foo<'a, 'b>(x: &'a i32, y: &'b i32)`처럼 매개변수가 두 개인 함수는 두 개의 개별 라이프타임 매개변수를 갖게 된다.
    2. 입력 라이프타임 매개변수가 딱 하나라면, 해당 라이프타임이 모든 출력 라이프타임에 대입된다.
        - `fn foo<'a>(x: &'a i32) -> &'a i32`가 예시이다.
    3. 입력 라이프타임 매개변수가 여러 개인데 그중 하나가 `&self`나 `&mut self`라면, 즉 **메서드**라면 `self`의 라이프타임이 모든 출력 라이프타임 매개변수에 대입된다.
```rust
fn longest(x: &str, y: &str) -> &str {
// 첫 번째 규칙 적용
fn longest<'a, 'b>(x: &'a str, y: &'b str) -> &str {
// 두 세번째 규칙 적용 불가
```
- 계속해서 봐왔던 예시는 첫 번째 규칙만 적용할 수 있으며, 적용 시 반환 값의 라이프타임을 알 수 없다.
    - 이 경우 라이프타임을 직접 명시해줘야 한다.

<!-- TOC --><a name="45-"></a>
### 4.5. 메서드 정의에 라이프타임 명시
- 라이프타임을 갖는 메서드를 구현하는 문법은 제네릭 매개변수 문법과 동일하다.
```rust
impl<'a> ImportantExcerpt<'a> {
    fn level(&self) -> i32 {
        3
    }
}
```
- 참조자를 가지는 구조체는 라이프타임을 무조건 가지며, 이 경우 라이프타임 매개변수는 구조체 타입의 일부가 된 상태이다.
    - 즉, 메서드 구현 시에도 꼭 라이프타임을 명시해줘야 한다.
- 다만 첫 번째 생략 규칙에 의해 메서드 시그니처엔 라이프타임이 생략된 상태이다.
```rust
impl<'a> ImportantExcerpt<'a> {
    fn announce_and_return_part(&self, announcement: &str) -> &str {
        println!("Attention please: {}", announcement);
        self.part
    }
}
```
- 이 경우엔, 세 번째 생략 규칙에 의해 모든 라이프타임이 `self`의 라이프타임에 맞춰 생략되었다.

<!-- TOC --><a name="46-"></a>
### 4.6. 정적 라이프타임
```rust
let s: &'static str = "I have a static lifetime.";
```
- 위와 같이 `'static` 라이프타임을 통해 변수를 선언할 경우, 프로그램 전체 생애 주기 동안 해당 변수가 유효함이 보장된다.
    - 모든 문자열 리터럴은 `'static` 라이프타임을 가지고 있다.

<!-- TOC --><a name="47-"></a>
### 4.7. 제네릭 타입 매개변수, 트레이트 바운드, 라이프타임을 한 곳에 사용해 보기
```rust
use std::fmt::Display;

fn longest_with_an_announcement<'a, T>(
    x: &'a str,
    y: &'a str,
    ann: T,
) -> &'a str
    where
        T: Display,
{
    println!("Announcement! {}", ann);
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```
- 이번 장에서 배운 내용을 총집합 해보았다.
    - <> 안에 라이프타임과 제네릭 타입을 같이 사용할 수 있음을 인지하자.