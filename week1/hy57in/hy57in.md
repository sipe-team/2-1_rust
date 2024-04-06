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

Rust에서 변수는 기본적으로 `불변(immutable)`이다.
변수가 불변일 때, 한 번 값이 이름에 할당되면 그 값을 변경할 수 없다.
변수 이름 앞에 `mut`을 추가하여 `가변(mutable)`으로 만들 수 있다.

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

### 4. if
