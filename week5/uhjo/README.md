### Rust - Move Semantics

---

```rust
fn main() {
    let s1: String = String::from("Hello!");
    let s2: String = s1;
    
    println!("{}, world!", s2);
    // println!("{}, world!", s1);
}
```

- **s1 변수에 `"Hello!"` 값이 메모리에 할당된 모습**
    ```rust
    let s1: String = String::from("Hello!");
    ```
  ![1.png](images%2F1.png)


- **s2 변수에 s1이 대입된 경우**
    ```rust
    let s2: String = s1;
    ```
  ![2.png](images%2F2.png)

    - **⭕️ 복사되는 것**
      - 포인터 / 길이 / 용량
    - **❌ 복사되지 않는 것**
      - 힙 메모리의 데이터

    > `IF` - 힙 메모리까지 복사하게 된다면?
    > - 어마무시한 데이터가 힙 메모리에 저장되어 있다면 복사 작업이 매우 느려지게 됨


- **변수에 대한 메모리 해제**
  1. s2에 s1이 대입될 경우 먼저 변수 s1을 **무효화**한다.
  ![3.png](images%2F3.png)
  2. s1 또는 s2가 scope에서 벗어나게 되는 경우 발생
  3. `drop` 함수를 호출하여 s2가 사용하고 있는 힙 메모리를 제거한다.

  > `IF` - s1 변수에 대해 무효화 하지 않는다면?
  > - 둘 다 같은 메모리를 해제하려고 할 것이다.
  > - 이는 `double free` 취약점 발생 
  > ```rust
  > println!("{}, world!", s1);
  > ```
  > ```rust
  > error[E0382]: use of moved value: `s1`
  > --> src/main.rs:4:27
  >   |
  > 3 |     let s2 = s1;
  >   |         -- value moved here
  > 4 |     println!("{}, world!", s1);
  >   |                            ^^ **value used here after move (여기서 이동한 value가 사용되고 있다고 경고)**
  >   |
  >   = note: move occurs because `s1` has type `std::string::String`,
  > which does not implement the `Copy` trait
  > ```

- **double free란 ?**
    - 메모리 해제를 두 번 반복하는 행위로 인해 발생하는 메모리 손상 오류를 뜻함
    - 왜인지는 다음 추후..


> **그럼 이게 왜 move semantics인거죠 ?**
> - 이것은 언뜻 보면 c++에서의 얕은 복사 개념과 비슷해 보일 수 있다.
> ![4.jpeg](images%2F4.jpeg)
> - s2에 s1을 대입하면 s1에 대하여 **우리는 사용할 수가 없는** 상황
> - 따라서 s1의 값은 s2만을 통해서 사용할 수 있다.
> - 이것을 가지고 우리는 `이동 의미론`이라고 말할 수 있다.

### Rust - Copy

---

```rust
fn main() {
    let x = 42;
    let y = x;
    println!("x: {x}");
    println!("y: {y}");
}
```

> 잠깐만요 ! 🙋🏻  
> 이거 위에서 `move`라면서 왜 또 `copy`인데요?

- 위의 `move` 코드와 `copy` 코드의 차이점이 무엇일까
    - String형과 정수형으로 **변수 값의 타입**에 차이가 존재한다.
    - Rust에서는 Stack에 저장되는 타입의 데이터들에게 `copy trait`이라는 특별한 `annotation`을 할당해준다.

    > **각 메모리에는 무엇이 저장되나요?**
    > - Heap Memory
    >     - **참조 타입**(String, Enum, Array…)의 데이터
    > - Stack Memory
    >     - `Copy`가 가능하다고 보면 됨
    >     - **원시 타입**(i32, u32, f64, bool, tuple(i32, i32)…)의 데이터

    - 만약 s1이 `copy trait`을 가지고 있다면 대입 후에도 해당 변수를 사용할 수 있다.
    ![5.png](images%2F5.png)


### Rust - Clone

---

- 만약 저는 느려지든 말든 해당 변수의 모든 데이터에 대한 복사를 하고 싶다면요?
- 그럴 때는 그냥 `clone()` 메소드를 사용하심 됩니다.

```rust
let s1 = String::from("hello");
let s2 = s1.clone();

println!("s1 = {}, s2 = {}", s1, s2);
```

### C++ - Copy

---

```cpp
std::string s1 = "Cpp";
std::string s2 = s1;
```

- **s1 변수에 `"Cpp"` 값이 메모리에 할당된 모습**
    ```cpp
    std::string s1 = "Cpp";
    ```
  ![6.png](images%2F6.png)

- **s2 변수에 s1이 대입된 경우**
    ```cpp
    std::string s2 = s1;
    ```
    - default로 `copy`를 수행
        - 이렇게 하지 않으면 기본적으로 소유권의 개념이 지금 구성에서 적용되지 않으므로
        - `double free` 취약점 발생
        ![7.png](images%2F7.png)

### C++ - Move

---

```cpp
std::string s1 = "Hello!";
std::string s2 = std::move(s1);
```

- **s1 변수에 `"Hello!"` 값이 메모리에 할당된 모습**
    ```rust
    std::string s1 = "Hello!";
    ```
    ![8.png](images%2F8.png)

- **s2 변수에 s1이 move(이동)된 경우**
    ```rust
    std::string s2 = std::move(s1);
    ```
    - s2는 s1이 가리키고 있던 **Heap 영역**의 데이터를 가리키게 된다. (s1의 데이터 소유권 이전)
    - **Stack 영역**의 s1은 **Heap 영역**의 어떠한 데이터도 지정하고 있지 않는다.
    - 사용자는 **Stack 영역**의 s1을 계속 **사용 할 수 있다.**
