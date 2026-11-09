# 付録B 主なイベント早見表

## この付録の使い方

SSP から SHIORI に届く主なイベントを、種類ごとに表にまとめました。イベントとは何か、Reference の読み方、`GET` と `NOTIFY` の違いは [第7章](07-shiori-events.md) を見てください。ここにないイベントや、Reference のくわしい意味は UKADOC の「SHIORI Event リスト」で確かめてください（[付録D](appendix-d-links.md)）。

- 「主な Reference」は、よく使うものだけを抜き出しています。番号は 0 から数えます。里々では `（Ｒ０）`、YAYA では `reference[0]` のように読みます。
- 「SSP」と書いたものは、SSP 以外のベースウェアでは届かないことがあります。
- 「通知」と書いたものは `NOTIFY` で届くため、セリフを返しても話しません。

## 起動・終了・切り替え

| イベント | いつ来るか | 主な Reference |
| --- | --- | --- |
| `OnFirstBoot` | はじめて起動したとき。SSP では、返事がないと続けて `OnBoot` が起きる | 0：消滅（アンインストール）された回数 |
| `OnBoot` | 起動したとき | 0：起動時のシェル名 |
| `OnClose` | 終了が指示されたとき | 0（SSP）：`user`（ユーザー操作）か `system`（Windows の終了） |
| `OnCloseAll` | SSP 全体が終了するとき（SSP）。返事がないと続けて `OnClose` が起きる | 0：`user` か `system` |
| `OnGhostChanging` | 別のゴーストに交代するとき（去る側）。SSP では、返事がないと続けて `OnClose` が起きる | 0：交代先の \0 側の名前、1：`manual`（手動）か `automatic`（自動） |
| `OnGhostChanged` | 別のゴーストから交代してきたとき（来た側）。SSP では、返事がないと続けて `OnBoot` が起きる | 0：交代前のゴーストの \0 側の名前、1：交代前のゴーストが交代のときに話したスクリプト |
| `OnGhostCalled` | ほかのゴーストに呼び出されて起動したとき（SSP）。返事がないと続けて `OnBoot` が起きる | 0：呼んだゴーストの \0 側の名前 |
| `OnShellChanged` | シェルが切り替わったとき | 0：いまのシェル名 |
| `OnWindowStateMinimize` | 最小化されたとき | 0（SSP）：最小化の理由 |
| `OnWindowStateRestore` | 最小化が元に戻ったとき | 0（SSP）：最小化された理由 |
| `OnInitialize` | 起動したとき（SSP・通知）。SHIORI の準備用 | 0：再読み込みのときは `reload` |
| `OnDestroy` | 終了が指示されたとき（SSP など・通知）。SHIORI の後片付け用 | 0：再読み込みのときは `reload` |

## マウス

マウスのイベントは、多くが同じ並びの Reference を持っています。

| イベント | いつ来るか | 主な Reference |
| --- | --- | --- |
| `OnMouseClick` | クリックされたとき | 0・1：x・y 座標、3：キャラクター（\0 側 `0`、\1 側 `1`）、4：当たり判定の名前、5：ボタン（左 `0`、右 `1`） |
| `OnMouseDoubleClick` | ダブルクリックされたとき | 同上 |
| `OnMouseMove` | キャラクターの上でマウスが動いたとき（とても多く届く） | 0・1：x・y 座標、3：キャラクター、4：当たり判定の名前 |
| `OnMouseWheel` | キャラクターの上でホイールが回ったとき | 2：回転の量と向き、3：キャラクター、4：当たり判定の名前 |
| `OnMouseEnter` | マウスが当たり判定に入ったとき（SSP など） | 3：キャラクター、4：当たり判定の名前 |
| `OnMouseLeave` | マウスが当たり判定から出たとき（SSP など） | 3：キャラクター、4：当たり判定の名前 |

`OnMouseMove` が要らないゴーストは、ゴースト側 `descript.txt` に `don't need onmousemove,1` と書くと届かなくなります。

## 時間

| イベント | いつ来るか | 主な Reference |
| --- | --- | --- |
| `OnSecondChange` | 1 秒ごと。話せないときは通知で届く | 3：話せるなら `1`、話せないなら `0`、4（SSP）：パソコンが操作されていない秒数 |
| `OnMinuteChange` | 1 分ごと。話せないときは通知で届く | 同上 |
| `OnHourTimeSignal` | 毎正時（SSP）。話せるようになるまで待って届く | 4：パソコンが操作されていない秒数 |
| `OnAITalk` | `\a` タグなどで「ランダムトークを話して」と求められたとき（SSP） | なし |

## 選択肢・アンカー

| イベント | いつ来るか | 主な Reference |
| --- | --- | --- |
| `OnChoiceSelectEx` | 選択肢が選ばれたとき（SSP）。`OnChoiceSelect` より先に届く | 0：表示の文字、1：ID、2 以降：`\q` の 3 番目以降の値 |
| `OnChoiceSelect` | 選択肢が選ばれたとき | 0：ID |
| `OnChoiceTimeout` | 選択肢が時間切れになったとき | 0：時間切れになったスクリプト |
| `OnAnchorSelect` | `\_a` のアンカーがクリックされたとき | 0：ID |
| （`\q` に書いた `On` で始まる名前） | ID が `On` で始まる選択肢が選ばれたとき | 0 以降：`\q` の 3 番目以降の値 |

## 入力・キーボード

| イベント | いつ来るか | 主な Reference |
| --- | --- | --- |
| `OnUserInput` | `\![open,inputbox,...]` の入力が決定されたとき | 0：ID、1：入力された文字 |
| `OnUserInputCancel` | 入力の箱が閉じられた・時間切れになったとき（SSP など） | 0：ID、1：`timeout` か `close` |
| `OnCommunicate` | コミュニケートボックスやほかのゴーストから話しかけられたとき | 0：`user` かゴーストの \0 側の名前、1：話しかけられた内容 |
| `OnKeyPress` | ベースウェア（ゴースト）がアクティブな状態でキーが押されたとき | 0：押されたキー |

## 消滅（アンインストール）

| イベント | いつ来るか | 主な Reference |
| --- | --- | --- |
| `OnVanishSelecting` | 消滅が指示されたとき | なし |
| `OnVanishSelected` | 確認で「はい」が選ばれたとき | なし |
| `OnVanishCancel` | 確認で「いいえ」が選ばれたとき | なし |
| `OnVanished` | 前のゴーストが消滅して、自分に交代してきたとき。SSP では、返事がないと続けて `OnBoot` が起きる | 0：消滅したゴーストの \0 側の名前 |

## そのほか

| イベント | いつ来るか | 主な Reference |
| --- | --- | --- |
| `OnSurfaceChange` | サーフェスが変わったとき（SSP など・通知） | 0：\0 側のいまのサーフェス番号、1：\1 側のいまのサーフェス番号 |
| `OnBalloonBreak` | 話している途中でユーザーに中断されたとき（SSP など） | 0：中断されたスクリプト、1：中断されたバルーンのキャラクター |
| `OnFileDrop2` | ファイルがドラッグ＆ドロップされたとき | 0：ファイルのパス、1：落とされたキャラクター |
| `OnInstallComplete` | ゴーストやバルーンなどのインストールが終わったとき | 1：インストールしたものの名前 |
| `OnUpdateComplete` | ネットワーク更新が成功して終わったとき | 0：成功の理由、1：更新されたファイルの一覧 |
| `OnTranslate` | SHIORI が返したスクリプトを、表示する前にもう一度手直しするためのイベント | 0：元のスクリプト |

---

[← 前の章](appendix-a-sakura-script.md) ｜ [目次](../README.md) ｜ [次の章 →](appendix-c-glossary.md)
