# 日本の国民負担率 見える化ダッシュボード（kklab-tax-burden）

財務省の公表データにもとづき、日本の国民負担率（租税負担率＋社会保障負担率）の推移・内訳・国際比較を可視化するダッシュボードです。ビルド不要の単一 HTML で、そのまま GitHub Pages で公開できます。

> 本サイトは **Gemini 3.8 Flash** でプロトタイプを作成し、**Claude Fable 5.1** で内容の精査・修正・拡充を行いました。

公開URL: https://katzkawai.org/kklab-tax-burden/ （GitHub Pages）／ https://kklab-tax-burden.katzkawai.workers.dev （Cloudflare Workers）

解説論文: [paper/paper.pdf](paper/paper.pdf)（指標の読み方、データの出所、仕組み、公開方法、開発過程、拡張の構想）

## 表示内容

- 最新指標（令和8年度見通し）: 国民負担率 45.7%、租税負担率 28.0%、社会保障負担率 17.6%、潜在的国民負担率 48.4%
- 国民負担率の推移（1970〜2026年度の全年度）: 国税・地方税・社会保障負担の積み上げと、潜在的国民負担率・対GDP比の折れ線。表示期間の切り替えつき
- 国民負担率の内訳（個人所得課税・法人所得課税・消費課税・資産課税等・社会保障負担）
- 国際比較（OECD加盟35カ国、2023年）: 対国民所得比／対GDP比の切り替え、主要国の潜在的国民負担率
- 国民負担率を読むうえでの注意点の解説（分母の違い、潜在的国民負担率の意味、個人の手取り率との違い）
- 給与所得者の負担概算シミュレーター（令和8年の税制・社会保険料率）
- 年度別データ表と CSV ダウンロード

## データの出典

| 内容 | 出典 |
|---|---|
| 国民負担率の推移・国際比較 | [財務省「令和8年度の国民負担率を公表します」（令和8年3月5日）](https://www.mof.go.jp/policy/budget/topics/futanritsu/20260305.html) |
| 租税負担率の内訳 | [財務省「負担率に関する資料」](https://www.mof.go.jp/tax_policy/summary/condition/a04.htm) |
| 所得税・住民税の控除額 | [財務省「令和8年度税制改正の大綱」](https://www.mof.go.jp/tax_policy/tax_reform/outline/fy2026/08taikou_01.htm) |
| 健康保険・介護保険・子ども・子育て支援金の料率 | [全国健康保険協会「令和8年度保険料率」](https://www.kyoukaikenpo.or.jp/about/business/insurance_rate/rate_prefectures/r08/index.html) |
| 雇用保険料率 | [厚生労働省「令和8年度 雇用保険料率のご案内」](https://www.mhlw.go.jp/content/001692566.pdf) |

令和6年度までは実績、令和7年度は実績見込み、令和8年度は見通しです。数値は各機関の公表資料から転記したもので、利用の際は出典元の利用規約に従ってください。

## シミュレーターの前提

個人の負担感をつかむための概算で、実際の税額・保険料とは一致しません。

- 令和8（2026）年の給与収入が対象。所得税は令和8年分（基礎控除62万円＋特例加算、給与所得控除の最低保障74万円、復興特別所得税を含む）、住民税はその所得に翌年度課税される額（所得割10%、均等割・森林環境税5,000円、調整控除）
- 社会保険料は協会けんぽ全国平均（9.90%）、介護保険（1.62%）、子ども・子育て支援金（0.23%）、厚生年金（18.3%）を労使折半、雇用保険は一般の事業（労働者0.5%）。賞与なしで年収を12等分した月給とみなし、標準報酬月額の上限（健康保険139万円、厚生年金65万円）を反映
- 配偶者特別控除、所得金額調整控除、生命保険料控除、iDeCo、住宅ローン控除、ふるさと納税、住民税の非課税限度額は対象外

## ファイル構成

```
index.html          ダッシュボード本体（HTML・データ・スクリプトをすべて含む）
README.md           このファイル
paper/paper.tex     解説論文（LuaLaTeX + jlreq）
paper/paper.pdf     解説論文のPDF
paper/figures/      論文に載せたアプリ画面の図
paper/tables/       論文の表の本体（公表資料とアプリのデータから生成）
wrangler.jsonc      Cloudflare Workers 用の設定（静的アセットのみ）
```

論文は `paper/` で `latexmk -lualatex paper.tex` を実行するとビルドできます。

外部ライブラリは CDN から読み込みます（Tailwind CSS、Chart.js 4.5.1、Google Fonts）。Chart.js を読み込めない環境では、グラフの代わりにデータ表を表示します。

## ローカルでの確認

`index.html` をブラウザで直接開くだけで動作します。

## Cloudflare Workers での公開

GitHub Pages のほか、Cloudflare Workers（静的アセット）でも公開できます。`dist/` に公開するファイルだけをコピーしてデプロイします。

```bash
mkdir -p dist/paper
cp index.html dist/
cp paper/paper.pdf dist/paper/
npx wrangler login      # 初回のみ
npx wrangler deploy     # https://kklab-tax-burden.katzkawai.workers.dev に公開される
```

手元での確認は `npx wrangler dev`。Git 連携（Workers Builds）や独自ドメインの設定は、解説論文の付録Cを参照してください。

## データの更新

財務省は毎年2〜3月ごろに新年度の見通しを公表し、過去の年度の値も改定します。更新時は `index.html` 内の次の箇所を書き換えてください。

1. `MOF_ROWS`: 「国民負担率（対国民所得比）の推移」の表を全年度ぶん差し替え（過年度も改定されるため）
2. `FORECAST_STATUS`: 実績見込み・見通しの年度
3. `BREAKDOWN`: 「国民負担率及び租税負担率の推移」の最新年度の内訳
4. `INTL`・`MAJOR`: 国際比較の2資料
5. `SIM` と控除額の関数: シミュレーターの税制・保険料率
6. 本文中の年度・数値（解説カード、内訳の凡例、見出し、出典リンク、フッターの確認日）
