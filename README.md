# canvas-page — Advanced Canvas support fork

Quartz Community の [`canvas-page`](https://github.com/quartz-community/canvas-page) をフォークし、Obsidian の **Advanced Canvas** で追加される表示設定の一部を Quartz v5 上でも再現できるようにした個人フォークです。

> This is an unofficial personal fork of `quartz-community/canvas-page`, focused on reproducing selected Advanced Canvas styles in Quartz v5.

このリポジトリは Quartz Community 公式版ではありません。Advanced Canvas の完全互換を目的としたものでもなく、主に自分が使用している機能を中心に実装しています。

## この fork について

元の `canvas-page` は、JSON Canvas (`.canvas`) を Quartz 上で表示する page type plugin です。

この fork ではその機能をベースに、Advanced Canvas が `.canvas` ファイル内の `styleAttributes` に保存する情報の一部を読み取り、Quartz 側の表示へ反映する処理を追加しています。

また、Advanced Canvas 対応とは別に、Obsidian と Quartz の見た目の差を小さくするため、ノード、グループ、エッジ、Canvas の初期表示なども調整しています。

## 主な変更点

### Group

- グループラベルを枠外ではなく枠内に表示
- グループラベルを左上に配置
- グループラベルの文字サイズを `1.5rem`、太字に設定
- グループの色を薄く背景へ反映
- `styleAttributes.border` を読み取るように変更
- `invisible` 指定時は枠線・背景・影を非表示
- デフォルトのグレーをノード側のグレーに近づけるよう調整

### Node

- テキストノードの文字サイズを `1rem` に設定
- テキストノードを上下左右中央揃えに変更
- ノード色を薄く背景へ反映
- Advanced Canvas の `styleAttributes.border` に対応
  - `solid`
  - `dashed`
  - `dotted`
  - `invisible`
- `invisible` 指定時は枠線・背景・影を非表示
- 画像ノードでは画像がノード枠内にずれず収まるよう表示を調整

### Edge

- エッジを三次ベジェ曲線で描画し、Obsidian Canvas と元の `canvas-page` の中間程度になるよう曲率を調整
- Advanced Canvas の `styleAttributes.path` に対応
  - `dotted`
  - `short-dashed`
  - `long-dashed`
- `styleAttributes.path` が未指定または `null` の場合は実線
- エッジラベルの背景サイズを固定値ではなく、実際の SVG テキストサイズから自動計算
- Web フォント読み込み後にもラベル背景サイズを再計算
- エッジラベル背景を不透明にし、背後のエッジが透けないよう変更
- エッジラベル背景に薄い枠線を追加
- エッジラベルの文字サイズを `1rem` に設定

### Canvas view / interaction

- 初期表示は Canvas 全体の高さではなく **横幅を基準**にフィット
- 初期ズームは横幅に対して約 90% とし、最大 `1` に制限
- Canvas の上端を表示開始位置に固定 (`panY = 0`)
- `ResizeObserver` により、画面幅変更時やモバイル表示時に初期ズームを再計算
- マウスホイール、ドラッグ、ピンチ操作によるズーム／パンに対応
- 埋め込み Canvas の右上ボタンは、元の Canvas 個別ページを開くリンクとして動作

## Advanced Canvas 対応状況

この fork で現在解釈している主な追加属性は次のとおりです。

| 対象 | `.canvas` 内の属性 | 対応値 |
| --- | --- | --- |
| Node / Group | `styleAttributes.border` | `solid`, `dashed`, `dotted`, `invisible` |
| Edge | `styleAttributes.path` | `dotted`, `short-dashed`, `long-dashed` |

Advanced Canvas のすべての追加属性を解釈するわけではありません。

## 表示上の独自設定

この fork では、JSON Canvas / Advanced Canvas の仕様そのものではなく、表示上の判断として次の値を採用しています。

| 対象 | 設定 |
| --- | --- |
| テキストノード | `1rem`、上下左右中央揃え |
| グループラベル | `1.5rem`、太字、左上・枠内表示 |
| エッジラベル | `1rem` |
| ノード背景 | ノード色を約 7% 混合 |
| グループ背景 | グループ色を約 7% 混合 |

これらは元の `.canvas` データに文字サイズなどが保存されているわけではなく、この fork 独自の表示方針です。

## Installation

Quartz v5 の plugin は `.quartz/plugins/` に展開され、使用するリポジトリと commit は `quartz.lock.json` に記録されます。

### 手順

#### 1. 新規にこの fork を使う場合

Quartz プロジェクトのルートで実行します。

```bash
npx quartz plugin add github:hyodoarch/canvas-page
```

その後、後述する `quartz.config.yaml` の設定を確認し、必要なら次を実行して config と plugin の状態を同期します。

```bash
npx quartz plugin install --from-config
```

インストール状況は次で確認できます。

```bash
npx quartz plugin list
```

#### 2. Quartz Community 公式版からこの fork へ切り替える場合

すでに `github:quartz-community/canvas-page` を使用している場合は、同名の `canvas-page` plugin を別ソースへ切り替えるため、いったん既存版を削除してから fork 版を追加する方法を推奨します。

```bash
npx quartz plugin remove canvas-page
npx quartz plugin add github:hyodoarch/canvas-page
```

その後、`quartz.config.yaml` の `source` が `github:hyodoarch/canvas-page` になっていることを確認します。

必要に応じて、

```bash
npx quartz plugin install --from-config
```

を実行してください。

> `remove` → `add` が必要なのは、Quartz Community 公式版からこの fork に初めて切り替えるときです。すでに fork 版を使用している場合、更新のたびにアンインストールする必要はありません。

#### 3. fork の最新版へ更新する場合

このリポジトリ側に新しい commit が追加された後、Quartz サイト側では次を実行します。

```bash
npx quartz plugin install --latest canvas-page
```

これにより `canvas-page` の最新版が取得され、`quartz.lock.json` の commit も更新されます。

その後、ローカルで確認します。

```bash
npx quartz build --serve
```

問題がなければ、少なくとも更新された `quartz.lock.json` をサイト側の Git リポジトリへ commit / push してください。

---

## ファイルの設定変更

### 1. `quartz.config.yaml`

Quartz Community 公式版から切り替える場合は、`canvas-page` の `source` を fork 版へ変更します。

```yaml
- source: github:hyodoarch/canvas-page
  enabled: true
  options:
    enableInteraction: true
    initialZoom: 1
    minZoom: 0.02
    maxZoom: 5
  order: 50
```

最低限の設定でよければ、次でも構いません。

```yaml
- source: github:hyodoarch/canvas-page
  enabled: true
```

このリポジトリで使用している `minZoom: 0.02` などの値はサイト側の設定であり、plugin 自体の必須値ではありません。

### 2. `quartz.lock.json`

`quartz.lock.json` は **手作業で編集しません**。

`plugin add` や `plugin install --latest` の実行によって自動的に更新されます。

fork 版を使用している場合、`canvas-page` の項目は概ね次のようになります。

```json
"canvas-page": {
  "source": "github:hyodoarch/canvas-page",
  "resolved": "https://github.com/hyodoarch/canvas-page.git",
  "commit": "<current commit>",
  "installedAt": "<timestamp>"
}
```

確認すべき重要な点は、

```json
"source": "github:hyodoarch/canvas-page"
```

となっていることです。

このファイルは GitHub Actions などの CI 環境でも同じ plugin commit を再現するために必要なので、更新後は site repository に commit してください。

### 3. `.github/workflows/deploy.yml`

通常、Quartz v5 の plugin は `quartz.lock.json` をもとに、

```bash
npx quartz plugin install
```

で CI 上にもインストールできます。

ただし、`.quartz/plugins` を GitHub Actions でキャッシュしている場合、同じ plugin 名 `canvas-page` のままソースを Quartz Community 版から fork 版へ切り替えると、古い plugin directory が残る可能性があります。

そのため、この fork を実際に使用しているサイトでは、`deploy.yml` で `canvas-page` を明示的に削除し、lockfile から clean install するようにしています。

例:

```yaml
- name: Cache Quartz plugins
  uses: actions/cache@v5
  with:
    path: .quartz/plugins
    key: ${{ runner.os }}-plugins-${{ hashFiles('quartz.lock.json') }}
    restore-keys: |
      ${{ runner.os }}-plugins-

- name: Install Quartz plugins
  run: |
    rm -rf .quartz/plugins/canvas-page
    npx quartz plugin install --clean canvas-page
    npx quartz plugin install
```

すでに他の fork plugin も同様に clean install している場合は、その処理の中へ `canvas-page` を追加してください。

このサイトで実際に使用している形は、概ね次のようになっています。

```yaml
- name: Install Quartz plugins
  run: |
    rm -rf .quartz/plugins/graph
    rm -rf .quartz/plugins/article-title
    rm -rf .quartz/plugins/folder-page
    rm -rf .quartz/plugins/tag-page
    rm -rf .quartz/plugins/recent-notes
    rm -rf .quartz/plugins/canvas-page

    npx quartz plugin install --clean graph
    npx quartz plugin install --clean article-title
    npx quartz plugin install --clean folder-page
    npx quartz plugin install --clean tag-page
    npx quartz plugin install --clean recent-notes
    npx quartz plugin install --clean canvas-page

    npx quartz plugin install
```

> `deploy.yml` の変更は、この plugin 自体の必須条件ではありません。plugin cache を使わない構成や、標準の `npx quartz plugin install` だけで正しく再現できる CI では追加不要です。キャッシュされた同名 plugin の取り違えを避けるための防御的な設定です。

## Configuration

現在の plugin options は次のとおりです。

| Option | Type | Default | Description |
| --- | --- | ---: | --- |
| `enableInteraction` | `boolean` | `true` | パン／ズーム操作を有効にする |
| `initialZoom` | `number` | `1` | 初期ズーム値 |
| `minZoom` | `number` | `0.1` | 最小ズーム値 |
| `maxZoom` | `number` | `5` | 最大ズーム値 |

> この fork では初期表示時に横幅基準のフィット処理を行うため、実際の開始倍率は Canvas と表示領域の幅に応じて再計算されます。

## 元の canvas-page から引き継いでいる主な機能

- JSON Canvas (`.canvas`) のページ表示
- Markdown を含むテキストノード
- Vault 内ファイルの埋め込み／リンク
- 画像ノード
- 外部 URL のリンクノード
- グループノード
- SVG によるエッジ、ラベル、矢印、色指定
- Canvas のパン／ズーム操作
- プリセット色 1–6 とカスタム HEX 色

外部 URL の iframe 表示は、リンク先サイトの CSP や `X-Frame-Options` によってブロックされる場合があります。

## Development

Requirements:

- Node.js `>= 22`
- npm `>= 10.9.2`

```bash
npm install
npm run build
```

チェック一式:

```bash
npm run check
```

主なスクリプト:

```bash
npm run build
npm run dev
npm run typecheck
npm run lint
npm run format
npm run test
```

## 解説記事

この fork を作成した経緯と、Obsidian / Advanced Canvas / Quartz での表示差については、無聊写記の記事にまとめています。

- [Advanced Canvas を Quartz で表示させる](https://blog.hyodo-arch.com/%E3%82%B3%E3%83%B3%E3%83%94%E3%83%A5%E3%83%BC%E3%82%BF/Advanced%20Canvas%20%E3%82%92%20Quartz%20%E3%81%A7%E8%A1%A8%E7%A4%BA%E3%81%95%E3%81%9B%E3%82%8B)

## Upstream / Credits

This repository is a fork of:

- [Quartz Community / canvas-page](https://github.com/quartz-community/canvas-page)

Related projects:

- [Quartz](https://quartz.jzhao.xyz/)
- [JSON Canvas](https://jsoncanvas.org/)
- [Obsidian](https://obsidian.md/)
- [Advanced Canvas](https://github.com/Developer-Mike/obsidian-advanced-canvas)

## Limitations

- Advanced Canvas の完全互換を目指したものではありません。
- 現在は、自分が利用・確認している表示機能を中心に対応しています。
- Advanced Canvas 側や Quartz / `canvas-page` upstream 側の仕様変更により、将来調整が必要になる可能性があります。
- テキストノードの左右揃えなど、Advanced Canvas の一部スタイル属性にはまだ対応していません。

Issues / pull requests は歓迎しますが、このリポジトリは個人利用を主目的とした fork です。

## License

MIT License.

This fork is based on Quartz Community's `canvas-page`. See [LICENSE](./LICENSE) for the full license text and copyright notice.
