# Cursor Rules Research

本調査では、Cursor AIの「cursor rules」に焦点を当て、以下の観点から調査結果をまとめました：

1. TypeScriptとGolangプロジェクトでのベストプラクティス
2. 精度を向上させるための方法
3. Clineのメモリバンク相当の機能実装の可能性

## ディレクトリ構成

- `/typescript_best_practices`: TypeScriptプロジェクトにおけるCursor Rulesのベストプラクティス
- `/golang_best_practices`: Golangプロジェクトにおけるベストプラクティス
- `/improving_accuracy`: Cursor AIの精度を向上させるためのガイドライン
- `/memory_bank_implementation`: Clineメモリバンク相当の機能をCursorで実装する可能性とアプローチ

## Cursor Rulesとは

Cursor Rulesは、Cursor AIの動作をカスタマイズするための設定ファイルまたはガイドラインです。これらは以下の方法で設定できます：

1. **グローバルルール**：Cursor設定の「General > Rules for AI」で設定され、すべてのAI操作に適用される基本原則
2. **プロジェクト固有ルール**：`.cursor/rules`ディレクトリに`.mdc`ファイルとして保存され、特定のコードベースに対するAIの理解とプロジェクト規約の遵守を支援

これらのルールは、コードの分析や生成時におけるAIの動作に対するガイドライン、制約、パターンを提供することで、コンテキストを考慮したアシスタンス機能を実現します。

## 調査概要

この調査では、各トピックに対する詳細な分析と実践的なガイドラインを提供します。各ディレクトリには、そのトピックに関連する詳細情報、例、ベストプラクティスが含まれています。
