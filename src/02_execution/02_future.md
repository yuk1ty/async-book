<!-- # The `Future` Trait -->
# `Future`トレイト

<!-- The `Future` trait is at the center of asynchronous programming in Rust. -->
<!-- A `Future` is an asynchronous computation that can produce a value -->
<!-- (although that value may be empty, e.g. `()`). A *simplified* version of -->
<!-- the future trait might look something like this: -->

`Future`トレイトはRustの非同期プログラミングの中心です。`Future`は値を返すことのできる（値は空かもしれませんが。例として`()`）非同期計算です。*簡略化された*バージョンのFutureトレイトは下記のようになるでしょう：

```rust
{{#include ../../examples/02_02_future_trait/src/lib.rs:simple_future}}
```

<!-- Futures can be advanced by calling the `poll` function, which will drive the -->
<!-- future as far towards completion as possible. If the future completes, it -->
<!-- returns `Poll::Ready(result)`. If the future is not able to complete yet, it -->
<!-- returns `Poll::Pending` and arranges for the `wake()` function to be called -->
<!-- when the `Future` is ready to make more progress. When `wake()` is called, the -->
<!-- executor driving the `Future` will call `poll` again so that the `Future` can -->
<!-- make more progress. -->

`poll`関数を呼び出すことでFutureを進めることができます。この関数は、完了まで進められる限りFutureの処理を進めます。Futureが完了すると、Futureは`Poll::Ready(result)`を返します。Futureがまだ完了できない場合、`Poll::Pending`を返し、`Future`がもう一度処理を進められる状態になった時に`wake()`関数が呼び出されるよう手配します。`wake()`が呼び出されると、`Future`を稼働するエグゼキュータは`poll`を再度呼び出し、`Future`がさらに処理を進められるようにします。

<!-- Without `wake()`, the executor would have no way of knowing when a particular -->
<!-- future could make progress, and would have to be constantly polling every -->
<!-- future. With `wake()`, the executor knows exactly which futures are ready to -->
<!-- be `poll`ed. -->

`wake()`がなければ、エグゼキュータは特定のFutureがいつ進行できるかを知るすべがありませなん。常にすべてのFutureをポーリングしなければならなくなります。`wake()`によって、エグゼキュータはFutureがポーリング可能かどうかを正確に知ることができるのです。

<!-- For example, consider the case where we want to read from a socket that may -->
<!-- or may not have data available already. If there is data, we can read it -->
<!-- in and return `Poll::Ready(data)`, but if no data is ready, our future is -->
<!-- blocked and can no longer make progress. When no data is available, we -->
<!-- must register `wake` to be called when data becomes ready on the socket, -->
<!-- which will tell the executor that our future is ready to make progress. -->
<!-- A simple `SocketRead` future might look something like this: -->

たとえば、データがあるかどうかわからないソケットからデータを読み出したいケースを考えてみましょう。もしデータがあるのであれば、データを読んで`Poll::Ready(data)`を返せば良いです。しかし準備完了したデータがない場合、Futureはブロックされ、これ以上進行できなくなります。データが利用可能でない場合、ソケットでデータの準備ができたときに呼び出されるよう`wake`を登録しておく必要があります。これにより、準備ができているということがエグゼキュータに通知されます。単純な`SocketRead`のFutureは次のようになるでしょう：

```rust,ignore
{{#include ../../examples/02_02_future_trait/src/lib.rs:socket_read}}
```

<!-- This model of `Future`s allows for composing together multiple asynchronous -->
<!-- operations without needing intermediate allocations. Running multiple futures -->
<!-- at once or chaining futures together can be implemented via allocation-free -->
<!-- state machines, like this: -->

`Future`のこのモデルは、中間アロケーションなしで複数の非同期操作をまとめて行うことも考慮しています。複数のFutureを一度に実行したり、複数のFutureをつないだりするには、下記のコード例のように　アロケーション不要の状態機械を使用して実装します：

```rust,ignore
{{#include ../../examples/02_02_future_trait/src/lib.rs:join}}
```

<!-- This shows how multiple futures can be run simultaneously without needing -->
<!-- separate allocations, allowing for more efficient asynchronous programs. -->
<!-- Similarly, multiple sequential futures can be run one after another, like this: -->

これは別々のアローケーションを必要とせずに同時に複数のFutureを実行し、より効率的な非同期プログラムを実現できることを示しています。同様に下記の例のように、複数の連続したFutureを次々に実行することもできます：

```rust,ignore
{{#include ../../examples/02_02_future_trait/src/lib.rs:and_then}}
```

<!-- These examples show how the `Future` trait can be used to express asynchronous -->
<!-- control flow without requiring multiple allocated objects and deeply nested -->
<!-- callbacks. With the basic control-flow out of the way, let's talk about the -->
<!-- real `Future` trait and how it is different. -->

これらの例は、複数の割り当てオブジェクトや深くネストされたコールバックを必要とせずに、`Future`トレイトを使用して非同期処理フローを表現できることを示しています。基本的な制御フローの説明は一旦ここまでにして、実際の`Future`トレイトとその違いについて説明します。

```rust,ignore
{{#include ../../examples/02_02_future_trait/src/lib.rs:real_future}}
```

<!-- The first change you'll notice is that our `self` type is no longer `&mut Self`, -->
<!-- but has changed to `Pin<&mut Self>`. We'll talk more about pinning in [a later -->
<!-- section][pinning], but for now know that it allows us to create futures that -->
<!-- are immovable. Immovable objects can store pointers between their fields, -->
<!-- e.g. `struct MyFut { a: i32, ptr_to_a: *const i32 }`. Pinning is necessary -->
<!-- to enable async/await. -->

最初の変更点は、`self`型が`&mut Self`ではなくなり、`Pin<&mut Self>`に変わったことです。ピン留めについては[後続の章][pinning]で説明しますが、とりあえずピン留めによってムーブしないFutureを作ることができることを知っておいてください。ムーブしないオブジェクトはフィールド間のポインターを格納することができます。たとえば`struct MyFut { a: i32, ptr_to_a: *const i32 }`のようにです。ピン留めはasync/awaitを可能にするためには必要不可欠です。

<!-- Secondly, `wake: fn()` has changed to `&mut Context<'_>`. In `SimpleFuture`, -->
<!-- we used a call to a function pointer (`fn()`) to tell the future executor that -->
<!-- the future in question should be polled. However, since `fn()` is just a -->
<!-- function pointer, it can't store any data about *which* `Future` called `wake`. -->

次に、`wake: fn()`は`&mut Context<'_>`へと変更されています。`SimpleFuture`では、関数ポインタの呼び出し（`fn()`）を使って、対象となるFutureがポーリングされるべきであるとFutureのエグゼキュータに対して伝達していました。しかし、`fn()`は単なる関数ポインタであるため、*どの*Futureが`wake`を呼び出したかについてのいかなるデータも格納しておくことができません。[^1]

<!-- In a real-world scenario, a complex application like a web server may have -->
<!-- thousands of different connections whose wakeups should all be -->
<!-- managed separately. The `Context` type solves this by providing access to -->
<!-- a value of type `Waker`, which can be used to wake up a specific task. -->

実際のシナリオでは、Webサーバーのような複雑なアプリケーションには何千もの異なる接続があり、それらのウェイクアップ（wakeup）はすべて個別に管理する必要があります。`Context`型は`Waker`型の値へのアクセスを提供することでこれを解決し、特定のタスクを起こすのに使うことができます。

[pinning]: ../04_pinning/01_chapter.md

---

## 訳註

[^1]: ここは少々わかりにくいが、要するに関数の中には構造体でいうフィールドのようなものは持たせられないため、`Context`のような型を用意してそこに現在の状態を持たせておくことで、「どのタスクを起こしたか」の状態の管理を実現している。実際後述されるように、`Context`型は`Waker`を中に持っていて、`wake`を呼び出す際にはこの`Context`を経由して行うことになる。
