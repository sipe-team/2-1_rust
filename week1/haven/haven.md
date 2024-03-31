# Week 1 Submission

![result.png](./images/result.png)

# TIL

## Rust의 primitive types

signed는 i prefix
* i8, i16, **i32**, i64
  
unsigned는 u prefix
* u8, u16, u32, u64

할당할 수 있는 메모리의 길이등을 타입으로 지정해야 할때는
* isize, usize

floating numers는
* f32, **f64**

C++의 경우 size_t를 많이 쓰는 편인데 요즘 시스템에서는 대부분 64-bit라서 큰 문제는 없지만 embedded 시스템에 들어가는 경우 간혹가다가 32-bit인 경우가 있어서 메모리 주소 계산할 때 망가지는 경우가 있다. 꼭 메모리 사이즈 할당 및 파일 처리를 위한 seek, tell 등 메모리 연산이 필요한 코드에서는 usize를 쓰자.

그리고 default 값이 **i32, f64**라는 점도 유의하자. Floating point 연산을 많이 해야하는 vector 연산등의 경우에는 4x4 Transformation Matrix는 8 * 16 = 108 byte라는 큰 크기를 가지게 되며, 3차원 vector 또한 8 * 3 = 24 byte나 된다. 물론 정밀도를 위해서 matrix는 4x4 double을 보통 사용하지만 vector의 경우 메모리 용량을 굳이 그정도의 자세한 정밀도를 위해 trade-off 할 필요가 없는 경우가 제법 많으니까.

## Tuple과 Array 문법

문법이 꽤 편하고 좋다고 생각했다. subarray indexing도 파이썬처럼 range 베이스 문법도 지원해주는 건 편하다.

* Array

```rust
let arr1 : [i32; 5] = [1, 2, 3, 4 ,5]; // 5 elements in fixed size array
let arr2 : [i32; 5] // Compile error; uninitialized
let arr3 = [3; 10]; // 10 elements filled with 3

let subarr1 = arr1[0..2] // [1, 2]
```

* Tuple

```rust
let tuple1 : (i32, i32) = (100, 200);
let x = tuple1.0;
let y = tuple1.1;
```

좀 더 직관적이고 짧은 코드 같기도...

C++에서는 코드가 약간 좀 더 길다.

```c++
std::tuple<int32_t, int32_t> tuple1 = std::make_tuple<>(100, 200);
int32_t x, y;
std::tie(x, y) = tuple1;

const auto& [x, y] = tuple1; // Since c++ 17
```

## 타입의 강제성

Rust는 언어의 설계 목적인 안정성을 지키기 위해 가변젹인 타입등은 전혀 지원하지 않고 유교보이 급의 보수적인 타입 제한을 자랑한다.

C++에서도 함수와 변수, iterator 등에 항상 const를 쓰는 것이 아주 안정적인 defensive programming으로 여겨지고, 그것에 매우 동감하는데 rust에서도 기본적으로 변수의 선언이 let이고 (사실상 변수가 아닌 상수가 된다), 가변할 수 있는 변수는 mutable을 넣어 let mut를 해야한다.

즉, 개발자로 하여금 진짜로 runtime중에 변해야만 하는 것인지 한 번 더 생각해보라는 철학을 엿볼 수 있다.

## 함수에서 return 생략

```rust
fn is_even(num: i32) -> bool {
    num % 2 == 0
}
```

뭔가 return이 없다는게 lambda 함수처럼 보이게 하려고 생략한건진 모르겠지만 거의 대부분의 함수에서 return을 써오던 버릇 때문인지 너무 어색하게 보인다...

하지만 그래도 명확한 룰이 있는 것은 lambda 처럼 return만 하기 위해 존재하는 pure function은 세미콜론을 없이 작성해야하는 규칙이 있으니 익숙해지면 문장이 간결해 질 것 같기도 하다.

## 마음에 들었던 것

* 모든 것에서 개발자를 귀찮게 한다는 점, 그리고 그만큼 신중한 코드를 작성하고 그 코드에 대한 이유를 스스로 한 번 더 생각하도록 만들어 준다는 점.
* cargo, docs 부터 개발자를 위한 자료가 제법 잘 구성되어 있다는 점을 꼽고 싶다.
* 아래와 같은 괴랄한 문법이 보이기 시작하는데 조금 무섭다. 찾아보니 string과 관련해서 아주 많은 타입이 있는 것을 보고 조금 겁먹었다.

```rust

pub fn animal_habitat(animal: &str) -> &'static str {
    animal
}
```