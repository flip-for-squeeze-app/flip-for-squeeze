# flip-for-squeeze 見直し調査（2026-10）

対象: `flip-for-squeeze-app/flip-for-squeeze`（master `df6a58b`、2026-10-07 時点）
性格: 公開リポジトリ。GitHub Pages（branch 方式、master の `/`）で配信される静的サイト。
注意: このリポジトリは公開で、Pages は master のルートをそのまま配信する。本レポートを master にマージすると `https://flip-for-squeeze-app.github.io/flip-for-squeeze/docs/improvement_research_2026-10.md` として誰でも読める状態になる。マージせずに PR を閉じる運用も検討してほしい。

---

## 0. 調べ方と限界

- 読み取り: 全追跡ファイル（7ファイル、202行）、全コミット履歴（`--all` の15コミット）、未マージブランチ `claude/ios-only-build-release-fcxn8k`、GitHub API（Pages 設定、ブランチ保護、リポジトリ設定）。
- 実行した検証:
  - Python 標準ライブラリだけで書いた簡易チェック（HTML のタグ対応、相対リンクの存在、WCAG のコントラスト比計算）。リポジトリには入れていない。
  - 公開 URL への HTTP GET（`curl`）で、配信中のページと `app-ads.txt` の状態を確認。
  - 履歴全体に対する秘密情報パターンの grep（Google API キー、`sk-`、`ghp_`、秘密鍵、AdMob の App ID・広告ユニット ID）。
- 実行しなかったもの: 正式な HTML バリデータ（`html-validate`、`tidy`、W3C Nu）は未導入のため実行していない。Lighthouse・スクリーンリーダー確認も未実施。
- 見ていないもの:
  - アプリ本体のソース（`flip-for-squeeze-src` ほか、別リポジトリ）。そのため「プライバシーポリシーの記載がアプリの実際の挙動と合っているか」は確認していない。
  - App Store Connect に登録された Privacy Policy URL・Support URL・Marketing URL（開発者サイト）。アクセス手段がないため [要確認]。
  - ユーザーサイト `flip-for-squeeze-app.github.io`（別リポジトリ）。リポジトリは読まず、公開 URL の応答だけを見た。
- このリポジトリにはテスト・lint・ビルド・アプリコードがないため、共通の根拠のうちコード健全性や flaky test に関する研究はほとんど当てはまらない（3章で適用度を明記）。

## 1. 結論（要点5つ）

1. **このリポジトリの本当の役割は、App Store 審査と AdMob が参照する公開 URL の置き場所。** ページのパスは外部との約束事なのに、どの URL をどこに登録したかの記録がない。ここが最大のリスク（4章 A1）。
2. **配信中のページに内容の食い違いがある。** `privacy.html:38-40` は「Web版で遊べます」と案内しているが、Web 版は `7f31373`（2026-07-26）で削除済みで、リンク先は案内ページになっている。Baccarat 側では同じリンクを `a722071` で消したのに、Flip 側には反映されていない（4章 A2）。
3. **`app-ads.txt` が2か所にあり、二重管理になっている。** AdMob が読むのはドメインのルート（`flip-for-squeeze-app.github.io/app-ads.txt`、別リポジトリ由来）で、このリポジトリの `/flip-for-squeeze/app-ads.txt` は AdMob の探索対象外。現在は改行コード以外同じ内容だが、片方だけ更新すると気づけない。ルートのトップページにある `privacy.html` へのリンクは 404 になっている（4章 A3）。
4. **検証の仕組みがない。** CI、リンク切れチェック、HTML 検証、README がいずれもない。7ファイルの静的サイトなので、数十行の GitHub Actions ワークフロー1本で足りる（4章 B1）。
5. **公開リポジトリ特有の衛生問題がある。** 2026-08-09 の未マージブランチが残っていること、大学ドメインの個人メールアドレスが公開履歴に残っていること（`7f31373`）、リポジトリの説明文が "web build" のままであることなど（4章 B2〜B4）。

## 2. 現状（監査結果）

### 2.1 技術スタックと規模

| 項目 | 内容 |
|---|---|
| 種類 | 静的 HTML（JavaScript なし）＋ `app-ads.txt` ＋ `.nojekyll` |
| 配信 | GitHub Pages、`build_type: legacy`（master の `/` から配信）、HTTPS 強制あり、カスタム 404 なし |
| 規模 | HTML 5ファイル 201行、txt 1行、合計 202行・約10.7KB |
| 依存 | なし（外部 JS・CSS・フォントも読み込んでいない） |
| 履歴 | 15コミット（master 14＋未マージ1）、PR は #1 の1件、pack は 10.81MiB（2026-07 の Flutter Web ビルドの名残。wasm 最大 7.2MB） |
| 経緯 | 07-14〜07-18 は Flutter Web ビルド成果物をそのまま master に置いて配信 → 07-26 に削除し、案内ページと `app-ads.txt` に置き換え → 08月以降は各アプリのプライバシーポリシー置き場になった |

### 2.2 ファイルと公開 URL の対応

| ファイル | 行数 | 変更回数 | 公開 URL の状態 | 用途（推定） |
|---|---|---|---|---|
| `index.html` | 18 | 3 | 200 | 案内ページ。CSS なし |
| `privacy.html` | 42 | 1 | 200 | Flip for Squeeze のポリシーとサポート |
| `baccarat/privacy.html` | 51 | 2 | 200 | Baccarat for Squeeze のポリシーとサポート |
| `chinchiro/index.html` | 23 | 1 | 200 | Toss for Chinchiro の紹介 |
| `chinchiro/privacy.html` | 67 | 1 | 200 | Toss for Chinchiro のポリシー |
| `app-ads.txt` | 1 | 1 | 200（ただし AdMob の探索対象外） | AdMob の販売者宣言 |
| （なし）`baccarat/` | - | - | **404** | Baccarat には紹介ページがない |

ホットスポット: 指定のコマンドで上位に出るのは `flutter_bootstrap.js`（8回）や `main.dart.js`（7回）など、すでに削除された Flutter ビルド成果物だけ。現存ファイルは最大でも3回しか変更されておらず、巨大ファイル・巨大関数もない。コードのホットスポット解消という観点では対象がない。

### 2.3 内容の食い違い・不具合

| # | 場所 | 内容 |
|---|---|---|
| 1 | `privacy.html:38-40` | 「Web版で遊べます」と案内しているが、Web 版は削除済み。Baccarat 側（`a722071`）とそろっていない |
| 2 | `privacy.html:20` | 最終更新日が 2026-07-18 のまま。#1 を直すなら日付も更新する |
| 3 | ルートドメイン | `https://flip-for-squeeze-app.github.io/` のトップには、このリポジトリの古い `index.html` とほぼ同じページがあり、その `privacy.html` リンクは **404**（ルートに `privacy.html` がない）。別リポジトリ側の問題 |
| 4 | `app-ads.txt` | このリポジトリとルートの2か所にある。改行コード（このリポジトリはチェックアウト時に CRLF、ルートは LF）以外は同一の内容 |
| 5 | GitHub のリポジトリ説明 | "Flip for Squeeze - web build" のまま。実態と合っていない |
| 6 | `index.html:10` と `privacy.html` | 「App Store で配信中」と書いてある。ストアの URL へのリンクはない |
| 7 | `chinchiro/index.html:18-19` | 「配信予定」「Coming to iOS」と書いてある。リリース時に更新が必要 |

### 2.4 プライバシーポリシーの記載（Apple ガイドライン 5.1.1(i) との照合）

アプリ本体は見ていないため、ここでは文面の網羅性だけを見た。

| 要求（Apple 5.1.1(i)） | Flip | Baccarat | Chinchiro |
|---|---|---|---|
| 収集するデータと用途 | 「収集しない」と明記 | AdMob と IDFA に言及 | AdMob、IP からのおおよその位置、操作情報まで明記 |
| 第三者が同等の保護をすることの確認 | 該当なし | **明記なし** | **明記なし**（Google のポリシーへのリンクのみ） |
| 保持・削除の方針、同意の撤回方法 | 該当なし | ATT の変更方法のみ。UMP（EEA などの同意）への言及なし | アプリ削除で消えること、ホーム画面から同意を変更できることを明記 |

Baccarat と Chinchiro はどちらも AdMob を使うが、書き方の詳しさがかなり違う。Chinchiro の文面が新しく網羅的なので、Baccarat を Chinchiro に合わせるのが自然。ただし、Baccarat が UMP を実装しているかは [要確認]。Chinchiro の「ATT を拒否してもパーソナライズされない広告が出る」という表現が Google の挙動と正確に一致するかも、公式文書では確認できていない [要原典確認]。

### 2.5 テスト・静的解析・CI

| 項目 | 状態 |
|---|---|
| テスト・CI（`.github/workflows`） | なし |
| lint・formatter・pre-commit | なし |
| `.gitattributes` | なし。インデックスは LF、Windows では `core.autocrlf=true` でチェックアウト時に CRLF になる（実害は小さい） |
| ブランチ保護 | なし（`Branch not protected`） |
| Secret scanning・push protection | 無効（公開リポジトリなら無料で有効化できる） |
| `delete_branch_on_merge` | false |

今回の簡易チェック結果:
- タグの対応: 5ファイルとも問題なし。
- 相対リンク: 4件すべて存在する（`index.html` → `privacy.html`、`baccarat/privacy.html`、`chinchiro/`、`chinchiro/index.html` → `privacy.html`）。
- コントラスト比（WCAG 2.2 の 1.4.3 で計算）:

| ページ | 前景 / 背景 | 比 | AA 判定 |
|---|---|---|---|
| Flip | 本文 `#E8ECF4` / `#101A2C` | 14.70 | 合格 |
| Flip | h2 `#E23B4E` / `#101A2C` | 4.12 | **不合格の可能性**。h2 は 1.15em（約18.4px、太字）で、大きい文字の基準（14pt 太字 ≒ 18.5px）にわずかに届かない |
| Flip | h1 `#E23B4E`（1.6em） | 4.12 | 大きい文字として合格（基準3:1） |
| Baccarat | リンク `#5B9BFF` / `#0A3D2E` | 4.41 | **不合格**（基準4.5:1） |
| Baccarat | 補足文 `#9AA6B8` / `#0A3D2E` | 4.95 | 合格 |
| Chinchiro | 全色 | 5.96〜15.26 | 合格 |

- 言語指定: 英日併記のページで `<html lang="en">` だけになっており、日本語の段落に `lang="ja"` がない（WCAG 3.1.2「部分の言語」）。

### 2.6 ドキュメント

- README、CLAUDE.md、AGENTS.md、引き継ぎメモはどれもない。過去にあった `.claude/agents/implementer.md`（34行）は `9933269` で削除済み。
- 未マージブランチ `claude/ios-only-build-release-fcxn8k`（`2827cad`、2026-08-09）に `docs/ios-mobile-release-setup.md` と `docs/templates/ios-testflight.yml` がある。このリポジトリにはアプリのソースがなく、文書は `main` ブランチや Xcode プロジェクト `FlipForSqueeze.xcodeproj` を前提にしている。置き場所も前提もこのリポジトリと合っていない。

### 2.7 セキュリティ・プライバシー

- 秘密情報: 履歴全体を grep した範囲では、API キー・トークン・秘密鍵・AdMob の App ID や広告ユニット ID は見つからなかった。
- `app-ads.txt` の AdMob パブリッシャー ID は公開が前提の値なので問題ない（本レポートには値を書いていない）。
- サポート用 Gmail アドレスが4ページに `mailto:` で載っている。Apple 1.5 で連絡手段が求められているので意図どおり。
- **コミット `7f31373`（2026-07-26）の author / committer に大学ドメインの個人メールアドレスが記録されている。** 公開リポジトリなので誰でも閲覧できる。他のコミットは noreply アドレスを使っている。
- 実行時のリスク: JavaScript もフォームもないため、XSS などの攻撃面はほぼない。GitHub Pages では CSP などのヘッダを設定できないが、このサイトでは実害がない。HSTS は GitHub 側で付与されている。

### 2.8 すでに良い点

- 外部依存がゼロで、JavaScript もない。壊れる場所がほとんどなく、長く放置しても動き続ける。
- 7.2MB の wasm を含む Flutter Web ビルドを、Web 版の終了と同時に削除し、案内ページに切り替えている。
- ポリシーが英日併記で、最終更新日も入っている。Chinchiro の文面は収集データ・第三者・保持・同意変更まで書いてあり、Apple 5.1.1(i) の項目をよくカバーしている。
- 直近の追加（#1）はブランチを切って PR 経由でマージしている。
- 不要になった `.claude/agents` を削除し、文脈を減らしている（持ち主の方針と一致）。

## 3. 根拠

| 根拠 | 内容 | 信頼度 | このリポジトリへの適用度と限界 |
|---|---|---|---|
| Google AdMob ヘルプ「app-ads.txt のトラブルシューティング」(https://support.google.com/admob/answer/9363762) | ファイルはドメインのルートに置く。クローラはストア掲載の「開発者ウェブサイト」を基準に探す。サブドメインは1段まで | 高（公式） | 高。`/flip-for-squeeze/app-ads.txt` は探索対象外と読める。App Store に登録した開発者サイトの URL は未確認 |
| Apple App Review Guidelines 5.1.1(i)・1.5 (https://developer.apple.com/app-store/review/guidelines/) | ポリシーへのリンクを App Store Connect とアプリ内の両方に置く。収集データ・第三者の同等保護・保持と削除・同意撤回の方法を書く。サポート URL から連絡できるようにする | 高（公式） | 高。このリポジトリのページがまさにその URL |
| WCAG 2.2 Understanding 1.4.3 (https://www.w3.org/WAI/WCAG22/Understanding/contrast-minimum.html)、3.1.2 | 通常の文字は 4.5:1、大きい文字（18pt、または 14pt 太字 ≒ 18.5px）は 3:1 | 高（W3C） | 中。法的義務ではなく品質目標。CJK フォントの換算は厳密ではない |
| GitHub Docs「Pages の公開元」(https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site) | Jekyll 以外のビルドや独自の手順が必要なら Actions ワークフローを推奨 | 高（公式） | 中。現状の branch 方式でも、PR 時の検証だけを別の Actions で回せばよい |
| GitHub Docs「カスタム 404」(https://docs.github.com/en/pages/getting-started-with-github-pages/creating-a-custom-404-page-for-your-github-pages-site) | `404.html` を置くだけで有効になる | 高（公式） | 低〜中。ポリシーページへの導線を残す程度の価値 |
| GitHub Docs「コミットメールアドレスの設定」「個人メールを含む push のブロック」 | noreply アドレスの利用と、個人メールを含む push のブロック設定 | 高（公式） | 中。既存履歴の書き換えは別問題（6章） |
| Anthropic "Claude Code best practices"・"memory"（共通の根拠） | 検証手段を与えるのが最重要。CLAUDE.md は推論できないことだけを書く | 中〜高（公式ベンダー文書） | 中。このリポジトリで推論できない情報は「どの URL を外部に登録したか」で、これを README か短い文書に書くのが合う |
| DORA 2025（共通の根拠） | 小さな変更単位・テスト・速いフィードバックが AI 活用の効果を左右する | 中（調査ベース、相関） | 中。直近の変更が AI 生成（`claude/` ブランチ由来）であることと、食い違い #1 が検出されなかったことは整合する（推論） |
| Tornhill & Borg「Code Red」、Khomh・Palomba、Luo ほか（共通の根拠） | コード健全性、スメル、flaky test | 中 | **低**。アプリコードもテストもないため、ほぼ適用されない |
| METR RCT・GitClear（共通の根拠） | AI 使用時の体感と実測のずれ、重複の増加 | 中・低 | 低。3ページの CSS がほぼ重複している程度で、規模が小さく問題にならない |

## 4. 改善候補

### 優先度 高

**A1. 「外部に登録している URL」の一覧を1か所に書く**（工数: 15〜30分）
- README を新設し、各ページがどのアプリのどの欄（App Store Connect の Privacy Policy URL・Support URL・Marketing URL、AdMob、アプリ内リンク）に登録されているかを表にする。あわせて「パスを変えない」「消す前に登録先を直す」と書く。
- 根拠: Apple 5.1.1(i)・1.5、AdMob のクローラ仕様。パスは推論できない外部仕様なので、文書化する価値が高い（Anthropic memory ガイダンス）。
- 登録 URL の実値は持ち主が App Store Connect で確認する必要がある [要確認]。

**A2. `privacy.html` から削除済み Web 版への案内を消し、最終更新日を更新する**（工数: 5分）
- `privacy.html:38-40` の3行を消し、`:20` の日付を更新する。`a722071` で Baccarat に対して行った変更と同じ内容。
- 根拠: Apple 1.5（正確で最新の情報）。リンク先は 200 を返すのでリンク切れチェックでは検出できない。内容の食い違いなので、人が見て直すしかない。

**A3. `app-ads.txt` の正本を1つに決める**（工数: 15分＋AdMob の再クロール最大24時間）
- AdMob が読むのはルート（`flip-for-squeeze-app.github.io` リポジトリ）なので、そちらを正本にする。このリポジトリの `app-ads.txt` は削除するか、README に「正本はルート、ここは参照されない」と書く。推奨は削除。
- 同時に、ルートのトップページにある `privacy.html` への 404 リンクを直す（`/flip-for-squeeze/privacy.html` へ向ける）。これは別リポジトリでの作業になる。
- 前提: App Store に登録した開発者サイトが `https://flip-for-squeeze-app.github.io/` 系であること [要確認]。削除前に AdMob 管理画面で app-ads.txt の検証状態を確認する。
- 根拠: AdMob ヘルプ（ルート配置、開発者サイトを基準に探索）。

### 優先度 中

**B1. PR 時に動く最小の検証ワークフローを置く**（工数: 1〜2時間）
- `.github/workflows/check.yml` で、(1) HTML 検証（例: `html-validate`。`package-lock.json` でバージョンを固定）、(2) 内部リンクの存在確認、(3) A1 の一覧にある公開 URL が 200 を返すかの確認（スケジュール実行、週1程度）を行う。
- 持ち主の「半年ごとにフックなどを外して素の挙動を見る」方針との関係: これは Claude Code のフックではなくリポジトリ側の CI なので、外す対象にならない。エージェントの有無に関係なく同じ網が効く。
- 根拠: Anthropic best practices（検証手段を与える）、DORA 2025（速いフィードバック）、GitHub Pages 公式（Actions の利用）。
- 限界: A2 のような意味の食い違いはこの CI では検出できない。

**B2. 未マージブランチ `claude/ios-only-build-release-fcxn8k` を整理する**（工数: 10分）
- 中身はアプリのソースリポジトリ向けの手順書とテンプレートなので、必要なら `flip-for-squeeze-src` 側へ移し、このブランチは削除する。あわせて `delete_branch_on_merge` を有効にする。
- 根拠: 置き場所と前提（`main` ブランチ、Xcode プロジェクト）がこのリポジトリと合っていない。公開リポジトリに古い計画が残り続ける。

**B3. Baccarat のポリシーを Chinchiro の書き方に合わせる**（工数: 30分＋アプリ側の挙動確認）
- 第三者による同等保護の文言、UMP による同意（実装していれば）、データ保持・削除の説明を追加する。Chinchiro にも第三者の同等保護の一文を足す。
- 根拠: Apple 5.1.1(i)。
- アプリ本体の挙動と照合する必要がある [要確認]。

**B4. 公開履歴の個人メールへの対策**（工数: 5分。履歴の書き換えは非推奨）
- 今後のために、GitHub の「Keep my email addresses private」と「Block command line pushes that expose my email」を有効化し、ローカルの `user.email` を noreply アドレスにする。
- 既存コミット `7f31373` の書き換えは force push が必要で、フォークやキャッシュには残るため効果が限られる。気になる場合だけ検討する。
- 根拠: GitHub Docs（コミットメール設定）。

### 優先度 低

- **C1. アクセシビリティの微修正**（工数: 15分）: Baccarat のリンク色を明るくして（例: `#7FB2FF` 系）4.5:1 以上にする。Flip の h2 は色を少し明るくするか、1.2em 以上にする。日本語の段落・節に `lang="ja"` を付ける。根拠は WCAG 2.2 の 1.4.3 と 3.1.2。
- **C2. リポジトリの説明文と案内ページを更新する**（工数: 10分）: 説明文を "Support & privacy pages for iOS apps" などに変える。`index.html` に App Store へのリンクを置くか検討する。Baccarat の紹介ページか `baccarat/index.html` を用意する（現在 `baccarat/` は 404）。
- **C3. `404.html` を追加する**（工数: 10分）: トップとポリシー一覧へのリンクだけを置く。
- **C4. Secret scanning と push protection を有効化する**（工数: 2分）: 公開リポジトリでは無料。現状は秘密情報がないので予防策。
- **C5. `.gitattributes`（`* text=auto eol=lf`）**（工数: 2分）: 実害は小さいので任意。
- **やらなくてよいもの**:
  - 3ページの CSS の共通化。ファイルが増えて壊れる箇所が増えるだけ。
  - 履歴から Flutter ビルド（pack 10.8MiB）を消すこと。公開リポジトリの履歴を書き換える費用に見合わない。
  - CLAUDE.md の新設。README の URL 一覧で足りる。

## 5. 進め方の提案

- **第1段: 文書と検証の仕組み**（半日以内）
  - A1（README と登録 URL の一覧）を作る。
  - App Store Connect と AdMob の登録値を持ち主が確認し、一覧に書く。
  - 同時に A2（Web 版への案内の削除）を直す。
- **第2段: 安全網**（1〜2時間）
  - B1 の CI を置く。
  - C4（Secret scanning）と B4（メール設定）を有効化する。
  - B2（古いブランチの整理）を行う。
- **第3段: ホットスポット解消に相当する作業**（各30分程度）
  - コードのホットスポットはないので、「二重管理と食い違いの解消」を行う。
  - A3: `app-ads.txt` を一本化し、ルートの 404 リンクを直す。
  - B3: ポリシーの文面をそろえる。
  - C1〜C3: 細かい修正。

## 6. 未確認・要注意事項

- App Store Connect に登録された各アプリの Privacy Policy URL・Support URL・開発者サイトの URL [要確認]。A1・A3 の前提になる。
- AdMob 管理画面での app-ads.txt の検証状態 [要確認]。
- 各アプリの実際の挙動（ATT・UMP の有無、通信先、保存データ）がポリシーの記載と一致しているか [要確認]。アプリのソースは今回の対象外。
- Flip for Squeeze が今も広告なし・通信なしか [要確認]。`app-ads.txt` があるのは Baccarat と Chinchiro のためと推定。
- Chinchiro の「ATT を拒否しても広告は出るが、パーソナライズされない」という表現の正確さ [要原典確認]。
- 共通の根拠のうち Gloaguen ほか（2026-02）は元の指示どおり [要原典確認]。本レポートの結論には使っていない。
- 本レポートは公開リポジトリの PR として公開される。秘密情報・個人メールアドレス・パブリッシャー ID の値は書いていない。
