# owning-ref-rs

[![Rust](https://github.com/corp-momenti/owning-ref-rs/actions/workflows/rust.yml/badge.svg?branch=main)](https://github.com/corp-momenti/owning-ref-rs/actions/workflows/rust.yml)
[![Build](https://github.com/corp-momenti/owning-ref-rs/actions/workflows/build.yml/badge.svg?branch=main)](https://github.com/corp-momenti/owning-ref-rs/actions/workflows/build.yml)
[![Test](https://github.com/corp-momenti/owning-ref-rs/actions/workflows/test.yml/badge.svg?branch=main)](https://github.com/corp-momenti/owning-ref-rs/actions/workflows/test.yml)
[![Audit](https://github.com/corp-momenti/owning-ref-rs/actions/workflows/audit.yml/badge.svg?branch=main)](https://github.com/corp-momenti/owning-ref-rs/actions/workflows/audit.yml)

A library for creating references that carry their owner with them.

This can sometimes be useful because Rust borrowing rules normally prevent
moving a type that has been borrowed from. For example, this kind of code gets rejected:

```rust
fn return_owned_and_referenced<'a>() -> (Vec<u8>, &'a [u8]) {
    let v = vec![1, 2, 3, 4];
    let s = &v[1..3];
    (v, s)
}
```

위와 같은 예제를 보면, 위의 코드는 컴파일이 되지 않는다.
왜냐면 v와 v에 의존성이 있는 v의 참조자인 s가 따로 변수화되어 반환될 수 없다.
borrow checker가 v가 함수의 범위를 벗어나면서 소멸될 때, s가 무효화되기 때문이다.

그렇지만 아래의 코드는 v(o)를 Deref하여 raw pointer의 안전한 참조(주소)를,
reference필드에, 즉 o의 메모리 값의 주소를 값으로 저장한 다음,
v에 대한 의존성을 떼어 놓는다.
즉 reference 필드는 v(o)에 대한 참조(주소)가 아니라, v(o)를 Deref한 메모리 값의
안전한 참조를 얻어 값으로 저장한다. 즉, OwningRef 커스텀 스마트 포인터의 주소값이 아니라,
스마트 포인터 내부의 값의 주소값을 reference 필드로 저장한다.


This library enables this safe usage by keeping the owner and the reference
bundled together in a wrapper type that ensure that lifetime constraint:

```rust
fn return_owned_and_referenced() -> OwningRef<Vec<u8>, [u8]> {
    let v = vec![1, 2, 3, 4];
    let or = OwningRef::new(v);
    let or = or.map(|v| &v[1..3]);
    or
}
```

## Getting Started

To get started, add the following to `Cargo.toml`.

```toml
owning-ref = "0.5.0"
```

...and see the docs for how to use it.

## Example

```rust
extern crate owning_ref;
use owning_ref::BoxRef;

fn main() {
    // Create an array owned by a Box.
    let arr = Box::new([1, 2, 3, 4]) as Box<[i32]>;

    // Transfer into a BoxRef.
    let arr: BoxRef<[i32]> = BoxRef::new(arr);
    assert_eq!(&*arr, &[1, 2, 3, 4]);

    // We can slice the array without losing ownership or changing type.
    let arr: BoxRef<[i32]> = arr.map(|arr| &arr[1..3]);
    assert_eq!(&*arr, &[2, 3]);

    // Also works for Arc, Rc, String and Vec!
}
```

## Acknowledgement

This repository is based on Marvin Löbel's [owning-ref-rs](https://github.com/Kimundi/owning-ref-rs). Thanks!

## License

<img align="right" src="https://opensource.org/trademarks/opensource/OSI-Approved-License-100x137.png">

The class is licensed under the [MIT License](http://opensource.org/licenses/MIT):

Copyright &copy; 2022 [Momenti Corp](https://github.com/corp-momenti).

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

---

This repository is based on Marvin Löbel's [owning-ref-rs](https://github.com/Kimundi/owning-ref-rs).

The MIT License (MIT)

Copyright (c) 2015 Marvin Löbel

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
