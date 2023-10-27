# `join!`

<!--
The `futures::join` macro makes it possible to wait for multiple different
futures to complete while executing them all concurrently.
-->

`futures::join`マクロは、複数の別々のFutureが並行に実行されている間、それらを待つことができます。

## `join!`

<!--
When performing multiple asynchronous operations, it's tempting to simply
`.await` them in a series:
-->

複数の非同期処理を行う際、単に一連の処理として`.await`したくなります：

```rust,edition2018,ignore
{{#include ../../examples/06_02_join/src/lib.rs:naiive}}
```

However, this will be slower than necessary, since it won't start trying to
`get_music` until after `get_book` has completed. In some other languages,
futures are ambiently run to completion, so two operations can be
run concurrently by first calling each `async fn` to start the futures, and
then awaiting them both:

しかし、これでは必要以上に遅くなります。というのも、`get_book`が完了するまで`get_music`が開始されないからです。他の言語では、Futureは完了するまでバックグラウンドで実行されます。そのため、まずそれぞれの`async fn`を呼びだしてFutureを開始し、それから両方を待機させることで、ふたつの処理を並行に実行することができます：

```rust,edition2018,ignore
{{#include ../../examples/06_02_join/src/lib.rs:other_langs}}
```

However, Rust futures won't do any work until they're actively `.await`ed.
This means that the two code snippets above will both run
`book_future` and `music_future` in series rather than running them
concurrently. To correctly run the two futures concurrently, use
`futures::join!`:

しかし、RustのFutureは、能動的に`.await`されるまで何もしません。

```rust,edition2018,ignore
{{#include ../../examples/06_02_join/src/lib.rs:join}}
```

The value returned by `join!` is a tuple containing the output of each
`Future` passed in.

## `try_join!`

For futures which return `Result`, consider using `try_join!` rather than
`join!`. Since `join!` only completes once all subfutures have completed,
it'll continue processing other futures even after one of its subfutures
has returned an `Err`.

Unlike `join!`, `try_join!` will complete immediately if one of the subfutures
returns an error.

```rust,edition2018,ignore
{{#include ../../examples/06_02_join/src/lib.rs:try_join}}
```

Note that the futures passed to `try_join!` must all have the same error type.
Consider using the `.map_err(|e| ...)` and `.err_into()` functions from
`futures::future::TryFutureExt` to consolidate the error types:

```rust,edition2018,ignore
{{#include ../../examples/06_02_join/src/lib.rs:try_join_map_err}}
```
