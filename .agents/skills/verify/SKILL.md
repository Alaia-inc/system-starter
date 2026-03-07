---
name: verify
description: Use when code changes are made, when the user asks to validate work, or before reporting completion. Runs the repository-required verification commands for the changed package and blocks completion if required checks fail or are skipped.
---

# Verify

この skill は、変更内容に応じて必須の検証コマンドを実行し、完了報告前に結果を確定させるための手順です。

## 使うタイミング

- backend のコードを変更したとき
- frontend のコードを変更したとき
- ユーザーが検証、確認、validate、verify を求めたとき
- 実装完了前に必須チェックを確定させるとき

## 基本方針

1. まず変更対象を backend / frontend / 両方 で判定する。
2. 対象パッケージごとに必須コマンドを順番に実行する。
3. 失敗したら完了扱いにしない。失敗したコマンド名と内容をそのまま共有する。
4. 時間や都合を理由に必須コマンドを省略しない。実行できない場合は、その理由を明示する。
5. package をまたぐ変更では、影響するすべての package を検証する。

## 必須コマンド

### backend を変更した場合

以下をすべて実行する。

```bash
pnpm -C backend run lint
pnpm -C backend run typecheck
pnpm -C backend run test:coverage
```

`backend/vitest.config.mts` では coverage threshold が `branches/functions/lines/statements` すべて 100 に設定されている。`test:coverage` の失敗は完了不可として扱う。

### frontend を変更した場合

以下をすべて実行する。

```bash
pnpm -C frontend run lint
pnpm -C frontend run typecheck
```

## 完了報告

- 実行したコマンドを列挙する
- 成功 / 失敗を package ごとに分けて書く
- 未実行がある場合は、理由を曖昧にせず明示する
- 失敗が残っている場合は「完了」と言わない
