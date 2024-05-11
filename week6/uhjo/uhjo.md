## Week 6 Result

### Two Sum
![week6_two-sum.png](images%2Fweek6_two-sum.png)
```rust
impl Solution {
    pub fn two_sum(nums: Vec<i32>, target: i32) -> Vec<i32> {
        let mut v: Vec<i32> = Vec::new();

        for i in 0..nums.len()-1 {
            for k in i+1..nums.len() {
                if nums[i] + nums[k] == target {
                    v.push(i as i32);
                    v.push(k as i32);
                    return v;
                }
            }
        }
        v
    }
}
```

### Palindrome Number
![week6_palindrome-number.png](images%2Fweek6_palindrome-number.png)
```rust
impl Solution {
    pub fn is_palindrome(x: i32) -> bool {
        let mut reverse = 0;
        let mut data = x;

        while data > 0 {
            reverse *= 10;
            reverse += data % 10;
            data /= 10;
        }
        if reverse == x {
            true
        } else {
            false
        }
    }
}
```

### Valid Parentheses
![week6_valid-parentheses.png](images%2Fweek6_valid-parentheses.png)
```rust
impl Solution {
    pub fn is_valid(s: String) -> bool {
        let mut stack: Vec<char> = Vec::new();

        for c in s.chars() {
            if c == '(' {
                stack.push(')');
            } else if c == '[' {
                stack.push(']');
            } else if c == '{' {
                stack.push('}');
            } else if stack.pop() != Some(c) {
                return false;
            }
        }
        stack.is_empty()
    }
}
```