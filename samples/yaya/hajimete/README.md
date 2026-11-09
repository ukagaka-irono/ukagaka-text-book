# はじめてのゴースト（YAYA 版）サンプル

[第10章 YAYA 入門](../../../docs/10-yaya-basics.md) と [第11章 YAYA 実践](../../../docs/11-yaya-practice.md) で作る辞書の完成形です。

## 同梱しているもの

```text
hajimete/
├─ install.txt          … インストール設定
├─ readme.txt           … ゴーストに添える説明書
└─ ghost/
    └─ master/
        ├─ descript.txt  … ゴーストの基本設定（shiori,yaya.dll）
        ├─ yaya.txt      … YAYA の基礎設定ファイル
        ├─ dic_boot.dic  … 起動・終了（第10章・第11章）
        ├─ dic_talk.dic  … ランダムトーク（第10章・第11章）
        ├─ dic_mouse.dic … つつかれ・なでられ（第11章）
        ├─ dic_time.dic  … 時報（第11章）
        └─ dic_menu.dic  … メニューとトーク間隔（第11章）
```

文字コードはすべて **UTF-8（BOM なし）**、改行は CRLF です。`yaya.txt` は `charset, UTF-8` を指定しており、ファイル自体は半角英数字（ASCII）だけで書いてあります。`descript.txt`・`install.txt`・`readme.txt` の 1 行目には `charset,UTF-8` を書いてあります。

## 同梱していないもの（自分で用意する）

動かすには、次の 3 つを自分で用意してください。

| 必要なもの | 入手先 | 置き場所 |
| --- | --- | --- |
| `yaya.dll`（YAYA 本体） | [YAYA-shiori/yaya-shiori のリリース](https://github.com/YAYA-shiori/yaya-shiori/releases) | `ghost/master/yaya.dll` |
| `yaya_base` フォルダ（システム辞書） | [YAYA-shiori/yaya-dic](https://github.com/YAYA-shiori/yaya-dic) | `ghost/master/yaya_base/` |
| シェル | 第12章を参照 | `shell/master/` |

### 手順

1. yaya-shiori のリリースページを開き、最新版の zip をダウンロードして展開します。中にある `yaya.dll` を `ghost/master/` にコピーします。
2. yaya-dic のページで「Code」→「Download ZIP」を選び、ダウンロードした zip を展開します。中にある `yaya_base` フォルダを、**フォルダごと** `ghost/master/` にコピーします。`yaya_base` の中に `shiori3.dic`・`config.dic`・`_loading_order.txt` などが入っていれば正しい形です。
3. シェルを `shell/master/` に用意します（作り方は第12章）。
4. `hajimete` フォルダを SSP の `ghost` フォルダに置き、SSP のゴースト切り替えメニューから「はじめてのゴースト」を選びます。

最終的に `ghost/master` はこうなります。

```text
ghost/master/
├─ descript.txt
├─ yaya.txt
├─ yaya.dll          ← 自分で入れる
├─ yaya_base/        ← 自分で入れる
│   ├─ _loading_order.txt
│   ├─ config.dic
│   ├─ shiori3.dic
│   └─ （ほかのシステム辞書）
└─ dic_*.dic
```

> **注意**
> - `yaya.dll` とシステム辞書は、できるだけ新しいもの同士を組み合わせてください。システム辞書の説明でも、更新するときは `yaya.dll` も同時に更新するようすすめられています。
> - 里々版のサンプル（`samples/satori/hajimete/`）も同じフォルダ名 `hajimete` を使っています。同じ SSP に両方入れるときは、どちらかのフォルダ名と `install.txt` の `directory` を変えてください。
> - 起動すると `ghost/master/yaya_variable.cfg`（変数のセーブファイル）ができます。配布するときは消してください。

## 当たり判定の名前

`dic_mouse.dic` は、第12章のシェルと同じ当たり判定の名前を使っています（ことね：`Head`・`Face`・`Bust`、ぴーすけ：`Head`）。反応を作っているのは、ことねの `Head`（つつかれ・なでられ）と `Face`（つつかれ）です。別のシェルを使うときは、`dic_mouse.dic` の `"Head"`・`"Face"` をそのシェルの名前に合わせて書き換えてください（大文字・小文字も区別されます）。
