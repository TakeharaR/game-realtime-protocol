ゲームリアルタイム通信プロトコル 第二回会合
===
当資料は "ゲームリアルタイム通信プロトコル 第二回会合" の議事録(メンバー向け)です。


# IETF 119 報告

- [ゲーム目線のインフラエンジニア IETFにおけるQUIC, HTTP3動向 2](/mtg002/quic_and_h3_at_ietf_2.pdf)
- [CCWGとmlcodecについての報告](/mtg002/ccwg_and_mlcodex_at_ietf119.pdf)


# 事前募集議題

### 参加者の当会合への貢献について

**議題補足**

ジュニアな参加者（モチベーションはある初学者）が当会合に貢献できることはあるか？
前回の議事録を見ていても分からない箇所があり、参加しようにも初学者には敷居が高い。
会の目的も初学者が議論に参加し辛い。

**議論**

- 参加して頂けるだけで十分な貢献になる
    - 前回の議論の中に「どう若手を育てるか」というものもあった
        - みんな教えたがりなので気軽に聞いて欲しい
    - もう少し参加し易い空気感を醸成する必要はありそう
        - 会の趣旨的にあえてハードルを上げている面はある(のでバランスを見つつ)
- あると良さそうな取り組み
    - Slack で自由に質問可能な部屋を作る
    - QUIC の仕様書読み合わせ会のようなコンテンツ
    - OSS 等の実装を繋げてみるようなハンズオン
- ユースケースや困っている事例を持ち寄るような、テクニカルな話以外を共有するのも非常に重要


### 携帯キャリアのデータ通信品質の低下

**議題補足**

どうしようというふわっとした問題意識がある。

**議論**

- いつ直るかもわからないので対策をしてゲームプレイできるようにしよう、と割り切るのも手
    - 下がった品質に合わせて作るしかない
    - 品質低下の方向性次第ではプロトコル面でケアできる可能性はある
- 低品質回線のテストをどうしているか
    - 会社の Wi-Fi 環境が悪い時期があり、会社での確認が低品質回線のテストを兼ねていたことがあった
        - 期せずして会社全体の通信エラーに関するハンドリングが向上した
        - 会社にそういう環境があるのも重要
    - パケット遅延マシンを用いるのも良い
        - KONAMI さんがラズベリーパイ上で動作するものを公開している
            - [EM-uNetPi](https://github.com/KONAMI/EM-uNetPi)
            - アップデートで機能追加予定
- 色々な会社がレポートをまとめて総務省に投げて国の力でご指導いただくのが正攻法
    - 国もふわっとした報告だと動けないのできちんとした報告を行う必要がある
    - クライアントアプリに新規に仕込むのが難しい問題
        - アプリ開発者にこの機能を入れるのを許容して貰えるかは企業文化に依存する
            - 全社共通基盤でログ送信を行う機構があるような場合はそこに載せればいけそう
    - アプリ側ではなく、 CDN ベンダーと協力してベンダー側でデータを採って貰う手も有る
        - パッチ配信のデータ取得等
    - 個人情報の収集に当たらないか？
        - ユーザに同意を得てまで収集する機能を追加するのは難しい
            - 包括同意でいけるかも？
            - キャリアを直接送れない場合でも、クライアントの IP アドレスからキャリアを逆引きする仕組みを入れれば特定はできそう
    - 結論 : 各社がそれぞれ対応するのは難しいので業界団体が主導で動ないと厳しそう


### KCPのC#実装 : KcpTransport(RUDP)

**議題補足**

- RUDP プロトコルとして KCP の C# 実装を作っている
    - https://github.com/Cysharp/KcpTransport/blob/main/src/KcpTransport/LowLevel/ikcpc.cs
    - KCP : https://github.com/skywind3000/kcp
- KCP が最適かどうかは分からないが、中国では数多くの実績がある
    - 国内の採用実績はある？
- KCP 自体はエンコードとデコードだけで通信層がないので逆に使いやすい
    - どうしても他のライブラリは通信層も抱えた全部盛りみたいなのが多い
- 発端 : メタバースプラットフォーム「バーチャルキャスト」の実例を見て
    - https://www.docswell.com/s/torisoup/5YWX2E-xrkaigi2023
    - アニメーションや位置指定の同期には C++ の RUDP 実装が併用されている
    - C# だけでもスループット優先のシナリオでも使えるようにしたい
- 「YetAnotherHttpHandler」は使われ出している
    - Rust の hyper, Rustls ベースの HTTP/2 通信ライブラリ
- KcpTransport は Pure C# での移植を試みている
    - ネイティブライブラリはコンソール機対応に難を抱えそうな気がする
        - YetAnotherHttpHandler も hyper, Rustls といった Rust ライブラリを動かしきれないとコンソールでは動かせない懸念がある
        - Pure C# だとその辺の不安がなくなるのが楽
    - QUIC を選択しなかったのもこの辺りの事情から
        - C#/.NET の場合は MsQuic が下地になっているのでビルドが重量級
            - Xbox は問題ない
        - 何も考えずに QUIC が使える状況なら QUIC で行きたいとは思っているが……
- 最終的には MagicOnion に乗せて gRPC over KCP にするつもり
    - MagicOnion が gRPC ベース
- gRPC over HTTP/3 は進捗がない
    - https://github.com/grpc/proposal/blob/master/G2-http3-protocol.md
    - gRPC のメインユースケース(安定したデータセンター間通信)的にそんなに需要もない？
    - 現状 .NET では使える（はず）だが、 Unreliable Datagram 対応とかも考えると、無視して（？）gRPC over QUIC みたいなオレオレの作りにしてしまったほうがいい（？）
- Reliable な通信において TCP じゃなくて RUDP が必要とされる条件を整理したい
    - パッと浮かぶ（期待してる）ところ
        - IP アドレスの変更でも繋がり続ける
        - HoL ブロッキングが防げる
        - アプリケーションの事情に沿った賢めの再送ができる
    - その他
        - UDP で繋がらないケースで TCP にフォールバックする必要は一般的にある？

**議論**

- QUIC 話
    - ゲームにおける実用しんどい話
        - Android/iOS は大変だけど何とかなるが、コンシューマ機(Xbox 除く)は NDA の関係で情報も出てこないので難しい
            - コンシューマ機の情報は現状この会でも取り扱うのが難しい
    - ゲームでの採用事例はある？
        - QUIC の仕様が最初に提出されて 10 年位経っていて拡張もどんどん提案されているのにまだゲームでの採用事例が無い
            - となるとこの先も採用されるのか疑問
            - 今まで QUIC の実装が出てからそこに乗っかる待ちの姿勢でいたが、このままだと進まなさそうなので別途 RUDP ベースのライブラリを作成することにした
        - 比較的回線環境が安定している(モバイル回線ではない)ので、コンシューマ側の QUIC 採用のモチベーションが低いのかもしれない
           - (主幹)プラットフォーマーの関係者の参加勧誘頑張ります
    - ゲームにおける採用のモチベーション
        - QUIC の機能自体がオーバースペックなケースもある
            - マルチパスがあれば QUIC でなくても良い等
        - パケットロス検知等、便利な機能は全部使いたいケースもある
            - QUIC 自体練り込まれた仕様なので、自前でそれを超える品質のプロトコルを作成するのは難しい(特にセキュリティ)
    - gRPC over HTTP/3 話
        - C# はマルチプラットフォームだが HTTP/3(QUIC) 実装に使われている MsQuic がそうではない
            - 別のプラットフォームで使おうとするとデフォルトでは警告が出る
            - MsQuic は Schannel を使っているので動かない環境ではその部分を OpenSSL に差し替える必要がある
        - gRPC は HTTP/2 の拡張のような側面があるので標準化しようとすると一から考え直す必要がある
            - 標準化せずに進める方が楽
    - Connection Migration
        - 誰が発動させるか問題
            - 意図的に発動するケースとコネクションを維持する為に発動するケースがある
            - ライブラリがどこまで実装するか
                - Chromium を用いた実装を行う場合、通信部は閉じられているので接続維持のようなケースではユーザは特に何もせずに発動する
                - 過去 IPv4-6 のデュアルスタック環境で複数のセッションを張り一番良い品質のもので接続する実装を作成したことがあるが、アプリ側に自分で選びたいという要望を貰ったことがある
                - より良い回線に乗り換えるようなケースでは、切り替え先の通信が本当に「そのアプリケーションにとって」品質が良いかを考慮すると、どうしてもアプリ側で判断しないといけないケースは出てしまう
                - iOS のマルチパスは四つ程ポリシーがあるのでそのような形に落とし込むのもあり
        - リアルタイム通信に使えるか
            - リアルタイム性がそこまで高くないゲームでは使える
            - リアルタイム性が高い場合は逆に入れないで欲しいようなケースもある
- KCP 話
    - パケロス時の対応等、自前で書かないといけない部分が結構あるが、その分柔軟性が高い
        - 原神も KCP を使っているが、魔改造しているらしい
        - 内容的にも 1500 行程度なので把握し易い
    - 国内採用実績
        - Tetris 99 のオープンソース一覧に載っていた
        - Python で任天堂コンソールのクライアントを実装しているプロジェクトに KCP を使っているらしきタイトルの一例がリストになっている
            - https://github.com/kinnay/NintendoClients/wiki/Eagle-Protocol
                - Tetris 99 / Super Mario Bros. 35 / PAC-MAN 99 / F-Zero 99
- RUDP 採用のモチベーション
    - Reliable/Unreliable を同じプロトコルで制御したい
        - 過去 Reliable 通信だけ遅延が大きすぎたことがあった
    - 再送等のアルゴリズムをカスタイマイズしているゲーム向け実装は結構ある
- UDP で繋がらないケースで TCP にフォールバックする必要
    - UDP が通らない環境はまだある
        - Proxy で止められている環境やマンションの NAT テーブルがあふれるようなケースは存在する
        - IPv4 は通るが IPv6 は通らない、またその逆のような環境もある
        - Google がデータを出さなくなったので詳細なデータは分からない
    - 通らない場合に TCP にフォールバックするかは難しいところ
        - 想定ユーザの環境ではないので諦める、サーバ経由にする等タイトルの規模や都合により変わる


### sendmmsg, recvmmsg

**議題補足**

- UDP サーバーを Linux で作るなら sendmmsg, recvmmsg を使うべきみたいな風潮がある（？）
    - ちなみに C# は対応していないのでそのままだとどうやっても叩けない
    - 3rd Party Library 経由でやるという手段は一応ある
        - https://github.com/tmds/Tmds.LibC
- 上記よりもオフロードのほうが圧倒的に効くぞ、みたいな話もある
    - https://twitter.com/majek04/status/1701132060897817007
    - Tailscale での QUIC/UDP のセグメンテーションオフロードで4倍高速化
        - https://tailscale.com/blog/quic-udp-throughput
- こうした設定や機器の構成において一般的なプラクティスやよく参照される事例があれば知りたい


**議論**

- sendmmsg/recvmmsg はあまり効く印象が無い
    - UDP の場合はパケット単位でルーティングを決めないといけないのが重い
    - sendmmsg, recvmmsg は複数のパケットを同時にカーネルに渡せるが、ルーティング情報はパケット毎の処理なので結局早くならない
- GSO や GRO は複数のパケットをルーティング情報もまとめて NIC にもっていけるので効く
- ユースケース : Golang で sendmmsg/recvmmsg を使ったらメモリの接合部でロスが発生してパフォーマンスが出なかった


# Slack 話題

### TWAMP（RFC5357）

**議題補足**

- TWAMP（RFC5357）を組み込んで使ってる人がいるか知りたい
    - リアルタイム通信できる環境か計測するためのプロトコル(L3)
- 片方向遅延も計測可能なので PING よりも有用な値が拾える
    - ただし NAT 対応してないので自力で Fork しないといけない

**議論**

- 遅延やジッタ、パケロス率等の一般的なネットワーク計測に必要とされる値を計測・グラフ化できる
    - PING で計測するより信頼できる値を取得できる
        - ゲーム業界的にいつまでも PING の値を参考値にするのはどうか、という問題がある
- TWANP を使って計測しておくとネットワーク系のベンダーに相談をする際に話がし易い
    - 5G 等も TWANP を使って計測されている
- 欠点
    - ベンダー実装の方言が普及している
        - RFC 通り実装しても動かないことがある
        - OSS をそのまま試しても動かないこともある
    - NAT 対応していない
- ゲーム向けの実装が欲しい
    - プラットフォーム側で用意されていると嬉しい


### QoE 計測のオンラインゲームのテストモデル

**議題補足**

- ネットワークの QoE 計測について、国内でも世界でも動画視聴が指標としてデファクトになりつつある
    - https://blog.nic.ad.jp/2024/9615/
- 特定のプロトコルを用いたオンラインゲームのテストモデルのようなものをゲーム業界から提供して、「対戦中の同期待ちフレーム数」とか「画面遷移でのウェイト発生回数」というような QoE 計測モデルを提示したい
    - そうしないとビデオ視聴の為のインターネットになってしまう

**議論**

- この計測値は ISP 等がユーザに安心してインターネットを使って貰う為のもの
- CDN ベンダーからはパケロス率は取れるが、ユーザがどういう通信をしているか、輻輳制御に何を用いているかが分からないと意味はない
    - ゲーム側で具体的な数値を出した方が CDN ベンダー的にも改善に取り組み易い
- 昔各社でネットワーク回線検証サービスが流行ったこともあったが今は無くなっている
    - 回線検証は帯域をかなり使うのでその利用料金が問題になって止めざるを得なかった
- (主幹)将来的に「ゲーム・エンタメNW接続性課題検討WG」と連携して取り組んでいきたい


### GDC2024

**議題補足**

- 鉄拳8 × Diarkis さんの動画は無料で誰でも閲覧可能
    - https://gdcvault.com/play/1034632/Diarkis-Inc-The-Team-Behind
    - 日本語字幕もある

**議論**

- 鉄拳8 × Diarkis さんの動画
    - リアルタイム通信(プロトコル)に関わる話はほとんどない
- その他リアルタイム通信に関わるセッションは無さそう


# 議論の整理

### 暗号化

- 第一回議事録より
    - 誰と誰の通信を何から守りたいのか、を明確にして分けて議論すべき
        - 暗号化と難読化で言葉を分けた方が良い
        - 権利の無い第三者からの攻撃を守りたいのか、通信当事者からの攻撃も守りたいのか
    - クライアント、サーバ、データセンター内等セグメントを分けて話した方が良い
    - 次回までに主幹で整理して、再び議論を実施
- 主幹整理
    - 暗号化と難読化で言葉を分ける
        - 暗号化
            - 通信データを暗号化する
            - 第三者からの盗聴や成りすまし、改ざんを防止する
        - 難読化
            - ドライブやメモリ等のローカルに保存されるゲームのデータそのものを暗号化する
            - ユーザ(通信者)本人からのデータの不正利用やチートを防止する
            - ※もっと良い表現あれば募集です……
    - 話題に挙げる際は発散しないようにスコープを限定する
        - 攻撃者
            - ユーザ(通信者)本人、第三者 等
        - 防ぎたい攻撃対象
            - チート、盗聴、成りすまし等
        - セグメント
            - クライアント、サーバ、データセンター 等

### リアルタイム通信まわりの現状整理

以下の項目に分けて整理・議論を実施していく。

- プロトコル
- 実装
- 暗号化・難読化
- 制御アルゴリズム (輻輳制御・フロー制御)
- その他
    - クロスプラットフォームの課題
    - (項目募集中)

今回は時間が無かったので、以下のプロトコルに関する補足を共有した所までで終了。

- プロトコルそのものの概要は知っている前提で、ゲームにおけるユースケース、問題点をまとめる
- 別途初学者向けに上記にプロトコルの概要を付け足した情報を公開するのも良いかも


# 主幹からの連絡

### Slack のログについて

まだ投稿が多くないので「ログ保存用」チャンネルに添付する形でテキスト保存している。

### 次回開催

- IETF 120(7/20-26) 終了の翌月(2024/8) に開催予定
- CEDEC 2024 と開催月が被っているので CEDEC 開催に合わせた懇親会実施は行わない
    - 第三回会合後の懇親会は実施予定
