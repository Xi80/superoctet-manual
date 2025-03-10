---
title: システムエクスクルーシブ仕様
---

# システムエクスクルーシブ仕様

この項では本音源が送受信するシステムエクスクルーシブについて説明します。

## はじめに

本音源の仕様は一部[XG 仕様書V1.35](https://jp.yamaha.com/files/download/other_assets/0/321740/xg_v135_j.pdf)に則っています。

これは、当サークル(fèng)がMIDIエクスクルーシブ会員ではないため自由にエクスクルーシブを作成できないことが理由です。

ここで、[MIDI1.0規格書](https://amei.or.jp/midistandardcommittee/MIDI1.0.pdf)の69ページを参照すると

> 公表されたエクスクルーシブのフォーマットは、一般の他のフォーマット（チャンネル・メッセージやコモン・メッセージ）とまったく同じ扱いとなり、そのフォーマットに忠実である限り、だれでも使用することができる

とあります。

そこで、本音源ではYAMAHAのシステムエクスクルーシブ・フォーマットに準拠することにしました。

YAMAHAのシステムエクスクルーシブのうち、送受信するのは次の2つです。

- パラメータチェンジ
- パラメータリクエスト(内部で使用)

なお、パラメータチェンジ・リクエストのアドレステーブルはXGのものとは一部を除いて異なりますのでご了承ください。

ここで、パラメータチェンジ・リクエストのフォーマットを示します。

|名称                  |データ(16進数)                    |備考          |
|----------------------|----------------------------------|--------------|
|XGパラメータチェンジ  |`F0 43 10 4C AH AM AL [データ] F7`|AH~AL:アドレス|
|XGパラメータリクエスト|`F0 43 30 4C AH AM AL [データ] F7`|AH~AL:アドレス|


### リセット系エクスクルーシブ

対応しているリセット系エクスクルーシブを示します。

|名称            |データ(16進数)                    |備考                                          |
|----------------|----------------------------------|----------------------------------------------|
|GMシステム・オン|`F0 7E 7F 09 01 F7`               |画面表示色：紫<br>基本はこれを使用してください|
|XGシステム・オン|`F0 43 10 4C 00 00 7E 00 F7`      |画面表示色：緑<br>表示色以外はGMリセットと同一|
|GSリセット      |`F0 41 10 42 12 40 00 7F 00 41 F7`|画面表示色：橙<br>表示色以外はGMリセットと同一|
|GSモード1       |`F0 41 10 42 12 00 00 7F 00 01 F7`|画面表示色：橙<br>表示色以外はGMリセットと同一|
|GSモード2       |`F0 41 10 42 12 00 00 7F 01 00 F7`|画面表示色：橙<br>表示色以外はGMリセットと同一|

### 画面表示系エクスクルーシブ

XG/GS両方のフォーマットに対応しています。

各フォーマットのデータの作り方についてはインターネット上に情報が多くありますのでここでの説明はしません。

#### GSフォーマット

`DISPLAYED LETTER(Address:10 00 00)`と`DISPLAYED DOT DATA(Address:10 01 00)`を受信します。

|名称                |データ(16進数)                           |備考                                                                 |
|--------------------|-----------------------------------------|---------------------------------------------------------------------|
|`DISPLAYED LETTER`  |`F0 41 10 45 12 10 00 00 [データ] sum F7`|途中のアドレスから送信しないでください<br>最大32文字まで             |
|`DISPLAYED DOT DATA`|`F0 41 10 45 12 10 01 00 [データ] sum F7`|途中のアドレスから送信しないでください<br>必ず1画面分送信してください|

!!! note inline end "情報"

    GSのシステムエクスクルーシブを受信時に**チェックサムのチェックは行っていません**。

#### XGフォーマット

`DISPLAY LETTER(Address:06 00 00)`と`DISPLAY BITMAP(Address:07 00 00)`を受信します。

|名称            |データ(16進数)                    |備考                                                                 |
|----------------|----------------------------------|---------------------------------------------------------------------|
|`DISPLAY LETTER`|`F0 43 10 4C 06 00 00 [データ] F7`|途中のアドレスから送信しないでください<br>最大32文字まで             |
|`DISPLAY BITMAP`|`F0 43 10 4C 07 00 00 [データ] F7`|途中のアドレスから送信しないでください<br>必ず1画面分送信してください|

!!! note inline end "情報"

    XGの仕様書には`DISPLAY BITMAP`は`07 vh 00`となっており、縦方向(v:0~7)・横方向(h:0~F)への拡張が可能ですが本音源では実装していません。

### パート設定系エクスクルーシブ

ドラムパートの設定とMultiモード時のユニットアサインを設定できます。
ドラムパートはXG/GS両方のエクスクルーシブを使用できます。

|名称            |データ(16進数)                    |備考                                                                 |
|----------------|----------------------------------|---------------------------------------------------------------------|
|`PART MODE(XG)`|`F0 43 10 4C 08 nn 07 mm F7`|`nn`:パート番号(`00`~`0F`)、`mm`:`0`がメロディーパート、`0`以外でリズムパート             |
|`PART MODE(GS)`|`F0 41 10 42 12 40 nn 15 mm sum F7`|`nn`:パート番号(`01`~`09`,`00`,`0A`~`0F)、`mm`:`0`がメロディーパート、`0`以外でリズムパート|

Multiモード時のユニットアサイン設定は`MMX PART ASSIGN(Address:08 xx 6F)`で行います。
なお、接続されているユニット以上の割り当ては無効となり、受信しません。

|名称            |データ(16進数)                    |備考                                                                 |
|----------------|----------------------------------|---------------------------------------------------------------------|
|`MMX PART ASSIGN`|`F0 43 10 4C 08 nn 6F mm F7`|`nn`:パート番号(`00`~`0F`)、`mm`:`0`がマスタ、`1`~`(スレーブの接続数)`で該当するスレーブ番号に割り当てる             |

### ユーザー音色転送

!!! warning inline end "注意"

    現在開発中の機能です。

    仕様が変更される可能性があります。

|アドレス<br>(H)|アドレス<br>(M)|アドレス<br>(L)|サイズ<br>(16進数)|データ範囲<br>(16進数)|パラメータ                   |備考                 |
|---------------|---------------|---------------|------------------|----------------------|-----------------------------|---------------------|
|`09`           |           `00`|     `音色番号`|`任意`            |`00-7F`               |`USER TIMBRE XFER PORT`      |                     |

なお音色ファイルのフォーマットは非公開です。音色エディタを使用してください。

データ量が大きいため転送後には十分時間を設けてください。

### その他

|アドレス<br>(H)|アドレス<br>(M)|アドレス<br>(L)|サイズ<br>(16進数)|データ範囲<br>(16進数)|パラメータ     |備考                 |
|---------------|---------------|---------------|------------------|----------------------|---------------|---------------------|
|`01`           |           `06`|           `00`|`1`               |`00-02`               |`SET UI COLOR` |リセットせずに画面配色のみを変更する<br>0:緑<br>1:橙<br>2:紫|