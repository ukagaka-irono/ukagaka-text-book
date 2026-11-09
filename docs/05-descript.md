# 第5章 descript.txt を書く

## この章で学ぶこと

- `descript.txt` の基本の書き方（「項目名,値」の形と文字コード）
- ゴースト側 `descript.txt` の必須項目と主な項目
- 里々版と YAYA 版で `shiori` の行がどう違うか
- `install.txt` の書き方
- シェル側 `descript.txt` の主な項目

## descript.txt とは

**descript.txt** は、ゴーストやシェルの「名札」にあたる設定ファイルです。名前、作者、使う SHIORI などを書いておくと、SSP がそれを読んでゴーストを認識します。第4章で見たとおり、`descript.txt` は二か所にあります。

- `ghost/master/descript.txt` … ゴースト（心）の設定
- `shell/master/descript.txt` … シェル（体）の設定

名前は同じですが、書く項目が違います。この章ではまずゴースト側を書き、次に `install.txt`、最後にシェル側を見ていきます。

### 基本の書き方

`descript.txt` は、1 行に 1 項目ずつ、**項目名と値をカンマでつないで** 書きます。

```text
項目名,値
```

- この本では、カンマの前後に余計な空白を入れずに書きます。
- 1 行目には **`charset`**（文字コード）を書きます。UKADOC では、古い環境との互換性を気にするなら `Shift_JIS`、そうでなければ `UTF-8` が推奨されています。この本では `UTF-8` を使います。
- `charset` に書いた文字コードと、実際にファイルを保存するときの文字コードは必ずそろえます。ずれていると、名前が文字化けします。
- 値の中で `\` や `%` から始まる文字列は、さくらスクリプトや環境変数とみなされ、安全のために捨てられます。文字としてそのまま使いたいときは `\\`、`\%` と書きます。

> **注意：保存するときの文字コード**
> Windows のメモ帳など、テキストエディタで保存するときに文字コードを選べます。`charset,UTF-8` と書いたら、UTF-8 で保存してください。

## ゴースト側の descript.txt

### 必須の項目

UKADOC では、ゴースト側の `descript.txt` について、次の項目が必須とされています（SSP では省略できるものもあります）。

| 項目 | 意味 | 書き方の例 |
| --- | --- | --- |
| `type` | ファイルの種類。ゴーストなら `ghost` | `type,ghost` |
| `name` | ゴーストの名前。メニューなどに出る | `name,はじめてのゴースト` |
| `sakura.name` | \0 側（メインキャラクター）の名前 | `sakura.name,ことね` |
| `kero.name` | \1 側（サブキャラクター）の名前 | `kero.name,ぴーすけ` |
| `craftman` | 作者名（半角英数のみ） | `craftman,yourname` |
| `craftmanw` | 作者名（日本語も使える） | `craftmanw,あなたの名前` |

`sakura.name` と `kero.name` という項目名は、昔の呼び名（さくら側・けろ側）の名残です。中身は「\0 側の名前」「\1 側の名前」と読み替えてください。SSP では、`kero.name` を省略すると `sakura.name` と同じ名前が使われます。

`craftman` には半角英数字しか使えないので、ローマ字のハンドルネームなどを書きます。日本語の名前は `craftmanw` のほうに書きます。

### よく使う項目

| 項目 | 意味 | 既定値・メモ |
| --- | --- | --- |
| `charset` | このファイルの文字コード | 1 行目に書く |
| `shiori` | 使う SHIORI の DLL のファイル名 | 省略すると `shiori.dll` |
| `craftmanurl` | 作者のサイトの URL | 省略可 |
| `homeurl` | ネットワーク更新用の URL（SSP） | 末尾の `/` を忘れない。第15章で設定する |
| `readme` | 説明書のファイル名（SSP） | 省略すると `readme.txt` |
| `icon` | タスクトレイのアイコン（`.ico`） | 省略すると SSP の標準アイコン |
| `balloon` | 標準で使うバルーンの名前 | 省略可 |
| `sakura.seriko.defaultsurface` | \0 側の最初のサーフェス番号（SSP） | 既定は 0 |
| `kero.seriko.defaultsurface` | \1 側の最初のサーフェス番号（SSP） | 既定は 10 |
| `don't need onmousemove` | `1` にすると `OnMouseMove` イベントを送らない | 既定は 0 |

**`shiori`** の行はとても大切です。SHIORI の DLL のファイル名が `shiori.dll` ではない場合、この行を書き忘れるとゴーストが SHIORI を読み込めず、まったくしゃべらなくなります。

**`homeurl`** は `descript.txt` に書くほか、SHIORI から返して設定する方法もあります。両方に書くと、サイトを引っ越したときに片方だけ直し忘れる事故が起きやすいので、どちらか一方にしておきましょう。

### 里々版と YAYA 版の shiori 行

里々と YAYA では、SHIORI の DLL の名前が違います。そのため、`shiori` の行だけが変わります。

| SHIORI | DLL のファイル名 | `shiori` の行 |
| --- | --- | --- |
| 里々 | `satori.dll` | `shiori,satori.dll` |
| YAYA | `yaya.dll` | `shiori,yaya.dll` |

YAYA は DLL のファイル名を自由に変えられ（たとえば `hoge.dll`）、そのときは基礎設定ファイルの名前も `hoge.txt` に変わります。この本では標準の `yaya.dll` のまま使います。

### 完成例：はじめてのゴースト

里々版の「はじめてのゴースト」の `descript.txt` です。

`ghost/master/descript.txt`

```text
charset,UTF-8
type,ghost
name,はじめてのゴースト
sakura.name,ことね
kero.name,ぴーすけ
craftman,yourname
craftmanw,あなたの名前
shiori,satori.dll
```

YAYA 版は、最後の 1 行だけが違います。

`ghost/master/descript.txt`（YAYA 版）

```text
charset,UTF-8
type,ghost
name,はじめてのゴースト
sakura.name,ことね
kero.name,ぴーすけ
craftman,yourname
craftmanw,あなたの名前
shiori,yaya.dll
```

`yourname` と `あなたの名前` は、自分のハンドルネームに置き換えてください。`homeurl` はまだ書きません。配布の準備をする第15章で追加します。

### 書いたら確かめる

1. `descript.txt` を UTF-8 で保存します。
2. SSP が起動していたら、右クリックメニューからいったん別のゴーストに切り替えるか、SSP を再起動します。開発者用機能を有効にしている場合は、開発用パレットの「リロード」でも読み直せます。
3. 右クリックメニューのゴースト一覧に「はじめてのゴースト」が出ていれば成功です。

名前が文字化けしているときは、`charset` の値と保存した文字コードが合っているかを確かめましょう。

## install.txt を書く

**install.txt** は、ゴーストフォルダの直下に置くインストール用の設定ファイルです。これがあると、ゴーストを nar ファイルにまとめたとき、SSP にドラッグ＆ドロップするだけでインストールできるようになります。書き方は `descript.txt` と同じ「項目名,値」です。

| 項目 | 意味 | メモ |
| --- | --- | --- |
| `charset` | 文字コード | 必ず 1 行目（Shift_JIS なら省略可） |
| `type` | 中身の種類。ゴーストなら `ghost` | 必須 |
| `name` | 名前 | 必須。ゴースト側 `descript.txt` の `name` とそろえる |
| `directory` | インストール先のフォルダ名 | ゴーストでは必須。半角英数推奨 |
| `accept` | 受け入れるゴーストの \0 側の名前 | 追加シェルなど、特定のゴースト向けのものに使う |

`type` には `ghost` のほかに、追加シェルの `shell`、バルーンの `balloon`、追加ファイルの `supplement` などがあります。

`ghost/master/descript.txt` の `name` と `install.txt` の `name` が違っていると、インストール直後にそのゴーストへ切り替える機能が正しく働かないことがあります。同じ名前にしておきましょう。

`install.txt`

```text
charset,UTF-8
type,ghost
name,はじめてのゴースト
directory,hajimete
```

> **コラム：追加シェルの install.txt**
> あとから追加シェルだけを配るときは、`type,shell` にして、`accept` に受け入れ側のゴーストの \0 側の名前を書きます。`accept` に書くのはゴースト名（`name`）ではなく、\0 側の名前（`sakura.name`）なので注意しましょう。はじめてのゴーストなら `accept,ことね` です。

## シェル側の descript.txt

シェル側の `descript.txt` は、シェルの名前と作者、それに見た目にかかわる設定を書きます。UKADOC では `type` と `name` が必須です。

| 項目 | 意味 | メモ |
| --- | --- | --- |
| `charset` | 文字コード | 1 行目に書く |
| `type` | 種類。シェルなら `shell` | 必須（SSP ではシェルフォルダ内なら自動で判別） |
| `name` | シェルの名前 | 必須。メニューに出る |
| `craftman` / `craftmanw` | シェルの作者名 | ゴースト側と同じ書き方 |
| `sakura.balloon.offsetx` / `offsety` | \0 側のバルーンの表示位置の調整 | 第12章 |
| `kero.balloon.offsetx` / `offsety` | \1 側のバルーンの表示位置の調整 | 第12章 |
| `seriko.use_self_alpha` | `1` にすると、PNG の透明度（アルファチャンネル）を使う（SSP） | 既定は 0 |

このほか、右クリックメニューの見た目や着せ替えの設定もシェル側の `descript.txt` に書きます。シェルの細かい設定は第12章で説明します。

`shell/master/descript.txt`

```text
charset,UTF-8
type,shell
name,master
craftman,yourname
craftmanw,あなたの名前
seriko.use_self_alpha,1
```

`seriko.use_self_alpha,1` を書いておくと、透明部分を含む PNG 画像をそのまま使えるので、画像を用意するときに楽になります（第12章）。

## まとめ

- `descript.txt` は「項目名,値」を 1 行ずつ書くファイルで、1 行目に `charset` を書き、同じ文字コードで保存します。
- ゴースト側では `type`、`name`、`sakura.name`、`kero.name`、`craftman`、`craftmanw` が必須です。
- `shiori` の行には SHIORI の DLL 名を書きます。里々なら `shiori,satori.dll`、YAYA なら `shiori,yaya.dll` です。
- `install.txt` はゴーストフォルダ直下に置き、`type`、`name`、`directory` を書きます。`name` はゴーストの `name` とそろえます。
- シェル側の `descript.txt` には `type`、`name` を書き、バルーン位置などの見た目の設定も置きます。

## 練習問題

1. 次の `descript.txt` には、里々のゴーストとして動かすうえで問題が一つあります。どこでしょうか。

```text
charset,UTF-8
type,ghost
name,テストゴースト
sakura.name,ことね
kero.name,ぴーすけ
craftman,yourname
craftmanw,あなたの名前
```

<details>
<summary>答え</summary>

`shiori` の行がありません。省略すると `shiori.dll` を探すので、里々（`satori.dll`）を使うなら `shiori,satori.dll` を書く必要があります。

</details>

2. 作者名を `craftman,あなたの名前` と書いてしまいました。どう直せばよいでしょうか。

<details>
<summary>答え</summary>

`craftman` は半角英数のみです。日本語の名前は `craftmanw,あなたの名前` に書き、`craftman` には `craftman,yourname` のように半角英数の名前を書きます。

</details>

3. 「はじめてのゴースト」用の追加シェル「なつふく」を配布したくなりました。フォルダ名を `natsufuku` とするとき、`install.txt` を書いてみましょう。

<details>
<summary>答えの例</summary>

```text
charset,UTF-8
type,shell
name,なつふく
directory,natsufuku
accept,ことね
```

`accept` には、ゴースト名ではなく \0 側の名前（`sakura.name`）を書きます。

</details>

---

[← 前の章](04-ghost-structure.md) ｜ [目次](../README.md) ｜ [次の章 →](06-sakura-script.md)
