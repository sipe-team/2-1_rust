## Week 4 Result

![week4_result.png](images/week4_result.png)

## What I learned


### ERROR

#### 복구 가능한 에러 핸들링

* Result Type
  * Result Type은 정상적으로 수행 되었을 경우 T 타입의 값을 리턴하고, 에러 발생시에는 E 타입의 값을 리턴한다.
```rust
// Result 열거형 타입의 정의
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```
```rust
pub fn test(status: String) -> Result<String, String> {
    if status.is_empty() {
        Err("It's Error".to_string())
    } else {
        Ok("It's OK".to_string())
    }
}
```

* 호출자에게 에러 전파
  * 본인이 에러를 처리하는 것이 아닌 상위 호출자(Caller)에게 에러를 전파
  * 호출자는 필요에 따라 Panic 에러를 일으키거나 다른 처리를 수행한다.
```rust
let mut f = match f {
    Ok(file) => file,
    Err(err) => return Err(err)  // 호출자에게 에러 return
};
```
* `?`
  * 위에서 매번 match를 사용하지 않고 `?` 연산자를 통해 간단하게 처리 가능
  * Result, Option, FromResidual(~~추후 알아보기~~) 타입을 리턴하는 함수에 사용
  * Return 값
    * `Result::Ok`인 경우 Ok 값을 변수에 Return
    * `Result::Err`인 경우 Error를 상위 호출자에게 Return
  * 주의점
    * Return 타입이 `?` 연산자의 Return 타입과 일치해야함
```rust
let mut f = File::open("a.txt")?; // 에러인 경우 호출자에게 에러 return
```
* Error 트레잇
  * 간단하게 Trait이란? (Trait은 추후 자세하게 다룸)
    * 여러 Type의 공통된 행위/특성을 표시한 것 (다른 언어의 Interface와 비슷한 개념)
  ```rust
  // catch-all error types like `Box<dyn error::Error> isn't recommended for library code
  ```
* Result 다루기
  * `unwrap()`
    * Result 타입에서 `Ok` 값을 안전하게 추출
      * Option 타입에서는 `Some` 값을 안전하게 추출
    * 값의 존재 여부를 검사하지 않고, 값을 강제로 추출함
      * 따라서, 값이 없는 경우(`None`, `Err`)에는 panic!을 발생 = `Err`가 발생하는 상황도 버그로 간주하여 고려하겠다는 의미
  ```rust
  // catch-all error types like `Box<dyn error::Error> isn't recommended for library code
  
  let x: Result<u32, &str> = Ok(2);
  assert_eq!(x.unwrap(), 2);
  ```
  ```rust
  // panics example (with `emergency failure`)
  let x: Result<u32, &str> = Err("emergency failure");
  x.unwrap();
  ```
  * `map`
    * Result<T, E>를 Result<U, E>로 변환
    * 에러가 발생했다면 아무런 일도 하지 않고, 성공했다면 성공한 결과를 다른 타입으로 변환시켜줌 (오류 검사를 나중으로 ‘미룰’ 수 있다는 점)
    ```rust
    let line = "1\n2\n3\n4\n";
  
    for num in line.lines() {
      match num.parse::<i32>().map(|i| i * 2) {
        Ok(n) => println!("{n}"),
        Err(..) => {}
      }
    }
    ```
    * `map_err`
      * Result<T, E>를 Result<U, F>로 변환
      * `map`에 대한 기능을 Err(E)에 수행 (`?`의 암시적인 `.into()` 대신 수동으로 E를 변환하고 싶을 때 같이 사용)
      ```rust
      fn stringify(x: u32) -> String { format!("error code: {x}") }

      let x: Result<u32, u32> = Ok(2);
      assert_eq!(x.map_err(stringify), Ok(2));
      
      let x: Result<u32, u32> = Err(13);
      assert_eq!(x.map_err(stringify), Err("error code: 13".to_string()));
      ```

  
### Generics

* Generic이란
  * 보통 함수, 메서드, struct, enum에 대해서는 구체적인 타입을 정의하는데 이를 대신하여 임의의 타입을 받을 수 있게 하는 것
  * <T>와 같은 타입 파라미터를 지정하여 정의할 수 있다.
  * 중복된 코드 작성 방지 및 유연한 코드 작성을 기대할 수 있다.
```rust
// pt1에서는 구조체를 'Point<i32>'로 사용하고
// pt2는 구조체를 'Point<f64>'로 사용할 것이다.
struct Point<T> {
    x: T,
    y: T
}
 
fn main() {
    let pt1 = Point { x: 1, y: 1 };
    let pt2 = Point { x: 2.0, y: 2.0 };
}
```
* 메서드 구현
  * impl<T> 와 같이 impl 뒤에 제네릭 타입 파라미터 T를 지정
  * 구조체(Point<T>)에 대한 메서드를 추가
```rust
struct Point<T> {
    x: T,
    y: T
}
 
impl<T> Point<T> {
    fn get_x(&self) -> &T {
        &self.x
    }
}
 
fn main() {
    let p = Point { x: 2.0, y: 2.0 };
    println!("{}", p.get_x());    
}
```
* 성능
  * 컴파일 타임에 구체적인 타입(concrete type)이 넣어지므로, 런타임 성능에 어떠한 영향도 미치지 않는다.


### Trait

* Trait이란
  * 여러 타입(type)의 공통된 행위/특성을 표시한 것
  * 다른 프로그래밍 언어의 Interface와 비슷한 개념이다.
  * trait의 키워드를 사용한다.
* Trait 선언하기
  * trait 블럭 안에 공통 메서드의 원형을 가짐
  * 실제로 구현된 것이 아닌, 메서드 이름 / 파라미터 / 리턴타입 만을 표시
```rust
trait Draw {
    fn draw(&self, x: i32, y: i32);
}
```
* Trait 구현하기
  * trait는 공통된 특성을 갖는 타입(들)에 붙여 사용될 수 있다.
  * `impl {trait_name} for {type_name}`로 정의 후 내부에서 메소드를 구현하면 된다.
  * 만약 trait에 여러 메서드가 존재한다면 (+ defaults trait 구현이 없다면) 모든 trait 메소드를 해당 타입에서 구현해야한다.
```rust
struct Rectangle {
  width: i32,
  height: i32
}

impl Draw for Rectangle {
  fn draw(&self, x: i32, y: i32) {
    let x2 = x + self.width;
    let y2 = y + self.height;
    println!("Rect({},{}~{},{})", x, y, x2, y2);
  }
}

struct Circle {
  radius: i32
}

impl Draw for Circle {
  fn draw(&self, x: i32, y: i32) {
    println!("Circle({},{},{})", x, y, self.radius);
  }
}
```
* Trait 메소드 사용하기
  * 트레이트를 갖는 서로 다른 타입들을 파라미터로 받아들여 공통 메서드 사용 가능하다.
  * `impl {trait_name}`의 표현으로 파라미터에 trait를 받아들인다.
```rust
fn draw_it(item: impl Draw, x: i32, y: i32) {
    item.draw(x, y);
}
 
fn main() {
    let rect = Rectangle { width: 20, height: 20 };
    let circle = Circle { radius: 5 };
 
    draw_it(rect, 1, 1);
    draw_it(circle, 2, 2);
}

/*
// traits 리턴 타입 명시
fn draw_basic_circle() -> impl Draw {
    Circle { radius: 1 }
}
*/
```
* Default Trait 구현
  * 일반적으로 Trait는 메서드를 구현하지 않는다.
  * 필요한 경우에는 Default로 실행되는 구현 추가 가능
```rust
trait Draw {
    fn draw(&self, x: i32, y: i32) {
        println!("draw at {},{}", x, y);
    }
}
 
struct Shape {
    name: String
}
 
impl Draw for Shape {}
```



### Lifetime

* Lifetime이란
  * Rust에서는 모든 참조자는 lifetime을 가진다.
    * 참조자가 유효한 scope를 뜻함
  * 대부분의 경우 타입이 추론되는 경우처럼 lifetime 또한 암묵적이고 추론된다.
  * 제너릭 lifetime 파라미터를 이용해서 관계들을 명시하는 것을 요구
    * lifetime 파라미터는 실제 lifetime에 영향을 미치지 않는다.
    * 단순히, 컴파일러에게 '최소한 파라미터' 만큼의 lifetime을 갖는다는 힌트를 제공
```rust
// Return 값이 x의 참조값일지 y의 참조값일지 알 수 없어서 컴파일 에러를 뱉는다.
fn longest(x: &str, y: &str) -> &str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```
```rust
// lifetime('a)을 명시해주면 해결된다.
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
  if x.len() > y.len() {
    x
  } else {
    y
  }
}
```
* 모든 부분에 lifetime을 명시하기엔 번거로움이 있어 다음 3가지 규칙에 부합하면 생략 가능하다.
  1. 모든 레퍼런스는 lifetime을 갖는다.
  2. 만약 인자로 입력받는 레퍼런스가 1개라면, 해당 레퍼런스의 lifetime이 모든 리턴값에 적용된다.
  3. 함수가 self 혹은 &mut self를 인자로 받는 메소드라면, self의 lifetime이 모든 리턴값에 적용된다.