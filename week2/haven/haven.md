# Week 2 Submission

![result.png](./images/result.png)

# TIL

## Collections

역시 시스템 프로그래밍 언어가 가장 중심에 있어서인지 C++과 유사한 문법들이 제법 있는 것 같다. 예를 들면 *를 가지고 iterator instance를 dereferencing 하는 것

```rust
for element in v.iter_mut() {
    *element = *element * 2;
}
```

## Lambda

람다식이 사실 이렇게 깔끔한건 본적이 많이 없는 듯.

```rust
v.iter().map(|elem| {elem * 2}).collect();
```

C++, 자바나 python이 그렇다고 이상한건 아니지만 뭔가 | 기호가 =>, -> 이런 것들과는 달라서 깔끔해 보였을지도 모르겠다.

## Ownership and move semantic

가장 꽃이자 잘 알아야 하는 부분이라고 생각한다. C++에서는 `std::move()`라는 것을 통해서 어느정도 비슷한 기능을 지원하고 있었다.

하지만 가장 큰 차이점이라면 c++은 아주 오래전에 설계된 언어이며, 기본적으로 default constructor를 사용하여 return하고 대입하는 언어이기 때문에 `const &` 문법을 알고 모르고의 차이가 그 사람의 숙련도를 대변하는 정도라고 생각한다. (실제로 아주 큰 데이터를 지속적으로 복사하거나 그대로 리턴하는 경우에는 정말 많은 메모리 복사가 이뤄지기 때문이다)

1. Default는 무조건 함수 호출이나 다른 변수에 대입하는 경우에는 메모리의 소유권이 이동한다.
2. 참조만(대여만) 하고 싶다면 & 연산자를 사용하여 일시적으로 빌려올 수 있음. 하지만 모든 것의 default behavior가 const이듯 & 연산자에도 `mut`를 붙이지 않는다면 변경이 기본적으로 불가능하다.
3. `immutable reference`는 얼마든지 생성되어도 좋지만 `mutable reference`가 하나라도 존재한다면 동일한 스코프내에서 사용되지 않은 reference가 하나 이상 존재할 수 없다. 

그리고 위의 모든 규칙이 compile 타임에 잡힐 수 있다는 것이 가장 마음에 들었다. 왜 시스템 프로그래밍 하는 사람들이 매력을 느끼는지 알 수 있는 부분이다.

성능 & 최적화 또는 low level에 가까운 코드를 작성해야 할 수록 다른 이의 실수가 치명적인 영향을 주기도 하고 메모리와 많이 싸우게 되는데 이러한 언어적인 특성은 runtime에 오기도 전에 static time에서 잡아주기 때문에 lint단에서 걸러지는 것만으로도 QA 테스트의 난이도를 기하급수적으로 낮춰줄 수 있다. C++에서는 어떻게 해서든 `const auto&` 를 어디서든 쓰도록 하고 있으나 그것이 기본 값은 아니다보니 여전히 실수 또는 해프닝이 생겨나기 마련이고 퀄리티가 낮은 코드가 생겨나는 것이 비일비재하기 때문이다.

## Struct

간결한 문법들을 잘 지원하는 것 같다.

```rust
struct VectorClassicStruct {
  x: f32,
  y: f32,
  z: f32,
}

struct VectorTupleStruct (f32, f32, f32);
```

그리고 진짜 편하게 구성해놓은 문법이 struct update syntax 인 것 같다. struct의 default가 named key 방식이다보니 가능한 것 같은데, 복사할 값들 중 변경이 필요한 것만 명시해주고 slice operator를 동일한 방식으로 사용하는 것... 

아름답다. C++에 올바른 느낌으로 현대 언어들의 컨셉이 잘 맞춰진 느낌이랄까

물론 C++에도 kargs 같은게 있긴 한데 좀 지저분한 감이 크고, 그건 struct에서 쓸 수도 없다. 항상 struct를 대입연산자로 복사한 다음에 직접 수정해야 하거나 생성자를 별도로 정의하는 방식이 쓰였을 뿐...

```rust
struct Color {
    r: u8,
    g: u8,
    b: u8,
}

fn main() {
    let color_a = Color { r: 0, g: 255, b: 0 };
    let color_b = Color { r: 255, ..color_a };
    
    println!("{}, {}, {}", color_a.r, color_a.g, color_a.b);
    println!("{}, {}, {}", color_b.r, color_b.g, color_b.b);
}
```

## enum

이렇게 값을 다양하게 가질 수 있는 enum은 좀처럼 적응이 안되는 문법이긴 한데, swift에서도 본적은 있었다. 그래도 여전히 이것 자체가 잘 와닿진 않는 것 같다. 그도 그럴게 enum은 자고로 하나의 그룹으로 묶인 self-stated constant에 불과한 편이었으니..

```c++
enum class Coin : uint_8 {
    Penny,
    Nickel,
    Dime,
    Quarter
}
```

하지만 쓰다보면 c++에서 불편함을 느끼고, 제발 있었으면 하는 기능들이 현대 언어에서는 rust의 enum과 비슷한 방식으로 해결 한 것 같다.

```rust
enum Coin {
    Penny,
    Nicket,
    Dime,
    Quarter,
}

impl Coin {

}
```

위처럼 implementation을 따로 빼놓다보니 enum의 타입과 자체적으로 가지고 있는 함수를 충분히 분리 할 수 있고, indentation도 한 단계 줄어드니 코드를 유지보수 하는데 제법 많은 이점이 있을 것으로 생각된다.

그리고 switch와 비슷하게 생겼지만 더 간결해진 `match`를 통해서 깔끔하게 정리 할 수 있는 것도 맘에 든다.

더 나아가서는 Option도 enum으로 구현되고, if let 문법도 같이 있다는 것은 마치 Dart, swift에서 보던 문법을 C++에서 예쁘게 짜놓은 느낌이달까?

## Worth noting

`if let` 문법은 사실 rust의 언어 철학과는 아주 살짝 위배되는 syntax sugar이다. 하지만 너무 딱딱하고 쓰기 어려운 언어이기 보다는 적당한 선에서 생산성 및 유지보수 & 코드 라인수를 컨트롤 해주고, 그와 동시에 안정성을 챙긴다는 점에서는 높게 살만한 언어 설계 방침이라고 느껴졌다.
