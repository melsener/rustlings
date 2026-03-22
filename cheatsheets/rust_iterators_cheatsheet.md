# Rust Iterators Cheatsheet

## 1. Iterator Creation

### From collections
```rust
let vec = vec![1, 2, 3];
let iter = vec.iter();        // &i32
let iter = vec.iter_mut();    // &mut i32 (requires mut vec)
let iter = vec.into_iter();   // i32 (consumes vec)
```

### From arrays
```rust
let arr = [1, 2, 3];
let iter = arr.iter();        // &i32
let iter = arr.into_iter();   // i32 (array is Copy)
```

### Range iterators
```rust
let iter = 0..10;             // 0 to 9
let iter = 0..=10;            // 0 to 10 (inclusive)
let iter = (0..10).rev();     // 9 to 0
```

### Other iterators
```rust
let iter = std::iter::repeat(5);              // infinite 5s
let iter = std::iter::repeat(5).take(3);      // [5, 5, 5]
let iter = std::iter::once(42);               // single value
```

## 2. Iterator Types

| Method | Iterator Type | Yields |
|--------|---------------|--------|
| `iter()` | Immutable | `&T` |
| `iter_mut()` | Mutable | `&mut T` |
| `into_iter()` | Consuming | `T` |

## 3. Consuming Adaptors (Terminal Operations)

### Collecting
```rust
let vec: Vec<_> = (0..5).collect();
let map: HashMap<_, _> = vec.iter().enumerate().collect();
```

### Reducing
```rust
let sum: i32 = vec.iter().sum();
let product: i32 = vec.iter().product();
let max = vec.iter().max();            // Option<&T>
let min = vec.iter().min();            // Option<&T>
```

### Finding
```rust
let found = vec.iter().find(|&&x| x > 2);       // Option<&T>
let pos = vec.iter().position(|&x| x > 2);      // Option<usize>
let any = vec.iter().any(|&x| x > 2);           // bool
let all = vec.iter().all(|&x| x > 0);           // bool
```

### Folding
```rust
let sum = vec.iter().fold(0, |acc, x| acc + x);
let result = vec.iter().reduce(|acc, x| acc + x);  // Option<T>
```

## 4. Iterator Adaptors (Lazy Operations)

### Transforming
```rust
let doubled: Vec<_> = vec.iter().map(|x| x * 2).collect();
let flattened: Vec<_> = vec_of_vecs.iter().flatten().collect();
let enumerated: Vec<_> = vec.iter().enumerate().collect();  // (index, value)
```

### Filtering
```rust
let filtered: Vec<_> = vec.iter().filter(|&&x| x > 2).collect();
let mapped_filtered: Vec<_> = vec.iter()
    .filter_map(|&x| if x > 0 { Some(x * 2) } else { None })
    .collect();
```

### Taking and skipping
```rust
let first_3: Vec<_> = vec.iter().take(3).collect();
let skip_2: Vec<_> = vec.iter().skip(2).collect();
let while_positive: Vec<_> = vec.iter().take_while(|&&x| x > 0).collect();
let skip_while: Vec<_> = vec.iter().skip_while(|&&x| x < 5).collect();
```

### Chaining and zipping
```rust
let chained: Vec<_> = vec1.iter().chain(vec2.iter()).collect();
let zipped: Vec<_> = vec1.iter().zip(vec2.iter()).collect();  // (T, U)
```

## 5. For Loop Equivalents

```rust
// for item in collection
for item in vec {}              // consumes vec
for item in &vec {}             // borrows vec (same as vec.iter())
for item in &mut vec {}         // mutably borrows vec (same as vec.iter_mut())

// Explicit iterator versions
for item in vec.into_iter() {}  // consumes
for item in vec.iter() {}       // borrows
for item in vec.iter_mut() {}   // mutably borrows
```

## 6. Common Patterns

### Transform and collect
```rust
let strings: Vec<String> = numbers.iter()
    .map(|n| n.to_string())
    .collect();
```

### Filter then map
```rust
let results: Vec<_> = items.iter()
    .filter(|item| item.is_valid())
    .map(|item| item.process())
    .collect();
```

### Chain multiple collections
```rust
let combined: Vec<_> = vec1.iter()
    .chain(vec2.iter())
    .chain(vec3.iter())
    .collect();
```

### Find first matching
```rust
let found = items.iter()
    .find(|item| item.matches_criteria())
    .map(|item| item.extract_value());
```

## 7. Performance Tips

- **Lazy evaluation**: Iterator adaptors do no work until consumed
- **Zero-cost abstractions**: Iterator chains compile to efficient loops
- **Avoid unnecessary collections**: Chain operations instead of collecting intermediate results
- **Use iterator methods**: Often faster than manual loops

## 8. Common Gotchas

```rust
// ❌ This doesn't work - need to collect or consume
(0..10).map(|x| println!("{}", x));

// ✅ These work
(0..10).for_each(|x| println!("{}", x));
(0..10).map(|x| println!("{}", x)).collect::<Vec<_>>();

// ❌ Borrowing issues
let vec = vec![1, 2, 3];
let iter = vec.iter();
drop(vec);  // Error: vec borrowed by iter

// ✅ Take ownership instead
let vec = vec![1, 2, 3];
let iter = vec.into_iter();  // vec consumed, iter owns elements
```