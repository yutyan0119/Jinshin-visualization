# 中村の進捗

# 11/7日分
- テーマが決まりきっていないのでテーマ決めから
前回出した案
- YouTubeの好きなチャンネルの情報を可視化 再生数＋タグ
- 去年の鉄道マップの全国版（ヤフー乗り換えとかぶらないようにしないといけない）
- 飛行機事故の可視化
- 鉄道人身事故のデータベースの視覚化

今日出した案
- 統計局の世界のデータベース的なのを紹介してそれをあさってみている


**一旦鉄道人身事故データベースの可視化でやる**

## 可視化する内容
- 日本地図と路線図
    - 去年の路線図のデータみたくどこかにデータが落ちているはず
- 日時情報
    - https://jinshinjiko.com/
    - ↑をひたすらスクレイピング
- 人身事故による影響の大きさ（Tweet数から算出）
    - 事故発生時 - 2時間後までの関連する路線名で検索したときのTweet数で出す
    - Twitter API申請済み
- ホームドアの設置状況（WebArchiveから出す）
    - https://www.mlit.go.jp/tetudo/tetudo_tk6_000022.html
    - web archiveに過去7年くらいのデータがある

TOP Page
![](shinchoku_image/nakamura/CamScanner%2011-07-2022%2015.55.jpg)

## どうやって情報を蓄積するかの案
- 基本CSVでよさそう（RDB使いたいなら後から使えばいい）
- 人身事故のデータは基本URL/IDになっているので簡単にスクレイピング出来る
    - curlで取得出来たのでseleniumもいらなさそう
    - https://jinshinjiko.com/accidents/14847
- 一覧については[https://jinshinjiko.com/accidents?limit=10&offset=0](https://jinshinjiko.com/accidents?limit=10&offset=0)のlimitを14898にすることで全件が一気に取得できた

- サクッと[csv](data/jinshin.csv)作成
    -  作成元は[このプログラム](nakamura/dataextract.ipynb)
    -  各事故の詳細は追ってスクレイピングするで良さそう
    -  駅間をどうするか、会社の表記ゆれをどう対応するか

# 11/8
[csv](data/jinshin_archive_sentence.csv)に
- 事故の文面（サイトに掲載のもの）
- ニュースサイトの魚拓URL（サイトからリンクであれば）
を追加するために追加でスクレイピングした。

路線図データを入手するべくgeojsonの入手先を発見

https://uedayou.net/jrslod-geojson-downloader/

ダウンロード自動化をしようと思ったが、複雑そうなので一旦断念してJR東日本までダウンロードしたところで加賀谷くんに頼む。

d3.jsで路線図のデータを描画するようにした。基本はjapan.htmlを参考にした。
![](shinchoku_image/nakamura/image%20(1).png)

全路線図のデータを描画することに成功した。駅のポイントの点も任意の形に変えられるようになった。
![](shinchoku_image/nakamura/rosenzu.png)

# 11/10
geojsonの構造を見ると、以下のようになっている。
propertiesには好きな情報を入れて良い。geometryのtypeで点なのか線なのか判別可能。
ホームドア設置状況はpoitn側のpropertiesの中に`"door": "2016-04-01"`のようにして追加できれば追加でcsvを読み込まなくても良いのでこれを行う。
```json
        {
            "properties": {
                "name": "盛岡",
                "uri": "https://uedayou.net/jrslod/IGRいわて銀河鉄道/いわて銀河鉄道線/盛岡",
                "color": "0000CD"
            },
            "type": "Feature",
            "geometry": {
                "type": "LineString",
                "coordinates": [
                    [
                        141.13802,
                        39.69969
                    ],
                    [
                        141.13473,
                        39.7027
                    ]
                ]
            }
        },
        {
            "properties": {
                "name": "盛岡",
                "uri": "https://uedayou.net/jrslod/IGRいわて銀河鉄道/いわて銀河鉄道線/盛岡"
            },
            "type": "Feature",
            "geometry": {
                "type": "Point",
                "coordinates": [
                    141.13473,
                    39.7027
                ]
            }
        },
```
次に人身事故のデータと、駅データの結びつけ方を考える必要がある。

- QAR
    - 時系列シークバー
        - 加賀谷くん
    - ズームインアウト、移動
        - ちょざい
    - 時計
        - 自分
    - 時系列グラフ
        - 簡単なのでパス

- 作業
    - geojsonのpropertiesに路線とホームドア設置日時を追加
        - 路線はちょざい？
        - ホームドアは一旦既存データの日時の変換を加賀谷くんがやる
    - 駅間で起きた人身事故の対処法を考える
        - 自分で考える

時計の進捗[はここ](nakamura/clock.html)
- まず時計を描画した（webからsampleを引っ張ってきて、バージョンの差異を修正した）
- 次に時計の針とかをぐるぐる回せるようにした
- 任意のデータ（時刻と値のペア）を、時計上に表示できるようにした
- 実際の人身事故データのうち、時刻のみを取り出して、時刻別に件数を計算してjsonに保存する[スクリプトをPythonで書いてjsonを保存](nakamura/dataextract.ipynb)
    - jsonは[ここ](nakamura/timeonly.json)
- d3.jsでjsonを読み込んで、実際のデータで描画出来るようにした。
![](shinchoku_image/nakamura/clock.png)
18:36注：
なんかバグってたので、2点を割り出すアルゴリズムと、元のデータを変更（事故の起こっていない時刻を0埋め）して、修正した。
![](shinchoku_image/nakamura/clock2.png)

見づらいので、15分か30分おきに4文意数を取って描画するほうがよさそう。
0, 25, 25, 75, 100%点をjsonに保存して（例によってnotebookでpythonで行った）、描画するようにした。

![](shinchoku_image/nakamura/clock3.png)

TODO:曜日ごとに変更出来るようにする。
- 曜日ごとのjsonを用意するところから必要

※前回の進捗で忘れてた分
- [このjsファイル](nakamura/datalist.js)は、[このスクリプト](nakamura/find_geojson.py)でgeojsonを再帰的に探索し、そのパスをjsファイルとして保存するもので、これによって一括でデータを管理することが出来るようになった。


# 11/14

- 加賀谷くんが前回のQARで実装してくれたグラフのシークバーを利用して、[時間経過を実装した](https://github.com/InfovisHandsOn/A-Pastani/issues/10)
    - 2010/01/01のUNIX TIMEを始点として、2022/11/10までをシークバーで移動することが出来る。
    - 現在の時刻がわかるように表示を追加した。
- 時間経過から、起こった人身事故を抽出するプログラムを実装した。
    - 移動開始時の時刻と、移動終了後の時刻の間の領域に該当する人身事故を抽出する。
    - どちらかの時刻にもっとも近い（超えない範囲で）人身事故を二分探査で求めて範囲で返すようにした。
        - 全探査でやるとサイト全体が重くなり使い物にならなかった
- ↑のプログラムから、人身事故の路線と場所がわかるようになったので、特定の駅で起きた人身事故について可視化するプログラムを仮で書いた。
    - 仮である理由は以下。
        1. おもすぎて使い物にならない
        2. 同名の駅が省くことが出来ていない
    - ちょざいが作成した辞書を利用することでずっと簡単に探索出来るようになるはず。

# 11/15
- 足りないgeojsonの追加
- [辞書データ](data/jinshin_to_geojson_linename_dic.py)の中身を検査して、足りない部分をピックアップした。
  - valueを直接ファイルの存在する部分へのpathへと変更した新たな[辞書](home/dict.js)を生成した
  - 使用したプログラムは[これ](nakamura/jinshin_to_geojson_linename_dic.py)
- 辞書データを使用して、ページの軽量化を行った。
  - https://github.com/InfovisHandsOn/A-Pastani/commit/9b81f3a53e9c1b044d765460311f62326a4e2eed#diff-dfabdc08c38c171a9fad6493031f1f7312d76c8fda2056b973cc83707e163201
- 人身事故発生時に出る円の大きさを、事故発生時の関連ツイートの多さに応じて大きくするようにした
  - https://github.com/InfovisHandsOn/A-Pastani/commit/27c3eda9064ed4511ef540a8551cf49763042c63
- 駅データのみのgeojsonを新たに生成し、これを使うことで駅の表示を残しながら大幅な軽量化をした
  - https://github.com/InfovisHandsOn/A-Pastani/commit/a753014d76e87d7a6b978ef4e9f9066a248afcaa
  - 作成したプログラムは[これ](nakamura/geojson_kairyo.ipynb)
  - 作成したデータは[これ](data/station_only.geojson)
- 時間を飛ばすと重くなる問題に対して、飛ばした後の時間から近いものから20個を表示する最大にすることで、軽量化した。
  - https://github.com/InfovisHandsOn/A-Pastani/commit/ac020bf63027de9d851ec33e015134cec96102d9
- zoomに対応して各種表示の調整を行った。

# 11/21
## 前回(11/15)から今日までの進捗
- ホームドア設置件数のグラフがバグっているのを修正した。[commit](https://github.com/InfovisHandsOn/A-Pastani/commit/96ac588a9db41cfd8c71bb5495fd610c071730c3)
- 路線ごとにフィルターをかけられるようにした。[commit](https://github.com/InfovisHandsOn/A-Pastani/commit/c52a4bd9c589227a2134ef3d452c07685a4b97b7)
- 任意の時刻に入力から飛べるようにした。[commit](https://github.com/InfovisHandsOn/A-Pastani/commit/e5bd27ebc030c83758a83a3783987097e10fd50f)
- geojsonを改良して、Pointのみのデータを生成し、サイトを軽量化した。[commit](https://github.com/InfovisHandsOn/A-Pastani/commit/f4cdaa23d7891d2430ac3a09e8dac5b288f9d5be)
- tooltipの消えるタイミングをmouseremoveから、ユーザーが地図を動かしたときに変更した。[commit](https://github.com/InfovisHandsOn/A-Pastani/commit/072250cd73b3b88f7d69856c8f05b27188215c3c)

## 今日の進捗
- flexboxに対応して表示を修正した。[commit](https://github.com/InfovisHandsOn/A-Pastani/commit/0658d7678ed9552633ddb7469aed5293780bd609)
- speedを任意の値に変えられるように変更した。[commit](https://github.com/InfovisHandsOn/A-Pastani/commit/0658d7678ed9552633ddb7469aed5293780bd609)
- より拡大できるようにした。[commit](https://github.com/InfovisHandsOn/A-Pastani/commit/5720d444935529290b6ef5a749726995e3341623)
- 操作しやすいようにUIの変更。[commit](https://github.com/InfovisHandsOn/A-Pastani/commit/e70ff5afe419f6f6e4431f8ba2fbfac2baa814d8)
- これまでの成果物として別にあった時計の表示をindex.htmlに統合。[commit](https://github.com/InfovisHandsOn/A-Pastani/commit/c9d5a6944ae060214dfa13357489761e38b1af54)
- homedoor表示の軽量化(CSVのソートとバイナリサーチの実装)。[commit](https://github.com/InfovisHandsOn/A-Pastani/commit/98cc60b06ca9f1d123cb2b35a7c3c7791389e826)
- ホームドア設置駅の表示の改良と、人身事故発生駅の発生件数に応じた円の表示

![](shinchoku_image/nakamura/jinshin_circle.png)

# 11/22
- 全体的なデザインの調整を行った
  - zoom時の表示の改良
  - ページ全体デザインの調整
- スライドを作った
  - https://docs.google.com/presentation/d/1y_yHa6pFSITmGuRu9BXcaB6W8Nmb7XwHwcQzzGtApK4/edit?usp=sharing  
- light-modeの追加
- 選択肢に人身事故を表示しないを追加