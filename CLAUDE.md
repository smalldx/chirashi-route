# チラシ配布ルート(岐阜市) — Claude Code 用メモ

## これは何か
岐阜市で「事務作業短縮DXサービス」のチラシを配るための、スマホ専用Webアプリ。
事務作業の多い事業所を業種で選び、出発地点(現在地・住所・地図タップ)から効率よく回る順番を作り、
Googleマップのナビに渡す。配布済みの記録もスマホ内に残す。

- 公開URL: https://smalldx.github.io/chirashi-route/ (GitHub Pages, main ブランチ / ルート)
- リポジトリ: https://github.com/smalldx/chirashi-route
- 利用者はスマホでしか使わない。PC表示は考慮不要。UI文言はすべて日本語。

## 構成
ビルドなし・依存インストールなしの **index.html 1ファイル** 構成(HTML/CSS/JSを全部この中に書く)。
外部ライブラリはCDNから読み込む:
- Leaflet 1.9.4(cdnjs)… 地図
- SortableJS 1.15.2(cdnjs)… 並べ替えのドラッグ
- 地図タイル: 国土地理院 淡色地図 `https://cyberjapandata.gsi.go.jp/xyz/pale/{z}/{x}/{y}.png`
- 住所検索: 国土地理院 地名検索API `https://msearch.gsi.go.jp/address-search/AddressSearch?q=`
APIキーは一切使っていない。今後もキー不要の構成を保つ。

## index.html の中身(上から順)
1. `<style>` … CSS変数(--bg, --card, --accent など)、ダークモード対応、下タブ、カード等
2. HTML … 3つのview(`#vRoute` ルート / `#vDone` 配布済み / `#vSet` 設定)+ 下タブ `.tabs`
   + 追加シート `#addSheet` + トースト `#toast`
3. `<script>`
   - `CATS` … 業種の定義(キー → label, color)
   - `PLACES` … 配布先データ。配列 `[id, cat, name, addr, tel, lat, lng]` を map でオブジェクト化
   - `PRESETS` … 出発地点の候補(JR岐阜駅など)
   - `store` … localStorage(キー `flyerRoute.v1`)に保存する状態
   - `km()` / `optimize()` … 距離計算と、最近傍法+2-opt による回る順番の最適化(出発地固定・戻らない片道)
   - `compute()` → `renderRoute()` / `drawMap()` / `renderDone()` … 描画。何か変えたら `render()` を呼ぶ

## store の主なキー
| キー | 内容 |
|---|---|
| done | `{id: "YYYY-MM-DD"}` 配布済みと配布日 |
| memo | `{id: "メモ"}` |
| skip | `{id: "YYYY-MM-DD"}` 今日だけスキップ(日付が変わると起動時に消える) |
| extra | `{id: "YYYY-MM-DD"}` 今日だけルートに追加(同上) |
| manual | 手動で並べた順番の id 配列。null なら自動の順番 |
| custom | ユーザーが現地で登録した事業所 `{id:"u…", cat, name, addr, tel, lat, lng, custom:true}` |
| start | 出発地点 `{name, lat, lng}` |
| cats / radius / limit / mode | 業種・距離・今回の件数・移動手段 |

store の形を変えるときは、既存ユーザーのデータが壊れないように読み込み時に補う処理を入れること
(例: 旧形式の日付 "M/D" を "YYYY-MM-DD" に変換している箇所がある)。

## 配布先データについて
- 2026年11月時点のGoogleマップ検索結果をもとに手で入れたもの(約80件)。
- 範囲は岐阜市と岐南町(岐南町の addr は「岐南町〜」で始まる)。
- 全国チェーンの店舗・大手の支店は除外する方針(決裁者に届きにくいため)。
- Googleの英語表記しか分からなかったものは名前に「(要確認)」を付けている。
- 追加するときは id の頭文字を業種ごとにそろえる(t=税理士, s=社労士/行政書士, j=司法書士/測量登記,
  r=不動産, i=保険, b=工務店/建設, c=介護, m=医療, l=法律, d=設計, g=運送, w=卸売, f=人材派遣, a=広告, v=自動車整備)。
  既存の id は配布記録と結びついているので変更・再利用しない。

## Googleマップへの受け渡し
- ナビボタンは `https://www.google.com/maps/dir/?api=1&destination=…&waypoints=…|…&travelmode=…`
- 経由地の上限があるので、未配布の先頭9件だけを渡す。配布済みにすると次の9件に進む。
- 出発地が「現在地」か、今日1件でも配布済みがあれば origin を付けない(=Googleマップが現在地から案内)。

## 動作確認のしかた
- 現在地ボタンと住所検索は https でないと動かない。ローカル確認は `python -m http.server` で開く
  (localhost は位置情報が使える)。
- スマホ幅(390×844前後)で表示崩れがないか必ず確認する。
- 変更後は main に push すれば1〜2分で GitHub Pages に反映される。

## これまでの経緯(ユーザーの要望)
スマホ専用 / トップはルートだけ / 配布済みは別ページ / ナビの「続き」ボタンは不要(1ボタンで次へ進む) /
今回配る件数を指定 / スキップ・並べ替え(ドラッグ)・近くの先の追加・新規登録 / 現在地から組み直し /
現地で手を止めない(1件目は `#nextBox` の「次はここ」に出し、スクロールなしで「配布した」を押せる)。
