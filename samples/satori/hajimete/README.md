# はじめてのゴースト（里々版）サンプル

[第8章 里々入門](../../../docs/08-satori-basics.md) と [第9章 里々実践](../../../docs/09-satori-practice.md) で作る辞書の完成形です。

## 同梱していないもの

このフォルダには、次の 2 つが **入っていません**。自分で用意してから動かしてください。

| 入っていないもの | 置く場所 | 入手方法 |
| --- | --- | --- |
| 里々の本体 `satori.dll` | `ghost/master/` | 下の「satori.dll を入手する」を参照 |
| シェル（立ち絵の画像と設定） | `shell/master/` | [第12章 シェルを作る](../../../docs/12-shell.md) を参照 |

`satori.dll` は里々の作者・開発チームが配布しているものなので、本書のリポジトリには含めていません。シェル画像も同じ理由で同梱していません。

## フォルダの中身

```text
hajimete/
  install.txt            … インストール設定（type,ghost / directory,hajimete）
  readme.txt             … ゴーストの説明
  ghost/
    master/
      descript.txt       … ゴーストの設定（shiori,satori.dll を指定）
      satori_conf.txt    … 里々の初期設定（＊初期化）
      dic_talk.txt       … ランダムトーク（第8章）
      dic_boot.txt       … 起動・終了・時報（第9章）
      dic_mouse.txt      … つつかれ・なでられ（第9章）
      dic_menu.txt       … メニュー・入力ボックス（第9章）
      satori.dll         … ★自分で置く
  shell/
    master/              … ★自分で用意する
```

## satori.dll を入手する

1. 里々の配布ページ（GitHub Releases）を開きます。
   <https://github.com/ukatech/satoriya-shiori/releases>
2. 最新版のファイルをダウンロードし、展開します。
3. 中にある `satori.dll` を、`hajimete/ghost/master/` にコピーします。

里々の説明書は <https://ukatech.github.io/satori-docs/> にあります。

## SSP に置いて動かす

1. このサンプルの `hajimete` フォルダを、SSP のフォルダの中の `ghost` フォルダにコピーします（`ssp/ghost/hajimete/` になるように）。
2. `ghost/master/` に `satori.dll` を置きます。
3. `shell/master/` にシェルを用意します（第12章）。
4. SSP を起動し、右クリックメニューからゴーストを「はじめてのゴースト」に切り替えます（切り替え方は [第2章](../../../docs/02-install-ssp.md) を参照）。

## 文字コード

- 辞書ファイル（`dic*.txt`、`satori_conf.txt`）と `descript.txt`・`install.txt`・`readme.txt` は、すべて **UTF-8（BOM なし）・改行 CRLF** で保存しています。
- `descript.txt`・`install.txt`・`readme.txt` の 1 行目には `charset,UTF-8` を書いてあります。
- 里々の **Unicode 版（バージョン記号 Mc2XX）** は、辞書の文字コードを自動で判定し、UTF-8 の辞書をそのまま読めます。上の配布ページから入手した最新版を使ってください。
- 古い **ACP 版（Mc1XX）** の `satori.dll` を使う場合は、UTF-8 の辞書を読めません。辞書ファイルをすべて Shift_JIS に変換して保存し直してください。
- 辞書の中のタブ文字（`＊名前`・`＠名前`・`＞名前`・`＿ラベル` のあとの区切り）は、半角スペースに置き換えないでください。エディタによっては、タブをスペースに変える設定があるので注意します。

## 動作の前提

- ベースウェアは SSP を前提にしています。時報（`OnHourTimeSignal`）と、入力ボックスの ID を `On` で始めてイベントを直接呼ぶ書き方は、SSP の機能です。
- 当たり判定の名前（`Head`・`Face`）は、使うシェルの `surfaces.txt` に合わせて書き換えてください。
