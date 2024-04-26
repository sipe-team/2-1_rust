# Week 4 Submission

![result.png](./images/result.png)

# TIL

## Error Handling

책에 있는 이 문장이 꽤 인상적이었다.

>러스트에는 예외 처리 기능이 없습니다.\
>하지만 복구가 가능한 에러를 위한 Result<T, E> enum과 복구 불가능한 상황에 프로그램을 강제 종료하는 panic! 매크로를 지원합니다.

이 일관된 언어의 컨셉이란... 이런 식으로 에러 처리마저도 확실하게 binary하게 구분하는 언어는 경험 해본 어느 언어에서도 본 적은 없었다. 뭐 가장 비슷 한 것은 어쩌면 node.js에서 response chaning 하는 거?? 대부분의 언어가 try-catch, try-except 같은 식으로 Try-Throw 형태의 에러 처리가 기본이었다보니 언제나 비슷한 컨셉으로 비슷한 코드를 짰었는데 되려 언어적으로 확실하게 상황을 구분시켜 버리니까 

* 이건 죽어야만 하는 코드인가?
* 이건 되살릴 수 있어야 하는 코드인가?

위 둘을 한 번은 더 고민하게 만든다는 점이 큰 특징인 것 같다. 최소한 한 번은 더 고민하게 만들었으니 `그만큼 로직이 탄탄해진다는 장점`이 보답으로 돌아온다.

```rust
fn main() {
    let result = File::open("result.csv");
    
    let file = match result {
        Ok(f) => f,
        Err(e) => panic!("Problem opening the file {:?}", e),
    };
}
```

enumeration이니 Option처럼 match 문법을 사용해서 코드를 표현할 수 있고 우리가 좋아하는 클로져(람다)를 이용해서 더 간결하고 깔끔하게 표현할 수 있다. 역시 들여쓰기는 없어야 제맛

```rust
fn main() {
    let file = File::open("result.csv").unwrap_or_else(|e| {
        panic!("Problem opening the file {:?}", e);
    });

    // 혹은 그냥 알아서 panic을 뱉도록 할 수 도 있다.
    let file2 = File::open("result.csv").unwrap();
    
    // 그것보다 좋은 건 그래도 메세지는 직접 출력해주는 것
    let file3 = File::open("result.csv").expect("The CSV file must exist before parsing.");

    // 이걸 싹다 줄여버리면...
    // 물론 아래 줄은 컴파일 되지 않는다. main 함수에서 직접 ?를 사용했기 때문인데 ? 언산자 자체가 match를 통해 값을 체크하고 반환을 하도록 되어 있기 때문. ? 연산자를 사용하면 panic과 동시에 함수를 빠져 나와야 하는데 이때 return 타입은 Result 또는 Option이어야 하기 때문이다.
    
    let file4 = File::open("result.csv")?;
}
```

위 예시 코드에서 가장 마지막 줄이 조금 좋으면서도 별로인 부분인데, 축약도 너무 줄이고 줄여서 코드의 읽기 난이도를 높이는 문법이지 않나 싶다.

그래도 존재 자체에는 의미가 있는데, `FromResidual` trait을 구현하면 하나의 함수 안에서 코드를 추가적으로 많이 작성할 필요 없이 바로 bypassing 해서 종료 시킬 수 있다.

```rust
fn last_char_of_first_line(text: &str) -> Option<char> {
    text.lines().next()?.chars().last()
}
```

이 코드를 match를 꼭 써야하며 라인 수가 더 늘어날 수 있었기 때문에 코드의 간결성이 떨어질 수 있다. 하지만 그 간결성의 trade-off는 확실성과 명확성 아닐까?

exercise 중에 해답이 두개인게 있는데, 그 해답을 같이 첨부해본다. 아래와 같은 상황에서는 ? 연산자를 쓰는 것이 오히려 더 이득이다.

```rust
pub fn total_cost(item_quantity: &str) -> Result<i32, ParseIntError> {
    let processing_fee = 1;
    let cost_per_item = 5;
    let qty = item_quantity.parse::<i32>();
    
    // parse()의 결과가 Result<>이기 때문에 match를 가지고 반환한다.
    match qty {
        Ok(qty) => Ok(qty * cost_per_item + processing_fee),
        Err(e) => Err(e),
    }
}
```

```rust
pub fn total_cost(item_quantity: &str) -> Result<i32, ParseIntError> {
    let processing_fee = 1;
    let cost_per_item = 5;
    // ? 연산자로 바로 Result<i32, ParseIntError>를 반환한다.
    let qty = item_quantity.parse::<i32>()?;
 
    Ok(qty * cost_per_item + processing_fee)
}
```

만약 main 함수안에서도 ? 연산자를 쓰고 싶다면 main이 Result를 반환하게 해준다.

```rust
fn main() -> Result<(), Box<dyn Error>> {
    let result = text.lines().next()?;

    Ok(());
}
```

## Generics

뭐 섹션을 나누어서 정리를 해야 하나 싶을 정도로 이미 코드에서 한참 많이 쓰고 기본적인 개념이라 스킵.

그나마 언어적인 특성으로 인해 한가지 남긴다면 struct 선언과 구현체를 구분해서 쓰기 때문에 Generic 이라는 것을 동네 방방곳곳 알려야 한다는 점.

```rust
struct Wrapper<T> {
}

impl<T> Wrapper<T> {
}
```

## Traits

내용을 읽어 보면서 이거 "규약"을 정한다는 의미가 interface와 동일한 개념인가? 하는 생각이 계속 들었는데 얼추 비슷한 역할을 한다.

문법상 차이라면 자바에서는 클래스 이름뒤에 implements를 붙이는 방식이고, rust는 impl...for을 사용한다. 한편으로 드는 생각은 이렇게 하면 해당 struct가 어느 trait이 있는지 한 눈에 보기 힘들지 않을까 하는 생각이 든다.

```rust
pub trait Summary {
    
}

struct Article {
    
}

impl Summary for Article {

}
```

### Trait을 사용하도록 제한하기

그리고 rust는 C++에서처럼 trait 자체가 메소드 정의도 할 수 있는데, 뭐 이건 장단점이 있을 것 같다.

매개변수로 trait을 구현하는 타입을 받으려면 `trait bound`를 사용한다.   

```rust
pub fn notify<T: Summary>(item: &T) {

}
```
`
### Syntax Sugar : impl trait

그리고 이는 syntax sugar로 `impl trait`가 주어진다. 훨씬 알아보기 쉽다.

```rust
pub fn notify(item: &impl Summary) {
    
}
```

대신 단점도 명확히 존재하는데, syntax sugar 이다보니 제약을 제대로 주지 못하는 경우도 생긴다. 아래처럼 Summary Trait을 구현한다고 하더라도 완전히 동일한 타입 하나만 사용하도록 강제하기는 힘들어지기 때문.

```rust
// 아래 두 코드는 동일하다.
pub fn notify(item1: &impl Summary, item2: &impl Summary) {
}

pub fn notify<T: Summary, F: Summary>(item1: &T, item2: &F) {
}
```

### Multi trait bounds

조금 어색한 문법이긴 한데 +를 사용해서 여러개의 trait을 동시에 구현하는 타입만 받도록 할 수 있다. 또는 Where를 사용해서 여러 개의 trait bound를 다른 줄로 뺄 수 있다.

```rust
// 아래 세 코드는 동일하다.
pub fn notify(item: &(impl Summary + Display)) {
}

pub fn notify<T: Summary + Display>(item: &T) {
}

pub fn notify<T>(item: &T)
where T: Summary + Display {
}
```

물론 여러 개도 가능하다.

```rust
pub fn notify<T, F>(item: &T, value: &F)
where
    T: Summary + Display,
    F: Error + Debug,
{
}
```

## Lifetime

Move semantic과 ownership에 의해서 생겨난 문법이라 그런지 정말 적응이 잘 되진 않는다. 이전에 공부하는 동안 자주 등장하지 않았던 이유는 단지 struct가 reference가 아닌 일반적인 변수 타입이었기 때문이다. 하지만 참조자도 포함 할 수 있다는 점은 이제 참조와 scope 수명의 문제가 붉어지기 때문에 필수적으로 생겨난 문법이라고 볼 수 있다.

라이프타임이 존재하는 가장 큰 이유는 `dangling reference`를 방지하기 위해서다. 이를 설명하는 하나의 문장은

> 라이프타임 명시는 함수 시그니쳐의 타입들과 마찬가지로 함수에 대한 계약서의 일부가 됩니다.

즉, 라이프타임을 타입에 명시해줌으로서 해당 타입들이 동일한 스코프 관계를 갖는다고 명시해주는 것이며 rust의 `borrow checker`가 충분히 검토할 수 있도록 해주는 것이다.

아래와 같은 코드에서 rust는 매개변수 x와 y 변수의 수명이 언제까지 살아 있을 것이며 누가 먼저 해제되고 죽을지 전혀 알 수 없다. 또한 그렇기에 반환을 시도할 때도 어떤 값이 들어갈지 몰라 dangling 이슈를 피하기 어려워진다. 

```rust
fn longest(x: &str, y: &str) -> &str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

따라서 아래 코드처럼 라이프타임이 동일할 것이라고 명시를 해주어 수명이 동일하다고 계약서를 맺고 그에 맞춰 코드를 사용하도록 스스로 강제하게 되는 것이다. 

```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

좀 귀여운 표현은 scope를 벗어나는 것을 컴파일러 메세지에서도 `live longer`로 표현한다 ㅋㅋ
