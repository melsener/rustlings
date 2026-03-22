# Rust Smart Pointers Cheatsheet

## 1. What Are Smart Pointers?

Smart pointers are data structures that act like pointers but have additional metadata and capabilities. Unlike references (which only borrow data), smart pointers often **own** the data they point to.

### Key smart pointer types in Rust:
- **`Box<T>`** - Heap allocation, enables recursive types
- **`Rc<T>`** - Reference counting for shared ownership (single-threaded)
- **`Arc<T>`** - Atomic reference counting for shared ownership (multi-threaded)
- **`Cow<T>`** - Clone-on-Write for efficient borrowing

## 2. Box<T> - Heap Allocation

### When to use Box<T>:
- Store data on the heap instead of stack
- Enable recursive types (unknown size at compile time)
- Transfer ownership of large data without copying

### Basic usage:
```rust
// Store a value on the heap
let boxed_num = Box::new(42);
println!("{}", boxed_num);  // Automatically dereferenced

// Explicit dereferencing
let value = *boxed_num;
```

### Recursive types (main use case):
```rust
// ❌ This won't compile - infinite size
// enum List {
//     Cons(i32, List),
//     Nil,
// }

// ✅ Box enables recursive types
#[derive(Debug)]
enum List {
    Cons(i32, Box<List>),
    Nil,
}

// Create a cons list: 1 -> 2 -> 3 -> Nil
let list = List::Cons(1, 
    Box::new(List::Cons(2, 
        Box::new(List::Cons(3, 
            Box::new(List::Nil))))));
```

### Performance characteristics:
- Single heap allocation
- Zero runtime overhead for dereferencing
- Move semantics transfer ownership

## 3. Rc<T> - Reference Counting (Single-threaded)

### When to use Rc<T>:
- Multiple owners need to share the same data
- Single-threaded context only
- Data that shouldn't be copied but shared

### Basic usage:
```rust
use std::rc::Rc;

let data = Rc::new(String::from("shared data"));
let data1 = Rc::clone(&data);  // Increment reference count
let data2 = Rc::clone(&data);  // Increment reference count

println!("Reference count: {}", Rc::strong_count(&data));  // 3
```

### Multiple ownership example:
```rust
use std::rc::Rc;

#[derive(Debug)]
struct Node {
    value: i32,
    children: Vec<Rc<Node>>,
}

let leaf = Rc::new(Node { value: 3, children: vec![] });
let branch = Node {
    value: 1,
    children: vec![Rc::clone(&leaf), Rc::clone(&leaf)],  // Same leaf, two parents
};
```

### Memory management:
```rust
{
    let data = Rc::new(42);
    let data_clone = Rc::clone(&data);
    println!("Count: {}", Rc::strong_count(&data));  // 2
}  // data_clone dropped, count becomes 1
// When last reference is dropped, data is deallocated
```

## 4. Arc<T> - Atomic Reference Counting (Multi-threaded)

### When to use Arc<T>:
- Share data across multiple threads
- Thread-safe reference counting
- Immutable data that needs concurrent access

### Basic usage:
```rust
use std::sync::Arc;
use std::thread;

let data = Arc::new(vec![1, 2, 3, 4, 5]);
let mut handles = vec![];

for i in 0..3 {
    let data_clone = Arc::clone(&data);
    let handle = thread::spawn(move || {
        println!("Thread {}: {:?}", i, data_clone);
    });
    handles.push(handle);
}

for handle in handles {
    handle.join().unwrap();
}
```

### Shared computation example:
```rust
use std::sync::Arc;
use std::thread;

let numbers = Arc::new((0..100).collect::<Vec<_>>());
let mut handles = vec![];

for offset in 0..4 {
    let numbers_clone = Arc::clone(&numbers);
    let handle = thread::spawn(move || {
        let sum: i32 = numbers_clone.iter()
            .filter(|&&n| n % 4 == offset)
            .sum();
        println!("Sum for offset {}: {}", offset, sum);
    });
    handles.push(handle);
}

for handle in handles {
    handle.join().unwrap();
}
```

### Arc vs Rc:
| Feature | Rc<T> | Arc<T> |
|---------|-------|--------|
| Thread Safety | Single-threaded only | Multi-threaded |
| Performance | Faster (no atomics) | Slower (atomic operations) |
| Use Context | Same thread sharing | Cross-thread sharing |

## 5. Cow<T> - Clone-on-Write

### When to use Cow<T>:
- Optimize for borrowing with occasional ownership
- Avoid unnecessary clones
- API flexibility (accept borrowed or owned data)

### Basic concept:
```rust
use std::borrow::Cow;

// Two variants:
// Cow::Borrowed(&T) - borrows data
// Cow::Owned(T)     - owns data
```

### Reading data (no cloning):
```rust
use std::borrow::Cow;

fn process_data(data: &Cow<str>) {
    println!("Data: {}", data);  // Works for both borrowed and owned
}

let owned = Cow::Owned("owned".to_string());
let borrowed = Cow::Borrowed("borrowed");

process_data(&owned);     // No clone
process_data(&borrowed);  // No clone
```

### Clone-on-write behavior:
```rust
use std::borrow::Cow;

fn make_ascii_uppercase(data: &mut Cow<str>) {
    if data.chars().any(|c| c.is_lowercase()) {
        // Only clone if mutation is needed
        let mut owned = data.to_mut();  // Clones if borrowed
        owned.make_ascii_uppercase();
    }
}

// Case 1: No mutation needed (already uppercase)
let mut cow1 = Cow::Borrowed("HELLO");
make_ascii_uppercase(&mut cow1);
assert!(matches!(cow1, Cow::Borrowed(_)));  // Still borrowed

// Case 2: Mutation needed
let mut cow2 = Cow::Borrowed("hello");
make_ascii_uppercase(&mut cow2);
assert!(matches!(cow2, Cow::Owned(_)));  // Now owned (cloned)
```

### Common patterns:
```rust
use std::borrow::Cow;

// Accept both &str and String
fn process_string(s: Cow<str>) -> String {
    if s.contains("URGENT") {
        format!("[PRIORITY] {}", s)
    } else {
        s.into_owned()  // Convert to String
    }
}

// Usage
let result1 = process_string("Hello".into());           // From &str
let result2 = process_string(String::from("Hi").into()); // From String
```

## 6. Smart Pointer Comparison

| Smart Pointer | Ownership | Thread Safe | Heap Allocated | Use Case |
|---------------|-----------|-------------|----------------|----------|
| `Box<T>` | Single owner | No* | Yes | Heap allocation, recursive types |
| `Rc<T>` | Multiple owners | No | Yes | Shared ownership (single-thread) |
| `Arc<T>` | Multiple owners | Yes | Yes | Shared ownership (multi-thread) |
| `Cow<T>` | Conditional | Depends on T | Conditional | Efficient borrowing/owning |

*Box<T> can be sent between threads, but not shared

## 7. When to Use Each

### Box<T>:
```rust
// Recursive data structures
enum BinaryTree {
    Leaf(i32),
    Branch(Box<BinaryTree>, Box<BinaryTree>),
}

// Large objects to avoid stack overflow
let big_array = Box::new([0; 1000000]);
```

### Rc<T>:
```rust
// Graph-like structures with shared nodes
struct Node {
    neighbors: Vec<Rc<Node>>,
}

// Sharing expensive-to-clone data
let config = Rc::new(load_large_config());
let worker1 = Worker::new(Rc::clone(&config));
let worker2 = Worker::new(Rc::clone(&config));
```

### Arc<T>:
```rust
// Sharing data across threads
let shared_state = Arc::new(vec![1, 2, 3]);
thread::spawn({
    let state = Arc::clone(&shared_state);
    move || process(state)
});
```

### Cow<T>:
```rust
// APIs that can work with both borrowed and owned data
fn normalize_path(path: Cow<Path>) -> PathBuf {
    if path.is_absolute() {
        path.into_owned()
    } else {
        env::current_dir().unwrap().join(path.as_ref())
    }
}
```

## 8. Common Patterns and Gotchas

### Reference cycles with Rc<T>:
```rust
// ❌ This creates a memory leak
use std::rc::Rc;
use std::cell::RefCell;

struct Node {
    next: Option<Rc<RefCell<Node>>>,
}

// Use Weak<T> to break cycles
use std::rc::Weak;
struct SafeNode {
    next: Option<Rc<RefCell<SafeNode>>>,
    parent: Option<Weak<RefCell<SafeNode>>>,  // Weak reference
}
```

### Performance tips:
```rust
// ✅ Prefer this
let data = Rc::new(expensive_data());
let clone1 = Rc::clone(&data);  // Cheap reference count increment

// ❌ Over this (if you meant to share)
let clone2 = data.clone();  // This might clone the data itself
```

### Cow<T> conversion:
```rust
use std::borrow::Cow;

let borrowed: Cow<str> = "hello".into();
let owned: Cow<str> = String::from("hello").into();

// Convert to owned
let string1: String = borrowed.into_owned();
let string2: String = owned.into_owned();

// Borrow as reference
let str_ref: &str = borrowed.as_ref();
```

## 9. Memory Layout

```
Stack          Heap
-----          ----
Box<T>   -->   [T]

Rc<T>    -->   [RefCount | WeakCount | T]
               
Arc<T>   -->   [AtomicRefCount | AtomicWeakCount | T]

Cow::Borrowed  -->  [&T]  (points to existing data)
Cow::Owned     -->  [T]   (owns the data)
```