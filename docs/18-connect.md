# 第18章 外の世界とつながる

## この章で学ぶこと

- **SSTP** で、ほかのプログラムからゴーストに話させたり、イベントを送ったりする方法
- ほかのゴーストの起動・終了・おしゃべりに反応する方法と、特定のゴーストにイベントを送る `\![raiseother]`
- **SAORI** を里々と YAYA から呼び出して、ゴーストにできることを広げる方法
- ゴーストの側から見たプラグインとヘッドラインのイベント
- 外から届くものを安全に扱うための注意と、ほかのゴーストやユーザーへのマナー

## ゴーストの「外」には何があるか

ここまでのことねとぴーすけは、SSP と SHIORI のやりとり（[第7章](07-shiori-events.md)）の中だけで暮らしてきました。でも、デスクトップにはほかのプログラムがあり、ほかのゴーストもいます。この章では、そうした「外の世界」とやりとりするしくみを見ていきます。

| 相手 | 使うしくみ |
| --- | --- |
| ほかのプログラム | SSTP |
| 同時に起動しているゴースト | `OnOtherGhost…` のイベント、`\![raiseother]` |
| SHIORI だけではできない処理 | SAORI |
| SSP に追加した部品 | プラグイン・ヘッドラインのイベント |

この章で書く辞書は、新しいファイルにまとめます。里々は `ghost/master/dic_connect.txt`（`dic` で始まり `.txt` で終わるので自動で読み込まれます）、YAYA は `ghost/master/dic_connect.dic` を作り、`yaya.txt` の `dic, …` の行の並びの最後に、`dic, dic_connect.dic` の 1 行を足してください（[第11章](11-yaya-practice.md)）。

> **注意**
> この章の里々のコードにも、見えない **タブ文字** が入っている行があります（`＊名前` のあとに条件を書く行）。半角スペースでは働かないので、コピーしたあとにタブが残っているか確かめてください（[第9章](09-satori-practice.md)）。

## SSTP：外からゴーストに話しかける

### SSTP のしくみ

**SSTP**（Sakura Script Transfer Protocol）は、ほかのプログラムやゴーストから SSP にさくらスクリプトやイベントを届けるための決まりです。時計ソフトが時刻を知らせたり、音楽プレイヤーが曲名を伝えたりするのに使われてきました。

SHIORI/3.0 と同じく、1 行目に命令、2 行目以降に「名前: 値」を並べたテキストでやりとりします。改行は CR+LF で、最後に空行を 1 行入れるとリクエストの終わりです。

```text
SEND SSTP/1.4
Charset: UTF-8
Sender: PowerShell
Script: \0\s[0]こんにちは。\e
```

| 行 | 意味 |
| --- | --- |
| `SEND SSTP/1.4` | 1 行目。**メソッド**（何をしてほしいか）とバージョン |
| `Charset` | 文字コード。**必ず書く**。特に理由がなければ `UTF-8` |
| `Sender` | 送り主の名前。自分のプログラム名などを書く |
| `Script` | 話してほしいさくらスクリプト |

| メソッド | すること |
| --- | --- |
| `SEND` | `Script` のさくらスクリプトを話させる。`Event` でイベントを送ることもできる |
| `NOTIFY` | イベントを知らせる。いまの仕様では `SEND` と同じ処理で、機能の差はない |
| `COMMUNICATE` | ユーザーの話しかけと同じように、文章を渡して反応してもらう |
| `EXECUTE` | 話させずに、名前やバージョンなどの情報を問い合わせる |

SSP は返事として、HTTP に似た番号つきの 1 行目を返します。`2` で始まれば成功です。

| 番号 | 意味 |
| --- | --- |
| `200 OK` / `204 No Content` | 成功（返す値あり／なし） |
| `400 Bad Request` | リクエストの書き方に不備がある |
| `404 Not Found` | 指定したゴーストが起動していない（SSP） |
| `409 Conflict` / `420 Refuse` | 割り込み禁止中や処理中／ゴーストの設定で受け取りを断られた |

送り方は 3 通りあります。TCP でポート 9801（SSP では 9821 も）に接続して上のテキストを送る **（Socket）SSTP**、`http://localhost:9801/api/sstp/v1` に HTTP の POST で送る **SSTP over HTTP**（SSP 2.6.32 以降）、Windows のウィンドウメッセージでゴーストに直接送る **Direct SSTP** です。

SSP は、設定を変えない限り **このパソコンの中（localhost）からの接続しか受け付けません**。待ち受けるポートは、本体設定の「接続(2)」の「SSTP通信」で確かめられます。

> **コラム：SSTP over HTTP の注意**
> SSTP over HTTP では `Content-Type: text/plain` と `Content-Length` が必要で、SSTP としては失敗していても HTTP の番号は `200 OK` で返ります。また、HTTP の `Origin` が localhost を指していないリクエストは、ほかのパソコンからのものとして扱われ、使えるタグや命令が強く制限されます。

### やってみよう：PowerShell からことねに話させる

Windows に最初から入っている PowerShell で、（Socket）SSTP を送ってみます。

1. SSP で「はじめてのゴースト」を起動しておきます。
2. スタートメニューから「Windows PowerShell」（または「ターミナル」）を開きます。
3. 次の関数を貼り付けて `Enter` を押します。何も表示されなければ準備完了です。

```powershell
function Send-Sstp([string[]]$lines) {
    $text  = ($lines -join "`r`n") + "`r`n`r`n"
    $bytes = [System.Text.Encoding]::UTF8.GetBytes($text)
    $client = New-Object System.Net.Sockets.TcpClient('127.0.0.1', 9801)
    try {
        $stream = $client.GetStream()
        $stream.Write($bytes, 0, $bytes.Length)
        $reader = New-Object System.IO.StreamReader($stream, [System.Text.Encoding]::UTF8)
        $reader.ReadToEnd()
    }
    finally {
        $client.Close()
    }
}
```

4. 続けて、次を貼り付けて実行します。

```powershell
Send-Sstp @(
    'SEND SSTP/1.4'
    'Charset: UTF-8'
    'Sender: PowerShell'
    'ReceiverGhostName: ことね'
    'Script: \0\s[0]PowerShell から声が届いたよ！\w9\1\s[10]外の世界とつながったぴ。\e'
)
```

ことねとぴーすけが話し、PowerShell に `SSTP/1.4 200 OK` のような返事が表示されれば成功です（バージョンの数字は違うことがあります）。

- `Send-Sstp` は、行を CR+LF でつなぎ、最後に空行を足して、UTF-8 で 127.0.0.1 の 9801 番に送ります。SSP は返事を返すと接続を切るので、`ReadToEnd` で返事を最後まで読めます。
- `ReceiverGhostName` は、送り先を \0 側の名前で決める SSP のヘッダです。ほかのゴーストを同時に起動していても、ことねに届きます。ことねが起動していなければ `404 Not Found` が返ります。
- PowerShell では `\` は特別な文字ではないので、さくらスクリプトをそのまま書けます。行はシングルクォート `'…'` で囲んだままにしてください。
- 「接続できませんでした」というエラーになったら、SSP が起動しているか、「接続(2)」で 9801 番がオンになっているかを確かめます。関数を `.ps1` ファイルに保存して使う場合、Windows PowerShell 5.1 では **UTF-8（BOM 付き）** で保存しないと日本語が化けます。

### イベントを送る

`Script` の代わりに `Event` と `Reference0`… を書くと、イベントを送れます。SSP はそれを SHIORI に渡し、返ってきたセリフを話させます。自作のイベント `OnHajimeteKnock`（ノックされた）を受け取れるようにしてみましょう。名前に `OnHajimete` を付けるのは、ほかのゴーストやソフトのイベントとぶつからないようにするためです。

`ghost/master/dic_connect.txt`（ノックの部分）

```text
＃ 外の世界とつながる（第18章）

＃ ---- SSTP で届く自作のイベント ----
＊OnHajimeteKnock
：（2）あれ？　だれかがノックしたよ。
：外の世界からのお客さんだぴ。
```

`ghost/master/dic_connect.dic`（ノックの部分）

```c
//==============================================================
// dic_connect.dic  外の世界とつながる（第18章）
//==============================================================

// SSTP で届く自作のイベント
OnHajimeteKnock
{
	"\0\s[2]あれ？　だれかがノックしたよ。\w9\1\s[10]外の世界からのお客さんだぴ。\e"
}
```

辞書を読み直したら（`\![reload,shiori]`、[第14章](14-debug.md)）、PowerShell から送ります。

```powershell
Send-Sstp @(
    'NOTIFY SSTP/1.1'
    'Charset: UTF-8'
    'Sender: PowerShell'
    'ReceiverGhostName: ことね'
    'Event: OnHajimeteKnock'
    'Script: \0\s[0]……だれか来たのかな？\e'
)
```

`Event` と `Script` をいっしょに書くと、`Script` は **保険** になります。ゴーストがイベントに何も返さなかったときだけ、`Script` のほうが話されます。辞書を読み直す前と後で送り比べると、違いがよくわかります。

### 受け取る側の設定

SSTP を受け取るかどうかは、ゴーストの作者とユーザーの両方が決められます。作者は `descript.txt`（[第5章](05-descript.md)）に次のように書けます。

| 設定 | 意味 |
| --- | --- |
| `sstp.allowunspecifiedsend,0` | 送り先のゴーストを指名していない SSTP を受け取らない（既定は `1`） |
| `sstp.allowcommunicate,0` | `COMMUNICATE` を受け取らない（SSP。既定は `1`） |
| `sstp.alwaystranslate,1` | SSTP で届いたスクリプトにも、つねに `OnTranslate` などのトランスレートを行う（SSP。既定は `0`） |

ユーザーの側では、本体設定の「接続(2)」に SSTP の設定がまとまっています。「SSTPのログを取る」をオンにすると、受け取ったスクリプトやイベントが `log\SSTP.log` に記録されます。また「SSTPのゴースト内フィルタ処理を完全に無効にする」で、ゴースト側の受け取り拒否を無効にすることもできます。ゴーストの設定は、最終的にはユーザーの判断で上書きされることがある、と覚えておきましょう。

## ほかのゴーストとかかわる

SSP では、複数のゴーストを同時に立たせられます（[第3章](03-ssp-features.md)）。ゴーストどうしがおたがいに気づいて反応すると、デスクトップがぐっとにぎやかになります。この節のイベントとタグは、どれも SSP（と一部のベースウェア）のものです。

### ほかのゴーストの起動と終了を知る

| イベント | いつ来るか | 主な Reference |
| --- | --- | --- |
| `OnOtherGhostBooted` | ほかのゴーストが起動したとき | 0：起動したゴーストの \0 側の名前、1：起動時のスクリプト、2：ゴースト名 |
| `OnOtherGhostClosed` | 同時に起動していたほかのゴーストが終了したとき | 0：終了したゴーストの \0 側の名前、1：最後のスクリプト、2：ゴースト名 |
| `OnOtherGhostChanged` | ほかのゴーストどうしが切り替わったとき | 0：切り替え前の \0 側の名前、1：切り替え後の \0 側の名前 |

`ghost/master/dic_connect.txt`（ほかのゴーストの起動の部分）

```text
＃ ---- ほかのゴーストの起動 ----
＊OnOtherGhostBooted
：（5）あ、（Ｒ０）さんが来たよ。
：にぎやかになるぴ。
```

`ghost/master/dic_connect.dic`（ほかのゴーストの起動の部分）

```c
// ほかのゴーストが起動した（SSP）
OnOtherGhostBooted
{
	"\0\s[5]あ、%(reference[0])さんが来たよ。\w9\1\s[10]にぎやかになるぴ。\e"
}
```

右クリックメニューの「他のゴーストを呼ぶ」で、もう一人呼んで試してみましょう。`OnOtherGhostClosed` も同じ形で書けるので、Reference0 の名前を使って「〇〇さん、帰っちゃった。」のようなセリフを足してみてください。

### ほかのゴーストのおしゃべりを聞く

**`OnOtherGhostTalk`** は、同時に起動しているほかのゴーストが話したときに届くイベントです。ただし最初は届きません。ゴーストが自分で `\![set,otherghosttalk,true]` を実行して、受け取りをオンにする必要があります。`true` なら相手が話し終わった直後に、`before` なら話す直前に届き、`false` で止まります。

この設定は **ゴーストが終了するまで** 有効なので、起動のトークに入れておきます。

- 里々では、`dic_boot.txt` の `＊OnFirstBoot` と、2 つある `＊OnBoot` のそれぞれの最後に、`\![set,otherghosttalk,true]` だけの行を足します（第9章の `\![open,inputbox,…]` と同じ書き方です）。
- YAYA では、`dic_boot.dic` の `OnFirstBoot` と `OnBoot` の先頭の `"\1\s[10]\0\s[0]` を、`"\![set,otherghosttalk,true]\1\s[10]\0\s[0]` に書き換えます。

| Reference | 中身 |
| --- | --- |
| 0・1 | 話したゴーストのゴースト名・\0 側の名前 |
| 2 | どういう発言か。SSTP で届いたものなら `sstp-send`、中断されたなら `break` などが、カンマ区切りで入る |
| 3・4 | 相手が話したときのイベント名・さくらスクリプト |

相手が話すたびに口をはさむと、うるさいうえに、相手も同じしくみで反応した場合にゴーストどうしで延々と話し続けてしまいます。ここでは **10 回に 1 回** だけ反応させます。

`ghost/master/dic_connect.txt`（おしゃべりを聞く部分）

```text
＃ ---- ほかのゴーストのおしゃべり（10回に1回だけ反応） ----
＊OnOtherGhostTalk	（乱数1～10）==1
：（Ｒ１）さん、楽しそうだね。
：オレたちも負けてられないぴ。
```

`ghost/master/dic_connect.dic`（おしゃべりを聞く部分）

```c
// ほかのゴーストのおしゃべり（10回に1回だけ反応）
OnOtherGhostTalk
{
	if RAND(10) == 0 {
		"\0\s[0]%(reference[1])さん、楽しそうだね。\w9\1\s[10]オレたちも負けてられないぴ。\e"
	}
}
```

- 里々の `（乱数1～10）` は 1〜10 の整数です（半角で書くと半角で返ります）。条件が成り立たなければ候補がないので、何も話しません。
- YAYA の `RAND(10)` は 0〜9 の整数です。`if` が成り立たなければ何も返さないので、何も話しません。
- Reference4 には相手のさくらスクリプトがそのまま入っています。自分のセリフに混ぜると、相手の表情の指定などまで実行されてしまうので、そのまま表示しないでください。

### 特定のゴーストにイベントを送る

自分から、起動中のほかのゴーストにイベントを送ることもできます。

| タグ | 意味 |
| --- | --- |
| `\![raiseother,ゴースト名,イベント名,r0,r1,…]` | 指定したゴーストにイベントを送る。`r0` 以降は相手の Reference0 以降になる |
| `\![notifyother,ゴースト名,イベント名,r0,r1,…]` | 同じだが、相手が何を返しても話さない（SSP） |

ゴースト名には、相手の `descript.txt` の `name` を書きます。`__SYSTEM_ALL_GHOST__` と書くと起動中のすべてのゴーストに送れますが、相手を選ばずに送ることになるので、使いどころはよく考えましょう。

受け取る側は、そのイベント名の文や関数を書いておくだけです。送れなかったときは、送った側に **`OnRaiseOtherFailure`**（`\![notifyother]` なら `OnNotifyOtherFailure`）が届きます。Reference0 には、相手が見つからなければ `notfound`、`\t` の割り込み禁止中なら `timecritical`、最小化中なら `minimized` などの理由が入ります。

あなたが将来作る 2 体目のゴースト（ここでは `ともだちのゴースト` という名前とします）に、あいさつを送るメニューを作ります。受け取る側の書き方も、ことねの辞書にいっしょに書いておきます。

`ghost/master/dic_connect.txt`（あいさつを送る部分）

```text
＃ ---- ほかのゴーストにイベントを送る ----
＊ともだちにあいさつ
：ともだちに、あいさつを送ってみるね。
\![raiseother,ともだちのゴースト,OnHajimeteGreeting]

＊OnRaiseOtherFailure
：（2）……あれ、ともだちに届かなかったみたい。
：起動してないか、いまは手がはなせないんだぴ。

＃ 受け取る側（ともだちのゴーストの辞書にも同じものを書く）
＊OnHajimeteGreeting
：（5）あ、あいさつが届いたよ！
：返事をしてあげるぴ。
```

`ghost/master/dic_connect.dic`（あいさつを送る部分）

```c
// ほかのゴーストにイベントを送る
ともだちにあいさつ
{
	"\0\s[0]ともだちに、あいさつを送ってみるね。\![raiseother,ともだちのゴースト,OnHajimeteGreeting]\e"
}

// 送れなかったとき（reference[0] に理由が入る）
OnRaiseOtherFailure
{
	if reference[0] == "notfound" {
		"\0\s[2]……あれ、ともだちが見つからないみたい。\w9\1\s[10]起動してないんじゃないかぴ？\e"
	}
	else {
		"\0\s[0]ともだちは、いまは手がはなせないみたい。\e"
	}
}

// 受け取る側（ともだちのゴーストの辞書にも同じものを書く）
OnHajimeteGreeting
{
	"\0\s[5]あ、あいさつが届いたよ！\w9\1\s[10]返事をしてあげるぴ。\e"
}
```

メニューから呼べるように、里々は `＊メニュー` に `＿ともだちにあいさつ` の行を、YAYA は `メニュー` に `\q[ともだちにあいさつ,menu_greet]` を、`OnChoiceSelect` に `when "menu_greet" { ともだちにあいさつ }` を足します（第9章・第11章）。`ともだちのゴースト` はまだいないので、選ぶと `OnRaiseOtherFailure` の返事が聞けるはずです。受け取る側は、スクリプト入力で `\![raise,OnHajimeteGreeting]` を実行すれば試せます。

## SAORI で機能を広げる

### SAORI とは

**SAORI** は、SHIORI から呼び出して使う追加の部品です。SHIORI だけではできない処理（ファイルの操作、音の再生、特別な計算など）を引き受けます。多くはゴーストの `ghost/master` の中（`saori` フォルダなど）に置く DLL で、実行ファイル（`.exe`）を使う **SAORI-basic** という形もあります。第9章で名前だけ紹介した里々の関数集 **ssu** も、じつは里々に内蔵された SAORI です。

SHIORI から SAORI への呼び出しも、SHIORI/3.0 によく似たテキストのやりとりです。引数を `Argument0`、`Argument1`… として渡し、結果を `Result` で受け取ります。返す値が複数あるときは、`Value0`、`Value1`… にも入ります。

| | 里々 | YAYA |
| --- | --- | --- |
| 準備 | `satori_conf.txt` の `＠SAORI` に `呼び出し名,相対パス,定義済み引数…` の形で 1 行ずつ登録する（区切りは半角の `,`） | 不要（システム辞書が読み込み・後片付けをする） |
| 呼び出し | `（呼び出し名、引数1、引数2）` | `FUNCTIONEX("saori/なにか.dll", "引数0", "引数1")` |
| 結果 | その場所が `Result` に置き換わる。`Value0`… は `（Ｓ０）`… | 返り値が `Result`。`Value0`… は `valueex[0]`… |

- 相対パスは、どちらも `ghost/master` からです。
- 里々の **定義済み引数** は、呼び出すたびに先頭に付けて渡される引数です（省略できます）。

どの引数を何番目に渡すかは、SAORI ごとにまったく違います。使いたい SAORI を見つけたら、**その SAORI の readme を読んで**、引数と返り値を確かめてください。

### 例：SSP の内蔵モジュールでさいころを振る

SSP 2.9.13 以降は、乱数・日時・ハッシュなどの機能を **内蔵モジュール** として持っていて、DLL を置かなくても SAORI と同じ書き方で呼べます。名前は `internal_` で始まります。ここでは `internal_random` の `Number`（最小以上・最大以下の整数を 1 つ選ぶ関数）で、さいころを作ってみましょう。対応している SHIORI は、里々は Mc205-1 以降、YAYA は内蔵モジュールに対応した新しい `yaya_base` です。

里々では、実在しない `saori/internal_random.dll` を DLL の場所に書いて登録します。関数名の `Number` は毎回同じなので、定義済み引数にしておきます。`satori_conf.txt` の最後に、次の 2 行を足してください。

`ghost/master/satori_conf.txt`（追加する部分）

```text
＠SAORI
さいころ,saori/internal_random.dll,Number
```

`ghost/master/dic_connect.txt`（さいころの部分）

```text
＃ ---- SAORI（SSP の内蔵モジュール）でさいころを振る ----
＊さいころをふる
：ころころ……（さいころ、1、6）が出たよ！
：オレにも振らせてほしいぴ。
```

`（さいころ、1、6）` は、`Argument0` が `Number`、`Argument1` が `1`、`Argument2` が `6` の呼び出しになり、1〜6 のどれかに置き換わります。

`ghost/master/dic_connect.dic`（さいころの部分）

```c
// SAORI（SSP の内蔵モジュール）でさいころを振る
さいころをふる
{
	_me = FUNCTIONEX("internal_random", "Number", 1, 6)
	if TOINT(_me) == 0 {
		"\0\s[2]あれ、さいころが見つからないよ……。\w9\1\s[10]SSP が古いのかもしれないぴ。\e"
	}
	else {
		"\0\s[0]ころころ……%(_me)が出たよ！\w9\1\s[10]オレにも振らせてほしいぴ。\e"
	}
}
```

YAYA の `FUNCTIONEX` は、内蔵モジュールが使えないとき（古い SSP など）に `0` を返し、失敗したときは `Result` が空になります。さいころの目は 0 にならないので、0 になったら「使えなかった」と判断しています。メニューへの足し方は「ともだちにあいさつ」と同じです。

> **コラム：内蔵モジュールの注意**
> 内蔵モジュールは、SHIORI が SSP からウィンドウの通知（`hwnd`）を受け取ってから使えるようになります。古い SSP や SSP 以外のベースウェアでは使えません。里々でうまく呼べなかったときは、その理由が里々のログに記録されます（[第14章](14-debug.md)）。ほかのモジュールと関数は、UKADOC の「SAORI over DirectSSTP」に一覧があります。

## プラグインとヘッドライン

[第3章](03-ssp-features.md) では、プラグインとヘッドラインを使う側から見ました。ここでは、ゴーストを作る側からできることを簡単に紹介します。

### プラグインとのやりとり

- **ゴーストからプラグインへ**：`\![raiseplugin,プラグインのIDまたは名前,イベント名,r0,r1,…]` で、プラグインにイベントを送れます（SSP）。返事のいらない `\![notifyplugin,…]` もあります。送れなかったときは `OnRaisePluginFailure` が届きます（Reference0 に `notfound` や `disabled`）。
- **プラグインからゴーストへ**：プラグインは、ゴーストにスクリプトを話させたり、そのプラグイン独自の名前のイベントを送ってきたりします。どんなイベントが来るかはプラグインの説明書で確かめます。よく知られたものは、UKADOC の「SHIORI Eventリスト（外部）」にもまとめられています。
- インストールされているプラグインの一覧は、`installedplugin` という通知（`NOTIFY`）で SHIORI に知らされます（SSP）。

### ヘッドラインのイベント

ユーザーがヘッドライン（RSS）を読み上げさせると、ゴーストに次のイベントが届きます。

| イベント | いつ来るか | 主な Reference |
| --- | --- | --- |
| `OnHeadlinesenseBegin` | 読み込みを始めるとき | 0：サイト名、1：URL |
| `OnHeadlinesense.OnFind` | 読み上げる内容が見つかったとき | 0：サイト名、1：URL、2：何ページ目か（`First`、`Next`、`Last`、`First and Last`）、3：内容 |
| `OnHeadlinesenseComplete` | 取得できたが、更新がなかったとき | 0：`no update` |
| `OnHeadlinesenseFailure` | 取得に失敗したとき | 0：理由（`can't download`、`can't analyze`） |

SSP では、RSS のときは先に `OnRSSBegin`・`OnRSSFailure` が届き、それに返事をしなければ `OnHeadlinesenseBegin`・`OnHeadlinesenseFailure` が届きます。読み上げの本体の `OnHeadlinesense.OnFind` は作り込むと奥が深いので、まずは `OnHeadlinesenseBegin` に「（Ｒ０）を見てくるね。」、`OnHeadlinesenseFailure` に「うまく読めなかったみたい……。」のような、始まりと失敗のひとことから書いてみましょう。書き方は、ほかのイベントとまったく同じです。

## 安全とマナー

外とつながると、できることが増えるぶん、気をつけることも増えます。

**外から届いた文字を、そのまま実行しない**

- SSTP やほかのゴーストから届いた Reference には、だれが何を入れてきたかわかりません。自分のセリフに混ぜると、中のタグ（ブラウザを開く、イベントを起こすなど）まで実行されてしまいます。`OnHajimeteKnock` で届いた値をあえて表示していないのはそのためです。
- どうしても表示したいときは、タグを取り除いてから使います。YAYA ならシステム辞書の `SHIORI3FW.RemoveAllTags` が使えます（練習問題 3）。
- 受け取った文字を `EVAL` で実行したり、`\![raise]` の引数にそのまま渡したりするのは避けましょう。YAYA のシステム辞書が「遅延 EVAL」を最初から無効にしているのも、同じ理由です。

**外からのイベントは「話すだけ」にする**

- SHIORI へのリクエストには `SecurityLevel` という行があり、パソコンの外から来たものは `external` になります（[第7章](07-shiori-events.md)）。
- 里々は、`external` のリクエストでは `set` や `call` などの一部の関数を実行しません。`＄外部から実行可能なイベントの接頭辞` でこの制限をゆるめられますが、指定するのは本当に必要なイベント名の接頭辞だけにし、`全部` は使わないでください。
- SSTP で受け取るイベントでは、大事な変数を書き換えたり SAORI でファイルを操作したりせず、反応のトークだけにしておくのが安全です。

**SAORI は信頼できるものだけを使う**

- DLL や実行ファイルの SAORI は、パソコンの上でそのまま動くプログラムです。配布元がはっきりしたものだけを使い、同梱して配布するときは、その SAORI の配布条件（readme）に従います。

**ほかのゴーストとユーザーへのマナー**

- イベント名には `OnHajimete…` のような自分らしい接頭辞を付け、ほかのゴーストのイベントとぶつからないようにします。
- `__SYSTEM_ALL_GHOST__` への送信や `OnOtherGhostTalk` への反応は控えめにし、相手のキャラクターやユーザーの静かな時間を壊さないようにします。`OnRaiseOtherFailure` の理由が `minimized` や `timecritical` なら、相手はいま話せない状態なので、何度も送り直さないようにします。
- `OnOtherGhostTalk` で聞こえた話には、ユーザーの名前などの個人的な内容が含まれることがあります。ファイルに書き出したり、外へ送ったりしないでください。
- SSTP を受け取るかどうかなどは、最終的にはユーザーが本体設定で決めます。ユーザーの設定を回避するようなしかけは作らないでください。

## まとめ

- SSTP は、外からゴーストにさくらスクリプトやイベントを届ける決まり。`SEND`・`NOTIFY` などのメソッドと、必須の `Charset`、送り主の `Sender` を書く。SSP は既定で localhost のポート 9801 で待ち受ける。
- `Event` と `Script` をいっしょに送ると、イベントに返事がないときだけ `Script` が話される。`ReceiverGhostName` で送り先を \0 側の名前で決められる（SSP）。
- ほかのゴーストの起動・終了は `OnOtherGhostBooted`・`OnOtherGhostClosed` で、おしゃべりは `\![set,otherghosttalk,true]` でオンにしてから `OnOtherGhostTalk` で受け取る。
- `\![raiseother,ゴースト名,イベント名,…]` で特定のゴーストにイベントを送り、失敗は `OnRaiseOtherFailure` で知る。
- SAORI は、里々では `＠SAORI` に登録して `（）` で、YAYA では `FUNCTIONEX` で呼ぶ。引数はその SAORI の readme で確かめる。SSP 2.9.13 以降は `internal_` で始まる内蔵モジュールも使える。
- 外から届いた文字はそのまま実行・表示しない。外からのイベントは話すだけにし、ほかのゴーストやユーザーの設定を尊重する。

## 練習問題

1. 本章の `Send-Sstp` を使って、ぴーすけ（\1 側）だけに「オレだけ呼ばれたぴ！」と話させてください。

   <details>
   <summary>答えの例</summary>

   ```powershell
   Send-Sstp @(
       'SEND SSTP/1.4'
       'Charset: UTF-8'
       'Sender: PowerShell'
       'ReceiverGhostName: ことね'
       'Script: \1\s[10]オレだけ呼ばれたぴ！\e'
   )
   ```

   `ReceiverGhostName` は \0 側の名前で指定するので、ぴーすけに話させたいときも `ことね` のままです。話し手はスクリプトの `\1` で決めます。

   </details>

2. `EXECUTE` メソッドの `GetName` コマンドを使うと、起動中のゴーストの名前を問い合わせられます。`Send-Sstp` で送って、返事を確かめてください。

   <details>
   <summary>答えの例</summary>

   ```powershell
   Send-Sstp @(
       'EXECUTE SSTP/1.1'
       'Charset: UTF-8'
       'Sender: PowerShell'
       'Command: GetName'
   )
   ```

   ゴーストは話さず、返事のヘッダと空行のあとに、`ことね,ぴーすけ` のように \0 側と \1 側の名前がカンマ区切りで返ってきます。

   </details>

3. SSTP で届くイベント `OnHajimeteMessage` の Reference0 に、送り主が書いたメッセージが入ってくるとします。これを `"\0\s[0]%(reference[0])\e"` のようにそのまま話させるのは、なぜ危ないのでしょう。YAYA でどう直せばよいでしょう。

   <details>
   <summary>答えの例</summary>

   Reference0 にさくらスクリプトのタグが入っていると、ことねのセリフの一部として実行されてしまうからです。タグを取り除いてから使います。

   ```c
   OnHajimeteMessage
   {
   	_text = SHIORI3FW.RemoveAllTags(reference[0])
   	"\0\s[0]こんなメッセージが届いたよ。\n\n%(_text)\e"
   }
   ```

   </details>

4. ことねとそっくりの「聞き耳」を持つゴーストが、もう一人いるとします。おたがいの `OnOtherGhostTalk` が、相手の発言に **毎回** 反応するように書かれていたら、何が起きるでしょう。

   <details>
   <summary>答え</summary>

   ことねが話すと相手が反応し、その反応にことねが反応し……と、二人が止まらずに話し続けてしまいます。反応を確率で減らす、Reference3（相手が話したイベント名）を見て反応するものを選ぶ、などの工夫で防ぎます。

   </details>

この章で、「はじめてのゴースト」の旅はひとまずおしまいです。ここから先は、付録の早見表や公式資料を手もとに置いて、あなたのゴーストを自由に育ててください。

---

[← 前の章](17-days-and-memory.md) ｜ [目次](../README.md) ｜ [次は付録 →](appendix-a-sakura-script.md)
