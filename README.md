# fizz-comment-dedupe

**Fizz** コメントフィルタ部品 — comment ストリームから既出 ID を落とす。
NDJSON in → NDJSON out の純粋なパイプフィルタ。

責務は一行: **comment stream → 重複を除いた comment stream**。
ソースの再送・再接続・polling の重なりをここで一手に吸収するので、
各ソース部品 (fizz-youtube-chat-source 等) は dedupe を実装しない。

## Usage

```bash
almide build src/main.almd -o fizz-comment-dedupe

fizz-youtube-chat-source | ./fizz-comment-dedupe | ...

# 複数ソースの合流にも (シェルの process substitution などで)
cat <(fizz-youtube-chat-source) <(fizz-twitch-chat-source) | ./fizz-comment-dedupe
```

不正な行は stderr に警告して読み飛ばす (パイプは止めない)。

## 挙動

- seen set は FIFO 上限つき (default 512)。上限を超えると最古 ID から
  忘れる — 上限より昔の再送は素通りする trade-off (旧 openaituber の
  youtube bridge と同じ方針)
- 状態は純粋関数 `push(state, comment) -> {state, fresh}` で更新され、
  CLI はその薄いラッパ

## Env

| var | 意味 | default |
|---|---|---|
| `FIZZ_DEDUPE_CAP` | seen set の上限 | 512 |

## Tests

```bash
almide test
```
