# 第7章 SHIORI とイベントのしくみ

## この章で学ぶこと

- ベースウェア（SSP）と SHIORI がどんな関係で働いているか
- SHIORI/3.0 のリクエストとレスポンスの形（`GET`、`ID`、`Reference0`…、`Value`）
- 起動・終了・マウス・時間・選択肢など、代表的なイベント
- ランダムトークがどんなしくみで動いているか
- Reference の読み方と、返事をしなくてよい「通知」のイベント

## SSP と SHIORI の関係

第1章で、ゴーストが話すまでの流れを見ました。この章では、その中心にある **SSP と SHIORI のやりとり** をもう少し詳しく見ていきます。ここがわかると、里々や YAYA で「何を書けばよいのか」がはっきりします。

### SHIORI は「問い合わせに答える係」

**SHIORI** は、ゴーストの `ghost/master` に置かれた DLL（里々なら `satori.dll`、YAYA なら `yaya.dll`）です。SSP はゴーストを起動するとき、`descript.txt` の `shiori` の行に書かれた DLL を読み込みます。

SSP と SHIORI の役割分担は、とてもはっきりしています。

| | SSP（ベースウェア） | SHIORI |
| --- | --- | --- |
| すること | 画面表示、マウスやタイマーの見張り、バルーンへの表示 | 辞書を調べて、返事のセリフを組み立てる |
| 自分から話しかけるか | 話しかける（「こんなことが起きました」） | 話しかけない。聞かれたときだけ答える |

SHIORI は、自分から勝手に動くことはありません。SSP から **リクエスト**（問い合わせ）が来たときだけ働き、**レスポンス**（返事）を返します。この一往復が、ゴーストの動きのすべての基本です。

```text
SSP ──リクエスト──▶ SHIORI   「OnBoot が起きました。何か言いますか？」
SSP ◀──レスポンス── SHIORI   「\0\s[0]おはよう、ユーザーさん！\e と言ってください」
```

> **コラム：DLL の入口はわずか**
> SHIORI の DLL には、読み込まれたときに呼ばれる `load`（SSP 2.6.92 以降は `loadu` も）、リクエストを受け取る `request`、終了時に呼ばれる `unload` という入口があります。ゴースト作者がこれを直接さわることはありませんが、「すべての問い合わせは `request` という一つの窓口を通る」と知っておくと、イメージしやすくなります。

## リクエストとレスポンスの形

SSP と SHIORI がやりとりする文章の決まりを **SHIORI/3.0** といいます。Web の HTTP に似た、1 行目に命令、2 行目以降に「名前: 値」を並べる形のテキストです。

### リクエスト（SSP → SHIORI）

たとえば、ゴーストが起動したときに SSP から送られるリクエストは、だいたい次のような形です。

```text
GET SHIORI/3.0
Charset: UTF-8
Sender: SSP
SecurityLevel: local
ID: OnBoot
Reference0: master
```

| 行 | 意味 |
| --- | --- |
| `GET SHIORI/3.0` | 1 行目。`GET` は「何か返してほしい」という問い合わせ |
| `Charset` | 文字コード |
| `Sender` | 送り主（SSP） |
| `SecurityLevel` | `local`（このパソコンの中から）か `external`（外部から） |
| `ID` | **イベント名**。何が起きたかを表す。ここでは `OnBoot`（起動した） |
| `Reference0`, `Reference1`, … | イベントについての **追加情報**。番号は 0 から始まる |

いちばん大事なのは **`ID`** と **`Reference`** です。SHIORI は `ID` を見て「どの出来事か」を知り、`Reference` を見て「どんな状況か」を知ります。

1 行目の命令には、`GET` のほかに **`NOTIFY`** があります。`NOTIFY` は「知らせるだけなので、返事はいらない」という意味です。`NOTIFY` で送られたイベントにセリフを返しても、SSP はそれを使いません。同じイベント名が、状況によって `GET` で来たり `NOTIFY` で来たりすることもあります。

### レスポンス（SHIORI → SSP）

SHIORI は、話したいことがあれば、次のように返します。

```text
SHIORI/3.0 200 OK
Charset: UTF-8
Value: \0\s[0]おはよう、ユーザーさん！\w9\1\s[10]オレもいるぴ。\e
```

| 行 | 意味 |
| --- | --- |
| `SHIORI/3.0 200 OK` | 成功して、返すもの（Value）がある |
| `SHIORI/3.0 204 No Content` | 成功したが、返すものはない（何も話さない） |
| `Value` | **話したいさくらスクリプト** |

実際には、このほかに送り主（SHIORI の名前）を表す `Sender` などの行が入ることもあります。

`Value` に入ったさくらスクリプトを、SSP が実行してバルーンに表示します。何も話さなくてよいときは `204 No Content` を返します。

> **コラム：本物のやりとりをのぞいてみる**
> SSP の開発用パレット（`Ctrl` + `Shift` + `D`、開発者用機能が必要）で「SHIORI呼出ログ有効」をオンにすると、SSP と SHIORI のやりとりが `ssp_SHIORI_log.txt` としてゴーストの `master` フォルダに記録されます。1 秒ごとに `OnSecondChange` が届いているようすなどが見られて面白いですよ。ただしファイルがどんどん大きくなるので、確かめ終わったらオフに戻し、配布物にも含めないようにしましょう。

### 里々や YAYA は何をしてくれるのか

このリクエストとレスポンスを毎回自分で組み立てるのは大変です。里々や YAYA は、その部分を引き受けてくれます。作者は「このイベントが来たら、このセリフを返す」とだけ書けばよいのです。

里々では、イベント名と同じ名前の見出しを `＊` で書きます。

```text
＊OnBoot
：おはよう、ユーザーさん！
：オレもいるぴ。
```

YAYA では、イベント名と同じ名前の関数を書き、返したいさくらスクリプトを文字列で書きます。

```c
OnBoot
{
	"\0\s[0]おはよう、ユーザーさん！\w9\1\s[10]オレもいるぴ。\e"
}
```

どちらも、「`ID: OnBoot` のリクエストが来たら、このセリフを `Value` に入れて返す」という意味になります。書き方の詳しいルールは、里々は [第8章](08-satori-basics.md)、YAYA は [第10章](10-yaya-basics.md) で学びます。

## 代表的なイベント

イベントはとても数が多いので、まずはよく使うものから覚えましょう。全体の一覧は [付録B](appendix-b-events.md) にまとめてあります。

### 起動と終了

| イベント | いつ来るか | 主な Reference |
| --- | --- | --- |
| `OnFirstBoot` | はじめて起動したとき | Reference0：消滅（アンインストール）された回数 |
| `OnBoot` | 起動したとき | Reference0：起動時のシェル名 |
| `OnClose` | 終了が指示されたとき | Reference0（SSP）：ユーザーの操作なら `user`、Windows の終了なら `system` |
| `OnGhostChanging` | 別のゴーストへ交代するとき（交代する側） | Reference0：交代先の \0 側の名前 |
| `OnGhostChanged` | 別のゴーストから交代してきたとき | Reference0：交代前のゴーストの \0 側の名前 |

SSP では、**返事がなかったときの代わり** が決まっているイベントがあります。たとえば `OnFirstBoot`、`OnGhostChanged` に何も返さなかった（204 を返した）ときは、続けて `OnBoot` が起きます。同じように、`OnGhostChanging` に何も返さなかったときは、続けて `OnClose` が起きます。つまり、最低限 `OnBoot` と `OnClose` を書いておけば、起動と終了のあいさつはひととおり話してくれます。

終了のあいさつの最後には、第6章で学んだ `\-` を書いて、話し終えたあとにゴーストを終了させます。

### マウス

| イベント | いつ来るか |
| --- | --- |
| `OnMouseClick` | クリックされたとき |
| `OnMouseDoubleClick` | ダブルクリックされたとき |
| `OnMouseMove` | キャラクターの上でマウスが動いたとき |
| `OnMouseWheel` | キャラクターの上でホイールが回されたとき |

マウスのイベントには、共通して次の Reference が入っています。

| Reference | 中身 |
| --- | --- |
| Reference0, 1 | マウスの x 座標、y 座標（キャラクターの画像の中での位置） |
| Reference3 | どのキャラクターか。\0 側なら `0`、\1 側なら `1` |
| Reference4 | **当たり判定** の名前（頭、顔など。シェルの `surfaces.txt` で決める） |
| Reference5 | どのボタンか。左なら `0`、右なら `1`（`OnMouseClick`、`OnMouseDoubleClick` など） |

たとえば「ことねの頭をダブルクリックされた」ときは、`OnMouseDoubleClick` の Reference3 が `0`、Reference4 が頭の当たり判定の名前になります。当たり判定の名前はシェル作者が決めるもので、作り方は第12章で扱います。

`OnMouseMove` は、マウスが動くたびにとてもたくさん届きます。「頭をなでられた」という反応は、SHIORI がこのイベントの回数を数えて作るのが一般的です（里々にはそのための機能があります。[第8章](08-satori-basics.md)・[第9章](09-satori-practice.md)）。`OnMouseMove` を使わないゴーストは、ゴースト側 `descript.txt` に `don't need onmousemove,1` と書くと、SSP がこのイベントを送らなくなります。

### 時間

| イベント | いつ来るか | 主な Reference |
| --- | --- | --- |
| `OnSecondChange` | 1 秒ごと | Reference3：話せる状態なら `1`、話せないなら `0` |
| `OnMinuteChange` | 1 分ごと | 同上 |
| `OnHourTimeSignal` | 毎正時（SSP） | 時報に使える |

`OnSecondChange` と `OnMinuteChange` は、ほかのセリフを話している途中など **話せない状態のとき**、Reference3 が `0` になり、`NOTIFY` で送られてきます。このときに返したセリフは無視されます。

### ランダムトークのしくみ

ゴーストが放っておいても話しかけてくる「**ランダムトーク**」には、実は専用のイベントがありません。多くの場合、SHIORI が `OnSecondChange` を受け取るたびに時間を数え、決めた間隔が過ぎたら、`OnSecondChange` への返事としてランダムに選んだセリフを返しています。

- 里々では、`＄喋り間隔` に秒数を設定すると、`OnSecondChange` のたびに残り時間を数え、時間が来たら名前のない `＊` に書いたトークから選んで話します（[第8章](08-satori-basics.md)）。
- YAYA では、システム辞書（`yaya_base`）がこのしくみを用意しています。システム辞書が `aitalkinterval` 秒ごとに `OnAiTalk` という関数を呼ぶので、ランダムトークはそこに書きます（[第10章](10-yaya-basics.md)）。

なお SSP には `OnAITalk` というイベントもあり、さくらスクリプトの `\a` タグを実行したときなどに「ランダムトークを話して」という意味で届きます。YAYA の `OnAiTalk` 関数とは名前が似ていますが、別物です。

### 選択肢

第6章の `\q[表示,ID]` で出した選択肢が選ばれると、次のイベントが起きます。

| イベント | いつ来るか | 主な Reference |
| --- | --- | --- |
| `OnChoiceSelectEx` | 選択肢が選ばれたとき（SSP。`OnChoiceSelect` より先） | Reference0：表示の文字、Reference1：ID |
| `OnChoiceSelect` | 選択肢が選ばれたとき | Reference0：ID |
| `OnChoiceTimeout` | 選ばれないまま時間切れになったとき | Reference0：時間切れになったスクリプト |
| （ID の名前のイベント） | ID が `On` で始まる選択肢が選ばれたとき | Reference0 以降：`\q` の 3 番目以降に書いた値 |

`\q[クッキー,cookie]` なら `OnChoiceSelect` の Reference0 に `cookie` が入るので、SHIORI 側で「Reference0 が `cookie` ならこう答える」と書き分けます。`\q[クッキー,OnOyatsu,クッキー]` のように ID を `On` で始めた場合は、`OnOyatsu` というイベントを書いておけば、それがそのまま呼ばれます。

## Reference の読み方

Reference は、イベントごとに **番号と意味が決まっています**。同じ Reference0 でも、`OnBoot` ならシェル名、`OnChoiceSelect` なら選択肢の ID です。新しいイベントを使うときは、かならず UKADOC の「SHIORI Event リスト」で、そのイベントの Reference の意味を確かめましょう。

- 番号は **0 から** 数えます。
- 決まっていない番号には何も入りません。
- 「SSP のみ」と書かれた Reference は、ほかのベースウェアでは入らないことがあります。

里々と YAYA では、Reference を次のように読みます。

| SHIORI | Reference0 の読み方 | 例 |
| --- | --- | --- |
| 里々 | `（Ｒ０）`（全角） | `（Ｒ０）さんから交代してきたよ。` |
| YAYA | `reference[0]` | `if reference[0] == "cookie" { ... }` |

たとえば `OnGhostChanged` の Reference0 には交代前のゴーストの \0 側の名前が入っているので、「〇〇さんから交代したんだね」と相手の名前を呼ぶセリフが作れます。

## 通知イベントと返事の扱い

イベントの中には、「話すため」ではなく **「SHIORI に状況を知らせるため」** のものがあります。こうしたイベントは `NOTIFY` で送られ、返事は使われません。

| イベント | 中身 |
| --- | --- |
| `OnInitialize` | 起動したとき（SSP）。SHIORI の準備用 |
| `OnDestroy` | 終了が指示されたとき。SHIORI の後片付け用 |
| `OnSurfaceChange` | サーフェスが変わったとき。Reference0・1 に \0 側・\1 側のいまのサーフェス番号 |
| `OnNotifyOSInfo` | OS の情報（SSP） |

返事の扱いをまとめると、次のようになります。

- `GET` のイベントでセリフを返す → SSP が話す。
- `GET` のイベントで何も返さない（204）→ 何も話さない。イベントによっては、代わりのイベントが続けて起きる（`OnFirstBoot` → `OnBoot` など）。
- `NOTIFY` のイベント → 何を返しても使われない。情報を受け取って変数に覚えておくなど、裏方の処理に使う。

知らないイベントに返事をしなくても、ゴーストが壊れることはありません。必要なイベントから少しずつ書き足していけば大丈夫です。

## まとめ

- SHIORI は、SSP からのリクエストに答えてレスポンスを返す係です。自分からは動きません。
- SHIORI/3.0 のリクエストでは `ID` がイベント名、`Reference0` 以降が追加情報です。レスポンスでは `Value` に話したいさくらスクリプトを入れます（何もなければ 204）。
- `GET` は返事を求める問い合わせ、`NOTIFY` は知らせるだけの通知です。
- よく使うイベントは `OnBoot`、`OnFirstBoot`、`OnClose`、`OnGhostChanged`、`OnMouseDoubleClick`、`OnMouseMove`、`OnSecondChange`、`OnMinuteChange`、`OnChoiceSelect` などです。
- ランダムトークは、SHIORI が `OnSecondChange` などで時間を数えて実現しています。
- Reference の意味はイベントごとに違うので、UKADOC で確かめてから使います。里々は `（Ｒ０）`、YAYA は `reference[0]` で読みます。

## 練習問題

1. 次のリクエストは、どんな出来事を知らせていますか。

```text
GET SHIORI/3.0
Charset: UTF-8
Sender: SSP
SecurityLevel: local
ID: OnMouseDoubleClick
Reference0: 52
Reference1: 30
Reference2: 0
Reference3: 1
Reference4: Head
Reference5: 0
```

<details>
<summary>答え</summary>

\1 側（Reference3 が `1`）のキャラクター、つまりぴーすけの `Head` という当たり判定（Reference4）が、左ボタン（Reference5 が `0`）でダブルクリックされたことを知らせています。Reference0・1 はクリックした位置の座標です。

</details>

2. `OnFirstBoot` にセリフを書かなかった場合、SSP ではじめて起動したときに何が起きますか。

<details>
<summary>答え</summary>

`OnFirstBoot` に返事がない（204）ので、SSP は続けて `OnBoot` を起こします。`OnBoot` に書いたふつうの起動あいさつが話されます。

</details>

3. ことねが話している途中に、ランダムトークの時間が来てしまいました。このとき `OnSecondChange` はどんな状態で届きますか。返したセリフはどうなりますか。

<details>
<summary>答え</summary>

話せない状態なので、Reference3 が `0` になり、`NOTIFY` で届きます。このとき返したセリフは無視されます。里々などの SHIORI はこれを見て、話せる状態になるまでランダムトークを待ちます。

</details>

---

[← 前の章](06-sakura-script.md) ｜ [目次](../README.md) ｜ [次の章 →](08-satori-basics.md)
