# TypeScriptプロジェクトにおけるCursor Rulesのベストプラクティス

## 概要

TypeScriptプロジェクトにおけるCursor Rulesは、AIアシスタントがTypeScriptコードを解析・生成する際の動作を定義します。適切に設定されたルールは、一貫性のあるコード生成、プロジェクト規約の遵守、および効率的な開発ワークフローに貢献します。

## ルールファイルの配置

```
PROJECT_ROOT/
  .cursor/
    rules/
      typescript-conventions.mdc
      typescript-project-rules.mdc
      ...
```

- ルールファイルは必ず`.cursor/rules/`ディレクトリに配置する
- ファイル名はケバブケース（kebab-case）で、`.mdc`拡張子を使用
- 名前はルールの目的を説明するものにする

## TypeScript固有のベストプラクティス

以下は、TypeScriptプロジェクトにおけるCursor Rulesの主要なベストプラクティスです：

### 1. 型システムに関するルール

```
- すべてのコードにTypeScriptを使用する
- 型（type）よりインターフェース（interface）を優先する
- enumを避け、代わりにマップを使用する
- 強力な型付けを使用する
- 明示的な戻り値の型を指定する
```

### 2. コンポーネントとモジュールに関するルール

```
- TypeScriptインターフェースを持つ関数コンポーネントを使用する
- コンポーネントには名前付きエクスポートを優先する
- 純粋な関数には「function」キーワードを使用する
```

### 3. コード構造とフォーマットに関するルール

```
- 一貫したインデントとスペースを使用する
- 条件式よりも早期リターンを優先する
- 明示的な型アノテーションを使用する
- any型の使用を避ける
```

## サンプルルールファイル

以下に、TypeScriptプロジェクト用の典型的なルールファイルの例を示します：

```markdown
# TypeScript プロジェクトルール

あなたは、TypeScriptプロジェクトの開発をサポートするAIアシスタントです。以下のルールに従ってください：

## 型システム

- すべてのコードにTypeScriptを使用し、強力な型付けを適用する
- 型（type）よりインターフェース（interface）を優先する
- enumを避け、代わりにマップまたはユニオン型を使用する
- any型の使用を避け、unknownを必要に応じて使用する
- 明示的な戻り値の型を常に指定する

## コンポーネントとモジュール

- TypeScriptインターフェースを持つ関数コンポーネントを使用する
- コンポーネントには名前付きエクスポートを優先する
- 純粋な関数には「function」キーワードを使用する
- モジュールを小さく保ち、単一責任の原則に従う

## コード構造とフォーマット

- 一貫したインデントとスペースを使用する
- 条件式よりも早期リターンを優先する
- 明示的な型アノテーションを使用する
- 複雑な条件式を避け、命名された中間変数を使用する

## プロジェクト固有のルール

- 状態管理にはReact HooksとContextを使用する
- APIリクエストにはAxiosを使用する
- 非同期処理にはasync/awaitを使用する
- テストにはJestとReact Testing Libraryを使用する
```

## プロジェクト規模に応じた推奨設定

### 小規模プロジェクト

小規模プロジェクトでは、基本的なTypeScriptの規約に焦点を当てたシンプルなルールセットが効果的です。

### 中規模プロジェクト

中規模プロジェクトでは、コンポーネント構造、モジュール分割、状態管理に関するより詳細なルールを追加します。

### 大規模プロジェクト

大規模プロジェクトでは、チーム間の一貫性を確保するための包括的なルールセットが必要です：

- 各機能モジュールについての詳細なドキュメント
- コンポーネント階層の明確な定義
- 共有ライブラリとユーティリティの使用に関するガイドライン
- 特定のデザインシステムやUIライブラリに関するルール

## ルールの定期的な見直し

プロジェクトの進化に合わせて、定期的にルールを見直し更新することが重要です。特に：

- 新しいプロジェクトの開始時
- 新しいコーディング規約の採用時
- チームメンバーからのフィードバック後
- 新しいライブラリやフレームワークの導入時

## 具体的な実装例

以下に、実際のTypeScriptプロジェクトで使用できるルールファイルの例を示します：

### typescript-conventions.mdc

```markdown
# TypeScript コーディング規約

あなたはTypeScriptエキスパートアシスタントとして、以下の規約に従ってコードを分析・生成してください：

## 型システム

- インターフェースを使用して型を定義する：
  ```typescript
  // 推奨
  interface User {
    id: string;
    name: string;
    email: string;
  }

  // 非推奨
  type User = {
    id: string;
    name: string;
    email: string;
  };
  ```

- enumではなくユニオン型またはマップを使用する：
  ```typescript
  // 推奨
  type Status = 'pending' | 'active' | 'completed';
  
  // または
  const StatusMap = {
    PENDING: 'pending',
    ACTIVE: 'active',
    COMPLETED: 'completed'
  } as const;
  type Status = typeof StatusMap[keyof typeof StatusMap];

  // 非推奨
  enum Status {
    PENDING = 'pending',
    ACTIVE = 'active',
    COMPLETED = 'completed',
  }
  ```

## 関数とコンポーネント

- 明示的な戻り値の型を指定する：
  ```typescript
  // 推奨
  function getUserName(user: User): string {
    return user.name;
  }

  // 非推奨
  function getUserName(user: User) {
    return user.name;
  }
  ```

- Reactコンポーネントには名前付き関数を使用する：
  ```typescript
  // 推奨
  export function UserProfile({ user }: UserProfileProps): JSX.Element {
    return <div>{user.name}</div>;
  }

  // 非推奨
  export const UserProfile = ({ user }: UserProfileProps): JSX.Element => {
    return <div>{user.name}</div>;
  };
  ```
```

## まとめ

TypeScriptプロジェクトにおけるCursor Rulesのベストプラクティスは、強力な型付け、一貫したコーディング規約、および明確なモジュール化に重点を置いています。これらのプラクティスを適用することで、Cursor AIはより正確かつプロジェクト固有のニーズに合致したコードを生成できるようになります。ルールは定期的に見直し、プロジェクトの進化に合わせて更新することが重要です。
