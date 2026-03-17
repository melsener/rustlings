# Rust Move Semantics Cheatsheet

## 1. Ownership Rules

-   Each value has one owner
-   When the owner goes out of scope, the value is dropped
-   Ownership can be:
    -   moved
    -   borrowed (`&T`)
    -   mutably borrowed (`&mut T`)
-   Some types implement `Copy`, so they are copied instead of moved

## 2. Move vs Copy

### Move (default for most types)

``` rust
let s1 = String::from("hello");
let s2 = s1;
```

### Copy types

``` rust
let x = 5;
let y = x;

println!("{}", x);
```

## 3. Taking Ownership

``` rust
fn take(s: String) {
    println!("{}", s);
}
```

## 4. Borrowing

Prefer &str and &\[T\]

``` rust
fn print_len(s: &str) {
    println!("{}", s.len());
}
```

## 5. Mutable Borrow

``` rust
fn add_world(s: &mut String) {
    s.push_str(" world");
}
```

## 6. Borrow Rules

-   many immutable OR one mutable
-   reference cannot outlive data

## 7. Ownership choices

  Need        Use
  ----------- ---------
  read        &T
  modify      &mut T
  own         T
  duplicate   clone()

## 8. Patterns

Take ownership:

``` rust
fn fill(mut v: Vec<i32>) -> Vec<i32> {
    v.push(1);
    v
}
```

Borrow and create new:

``` rust
fn filled(slice: &[i32]) -> Vec<i32> {
    let mut v = slice.to_vec();
    v.push(1);
    v
}
```

Mutate in place:

``` rust
fn fill(v: &mut Vec<i32>) {
    v.push(1);
}
```

## 9. Conversions

``` rust
let s: String = "hi".to_string();

let s: &str = &String::from("hi");

let v = vec![1,2,3];

let s: &[i32] = &v;

let v2 = s.to_vec();
```

## 10. Guarantees

Safe Rust prevents:

-   use after free
-   double free
-   data races
-   dangling refs
