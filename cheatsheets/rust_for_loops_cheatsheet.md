# Rust For Loop Cheatsheet

## 1. Basic For Loop Syntax

### Iterate over collections
```rust
let vec = vec![1, 2, 3, 4, 5];

// Consume the collection (moves ownership)
for item in vec {
    println!("{}", item);  // item: i32
}
// vec is no longer accessible after this

// Borrow immutably (iterate over references)
for item in &vec {
    println!("{}", item);  // item: &i32
}
// vec is still accessible

// Borrow mutably (iterate over mutable references)
for item in &mut vec {
    *item *= 2;  // item: &mut i32
}
```

## 2. Range Syntax

### Exclusive ranges (start..end)
```rust
// 0, 1, 2, 3, 4
for i in 0..5 {
    println!("{}", i);
}

// Empty range
for i in 5..5 {  // Nothing happens
    println!("{}", i);
}

// Reverse won't work directly
// for i in 5..0 {}  // ❌ Empty range
```

### Inclusive ranges (start..=end)
```rust
// 0, 1, 2, 3, 4, 5
for i in 0..=5 {
    println!("{}", i);
}

// Single item
for i in 5..=5 {  // Just 5
    println!("{}", i);
}
```

### Reverse ranges
```rust
// 5, 4, 3, 2, 1, 0
for i in (0..=5).rev() {
    println!("{}", i);
}

// 4, 3, 2, 1, 0
for i in (0..5).rev() {
    println!("{}", i);
}
```

## 3. Common Range Patterns

### Array/vector indices
```rust
let arr = [10, 20, 30, 40];

// Iterate over indices
for i in 0..arr.len() {
    println!("arr[{}] = {}", i, arr[i]);
}

// Better: iterate with enumerate
for (i, value) in arr.iter().enumerate() {
    println!("arr[{}] = {}", i, value);
}
```

### Step by
```rust
// Every 2nd number: 0, 2, 4, 6, 8
for i in (0..10).step_by(2) {
    println!("{}", i);
}

// Reverse with step: 10, 8, 6, 4, 2, 0
for i in (0..=10).rev().step_by(2) {
    println!("{}", i);
}
```

### Conditional ranges
```rust
let start = 2;
let end = 8;

for i in start..end {
    println!("{}", i);  // 2, 3, 4, 5, 6, 7
}

// Dynamic range based on condition
let limit = if condition { 10 } else { 5 };
for i in 0..limit {
    println!("{}", i);
}
```

## 4. Collection Iteration Patterns

### Arrays and slices
```rust
let arr = [1, 2, 3, 4, 5];

for item in arr {}              // Copy semantics (item: i32)
for item in &arr {}             // item: &i32
for item in arr.iter() {}       // item: &i32 (explicit)
```

### Vectors
```rust
let vec = vec![1, 2, 3, 4, 5];

for item in vec {}              // Consumes vec (item: i32)
for item in &vec {}             // item: &i32
for item in vec.iter() {}       // item: &i32
for item in vec.into_iter() {}  // Consumes vec (item: i32)
```

### Strings
```rust
let s = String::from("hello");

// Iterate over bytes
for byte in s.bytes() {
    println!("{}", byte);  // u8
}

// Iterate over chars  
for ch in s.chars() {
    println!("{}", ch);  // char
}

// Iterate over lines
for line in s.lines() {
    println!("{}", line);  // &str
}
```

### Hash maps
```rust
use std::collections::HashMap;
let mut map = HashMap::new();
map.insert("a", 1);
map.insert("b", 2);

// Iterate over key-value pairs
for (key, value) in &map {
    println!("{}: {}", key, value);
}

// Iterate over keys only
for key in map.keys() {
    println!("{}", key);
}

// Iterate over values only  
for value in map.values() {
    println!("{}", value);
}
```

## 5. Loop Control

### Break and continue
```rust
for i in 0..10 {
    if i == 3 {
        continue;  // Skip to next iteration
    }
    if i == 7 {
        break;     // Exit loop completely  
    }
    println!("{}", i);  // Prints: 0, 1, 2, 4, 5, 6
}
```

### Labeled breaks
```rust
'outer: for i in 0..3 {
    for j in 0..3 {
        if i == 1 && j == 1 {
            break 'outer;  // Breaks out of outer loop
        }
        println!("({}, {})", i, j);
    }
}
```

### Return from loops
```rust
fn find_first_even(numbers: &[i32]) -> Option<i32> {
    for &num in numbers {
        if num % 2 == 0 {
            return Some(num);  // Return early
        }
    }
    None
}
```

## 6. Nested Loops

### 2D iteration
```rust
let matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]];

for row in &matrix {
    for &item in row {
        print!("{} ", item);
    }
    println!();
}
```

### Cartesian product
```rust
let letters = ['a', 'b', 'c'];
let numbers = [1, 2, 3];

for letter in &letters {
    for number in &numbers {
        println!("{}{}", letter, number);  // a1, a2, a3, b1, b2, b3, c1, c2, c3
    }
}
```

## 7. Common Patterns

### Index tracking
```rust
let items = ["apple", "banana", "cherry"];

// Method 1: Manual counter
let mut index = 0;
for item in &items {
    println!("{}: {}", index, item);
    index += 1;
}

// Method 2: Enumerate (preferred)
for (index, item) in items.iter().enumerate() {
    println!("{}: {}", index, item);
}
```

### Chunking
```rust
let data = [1, 2, 3, 4, 5, 6, 7, 8];

for chunk in data.chunks(3) {
    println!("{:?}", chunk);  // [1, 2, 3], [4, 5, 6], [7, 8]
}
```

### Zipping collections
```rust
let names = ["Alice", "Bob", "Charlie"];
let ages = [25, 30, 35];

for (name, age) in names.iter().zip(ages.iter()) {
    println!("{} is {} years old", name, age);
}
```

## 8. Performance Notes

- **Zero-cost abstraction**: For loops compile to efficient machine code
- **Iterator methods**: Often more efficient than manual indexing
- **Bounds checking**: `for item in collection` avoids bounds checks
- **Manual indexing**: `for i in 0..len { arr[i] }` includes bounds checks

## 9. When to Use What

| Pattern | Use Case |
|---------|----------|
| `for item in collection` | Want to consume/move the collection |
| `for item in &collection` | Want to read items, keep collection |
| `for item in &mut collection` | Want to modify items in place |
| `for i in 0..len` | Need indices for complex logic |
| `for (i, item) in enumerate()` | Need both index and item |
| `for _ in 0..n` | Repeat something n times |
| `for i in (0..n).rev()` | Count down |