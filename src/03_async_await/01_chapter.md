# `async`/`.await`

<!-- In [the first chapter], we took a brief look at `async`/`.await`. -->
<!-- This chapter will discuss `async`/`.await` in -->
<!-- greater detail, explaining how it works and how `async` code differs from -->
<!-- traditional Rust programs. -->

[最初の章]で、`async`/`.await`を手短に確認しました。この章ではかなり詳しく`async`/`.await`を見ていきます。具体的には`async`/`.await`がどのような仕組みになっているのかや、従来のRustのコードとはどのように`async`のコードが異なっているかについて説明します。

<!-- `async`/`.await` are special pieces of Rust syntax that make it possible to -->
<!-- yield control of the current thread rather than blocking, allowing other -->
<!-- code to make progress while waiting on an operation to complete. -->

`async`/`.await`は、使用中のスレッドをブロッキングせず、スレッドの制御を委譲することを可能にするRustの特別な構文で、操作の完了を待っている間であっても他のコードが処理を進行できるようにします。

<!-- There are two main ways to use `async`: `async fn` and `async` blocks. -->
<!-- Each returns a value that implements the `Future` trait: -->

`async`には2通りの使用法があります：`async fn`と`async`ブロックです。どちらも`Future`トレイトを実装した値を返します：

```rust,edition2018,ignore
{{#include ../../examples/03_01_async_await/src/lib.rs:async_fn_and_block_examples}}
```

<!-- As we saw in the first chapter, `async` bodies and other futures are lazy: -->
<!-- they do nothing until they are run. The most common way to run a `Future` -->
<!-- is to `.await` it. When `.await` is called on a `Future`, it will attempt -->
<!-- to run it to completion. If the `Future` is blocked, it will yield control -->
<!-- of the current thread. When more progress can be made, the `Future` will be picked -->
<!-- up by the executor and will resume running, allowing the `.await` to resolve. -->

最初の章で確認したように、`async`のボディ部とその他のFutureは遅延評価です。つまり、実行されるまで何もしません。`Future`を実行させるもっとも一般的な方法は、`.await`です。`Future`に対して`.await`が呼び出されると、完了するまでそのFutureを実行しようとします。実行した`Future`がブロックされる際には、使用中のスレッドの制御を放棄します。さらにそのFutureの処理が進むと、`Future`はエグゼキュータ（executor）に拾われて実行を再開し、`.await`を解決できるようになります。

<!-- ## `async` Lifetimes -->
## `async`のライフタイム

<!-- Unlike traditional functions, `async fn`s which take references or other -->
<!-- non-`'static` arguments return a `Future` which is bounded by the lifetime of -->
<!-- the arguments: -->

従来の関数とは異なり、参照をとったり、他の`'static`でない引数をとったりする`async fn`は、引数のライフタイムに制限された`Future`を返します：

```rust,edition2018,ignore
{{#include ../../examples/03_01_async_await/src/lib.rs:lifetimes_expanded}}
```

<!-- This means that the future returned from an `async fn` must be `.await`ed -->
<!-- while its non-`'static` arguments are still valid. In the common -->
<!-- case of `.await`ing the future immediately after calling the function -->
<!-- (as in `foo(&x).await`) this is not an issue. However, if storing the future -->
<!-- or sending it over to another task or thread, this may be an issue. -->

つまり、`async fn`から返されるFutureは、`'static`でない引数のライフタイムが続く限りは`.await`されなければなりません。関数を呼び出した直後に`.await`する一般的なケース（`foo(&x).await`のように）においては、これは問題になりません。しかし一方で、Futureを格納したり、Futureを別のタスクやスレッドに送ったりする場合には問題になる可能性があります。

<!-- One common workaround for turning an `async fn` with references-as-arguments -->
<!-- into a `'static` future is to bundle the arguments with the call to the -->
<!-- `async fn` inside an `async` block: -->

参照を引数に持つ`async fn`を`'static`なFutureにするための一般的な回避策の一つは、引数を`async`ブロック内の`async fn`への呼び出しに巻き込んでおくことです：

```rust,edition2018,ignore
{{#include ../../examples/03_01_async_await/src/lib.rs:static_future_with_borrow}}
```

<!-- By moving the argument into the `async` block, we extend its lifetime to match -->
<!-- that of the `Future` returned from the call to `good`. -->

引数を`async`ブロックの中に移動することにより、`good`の呼び出しから返される`Future`のライフタイムと同じになるように、ライフタイムを引き延ばします。

## `async move`

<!-- `async` blocks and closures allow the `move` keyword, much like normal -->
<!-- closures. An `async move` block will take ownership of the variables it -->
<!-- references, allowing it to outlive the current scope, but giving up the ability -->
<!-- to share those variables with other code: -->

通常のクロージャーと同様に、`async`ブロックとクロージャーは`move`キーワードを使用することができます。`async block`はそのブロックが参照する変数の所有権をとり、現在のスコープより長生きさせられますが、変数を他のコードと共有することはできなくなります：

```rust,edition2018,ignore
{{#include ../../examples/03_01_async_await/src/lib.rs:async_move_examples}}
```

<!-- ## `.await`ing on a Multithreaded Executor -->
## マルチスレッド化されたエグゼキュータで`.await`する

<!-- Note that, when using a multithreaded `Future` executor, a `Future` may move -->
<!-- between threads, so any variables used in `async` bodies must be able to travel -->
<!-- between threads, as any `.await` can potentially result in a switch to a new -->
<!-- thread. -->

マルチスレッドの`Future`のエグゼキュータを使用する場合、`Future`がスレッドをまたいでムーブする可能性があることに注意しましょう。そのため、`async`のボディで使用される変数はスレッド間を移動できなければなりません。なぜなら、`.await`は新しいスレッドに切り替わる可能性があるからです。

<!-- This means that it is not safe to use `Rc`, `&RefCell` or any other types -->
<!-- that don't implement the `Send` trait, including references to types that don't -->
<!-- implement the `Sync` trait. -->

つまり、`Rc`や`&RefCell`など`Send`トレイトを実装していない型や、`Sync`トレイトを実装しない型への参照などは安全ではありません。

<!-- (Caveat: it is possible to use these types as long as they aren't in scope -->
<!-- during a call to `.await`.) -->

（注意：`.await`の呼び出し中にスコープ内にない限りは、これらの型を使用可能です。）

<!-- Similarly, it isn't a good idea to hold a traditional non-futures-aware lock -->
<!-- across an `.await`, as it can cause the threadpool to lock up: one task could -->
<!-- take out a lock, `.await` and yield to the executor, allowing another task to -->
<!-- attempt to take the lock and cause a deadlock. To avoid this, use the `Mutex` -->
<!-- in `futures::lock` rather than the one from `std::sync`. -->

同様に、`.await`の呼び出し中に従来の、Futureを意識しないロックを保持するのはよくありません。スレッドプールがロックされる可能性があるためです。あるタスクがロックを取得して`.await`し、エグゼキュータに委譲すると、別のタスクがロックを取得しようとしてデッドロックを引き起こす可能性があります。これを避けるには、`std::sync`の`Mutex`ではなく、`futures::lock`の`Mutex`を使用します。

[最初の章]: ../01_getting_started/04_async_await_primer.md
