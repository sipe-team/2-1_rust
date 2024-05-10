# week2

![image](./images/week2.png)

### primitive_types

- `panic` : println과 달리 터미널 창에 빨간 글자로 error 메세지를 보여준다.
- `&` : & 연산자는 참조를 생성한다. 해당 값에 대한 참조를 만들고, 그 값을 대신하여 해당 참조를 사용할 수 있게 해준다. rust 슬라이스는 컴파일 시간에 크기를 알 수 없고 런타임에 크기가 결정된다.

### vector

- vector 선언 방법

```rust
  let v: Vec<i32> = Vec::new();
  // or
  let v = vec![1, 2, 3];
```

- https://doc.rust-lang.org/stable/book/ch08-01-vectors.html

### move_semantics

#### 소유권 규칙
- 러스트에서, 각각의 값은 소유자 (owner) 가 정해져 있다.
- 한 값의 소유자는 동시에 여럿 존재할 수 없다.
- 소유자가 스코프 밖으로 벗어날 때, 값은 버려진다 (dropped).

#### 변수의 스코프

```rust
    {                      // s는 아직 선언되지 않아서 여기서는 유효하지 않습니다
        let s = "hello";   // 이 지점부터 s가 유효합니다

        // s로 어떤 작업을 합니다
    }                      // 이 스코프가 종료되었고, s가 더 이상 유효하지 않습니다
```

#### String 타입

```rust
    {
        let s = String::from("hello"); // s는 이 지점부터 유효합니다

        // s를 가지고 무언가 합니다
    }                                  // 이 스코프가 종료되었고, s는 더 이상
                                       // 유효하지 않습니다.
```

### structs

- 구조체는 각각 다른 타입을 가질 수 있다
```rust
// 구조체 선언
struct User {
    active: bool,
    username: String,
    email: String,
    sign_in_count: u64,
}

// 구조체 인스턴스 생성
fn main() {
    let mut user1 = User {
        active: true,
        username: String::from("someusername123"),
        email: String::from("someone@example.com"),
        sign_in_count: 1,
    };

    user1.email = String::from("anotheremail@example.com");
}
```

### enums

- enums 정의
```rust
enum IpAddrKind {
    V4,
    V6,
}
```

- enums 값. 
- 아래처럼 IpAddrKind 의 두 개의 배리언트에 대한 인스턴스를 만들 수 있다.
- 열거형을 정의할 때의 식별자로 네임스페이스가 만들어져서, 각 배리언트 앞에 이중 콜론(::)을 붙여야 한다
```rust
let four = IpAddrKind::V4;
let six = IpAddrKind::V6;
```

#### Option

```rust
enum Option<T> {
    None,
    Some(T),
}
```

#### match

- match 표현식이 실행될 때, 결괏값을 각 갈래의 패턴에 대해서 순차적으로 비교한다. 만일 어떤 패턴이 그 값과 매칭되면, 그 패턴과 연관된 코드가 실행되고, 매칭되지 않는다면, 다음 갈래로 실행된다.

- 각 갈래와 연관된 코드는 표현식이고, 이 매칭 갈래에서의 표현식의 결과로써 생기는 값은 전체 match 표현식에 대해 반환되는 값

```rust
enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter,
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => {
            println!("Lucky penny!");
            1
        }
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}
```

#### if let
```rust
let config_max = Some(3u8);
if let Some(max) = config_max {
    println!("The maximum is configured to be {}", max);
}

let config_max = Some(3u8);
if let Some(max) = config_max {
    println!("The maximum is configured to be {}", max);
}

```