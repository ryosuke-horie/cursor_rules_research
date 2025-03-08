# Cursor Rules Research

本調査では、Cursor AIの「cursor rules」に焦点を当て、以下の観点から調査結果をまとめました：

1. TypeScriptとGolangプロジェクトでのベストプラクティス
2. 精度を向上させるための方法
3. Clineのメモリバンク相当の機能実装の可能性
4. MDCファイルの規約と構造

## ディレクトリ構成

- `/typescript_best_practices`: TypeScriptプロジェクトにおけるCursor Rulesのベストプラクティス
- `/golang_best_practices`: Golangプロジェクトにおけるベストプラクティス
- `/improving_accuracy`: Cursor AIの精度を向上させるためのガイドライン
- `/memory_bank_implementation`: Clineメモリバンク相当の機能をCursorで実装する可能性とアプローチ
- `mdc_file_conventions.md`: MDCファイルの規約と構造に関する詳細ガイド

## Cursor Rulesとは

Cursor Rulesは、Cursor AIの動作をカスタマイズするための設定ファイルまたはガイドラインです。これらは以下の方法で設定できます：

1. **グローバルルール**：Cursor設定の「General > Rules for AI」で設定され、すべてのAI操作に適用される基本原則
2. **プロジェクト固有ルール**：`.cursor/rules`ディレクトリに`.mdc`ファイルとして保存され、特定のコードベースに対するAIの理解とプロジェクト規約の遵守を支援

これらのルールは、コードの分析や生成時におけるAIの動作に対するガイドライン、制約、パターンを提供することで、コンテキストを考慮したアシスタンス機能を実現します。

## MDCファイルについて

MDC（Multi-Document Context）ファイルは、Cursor AIにプロジェクト固有の情報を構造化して提供するために使用されます。これらのファイルはマークダウン形式をベースにしており、AIに対する詳細な指示や制約を含みます。MDCファイルの命名規則、構造、ベストプラクティスについては`mdc_file_conventions.md`を参照してください。

## 調査概要

この調査では、各トピックに対する詳細な分析と実践的なガイドラインを提供します。各ディレクトリには、そのトピックに関連する詳細情報、例、ベストプラクティスが含まれています。また、実際のプロジェクトですぐに使用できるサンプルのMDCファイルも提供しています。
