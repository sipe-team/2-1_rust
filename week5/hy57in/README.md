## error handling

Rust에서 오류는 아래 표와 같이 크게 두 가지 범주로 분류

![](https://miro.medium.com/v2/resize:fit:1400/1*mn1jkegNpeSHfwkmbD1G8A.png)

`Recoverable Error`는 프로그램은 실패한 작업을 재시도하거나 복구 가능한 오류가 발생했을 때 다른 조치를 지정 가능  
`Recoverable Error`로 인해 프로그램이 갑자기 종료되지는 않음  
`UnRecoverable`는 오류로 인해 프로그램이 갑자기 종료됨  
`Recoverable Error`가 발생하면 프로그램을 정상 상태로 되돌릴 수 없음  
`Recoverable Error`는 실패한 작업을 다시 시도하거나 오류를 실행 취소할 수 없음

**_다른 프로그래밍 언어와 달리 Rust에는 예외가 없음_**

Recoverable Error에 대해서는 열거형 Result<T, E>를 반환  
UnRecoverable가 발생하면 패닉 매크로를 호출  
패닉 매크로는 프로그램이 갑자기 종료되도록 함

### panic! macro
`panic!` : 명시적으로 에러를 발생시키는 매크로.
panic! 매크로를 사용하면 프로그램이 즉시 종료되고 피드백을 제공  
프로그램이 unrecoverable 상태일때 사용

panic! 이 발생하면, 프로그램은 되감기 (unwinding)를 시작하는데, 패닉을 발생시킨 함수로부터 스택을 거꾸로 올라가며 데이터를 청소한다는 뜻. 프로그램이 데이터 정리 작업 없이 즉각 종료되는 aborting을 사용할 수도 있음.

Cargo.toml 에 다음과 같이 바꾸면 됨 (릴리즈 모드에 주로 사용)
```rust
[profile.release]
panic = 'abort'
```

### Result<T, E> enum
```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```
Enum Result<T,E>는 recoverable error를 처리하는 데 사용
T는 성공한 경우 Ok 에 반환될 타입, E는 실패한 경우 Err에 반환될 타입을 나타냄

```rust
use std::fs::File;

fn main() {
    let greeting_file_result = File::open("hello.txt");

    let greeting_file = match greeting_file_result {
        Ok(file) => file,
        Err(error) => panic!("Problem opening the file: {:?}", error),
    };
}
```
`File::open` 성공시 Ok, 실패시 Err 반환

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
error.kind()를 이용하여 에러의 종류에 따라 다른 에러를 발생시키는 방식으로 에러 핸들링하기


```rust
use std::fs::File;
use std::io::ErrorKind;

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
중첩 `match` 대신 `unwrap_or_else` 를 사용하면 코드가 깔끔해짐

### unwrap 와 expect
표준 라이브러리에는 열거형인 Result<T,E> 및 Option< T >과 연관된 몇 가지 도우미 메서드 제공
![[Pasted image 20240503205300.png]]

### unwrap
unwrap() 함수는 작업이 성공한 실제 결과를 반환  
작업이 실패하면 기본 오류 메시지와 함께 패닉을 반환

`Result` 값이 `Ok` 라면, `unwrap`은 `Ok` 내의 값을 반환하고,
`Result`가 `Err` 라면 `unwrap`은 `panic!` 호출
```rust
use std::fs::File;

fn main() {
    let greeting_file = File::open("hello.txt").unwrap();
}
```

### expect
패닉이 발생할 경우 프로그램에서 사용자 지정 오류 메시지를 반환

`expect`는 `panic!` 에러 메시지도 선택할 수 있도록 해줌.
`unwrap`은 `panic!`의 기본 메시지가 출력되지만, `expect`는 매개변수로 전달한 에러 메시지를 출력함.
보통 expect가 더 유용!
```rust
use std::fs::File;

fn main() {
    let greeting_file = File::open("hello.txt")
        .expect("hello.txt should be included in this project");
}
```

### error propagating
```rust
use std::fs::File;
use std::io::{self, Read};

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

```rust
use std::fs::File;
use std::io::{self, Read};

fn read_username_from_file() -> Result<String, io::Error> {
    let mut username_file = File::open("hello.txt")?;
    let mut username = String::new();
    username_file.read_to_string(&mut username)?;
    Ok(username)
}

fn last_char_of_first_line(text: &str) -> Option<char> {
    text.lines().next()?.chars().last()
}
```
에러 전파를 위한 숏컷인 `?`를 이용하면 간결하게 작성할 수 있음

- ?는 match 와 거의 같은 방식으로 동작함. Result의 값이 Ok이면 Ok 안의 값이 얻어지고 프로그램 계속 실행, 만약 Err 라면, Err 값 반환됨
- ? 연산자를 사용할 때의 에러 값들은 from 함수를 거침. `from` 함수는 표준 라이브러리 내의 `From` 트레이트에 정의되어 있으며 어떤 값의 타입을 다른 타입으로 변환하는 데에 사용합니다. `?` 연산자가 `from` 함수를 호출하면, `?` 연산자가 얻게 되는 에러를 `?` 연산자가 사용된 **현재 함수의 반환 타입에 정의된 에러 타입으로 변환**함. 어떤 함수가 다양한 종류의 에러로 인해 실패할 수 있지만, 모든 에러를 하나의 에러 타입으로 반환할 때 유용.
- `?`는 `?`이 사용된 값과 호환 가능한 반환 타입을 가진 함수에서만 사용될 수 있음

main은 Result<(), E>를 반환할 수 있음
```rust
use std::error::Error;
use std::fs::File;

fn main() -> Result<(), Box<dyn Error>> {
    let greeting_file = File::open("hello.txt")?;

    Ok(())
}
```
- `Box<dyn Error>`는 ‘어떠한 종류의 에러’를 의미