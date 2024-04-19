# week3

![alt text](./image/image.png)

### strings

```rust
fn main() {
    let word = String::from("green"); // Try not changing this line :)
    // if is_a_color_word(&word) {
    if is_a_color_word(word.clone()) {
        println!("That is a color word I know!");
    } else {
        println!("That is not a color word I know.");
    }
}

// fn is_a_color_word(attempt: &str) -> bool {
fn is_a_color_word(attempt: String) -> bool {
    attempt == "green" || attempt == "blue" || attempt == "red"
}
```

두가지 방법으로 수정이 가능한데, `&word`는 `String`의 참조를 전달하게 되어, `is_a_color_word` 함수에서도 해당 `String`을 참조하여 사용할 수 있게 된다.   
Rust에서는 가능한 경우에는 소유권을 이전하지 않고도 데이터에 접근할 수 있도록 참조를 사용하는 것이 권장. 그래서 `word.clone()`을 통해 main 함수 내에서 `word` 변수의 소유권을 보존하는 것이 좋음.   

문자열 compose 하는 방법

```rust
fn compose_me(input: &str) -> String {
  input.to_string() + " world!"
  format!("{} world!", input)
}
```

- `to_owned()`
  - 메서드는 문자열을 소유하는 `String` 타입으로 변환하는 메서드
- `into()`
  - `into()` 메서드는 Rust에서 소유권을 전환하는 역할을 함. 이 메서드는 원래 값의 소유권을 포기하고, 대신 새로운 타입의 소유권을 얻게 된다. 주로 타입 변환시 사용.
  - `into()` 메서드는 `to_owned()` 메서드와 유사하게 동작하지만, `to_owned()`는 명시적으로 `String` 타입으로 변환하는 데 사용되는 반면, `into()`는 일반적으로 컴파일러에게 타입을 추론하도록 하여 코드를 더 간결하게 만듬.

```rust
let str_literal: &str = "nice weather";
let owned_string: String = str_literal.into();
```

### modules
```rust
mod sausage_factory {
    // Don't let anybody outside of this module see this!
    fn get_secret_recipe() -> String {
        String::from("Ginger")
    }

    pub fn make_sausage() {
        get_secret_recipe();
        println!("sausage!");
    }
}
```

- 모듈에 `pub`키워드를 사용하여 모듈을 공개해도 모듈의 내용은 비공개.
- 모듈을 공개했다고 해서 내용까지 공개되지는 않는다. 모듈의 `pub`키워드는 상위 모듈이 해당 모듈을 가리킬 수 있도록 할 뿐, 그 내부 코드에 접근하도록 하는 것은 아님. 모듈은 단순한 컨테이너이기 때문에 모듈을 공개하는 것만으로는 할 수 있는 것은 별로 없음.
- 모듈이 가지고 있는 아이템도 마찬가지로 공개해야함.
- 크레이트: 라이브러리나 실행가능한 모듈로 구성된 트리 구조로 러스트가 한 번의 컴파일 시 고려하는 가장 작은 코드 단위.

```rust
// 절대 경로: 크레이트 루트로부터 시작되는 전체 경로. 외부 크레이트로부터의 코드에 대해서는 해당 크레이트 이름으로 절대 경로가 시작되고 현재의 크레이트로부터의 코드에 대해서는 crate리터럴로부터 시작된다.
crate::sausage_factory::make_sausage();
// 상대 경로
sausage_factory::make_sausage();
```

- `use` 키워드를 사용하면 어떤 경로의 단축경로(shortcut)를 만들 수 있고, 그러면 스코프 안쪽 어디서라도 짧은 이름을 사용할 수 있다. 
- `use`가 사용된 특정 스코프에서만 단축경로가 만들어진다.
- `use` 키워드로 동일한 이름의 타입을 스코프로 여러 개 가져올 경우 경로 뒤에 `as` 키워드를 작성하고, 새로운 이름이나 타입 별칭을 작성하면 된다.
- `pub use`로 다시 내보내기 가능.

```rust
pub use self::fruits::PEAR as fruit;
pub use self::veggies::CUCUMBER as veggie;
```

```rust
// glob 연산자 *를 붙이면 경로 안에 정의된 모든 공개 아이템을 가져올 수 있다. 
use std::time::*;
// 두 경로를 use 구문 하나로 합치기
use std::time::{ UNIX_EPOCH, SystemTime };
```

### hashmaps
- 서로 연관된 친구들 저장.
- 임의 타입으로 된 키를 이용하여 데이터를 찾을 때 유용.

```rust

fn fruit_basket() -> HashMap<String, u32> {
  // hashMap 생셩
  let mut basket = HashMap::new();

  // 요소 추가
  basket.insert(String::from("banana"), 2);

  basket
}
```

- 요소에 접근하려면 `get` 메서드 사용.
- `get`메서드는 Option<&V>를 반환함. 만약에 해당 키에 대한 값이 없으면 `get`은 `None`을 반환함.
- 따라서 `copied`를 호출하여 `Option<&u32>`이 아닌 `Option<u32>`를 얻어온 다음, `unwrap_or`를 써서 해당 키에 대한 아이템을 가지고 있지 않응 경우 0을 설정하도록 처리할 수 있음.

```rust
basket.get("apple").copied().unwrap_or(0);
```

- `entry` 메서드를 사용하면 키가 없을때만 키와 값 추가할 수 있다.

```rust
basket.entry(fruit).or_insert(1);
```

- 해시 맵은 키에 대한 값을 찾아서 예전 값에 기초하여 값을 업데이트함.

```rust
let team_1_scores = scores.entry(team_1_name).or_insert(Team {
    goals_scored: 0,
    goals_conceded: 0
});

team_1_scores.goals_scored += team_1_score;
team_1_scores.goals_conceded += team_2_score;
```

### options
- `Option` 타입은 값이 있거나 없을 수 있다.
- 러스트는 null 값으로 발생하는 문제를 제거하기 위해서 널 개념이 없다.
- 대신에 러스트에는 널이 없지만, 값의 존재 혹은 부재의 개념을 표현할 수 있는 열거형이 있다. 이 열거형이 바로 `Option<T>`이며, 표준 라이브러리에 정의되어 있다.

```rust
enum Option<T> {
  None,
  Some(T),
}
```

- `if let` 구문을 사용해서 `optional_target`이 `Some` 유형인 경우에만 실행된다.

```rust
fn simple_option() {
    let target = "rustlings";
    let optional_target = Some(target);

    if let Some(word) = optional_target {
        assert_eq!(word, target);
    }
}
```

- `ref` 키워드는 `match` 패턴에서 변수를 참조로 바인딩하는 데 사용.
- `match` 패턴에서 변수를 바인딩할 때, 해당 변수는 값을 소비되는데, 변수가 패턴에 바인딩되면 그 변수는 소비되어 원래의 값이 이동(move)하게 된다. 하지만 `ref` 키워드를 사용하면 변수를 참조로 바인딩하여 원래 값이 소비되지 않고 참조만 바인딩된다.
- 이렇게 `ref`를 사용하면 원래 값이 유지되어야 하는 상황에서 유용한데, 아래 코드에서 `y` 변수는 여전히 원래의 `Option<Point>` 값을 소유하게 된다.

```rust
fn main() {
    let y: Option<Point> = Some(Point { x: 100, y: 200 });

    match y {
        Some(ref p) => println!("Co-ordinates are {},{} ", p.x, p.y),
        _ => panic!("no match!"),
    }
    y; // Fix without deleting this line.
}
```

- `&`와 `ref`의 차이점
  - `&`는 참조 연산자로 변수 앞에 `&`를 사용하면 해당 변수의 참조를 만든다. 이 경우에는 변수를 참조로 바인딩하게 된다.
  - `ref`는 패턴 매칭에서 변수를 참조로 바인딩하는데 `ref` 키워드는 `match` 패턴에서만 사용된다. `match` 패턴에서 변수를 바인딩할 때, 그 변수가 값을 소비하는 대신 참조로 바인딩되도록 한다.
  - 주요 차이점은 `&`는 참조를 생성하는 연산자이고, `ref`는 `match` 패턴에서 변수를 참조로 바인딩하는 데 사용되는 키워드.

### week3 느낀점
이번주차는 책이 있어서 더 수월하게 문제를 풀고 배운것도 많았던 것 같다. 이미 있는 거긴했지만.. 번역에 잘 정리되어있어서 더 수월하게 읽고 이해하는데 도움이 되었다. (나는 책이 더 좋은가보다.. 늙었나..)   

`Option<T>`는 함수형 프로그래밍에서 자주보이는 패턴인데 러스트에서 보니 신기했다. rust는 객체지향과 함수형이 잘 공존되어있는 언어인 것 같다.   
자바스크립트를 함수형 개넘으로 만든 rescript를 한창 공부했을때 써봐서 그런지 이부분은  익숙했다.

```rescript
// rescript
type option<'a> = None | Some('a)

let licenseNumber =
  if personHasACar {
    Some(5)
  } else {
    None
  }

switch licenseNumber {
| None =>
  Console.log("The person doesn't have a car")
| Some(number) =>
  Console.log("The person's license number is " ++ Int.toString(number))
}
```

이번주차는 책이 있어서 더 많은 것들을 이해할 수 있어서 좋았다. 러스트는 좋은 기능들만 가져다놓은 집합체같은 느낌을 받았다.
