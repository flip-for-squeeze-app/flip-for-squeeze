# iPhoneだけでビルド〜リリースを回すためのセットアップ手順

目的: Mac を開かずに、出先の iPhone から「修正指示 → 自動ビルド → TestFlight → 審査提出 → リリース」を完結できるようにする。

## 全体像

```
iPhone (claude.ai で Claude に修正指示)
   │  Claude がこのリポジトリに push
   ▼
GitHub (main ブランチ)
   │  push をトリガーに自動ビルド
   ▼
Xcode Cloud (Apple のクラウド Mac / 無料枠 月25時間)
   │  ビルド・署名・アップロードまで自動
   ▼
TestFlight
   │  iPhone の App Store Connect アプリで操作
   ▼
審査提出 → リリース
```

## STEP 1: アプリ本体のソースを GitHub に push する(Mac で1回だけ)

Mac のターミナルで、Xcode プロジェクトのフォルダに移動して実行:

```sh
cd /path/to/FlipForSqueeze   # ← Xcode プロジェクトのフォルダ

# まだ git 管理していない場合
git init
git add .
git commit -m "Initial commit of iOS app source"

# GitHub に新しいリポジトリを作って push
# (リポジトリ名は例: flip-for-squeeze-ios)
git remote add origin https://github.com/flip-for-squeeze-app/flip-for-squeeze-ios.git
git branch -M main
git push -u origin main
```

- GitHub 上のリポジトリ作成は https://github.com/new から(iPhone のブラウザでも可)。
- `.gitignore` が無い場合は Xcode 用のものを入れる(`xcuserdata/`, `DerivedData/`, `*.xcuserstate` など)。

## STEP 2: Xcode Cloud のワークフローを作成(1回だけ)

推奨は Apple 公式の **Xcode Cloud**。署名(証明書・プロビジョニング)を Apple 側が自動管理してくれるので、証明書ファイルの手動エクスポートが不要。

1. Mac の Xcode でプロジェクトを開き、メニュー **Integrate → Create Workflow…**
   (または App Store Connect の Web → 対象アプリ → 「Xcode Cloud」タブ)
2. GitHub アカウントを接続し、STEP 1 のリポジトリを選択
3. ワークフロー設定:
   - **Start Condition**: Branch Changes → `main`
   - **Archive** アクション → **TestFlight (Internal Testing)** に配布
4. 初回ビルドを実行して TestFlight に届くことを確認

これ以降、`main` に push されるたびに自動で TestFlight に新ビルドが上がる。

## STEP 3: iPhone 側の準備

- **App Store Connect** アプリをインストール(TestFlight 配布・審査提出・リリースが全部できる)
- **TestFlight** アプリをインストール(新ビルドの動作確認用)
- **claude.ai** から Claude Code セッションを開けば、コード修正の指示 → push まで完結

## 運用フロー(セットアップ後)

1. iPhone で claude.ai を開き「〇〇を修正して push して」と指示
2. Xcode Cloud が自動ビルド → 15〜30分ほどで TestFlight に着弾
3. TestFlight アプリで動作確認
4. 問題なければ App Store Connect アプリから審査提出 → リリース

## 代替案: GitHub Actions を使う場合

Xcode Cloud の無料枠(月25時間)を超える場合や GitHub に寄せたい場合は、
macOS ランナー + fastlane で同じことができる。テンプレートは
`docs/templates/ios-testflight.yml` に用意してある(アプリのリポジトリの
`.github/workflows/` にコピーして使う)。

必要な GitHub Secrets:

| Secret | 内容 |
|---|---|
| `ASC_KEY_ID` | App Store Connect API キーの Key ID |
| `ASC_ISSUER_ID` | 同 Issuer ID |
| `ASC_KEY_P8` | .p8 キーの中身(base64) |
| `BUILD_CERTIFICATE_P12` | 配布用証明書(base64) |
| `P12_PASSWORD` | 証明書のパスワード |
| `PROVISIONING_PROFILE` | プロビジョニングプロファイル(base64) |

※ 証明書まわりの準備に一度 Mac が必要になるため、まずは Xcode Cloud を推奨。
