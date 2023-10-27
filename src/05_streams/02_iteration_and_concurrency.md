<!-- # Iteration and Concurrency -->

# イテレーションと並行性

<!--
Similar to synchronous `Iterator`s, there are many different ways to iterate
over and process the values in a `Stream`. There are combinator-style methods
such as `map`, `filter`, and `fold`, and their early-exit-on-error cousins
`try_map`, `try_filter`, and `try_fold`.
-->

同期イテレータと同様に、`Stream`内の値をイテレーションする方法にはさまざまなものがあります。`map`、`filter`、`fold`といったコンビネータ形式のメソッドや、エラー時に早期終了する`try_map`、`try_filter`、`try_fold`といった親類のメソッドがあります。

<!--
Unfortunately, `for` loops are not usable with `Stream`s, but for
imperative-style code, `while let` and the `next`/`try_next` functions can
be used:
-->

残念ながら`for`ループは`Stream`では使えませんが、命令型のコードでは、`while let`や`next/try_next`関数を利用することができます：

```rust,edition2018,ignore
{{#include ../../examples/05_02_iteration_and_concurrency/src/lib.rs:nexts}}
```

<!--
However, if we're just processing one element at a time, we're potentially
leaving behind opportunity for concurrency, which is, after all, why we're
writing async code in the first place. To process multiple items from a stream
concurrently, use the `for_each_concurrent` and `try_for_each_concurrent`
methods:
-->

しかしもし仮に一度にひとつだけの要素を処理している場合、並行処理の機会を逃してしまってる可能性があります。これがそもそも非同期処理コードを書いている理由です。ストリームから複数のアイテムを並行的に処理するには、`for_each_concurrent`ならびに`try_for_each_concurrent`メソッドを使用します：

```rust,edition2018,ignore
{{#include ../../examples/05_02_iteration_and_concurrency/src/lib.rs:try_for_each_concurrent}}
```
