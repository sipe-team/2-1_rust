# Week 1 Submission

![week1-progress.png](./images/week1-progress.png)

## TIL

ruslings는 rust 언어에 대한 간단한 퀴즈를 풀며 문법을 익힐 수 있도록 도와주는 학습 도구다. 처음부터 문법을 공부해도 좋지만, 퀴즈를 풀며 관련 내용을 학습하는 방법도 꽤 좋아보인다. 퀴즈를 풀다가 hint 에 나오는 관련 내용을 학습할 때 아래 문서를 찾아보면 좋다.

**rust 공부를 위한 학습 자료**

- [The Book](https://doc.rust-lang.org/book/index.html)
- [Rust By Example](https://doc.rust-lang.org/rust-by-example/index.html)

### Installation

**MacOS 기준**

1. 터미널에서 실행

```
curl -L https://raw.githubusercontent.com/rust-lang/rustlings/main/install.sh | bash
```

2. rustlings clone

```
# find out the latest version at https://github.com/rust-lang/rustlings/releases/latest (on edit 5.6.1)
git clone -b 5.6.1 --depth 1 https://github.com/rust-lang/rustlings
cd rustlings
cargo install --force --path .
```

이때, cargo가 설치가 안되어 있다면 [cargo](https://doc.rust-lang.org/cargo/getting-started/installation.html)를 설치하자.

```
curl https://sh.rustup.rs -sSf | sh
```

3. rustlings 사용법

- `rustlings watch` : 코드를 계속 watch 하면서 수정 사항을 바로 컴파일하여 에러를 잡아준다. 문제 풀 때 무조건 켜는게 좋다.

- `rustlings run myExercise1` : /exercises/myExercise1 디렉토리 안에 있는 문제를 컴파일 한다.

- `rustlings hint myExercise1` : hint를 보여준다.

그 밖의 명령어는 문제를 풀어보면서 익히며 추가할 예정..!

### 1. intro

### 2. variables

- Rust에서 변수는 기본적으로 `불변(immutable)`이다.
- 변수가 불변일 때, 한 번 값이 이름에 할당되면 그 값을 변경할 수 없다.
- (⭐️) 변수 이름 앞에 `mut`을 추가하여 `가변(mutable)`으로 만들 수 있다.

```rust
let x = 5;      // 불변 변수
let mut y = 5;  // 가변 변수
y = 6;          // 값 변경 가능
```

```rust
// variables4.ts
fn main() {
    let mut x = 3;
    println!("Number {}", x);
    x = 5; // don't change this line
    println!("Number {}", x);
}

```

- [Variables and Mutability](https://doc.rust-lang.org/book/ch03-01-variables-and-mutability.html)

### 3. functions

- `fn` 키워드 사용하여 함수 선언
- `pub fn` 은 public 인가보다
- `->` 를 사용하여 return value type 지정

```rust
fn is_even(num: i32) -> bool {
    num % 2 == 0
}
```

### 4. if

- 괄호를 작성하지 않아도 된다.
- (⭐️) `let` 구문에서 `if` 구문 사용 가능하고, if 의 결과를 할당 가능하다. return 이나 ; 필요 없이 return 하는 값만 명시한다. if 구문의 결과값이 반환되어 let에 할당되므로!

```rust
pub fn animal_habitat(animal: &str) -> &'static str {
    let identifier = if animal == "crab" {
        1
    } else if animal == "gopher" {
        2
    } else if animal == "snake" {
        3
    } else {
        0
    };

    let habitat = if identifier == 1 {
        "Beach"
    } else if identifier == 2 {
        "Burrow"
    } else if identifier == 3 {
        "Desert"
    } else {
        "Unknown"
    };

    habitat
}
```

### 그외

#### Scalar Types

- `Signed integers`: i8, i16, i32, i64, i128 and isize (pointer size)
    - 부호가 있는 정수 (음수, 0, 양수)
- `Unsigned integers`: u8, u16, u32, u64, u128 and usize (pointer size)
    - 부호가 없는 정수 (0, 양수)

- `Floating point`: f32, f64
- `boolean` : true, false
- `char`: a (4 bytes each)


### 느낀점
새로운 언어를 오랜만에 배우는데, 예전에 언어 공부를 할 때의 기억이 새록새록 떠올랐다. rustlings와 함께 공부하니 코테푸는 것 같이 재미있었고 더 깊게 공부하고 싶어지는 생각도 들었다. 아직 가벼운 문법 수준의 학습이지만, 얼른 학습해서 rust로 코드를 짜보고 싶어졌다. 실제로 여러 오픈소스에서 내가 자주 사용하는 javascript의 한계점을 보완하여 rust로 짠 코드들이 많은데 어떤 점이 다른지 비교해보고 싶고 어떻게 짜여졌는지 분석해보고싶다. 다음 시간에는 좀 더 시간을 투자하여 꼼꼼히 공부해봐야겠다!