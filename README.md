# Vue + Vite 試験対策サイト

## 概要
GitHub Copilot 認定試験対策のためのクイズアプリケーションです。
- 複数のCSVファイルから問題データを読み込み、ランダムに出題します。
- 問題、選択肢、正解、解説の各項目を型安全に扱っています。
- シンプルで見やすいUIを提供し、効率的な学習をサポートします。

## 機能
- 📚 **複数の問題セット**: questions.csv、Book1.csv、BK_questions.csvから問題を選択
- 🎲 **ランダム出題**: 問題順序をシャッフルして出題
- ✅ **即座のフィードバック**: 回答後に正解・不正解を表示
- 📊 **進捗管理**: 現在の問題番号と正答数を表示
- 📖 **詳細解説**: 各問題に対する解説を提供
- 🔄 **復習機能**: 最初からやり直し可能
- 📱 **レスポンシブデザイン**: モバイルデバイスにも対応

## 使い方
1. `npm install` で依存パッケージをインストール
2. `npm run dev` でローカルサーバーを起動
3. ブラウザで `http://localhost:5173` にアクセス
4. オープニングページで問題セットを選択
5. クイズを開始して学習を進める

## プロジェクト構成
```
src/
├── App.vue                 # ルートコンポーネント
├── main.js                 # エントリポイント
├── style.css              # グローバルスタイル
└── components/
    ├── OpeningPage.vue     # 問題セット選択画面
    ├── QuestionQuiz.vue    # クイズメイン画面
    └── HelloWorld.vue      # 未使用コンポーネント

public/
├── questions.csv           # 基本問題セット
├── Book1.csv              # 追加問題セット1
├── BK_questions.csv       # 追加問題セット2
└── staticwebapp.config.json # Azure Static Web Apps設定
```

## CSVフォーマット
問題データは以下の形式で記述してください：

```csv
question,choices,answer,explanation
質問文,A.選択肢1;B.選択肢2;C.選択肢3;D.選択肢4,正解のキー(例:A),解説文
```

### 記述例
```csv
GitHub Copilot はコード リファクタリング タスクをどのように支援しますか?,A.GitHub Copilot は、ユーザー入力なしで構文エラーを修正できます。;B.GitHub Copilot は、ベストプラクティスに従ってコードを自動的に書き換えることができます。;C.GitHub Copilot は、コード品質を向上させるためのリファクタリングの改善を提案できます。;D.GitHub Copilot はプロジェクト ディレクトリから不要なファイルを削除できます。,C,GitHub Copilotは既存のコードを分析し、コードの品質、可読性、保守性を向上させるためのリファクタリングの改善を提案します。
```

## 技術スタック
- **Vue 3** - プログレッシブフレームワーク
- **Vite** - 高速ビルドツール
- **JavaScript** - プログラミング言語
- **PapaParse** - CSVパースライブラリ
- **CSS3** - スタイリング

## デプロイ
このプロジェクトはAzure Static Web Appsでのデプロイに対応しています。
- `.github/workflows/` にGitHub Actionsワークフローを設定済み
- `public/staticwebapp.config.json` でルーティング設定済み

## 開発
```bash
# 依存関係のインストール
npm install

# 開発サーバー起動
npm run dev

# 本番ビルド
npm run build

# ビルド結果のプレビュー
npm run preview
```

## ライセンス
このプロジェクトは学習目的で作成されています。
