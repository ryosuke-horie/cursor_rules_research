# Golangプロジェクトにおける Cursor Rules のベストプラクティス

## 概要

Golangプロジェクトに対するCursor Rulesの設定は、GoのイディオムやベストプラクティスをAIアシスタントに理解させ、プロジェクト固有の要件に合わせたコード生成を実現するために重要です。この文書では、Golangプロジェクトで効果的なCursor Rulesを作成・使用するためのベストプラクティスを提供します。

## ルールファイルの基本構造

```
PROJECT_ROOT/
  .cursor/
    rules/
      golang-conventions.mdc
      golang-project-rules.mdc
      ...
```

- ルールファイルは`.cursor/rules/`ディレクトリに配置
- ファイル名はケバブケース（kebab-case）で`.mdc`拡張子を使用
- 名前はルールの目的を反映したものにする

## Golang固有のベストプラクティス

### 1. コーディング規約とフォーマット

```
- 常に最新の安定版Goを使用する
- 標準のGoフォーマット規約に従う（gofmt/goimports）
- エラー処理にはイディオマティックなアプローチを使用する
- 意味のあるエラーメッセージを返す
- メソッドレシーバーには一貫した命名規則を使用する
```

### 2. パッケージと依存関係の管理

```
- 意味のあるパッケージ名を使用する
- 循環依存を避ける
- 標準のGoプロジェクトレイアウトに従う
- 依存関係の管理にはGo Modulesを使用する
```

### 3. インターフェースと抽象化

```
- 継承よりコンポジションを優先する
- 抽象化にはインターフェースを使用する
- インターフェースは使用する側で定義する
- 小さくフォーカスしたインターフェースを作成する
```

### 4. エラー処理と例外管理

```
- panicよりも明示的なエラーリターンを優先する
- 意味のあるカスタムエラー型を作成する
- エラーラッピングには標準のfmt.Errorfとerrors.Unwrapを使用する
- エラー処理をスキップしない
```

## サンプルルールファイル

以下に、Golangプロジェクト用の典型的なルールファイルの例を示します：

```markdown
# Golang プロジェクトルール

あなたはGolangプロジェクトの開発をサポートするAIアシスタントです。以下のルールに従ってください：

## コーディング規約とフォーマット

- 常に最新の安定版Goを使用する
- すべてのコードはgofmtでフォーマットしたとみなす
- 変数名には短くても説明的な名前を使用する（例：i -> idx, s -> str）
- パッケージ名はシンプルで説明的なものにする
- メソッドレシーバーには一貫した命名規則を使用する（1-2文字の略語）

## エラー処理

- エラーは常に明示的に処理し、nilチェックを行う
- エラーは最後のリターン値として返す
- panicよりもエラーリターンを優先する
- エラーラッピングにはfmt.Errorfを使用する: fmt.Errorf("failed to process: %w", err)
- 意味のあるエラーメッセージを返す

## パッケージとプロジェクト構造

- 標準のGoプロジェクトレイアウトに従う
- パッケージは単一の責任を持つようにする
- 循環依存を避ける
- テストファイルは実装と同じディレクトリに配置する

## インターフェースと抽象化

- 継承よりコンポジションを優先する
- インターフェースは使用する側で定義する
- 小さく、フォーカスしたインターフェースを作成する
- モックテストのためにインターフェースを使用する

## その他のベストプラクティス

- context.Contextを適切に使用する
- ゴルーチンではチャネルを適切に管理し、リークを防ぐ
- reflectパッケージの使用を最小限に抑える
- 効率的なメモリ使用のためにポインタとバリューを適切に選択する
```

## プロジェクト規模に応じた推奨設定

### 小規模プロジェクト

小規模プロジェクトでは、基本的なGoの規約とエラー処理に焦点を当てた簡潔なルールセットが適しています。

### 中規模プロジェクト

中規模プロジェクトでは、パッケージ構造、テスト戦略、およびドキュメンテーションに関するより詳細なルールを追加します。

### 大規模プロジェクト

大規模プロジェクトでは、マイクロサービスアーキテクチャ、API設計、パフォーマンス最適化などに関する包括的なルールが必要です：

- 各サービスの責任範囲の明確な定義
- API契約とバージョニング戦略
- パフォーマンスベンチマークと最適化ガイドライン
- 分散システム設計パターン

## 具体的な実装例

以下に、実際のGolangプロジェクトで使用できるより詳細なルールファイルの例を示します：

### golang-conventions.mdc

```markdown
# Golang コーディング規約

あなたはGolangエキスパートアシスタントとして、以下の規約に従ってコードを分析・生成してください：

## エラー処理パターン

- エラーは常に明示的に処理し、適切なリターン値として処理する：
  ```go
  // 推奨
  func processFile(fileName string) ([]byte, error) {
      file, err := os.Open(fileName)
      if err != nil {
          return nil, fmt.Errorf("failed to open file %s: %w", fileName, err)
      }
      defer file.Close()

      data, err := ioutil.ReadAll(file)
      if err != nil {
          return nil, fmt.Errorf("failed to read file %s: %w", fileName, err)
      }

      return data, nil
  }

  // 非推奨
  func processFile(fileName string) []byte {
      file, _ := os.Open(fileName)
      defer file.Close()
      data, _ := ioutil.ReadAll(file)
      return data
  }
  ```

## インターフェース定義

- インターフェースは使用する側で定義し、小さく保つ：
  ```go
  // 推奨
  type DataStore interface {
      Get(id string) ([]byte, error)
      Set(id string, data []byte) error
  }

  // 非推奨 (大きすぎるインターフェース)
  type DataStore interface {
      Get(id string) ([]byte, error)
      Set(id string, data []byte) error
      Delete(id string) error
      List() ([]string, error)
      Count() (int, error)
      Clear() error
      // ...その他多数のメソッド
  }
  ```

## 構造体と関数設計

- メソッドレシーバーには一貫した命名を使用：
  ```go
  // 推奨
  type User struct {
      ID   string
      Name string
  }

  func (u *User) UpdateName(name string) {
      u.Name = name
  }

  func (u *User) GetID() string {
      return u.ID
  }

  // 非推奨 (一貫性のない命名)
  func (user *User) UpdateName(name string) {
      user.Name = name
  }

  func (self *User) GetID() string {
      return self.ID
  }
  ```
```

## ルールの定期的な見直し

Golangプロジェクトの進化に合わせて、定期的にルールを見直し、更新することが重要です。特に：

- 新しいGoのバージョンがリリースされたとき
- 新しいツールやライブラリを導入するとき
- プロジェクト構造が大きく変わるとき
- パフォーマンスや保守性の問題が発生したとき

## まとめ

GolangプロジェクトにおけるCursor Rulesのベストプラクティスは、Goの標準的な規約、イディオム、およびエラー処理パターンを重視しています。適切に設定されたルールを使用することで、Cursor AIはGolangの特性を活かした、クリーンで効率的なコードを生成できるようになります。
