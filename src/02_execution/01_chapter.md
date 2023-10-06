<!-- # Under the Hood: Executing `Future`s and Tasks -->
# `Future`とタスクを実行する内部構造

<!-- In this section, we'll cover the underlying structure of how `Future`s and -->
<!-- asynchronous tasks are scheduled. If you're only interested in learning -->
<!-- how to write higher-level code that uses existing `Future` types and aren't -->
<!-- interested in the details of how `Future` types work, you can skip ahead to -->
<!-- the `async`/`await` chapter. However, several of the topics discussed in this -->
<!-- chapter are useful for understanding how `async`/`await` code works, -->
<!-- understanding the runtime and performance properties of `async`/`await` code, -->
<!-- and building new asynchronous primitives. If you decide to skip this section -->
<!-- now, you may want to bookmark it to revisit in the future. -->

このセクションでは、`Future`と非同期タスクがスケジューリングされる仕組みについて解説します。もし仮に既存の`Future`型を使った高レイヤーでのコードの書き方にのみ興味があり、`Future`型がどのように動作するかについてはそこまで興味がない場合、`async`/`await`の章まで読み飛ばすことができます。しかし、この章でかわされるいくつかのトピックは`async`/`await`のコードの仕組みを理解する上で役立ちますし、ランタイムや`async`/`await`コードのパフォーマンス特性や新しい非同期プリミティブを構築する際に役に立ちます。読み飛ばしすると決めた場合でも、ここにしおりを挟んでおいて、将来もう一度戻ってきたくなるかもしれませんね。

<!-- Now, with that out of the way, let's talk about the `Future` trait. -->

さて、それはさておき、`Future`トレイトを解説していきましょう。
