# Week 3 Submission

![result.png](./images/result.png)

# TIL

## String

string에 아주 다양한 함수가 있는데, 역시 어느 언어에서나 문자열이 가장 문제인 것 같다.

기본적으로 무조건 UTF-8으로 저장하도록 설계해놨고, byte vector로 구현되어 있어서 일관성을 유지하려고 했던 노력이 엿보인다. 또한 최신 트렌드에 맞게 char type 자체를 4바이트로 만들어서 Unicode 그 이상의 모든 것을 표현 할 수 있도록 한 점이 큰 장점이라고 볼 수 있을 것 같다.

`문자열` 이라는 것 자체가 대부분의 언어에서 compile 타임에 선언된 constant string literal과 가변이 가능한 string 클래스로 보통 구현되는데 이는 rust에서도 동일하게 볼 수 있다. 대신 그 문자열 데이터에 대한 소유권과 **string slice라고 불리는 string reader pointer**를 가지고 필요에 따라 사용할 수 있도록 한 점이 핵심 포인트라고 할 수 있다. 물론 이 역시 rust에서만 새로운 개념은 아니고 많은 언어들에도 채용하고 있는 개념이다. C++에서는 최근에서야 `std::string_view`라는 녀석이 생겼고, 아직도 production 코드에서 쉽게 보긴 힘든 편이다.

뭐 다른 string 함수들은 흔히 쓰는 이름들과 비슷하니 소유권과 관련된 함수만 정리해본다.

#### [to_owned](https://doc.rust-lang.org/std/string/struct.String.html)

새로운 인스턴스를 생성한다. 보통은 복제를 한다.

#### [into](https://doc.rust-lang.org/std/convert/trait.Into.html#tymethod.into)

이 Trait을 구현하고 있어서 원하는 타입의 `from()`을 사용 할 수 있다.

#### !format

String은 아무래도 출력에 가장 많이 쓰이는 만큼 formatting도 중요한데, 아주 당연하게도 !format 매크로는 참조만 하기 때문에 string slice를 매개변수로 받는다.

#### Concatenation

가장 중요한 부분인데, 많은 언어에서는 + 연산이 left operand와 right operand를 가지고 아에 새로운 문자열을 만들어낸다.

참고: [Performance Comparison Between Different Java String Concatenation Methods](https://www.baeldung.com/java-string-concatenation-methods)

```java
String a = "Hello, ";
String b = "World!";
String c = a + b;
```

위와 같은 java 코드에서 heap에는 "Hello, ", "World!", 그리고 "Hello, World!" 까지 존재하는 것이 대부분 언어에서 공통적인 편이다. 그래서 GC의 처리를 받기 전에는 heap이 계속 불어나는 점이 있으며, 위 글의 [Batch Processing](https://www.baeldung.com/java-string-concatenation-methods#2-batch-processing)에서 overflow가 나는 것은 당연한 일.

Rust에서는 소유권 모델이 언어의 핵심축에 있다보니 애초에 + 연산을 함부로 하지도 못한다.

```rust
let a = String::from("Hello, ");
let b = String::from("World!");
//let c = a + b;  // 컴파일 에러
let c = a + &b;
```

add 함수의 시그니쳐 자체가 아래처럼 string slice만 받게 되어 있기 때문이며 반환 타입에서 볼 수 있듯 함수 호출의 결과, 원본 데이터가 옮겨진다. 즉 동일한 데이터가 중복하여 heap에 존재하지 않으며 그로인해 애초에 GC에 의존하는 코드가 작성되지 않는 것.

```rust
fn add(self, s: &str) -> String {}
```

그래서 Java의 StringBuilder나 C의 std::stringstream 등을 쓰는 것이 그다지 강제되진 않는 것 같다.

## Modules

언어에서 풍기는 느낌에 고급진 C++이 있는게 module 관리 부분에서도 느껴진다. 처음에는 Binary Crate, Library Crate 할 때 뭔가 와닿지 않았는데 사실상 header가 하는 역할이 이 crate와 동일하다. (자꾸 crate말고 create 습관적으로 치는거 진짜 킹받음)

그리고 유교 언어답게 심지어 같은 코드 안에 있는 패키지 끼리도 private이 기본 값이라서 pub을 꼭 써줘야만 사용할 수 있다는 점 아주 맘에 든다.

자바나 C++에서는 default accessibility가 pacakge 레벨인데, 이게 생각보다 좀 애매하고 간혹가다 없으면 불안하게 만드는게 있기 때문. 결국 앵간하면 private & public을 가장 많이 쓰는데 예외적인 경우가 있기도 하다. 부모가 자식에게는 보여주고 싶은 것도 있기 때문.

Rust에서는 pub enum으로 이를 구현하며, [pub accessbility 문서](https://doc.rust-lang.org/reference/visibility-and-privacy.html#pubin-path-pubcrate-pubsuper-and-pubself)에서 볼 수 있듯 public의 범위를 정해줄 수 있어 필요에 따라 유연하게 대처할 수 있다. 

* default pub = 완전 공개
* private = pub(self), pub(in self)와 같다.
* protected = 없음.

궁금해서 찾아봤는데 기존의 상속 중심의 객체지형 언어보다 현대 언어에서는 [composition over inheritance](https://www.reddit.com/r/rust/comments/o4oxyj/comment/h2igip3/?utm_source=share&utm_medium=web3x&utm_name=web3xcss&utm_term=1&utm_content=share_button)라고 한다. 확실히 공감가는 부분이기도 한게 inheritance가 가져다주는 polymorphism도 좋지만 composition으로도 해결이 충분이 가능한 경우가 더 많아서 인 듯 하다. method overriding이 가져다주는 크고 작은 실수도 포함해서.

## HashMap

딱히 특별할 것은 없는데 Quality of Life 함수로 `or_insert`를 지원해주는 건 진짜 행복하다.

## Options

Some<T>와 None element를 가지는 enum type. 현대 언어에서 가장 많이 볼 수 있으며 아주 좋아하는 문법 특성중 하나라서 반가웠다.

딱 한가지를 기록하고 넘어가자면 

* ref keyword : 값을 바인딩할 떄, reference로 대여함
=> 그 말은 즉 ref를 쓰지 않으면 역시나 move가 이뤄진다는 것을 확인할 수 있다.

```rust
match y {
    Some(p) => println!("Co-ordinates are {},{} ", p.x, p.y),
    _ => panic!("no match!"),
}

// 예제에 있는 함수에서는 Option 변수 y 자체를 반환시켰다.
// 뭔가 조금 모호했어서 아에 동일한 함수를 두번 호출해봤다.
match y {
    Some(p) => println!("Co-ordinates are {},{} ", p.x, p.y),
    _ => panic!("no match!"),
}
```