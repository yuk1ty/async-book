<!-- # The `Stream` Trait -->

# `Stream`トレイト

<!--
The `Stream` trait is similar to `Future` but can yield multiple values before
completing, similar to the `Iterator` trait from the standard library:
-->

`Stream`トレイトは`Future`に似ていますが、標準ライブラリの`Iterator`に似たもので、完了するまでに複数の値を返すことができます：

```rust,ignore
{{#include ../../examples/05_01_streams/src/lib.rs:stream_trait}}
```

<!--
One common example of a `Stream` is the `Receiver` for the channel type from
the `futures` crate. It will yield `Some(val)` every time a value is sent
from the `Sender` end, and will yield `None` once the `Sender` has been
dropped and all pending messages have been received:
-->

`Stream`のもっとも一般的な例は、`futures`クレートにある`Receiver`というチャネル型でしょう。`Receiver`は`Sender`から値が送信されるたびに`Some(val)`を返し、`Sender`が削除され、保留されていたメッセージがすべて受信されると`None`を返します。

```rust,edition2018,ignore
{{#include ../../examples/05_01_streams/src/lib.rs:channels}}
```
