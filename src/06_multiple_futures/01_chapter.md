<!-- # Executing Multiple Futures at a Time -->

# 複数のFutureを同時に実行する

<!--
Up until now, we've mostly executed futures by using `.await`, which blocks
the current task until a particular `Future` completes. However, real
asynchronous applications often need to execute several different
operations concurrently.
-->

これまでのところ、特定の`Future`が完了するまで現在のタスクをブロックする`.await`を使ってFutureを実行してきました。しかし、実際の非同期アプリケーションでは、異なるいくつかの操作を並行に実行する必要があります。

<!--
In this chapter, we'll cover some ways to execute multiple asynchronous
operations at the same time:
-->

この章では、複数の非同期操作を同時実行するいくつかの方法について説明します：

<!--
- `join!`: waits for futures to all complete
- `select!`: waits for one of several futures to complete
- Spawning: creates a top-level task which ambiently runs a future to completion
- `FuturesUnordered`: a group of futures which yields the result of each subfuture
-->

- `join!`: すべてのFutureが完了するのを待つ。
- `select!`: いくつかのFutureのうちひとつが完了するのを待つ。
- スポーン（Spawning）: あるFutureが完了するまで実行されるトップレベルのタスクを生成する。
- `FuturesUnordered`: 各子Futureの結果を返すFutureのグループ。
