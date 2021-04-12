---
title: メディア・コミュニケーション
type: docs
weight: 7
---

# WebRTCのメディア通信では何ができるのですか？

WebRTCでは、オーディオやビデオのストリームを無制限に送受信することができます。これらのストリームは、通話中にいつでも追加・削除することができます。これらのストリームはすべて独立していることもあれば、まとめて送信することもできます。例えば、自分のデスクトップのビデオフィードを送信し、ウェブカムからのオーディオ／ビデオを含めることができます。

WebRTCプロトコルは、コーデックに依存しません。基礎となるトランスポートは、まだ存在しないものも含めて、すべてをサポートしています。ただし、通信相手であるWebRTCエージェントが、それを受け入れるために必要なツールを持っていない場合もあります。

また、WebRTCは、動的なネットワーク状況に対応できるように設計されています。通話中に帯域が増えたり減ったりすることがあります。また、突然パケットロスが多発することもあります。WebRTCはこのような状況にも対応できるように設計されています。WebRTCはネットワークの状態に対応し、利用可能なリソースで最高の体験を提供しようとします。

## どのような仕組みになっているのですか？

WebRTCは、[RFC 1889](https://tools.ietf.org/html/rfc1889)で定義されている2つの既存のプロトコルRTPとRTCPを使用しています。

RTP（Real-time Transport Protocol）は、メディアを伝送するプロトコルです。動画をリアルタイムに配信することを目的に設計されています。遅延や信頼性に関するルールは規定されていませんが、それらを実装するためのツールが提供されています。RTPはストリームを提供し、1つの接続で複数のメディアフィードを実行することができます。また、メディアパイプラインに供給するために必要な、タイミングや順序の情報も提供します。

RTCP（RTP Control Protocol）は、コールに関するメタデータを通信するためのプロトコルです。このフォーマットは非常に柔軟で、必要なメタデータを追加することができます。通話に関する統計情報を通信するために使用されます。また、パケットロスの処理や輻輳制御の実装にも使用されます。これにより、変化するネットワークの状況に対応するために必要な双方向の通信が可能になります。

## レイテンシー vs クオリティ

リアルタイムメディアは、遅延と品質のトレードオフの関係にあります。遅延を許容すればするほど、高品質な映像が期待できます。

### 現実の制約

これらの制約は、すべて現実世界の限界に起因するものです。これらの制約は、すべて現実世界の制約に起因するもので、お客様が克服しなければならないネットワークの特性です。

### ビデオは複雑

動画の転送は簡単ではありません。30分の非圧縮720 8bitビデオを保存するには、約110Gb必要です。この数字では、4人での電話会議は不可能です。もっと小さくする方法が必要ですが、その答えは映像の圧縮です。しかし、これにはデメリットもあります。

## ビデオ101

ここでは、動画圧縮について詳しく説明しませんが、RTPがなぜこのように設計されているのかを理解するには十分です。動画圧縮とは、動画を新しいフォーマットにエンコードすることで、同じ動画をより少ないビット数で表現することです。

### 可逆圧縮と可逆圧縮

動画のエンコードは、ロスレス（情報が失われない）とロッシー（情報が失われる可能性がある）の2種類があります。ロスレス圧縮の場合、相手に送るデータ量が多くなり、ストリームの遅延が大きくなったり、パケットの損失が多くなるため、RTPでは映像の品質が悪くなってもロスレス圧縮を行うのが一般的です。

### イントラフレームとインターフレームの圧縮

動画の圧縮には2種類あります。1つ目はイントラフレームです。フレーム内圧縮では、1つのビデオフレームを記述するためのビットを削減します。静止画の圧縮にも同じ手法が使われており、JPEG圧縮法などがあります。

2つ目は、フレーム間圧縮です。動画は多くの画像で構成されているので、同じ情報を2度送らない方法を考えます。

### フレーム間の種類

フレームには3つの種類があります。

* **I-Frame** - 完全な画像で、何もなくてもデコードできます。
* **P-Frame** - 部分的な画像で、前の画像を修正したもの。
* **B-Frame** - 部分的な画像で、以前の画像と未来の画像を組み合わせたもの。

3つのフレームタイプを視覚化すると以下のようになります。

![Frame types](../images/06-frame-types.png "Frame types")

### 動画はデリケート
動画の圧縮は非常にステートフルであり、インターネットでの転送は困難です。I-Frameの一部が失われるとどうなるのか？P-Frameはどうやって修正すべき箇所を知るのでしょうか？映像圧縮がより複雑になるにつれ、この問題はさらに深刻になっています。幸いなことに、RTPとRTCPには解決策があります。

## RTP

### パケットフォーマット

すべてのRTPパケットは、以下のような構造になっています。

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|V=2|P|X|  CC   |M|     PT      |       Sequence Number         |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                           Timestamp                           |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|           Synchronization Source (SSRC) identifier            |
+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+
|            Contributing Source (CSRC) identifiers             |
|                             ....                              |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                            Payload                            |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

#### バージョン (V)
`バージョン` は常に `2` です。

#### パディング (P)

`パディング` はペイロードにパディングがあるかどうかを制御する bool です。

ペイロードの最後のバイトには、何バイトのパディングが追加されたかのカウントが入っています。

#### 拡張 (X)

セットされている場合、RTPヘッダーは拡張機能を持つことになります。これについては、以下で詳しく説明します。

#### CSRC 数 (CC)

`SSRC`の後、ペイロードの前に続く`CSRC`の識別子の数です。

#### マーカー (M)

マーカービットには事前に設定された意味はなく、ユーザーが好きなように使うことができます。

場合によっては、ユーザーが話しているときに設定されることもあります。また、キーフレームのマークとしてもよく使われます。

#### ペイロードタイプ (PT)

`ペイロードタイプ` は、このパケットで伝送されるコーデックを示す一意の識別子です。

WebRTCでは、`ペイロードタイプ`は動的なものです。ある通話での VP8 は、別の通話では異なる可能性があります。通話中の提供者は、`セッション記述`の中で`ペイロードタイプ`とコーデックのマッピングを決定します。

#### シーケンス番号

`シーケンス番号`は、ストリームのパケットの順序付けに使用されます。パケットが送信されるたびに、`シーケンス番号`は1ずつ増加します。

RTPは、損失の多いネットワーク上で役立つように設計されています。これにより、受信者はパケットが失われたことを検出することができます。

#### タイムスタンプ

このパケットのサンプリングの瞬間です。これはグローバルクロックではなく、メディアストリームの中でどれだけの時間が経過したかを示すものです。

#### 同期ソース (SSRC)

`SSRC`は、このストリームの一意の識別子です。これにより、複数のメディアストリームを1つのストリーム上で実行することができます。

#### コントリビューションソース(CSRC)

どの `SSRC` がこのパケットに貢献したかを伝えるリストです。

これは一般的にトーキングインジケーターに使用されます。例えば、サーバー側で複数のオーディオフィードを1つのRTPストリームにまとめたとします。このフィールドを使用して、「入力ストリームAとCがこの瞬間に話していた」と言うことができます。

#### ペイロード
実際のペイロードデータです。パディングフラグが設定されている場合は、何バイトのパディングが追加されたかが最後に表示されます。

### 拡張機能

## RTCP

### Packet Format

RTCPのパケットは、以下のような構造になっています。

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|V=2|P|    RC   |       PT      |             length            |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                            Payload                            |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

#### バージョン (V)

`バージョン` は常に `2` です。

#### パディング (P)

`パディング` は bool で、ペイロードにパディングがあるかどうかを制御します。

ペイロードの最後のバイトには、何バイトのパディングが追加されたかのカウントが含まれています。

#### 受信レポート数 (RC)

このパケットに含まれるレポートの数です。1つのRTCPパケットに複数のイベントを含めることができます。

#### パケットタイプ (PT)

この RTCP パケットがどのタイプであるかを示す一意の識別子です。WebRTCエージェントは、これらのタイプをすべてサポートする必要はなく、エージェントによってサポートが異なる場合があります。しかし、一般的に目にするのはこれらのタイプです。

* Full INTRA-frame Request (FIR) - `192`
* Negative ACKnowledgements (NACK) - `193`
* Sender Report - `200`
* Receiver Report - `201`
* Generic RTP Feedback - `205`

これらのパケットタイプの意義については、以下で詳しく説明します。

### フルイントラフレームリクエスト (Full INTRA-frame Request)

このRTCPメッセージは、送信者がフル画像を送信する必要があることを通知します。エンコーダーから部分的なフレームが送られてきて、それをデコードできない場合に使用します。

これは、パケットロスが多かったか、デコーダがクラッシュしたことが原因と考えられます。

### Negative ACKnowledgements

NACKは、送信者に1つのRTPパケットの再送を要求するものです。これは通常、RTP パケットが失われたときに発生しますが、遅延した場合にも発生します。

NACKは、フレーム全体の再送信を要求するよりも、はるかに帯域幅を効率的に利用できます。RTPはパケットを非常に小さなチャンクに分割するので、実際には1つの小さな欠落部分を要求しているに過ぎません。

### 送信者/受信者レポート

これらのレポートは、エージェント間で統計情報を送信するために使用します。このレポートでは、実際に受信したパケット量やジッターを伝えます。

このレポートは、診断や輻輳制御に利用できます。

## RTP/RTCPが共に問題を解決する方法

このように、RTPとRTCPが連携することで、ネットワークに起因するあらゆる問題を解決することができます。これらの技術は今でも常に変化しています。

### Negative Acknowledgment

NACKとも呼ばれます。これは、RTPのパケットロスに対処する方法のひとつです。

NACKは、再送信を要求するために送信者に送り返されるRTCPメッセージです。受信者は、SSRCとシーケンス番号を含むRTCPメッセージを作ります。送信者は、再送信可能なこのRTPパケットを持っていない場合、そのメッセージを無視します。

### 前方誤り訂正 (Forward Error Correction)

FECとも呼ばれます。パケットロスに対処するもう一つの方法です。FEC は、同じデータを要求されてもいないのに複数回送信することです。これは、RTP レベルで行われ、さらに下位のコーデックでも行われます。

通話中のパケットロスが安定している場合、FECはNACKよりもはるかに低遅延のソリューションです。NACKの場合は、パケットを要求してから再送信するまでの往復時間が大きくなります。

### 適応型ビットレートと帯域幅の推定 (Adaptive Bitrate and Bandwidth Estimation)
[リアルタイムネットワーキング](../05-real-time-networking/)で説明したように、ネットワークは予測不可能で信頼性がありません。帯域幅の利用可能性は、セッション中に何度も変化する可能性があります。
利用可能な帯域幅が1秒以内に劇的に（桁違いに）変化することも珍しくありません。

主なアイデアは、予測される、現在および将来の利用可能なネットワーク帯域幅に基づいて、エンコーディングのビットレートを調整することです。
これにより、可能な限り最高の品質の映像・音声信号を伝送し、ネットワークの混雑によって接続が切断されることがないようにします。
ネットワークの挙動をモデル化し、それを予測するヒューリスティックな手法を「帯域推定」といいます。

これには様々なニュアンスがありますので、詳しくご紹介しましょう。

## ネットワークの状態を伝える
輻輳制御を実装する上での最初の障害は、UDPとRTPがネットワークの状態を通信しないことです。送信者としては、自分のパケットがいつ到着したのか、あるいは到着しているのかどうか、まったくわかりません。

RTP/RTCPには、この問題に対する3つの異なるソリューションがあります。それぞれに長所と短所があります。どのソリューションを使用するかは、どのようなクライアントを使用するかによります。トポロジーはどうなっているのか。あるいは、どれだけの開発時間を確保できるかによっても異なります。

### 受信者レポート (Receiver Reports)

受信者レポートはRTCPメッセージであり、ネットワークステータスを伝達するための元の方法です。 それらは[RFC1889](https://tools.ietf.org/html/rfc1889)で見つけることができます。 これらは各SSRCのスケジュールで送信され、次のフィールドが含まれています。

* **Fraction Lost** -- 前回のReceiver Report以降、何パーセントのパケットが失われたか。
**Cumulative Number of Packets Lost** -- 通話全体で失われたパケット数。
**Extended Highest Sequence Number Received** -- 最後に受信したシーケンス番号と、それが何回ロールオーバーしたかを示しています。
**Interarrival Jitter** -- 呼全体のローリングジッターです。

### TMMBR, TMMBN, REMB

次世代のネットワーク・ステータス・メッセージでは、すべての受信者がRTCPを介して送信者に明示的なビットレート要求をメッセージします。

* **Temporary Maximum Media Stream Bit Rate Request** - 1つのSSRCに対する要求ビットレートの仮数/指数です。
* **Temporary Maximum Media Stream Bit Rate Notification** - TMMBRを受信したことを通知するメッセージです。
* **Receiver Estimated Maximum Bitrate** - セッション全体に対して要求されたビットレートの仮数/指数です。

TMMBRとTMMBNが先に登場し、[RFC 5104](https://tools.ietf.org/html/rfc5104)で定義されています。REMBは後に登場し、[draft-alvestrand-rmcat-remb](https://tools.ietf.org/html/draft-alvestrand-rmcat-remb-03)でドラフトが提出されましたが、標準化されませんでした。

REMBを使用したセッションは以下のようになります。

![REMB](../images/06-remb.png "REMB")

### トランスポートワイド輻輳制御 (Transport Wide Congestion Control)

トランスポートワイド輻輳制御は、RTCPのネットワークステータス通信の最新の開発です。

TWCCは非常にシンプルな原理を使用しています。

![TWCC](../images/06-twcc-idea.png "TWCC")

REMBとは異なり、TWCC受信機は自分の受信ビットレートを推定しようとはしません。TWCC受信機は、どのパケットがいつ受信されたかを送信者に知らせるだけです。送信者は、これらのレポートに基づいて、ネットワークで何が起こっているかを最新の状態で把握することができます。

- 送信者は、パケットシーケンス番号のリストを含む、特別なTWCCヘッダー拡張を持つRTPパケットを作成します。
- 受信者は、各パケットがいつ受信されたかを送信者に知らせる特別なRTCPフィードバックメッセージで応答します。

送信者は、送信したパケット、そのシーケンス番号、サイズ、タイムスタンプを記録します。
送信者は、受信者からRTCPメッセージを受信すると、送信側のパケット間遅延と受信側の遅延を比較します。
受信遅延が増加した場合は、ネットワークの混雑が発生していることを意味し、送信者はそれに対処する必要があります。

下の図では、インターパケット遅延の増加の中央値は+20ミリ秒で、ネットワークの混雑が起きていることを明確に示しています。

![TWCC with delay](../images/06-twcc.png "TWCC with delay")

TWCCは生のデータを提供し、ネットワークの状態をリアルタイムに把握することができます:

- パケットロスの統計情報をほぼ瞬時に確認でき、ロスの割合だけでなく、ロスした正確なパケットも確認できます。
- 正確な送信ビットレート。
- 正確な受信ビットレート
- ジッターの推定値。
- 送信パケットと受信パケットの遅延時間の違い。

送信側から受信側への受信ビットレートを推定するための些細な輻輳制御アルゴリズムは、受信したパケットサイズを合計し、それを経過したリモートタイムで割ることです。

## 帯域幅の推定値の生成

ネットワークの状態に関する情報が得られたので、利用可能な帯域幅を推定することができます。2012年、IETFはRMCAT（RTP Media Congestion Avoidance Techniques）ワーキンググループを立ち上げました。
このワーキンググループには、輻輳制御アルゴリズムに関する複数の規格が提出されています。それ以前は、すべての輻輳制御アルゴリズムは独自のものでした。

最も導入されているのは、[draft-alvestrand-rmcat-congestion](https://tools.ietf.org/html/draft-alvestrand-rmcat-congestion-02)で定義されている「A Google Congestion Control Algorithm for Real-Time Communication」です。
このアルゴリズムは、2つのパスで実行されます。まず、受信機レポートのみを使用する「損失ベース」のパス。TWCCが利用可能な場合は、その追加データも考慮されます。
また、[カルマンフィルター](https://en.wikipedia.org/wiki/Kalman_filter)を使用して、現在および将来のネットワーク帯域幅を予測します。

GCCに代わるものとしては、[NADA: A Unified Congestion Control Scheme for Real-Time Media](https://tools.ietf.org/html/draft-zhu-rmcat-nada-04)や[SCReAM - Self-Clocked Rate Adaptation for Multimedia](https://tools.ietf.org/html/draft-johansson-rmcat-scream-cc-05)などがあります。