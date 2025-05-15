# Vue + Vite 試験対策サイト

## 概要
- CSVファイル（questions.csv）から問題データを読み込み、ランダムに出題します。
- 問題、選択肢、正解、解説は型安全に扱っています。
- UIはシンプルで見やすい設計です。

## 使い方
1. `npm install` で依存パッケージをインストール
2. `npm run dev` でローカルサーバーを起動
3. ブラウザで `http://localhost:5173` にアクセス

## CSVフォーマット例
```
question,choiceA,choiceB,choiceC,choiceD,answer,explanation
GitHub Copilot はコード リファクタリング タスクをどのように支援しますか?,GitHub Copilot は、ユーザー入力なしで構文エラーを修正できます。,GitHub Copilot は、ベストプラクティスに従ってコードを自動的に書き換えることができます。,GitHub Copilot は、コード品質を向上させるためのリファクタリングの改善を提案できます。,GitHub Copilot はプロジェクト ディレクトリから不要なファイルを削除できます。,C,GitHub Copilotは既存のコードを分析し、コードの品質、可読性、保守性を向上させるためのリファクタリングの改善を提案します。
```

## 技術スタック
- Vue 3
- Vite
- TypeScript
- papaparse（CSVパース用）
