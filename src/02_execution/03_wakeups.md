<!-- # Task Wakeups with `Waker` -->
# `Waker`でタスクを起こす

<!-- It's common that futures aren't able to complete the first time they are -->
<!-- `poll`ed. When this happens, the future needs to ensure that it is polled -->
<!-- again once it is ready to make more progress. This is done with the `Waker` -->
<!-- type. -->

Futureが最初の`poll`でタスクを完了できないのはよくあることです。このようなケースでは、Futureが処理を進める準備ができたら、再びポーリングされるようにする必要があります。これは`Waker`型により達成されます。

<!-- Each time a future is polled, it is polled as part of a "task". Tasks are -->
<!-- the top-level futures that have been submitted to an executor. -->

Futureはポーリングされるたび、「タスク」の一部としてポーリングされます。タスクはエグゼキュータに送られたトップレベルのFutureです。

<!-- `Waker` provides a `wake()` method that can be used to tell the executor that -->
<!-- the associated task should be awoken. When `wake()` is called, the executor -->
<!-- knows that the task associated with the `Waker` is ready to make progress, and -->
<!-- its future should be polled again. -->

`Waker`は`wake()`メソッドを提供しており、このメソッドを使って、紐づけられたタスクを起動するようエグゼキュータに指示することができます。`wake()`が呼び出されると、エグゼキュータは`Waker`と紐づけられたタスクが実行可能になったと知ります。そして、フューチャーは再びポーリングされなければなりません。

<!-- `Waker` also implements `clone()` so that it can be copied around and stored. -->

`Waker`は、コピーしたり保存したりできるよう、`clone()`も実装しています。

<!-- Let's try implementing a simple timer future using `Waker`. -->

`Waker`を使った簡単なタイマーを実装してみましょう。

<!-- ## Applied: Build a Timer -->

## 応用編：タイマーを作る

<!-- For the sake of the example, we'll just spin up a new thread when the timer -->
<!-- is created, sleep for the required time, and then signal the timer future -->
<!-- when the time window has elapsed. -->

この例では、タイマーガー生成された時に新しいスレッドを立ち上げ、必要な時間スリープさせ、一定時間経過したところでタイマー用のFutureにシグナルを送ります。

<!-- First, start a new project with `cargo new --lib timer_future` and add the imports -->
<!-- we'll need to get started to `src/lib.rs`: -->

まずはじめに、`cargo new --lib timer_future`で新しいプロジェクトを立ち上げ、`src/lib.rs`に作業を開始するのに必要なインポート類を追加しておきましょう。

```rust
{{#include ../../examples/02_03_timer/src/lib.rs:imports}}
```

<!-- Let's start by defining the future type itself. Our future needs a way for the -->
<!-- thread to communicate that the timer has elapsed and the future should complete. -->
<!-- We'll use a shared `Arc<Mutex<..>>` value to communicate between the thread and -->
<!-- the future. -->

Future自体を定義するところからはじめましょう。このフューチャーには、スレッドが、タイマーが経過してFutureが完了したということを伝えるための方法が必要です。スレッドとFutureの間の通信には、共有された`Arc<Mutex<...>>`を用います。

```rust,ignore
{{#include ../../examples/02_03_timer/src/lib.rs:timer_decl}}
```

<!-- Now, let's actually write the `Future` implementation! -->

ここで、実際に`Future`の実装を書いてみましょう。

```rust,ignore
{{#include ../../examples/02_03_timer/src/lib.rs:future_for_timer}}
```

<!-- Pretty simple, right? If the thread has set `shared_state.completed = true`, -->
<!-- we're done! Otherwise, we clone the `Waker` for the current task and pass it to -->
<!-- `shared_state.waker` so that the thread can wake the task back up. -->

とてもシンプルではありませんか？スレッドに`shared_state.completed = true`がセットされれば完了です。セットできない場合には、現在のタスクの`Waker`をクローンして`shared_state.waker`に渡し、スレッドがタスクを起こせるようにしておきます。

<!-- Importantly, we have to update the `Waker` every time the future is polled -->
<!-- because the future may have moved to a different task with a different -->
<!-- `Waker`. This will happen when futures are passed around between tasks after -->
<!-- being polled. -->

重要なのは、Futureがポーリングされるたびに`Waker`を更新しなければならないということです。Futureは別の`Waker`とともに別のタスクにムーブするかもしれないからです。これはポーリングされたFutureがタスク間で受け渡しされる際に怒ります。

<!-- Finally, we need the API to actually construct the timer and start the thread: -->

最後に、実際にタイマーを構築してスレッドをスタートさせるAPIが必要です：

```rust,ignore
{{#include ../../examples/02_03_timer/src/lib.rs:timer_new}}
```

<!-- Woot! That's all we need to build a simple timer future. Now, if only we had -->
<!-- an executor to run the future on... -->

やりましたね。シンプルなタイマー型のFutureを構築するために必要なことは、これで全てです。これであとは、Futureを実行するエグゼキュータがあれば…。
