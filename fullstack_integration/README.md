# Next.js、Go、Node.js(Hono)のフルスタック連携におけるCursor Rules

## 概要

このドキュメントでは、Next.js（フロントエンド）とGo/Node.js(Hono)（バックエンド）を組み合わせたフルスタック開発におけるCursor Rulesの設定例と最適なプラクティスを提供します。フルスタック開発特有の課題に焦点を当て、型の共有、API契約、およびプロジェクト構造の一貫性を確保するためのガイドラインを示します。

## フルスタック開発の主要課題

フルスタック開発では、フロントエンドとバックエンドの連携が重要な課題となります。特に：

1. **型の共有と一貫性** - フロントエンドとバックエンド間でのデータ構造と型定義の一貫性確保
2. **API契約の維持** - エンドポイント、リクエスト/レスポンス形式の一貫した実装
3. **開発ワークフローの統合** - 異なる言語環境での効率的な開発プロセス
4. **コード生成の整合性** - AIが生成するコードがフロントエンドとバックエンドで適切に連携すること

## ルールファイルの配置構造

フルスタックプロジェクトでは、複数のルールファイルを組み合わせて使用することをお勧めします：

```
PROJECT_ROOT/
  .cursor/
    rules/
      nextjs-frontend.mdc        # Next.js固有のルール
      golang-backend.mdc         # Go固有のルール
      nodejs-hono-backend.mdc    # Node.js(Hono)固有のルール
      fullstack-integration.mdc  # フロントエンドとバックエンドの連携ルール
      api-contracts.mdc          # API契約に関するルール
```

## 連携開発のためのルール設定例

### 1. Next.jsとGolangの連携

```markdown
# Next.js + Golang フルスタック連携ルール

あなたは、Next.jsとGolangを使用したフルスタックアプリケーション開発をサポートするAIアシスタントです。以下のルールに従って、一貫性のあるコードを生成してください。

## プロジェクト構造

このプロジェクトは以下の構造を持っています：

```
/
├── frontend/                # Next.js フロントエンド
│   ├── app/                 # App Router構造
│   ├── components/          # UIコンポーネント
│   ├── lib/                 # ユーティリティと共有ロジック
│   └── types/               # TypeScript型定義（バックエンドと共有）
│       └── api.ts           # API型定義
│
├── backend/                 # Golang バックエンド
    ├── cmd/                 # メインエントリポイント
    ├── internal/            # 内部パッケージ
    │   ├── api/             # APIハンドラー
    │   ├── models/          # データモデル
    │   └── services/        # ビジネスロジック
    └── pkg/                 # 共有パッケージ
```

## 型の共有と一貫性

- フロントエンドの `types/api.ts` で定義された型とバックエンドの構造体を一致させること
- API境界を通過するすべてのデータ構造は、両方の言語で同じ構造と命名規則に従うこと
- フロントエンドではzodスキーマを使用して型検証を行い、バックエンドでは対応する検証を実装すること

例（フロントエンド）:
```typescript
// frontend/types/api.ts
export interface User {
  id: string;
  name: string;
  email: string;
  createdAt: string; // ISO 8601形式の日付文字列
}

// zodスキーマ
export const userSchema = z.object({
  id: z.string().uuid(),
  name: z.string().min(1),
  email: z.string().email(),
  createdAt: z.string().datetime()
});
```

対応するバックエンド（Golang）:
```go
// backend/internal/models/user.go
type User struct {
    ID        string    `json:"id"`
    Name      string    `json:"name"`
    Email     string    `json:"email"`
    CreatedAt time.Time `json:"createdAt"`
}

// 検証関数
func ValidateUser(u User) error {
    if u.ID == "" {
        return errors.New("id is required")
    }
    if u.Name == "" {
        return errors.New("name is required")
    }
    // メールアドレス検証
    if !regexp.MustCompile(`^[a-z0-9._%+\-]+@[a-z0-9.\-]+\.[a-z]{2,4}$`).MatchString(u.Email) {
        return errors.New("invalid email format")
    }
    return nil
}
```

## API契約

- RESTful APIエンドポイントは一貫した命名規則に従う
  - リソース名には複数形の名詞を使用（例：/api/users）
  - 適切なHTTPメソッドを使用（GET/POST/PUT/DELETE）
- エンドポイントの変更は両方の言語環境で同時に更新する
- エラーレスポンスは一貫したフォーマットに従う

フロントエンドでのAPI呼び出し例:
```typescript
// 一貫したAPIクライアント
export async function getUser(id: string): Promise<User> {
  const response = await fetch(`/api/users/${id}`);
  
  if (!response.ok) {
    const error = await response.json();
    throw new Error(error.message || 'Failed to fetch user');
  }
  
  const data = await response.json();
  return userSchema.parse(data);
}
```

バックエンドでの対応するハンドラー:
```go
// backend/internal/api/user.go
func GetUserHandler(w http.ResponseWriter, r *http.Request) {
    id := chi.URLParam(r, "id")
    
    user, err := services.GetUser(r.Context(), id)
    if err != nil {
        api.SendErrorResponse(w, err, http.StatusNotFound)
        return
    }
    
    api.SendJSONResponse(w, user, http.StatusOK)
}
```

## エラー処理

- フロントエンドとバックエンドでエラーメッセージと構造を一貫させる
- バックエンドではHTTPステータスコードを適切に設定し、構造化されたエラーレスポンスを返す
- フロントエンドではエラーハンドリングを適切に実装し、ユーザーに意味のあるメッセージを表示する

エラーレスポンス構造:
```typescript
// frontend/types/api.ts
export interface ApiError {
  code: string;      // エラーコード
  message: string;   // ユーザー向けメッセージ
  details?: any;     // 追加の詳細情報（オプション）
}
```

## 開発ガイドライン

- バックエンドAPIの変更は、常に型定義の更新と合わせて行う
- フロントエンドでNext.jsのAPI Routesを使用する場合、それはBFFとして機能し、実際のバックエンドAPIを呼び出す
- CSRFトークンやJWTなどの認証メカニズムは両方の環境で一貫して実装する
- 環境変数名は両方の環境で同じ命名規則を使用する

## 技術スタック統合

- Next.jsはReact Server ComponentsとClient Componentsを適切に使い分ける
- Golangバックエンドはパフォーマンスが重要なAPIに使用する
- フロントエンドはTypeScriptの厳格モードを有効にする
- バックエンドでのモデル変更は、フロントエンドの型定義を必ず更新する
```

### 2. Next.jsとNode.js(Hono)の連携

```markdown
# Next.js + Node.js(Hono) フルスタック連携ルール

あなたは、Next.jsとNode.js(Hono)を使用したフルスタックアプリケーション開発をサポートするAIアシスタントです。以下のルールに従って、一貫性のあるコードを生成してください。

## プロジェクト構造

このプロジェクトは以下の構造を持っています：

```
/
├── frontend/                # Next.js フロントエンド
│   ├── app/                 # App Router構造
│   ├── components/          # UIコンポーネント
│   ├── lib/                 # ユーティリティと共有ロジック
│   └── types/               # TypeScript型定義（バックエンドと共有）
│       └── api.ts           # API型定義
│
├── backend/                 # Node.js(Hono) バックエンド
    ├── src/                 # ソースコード
    │   ├── index.ts         # エントリポイント
    │   ├── routes/          # APIルート定義
    │   ├── handlers/        # リクエストハンドラー
    │   ├── services/        # ビジネスロジック
    │   ├── models/          # データモデル
    │   └── types/           # TypeScript型定義（フロントエンドと共有）
    └── tsconfig.json        # TypeScript設定
```

## 型の共有

Node.jsとNext.jsはどちらもTypeScriptを使用するため、型定義を直接共有できる大きな利点があります。

### アプローチ1: パッケージとしての共有型

共有型を別のパッケージとして分離する：

```
/shared-types/              # 共有型定義パッケージ
  ├── index.ts              # メインエクスポート
  ├── models.ts             # データモデル
  ├── requests.ts           # リクエスト型
  └── responses.ts          # レスポンス型
```

例（共有型）:
```typescript
// shared-types/models.ts
export interface User {
  id: string;
  name: string;
  email: string;
  createdAt: string;
}

// shared-types/requests.ts
export interface CreateUserRequest {
  name: string;
  email: string;
  password: string;
}

// shared-types/responses.ts
export interface ApiResponse<T> {
  data: T;
  meta?: {
    total?: number;
    page?: number;
  };
}

export interface ApiError {
  code: string;
  message: string;
  details?: any;
}
```

### アプローチ2: プロジェクト間でファイルを同期

型定義ファイルを同期するスクリプトやツールを使用する方法：

```typescript
// frontend/scripts/sync-types.js
// backend/src/types/からfrontend/types/へ型定義をコピーするスクリプト
```

## API契約

Honoは型安全なAPIルーティングをサポートしています。これを活用して、フロントエンドとバックエンド間の一貫性を確保します：

バックエンド（Hono）での定義:
```typescript
// backend/src/routes/users.ts
import { Hono } from 'hono';
import { z } from 'zod';
import { zValidator } from '@hono/zod-validator';
import type { User, CreateUserRequest } from '../types';

const userRouter = new Hono();

const createUserSchema = z.object({
  name: z.string().min(1),
  email: z.string().email(),
  password: z.string().min(8)
});

userRouter.post(
  '/',
  zValidator('json', createUserSchema),
  async (c) => {
    const data = c.req.valid('json') as CreateUserRequest;
    // 処理ロジック...
    return c.json({ id: 'new-id', ...data, createdAt: new Date().toISOString() });
  }
);

userRouter.get('/:id', async (c) => {
  const id = c.req.param('id');
  // ユーザー取得ロジック...
  const user: User = {
    id,
    name: 'John Doe',
    email: 'john@example.com',
    createdAt: new Date().toISOString()
  };
  return c.json(user);
});

export default userRouter;
```

フロントエンドでの使用:
```typescript
// frontend/lib/api.ts
import type { User, CreateUserRequest, ApiResponse } from '../types/api';

export async function createUser(data: CreateUserRequest): Promise<User> {
  const response = await fetch('/api/users', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json'
    },
    body: JSON.stringify(data)
  });
  
  if (!response.ok) {
    const error = await response.json();
    throw new Error(error.message || 'Failed to create user');
  }
  
  return await response.json();
}

export async function getUser(id: string): Promise<User> {
  const response = await fetch(`/api/users/${id}`);
  
  if (!response.ok) {
    const error = await response.json();
    throw new Error(error.message || 'Failed to fetch user');
  }
  
  return await response.json();
}
```

## TypeScriptの設定統一

フロントエンドとバックエンドで一貫したTypeScript設定を使用することで、型の互換性を確保します：

```json
// 共通のtsconfig設定（ベース）
{
  "compilerOptions": {
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "noUncheckedIndexedAccess": true
  }
}
```

## エラー処理

統一されたエラー処理アプローチを使用します：

```typescript
// backend/src/utils/errors.ts
export class ApiError extends Error {
  constructor(
    public readonly code: string,
    public readonly message: string,
    public readonly statusCode: number = 400,
    public readonly details?: any
  ) {
    super(message);
    this.name = 'ApiError';
  }
  
  toResponse() {
    return {
      code: this.code,
      message: this.message,
      details: this.details
    };
  }
}

// Honoミドルウェアでエラーハンドリング
export const errorHandler = async (c, next) => {
  try {
    await next();
  } catch (err) {
    if (err instanceof ApiError) {
      return c.json(err.toResponse(), err.statusCode);
    }
    console.error(err);
    return c.json(
      { code: 'INTERNAL_ERROR', message: 'Internal server error' },
      500
    );
  }
};
```

## 開発ワークフロー統合

- TypeScriptの型チェックをビルドプロセスと統合
- ESLintとPrettierの設定を統一
- コミット前のバリデーションフックを設定
- 共有型の変更を検出して同期するスクリプトを実行

## APIテストと検証

型安全性を確保するためのテスト戦略：

```typescript
// backend/tests/api/users.test.ts
import { describe, it, expect } from 'vitest';
import app from '../../src/index';
import type { User } from '../../src/types';

describe('Users API', () => {
  it('should create a user', async () => {
    const req = new Request('http://localhost/users', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        name: 'Test User',
        email: 'test@example.com',
        password: 'password123'
      })
    });
    
    const res = await app.fetch(req);
    expect(res.status).toBe(200);
    
    const data = await res.json();
    expect(data).toHaveProperty('id');
    expect(data.name).toBe('Test User');
    
    // 型チェック - データがUser型と互換性があることを確認
    const user = data as User;
    expect(typeof user.id).toBe('string');
  });
});
```

## 認証とセッション管理

認証ロジックの一貫性を確保:

```typescript
// backend/src/middlewares/auth.ts
import { jwt } from 'hono/jwt';
import type { JwtPayload } from '../types';

export const authMiddleware = jwt({
  secret: process.env.JWT_SECRET || 'default-secret'
});

// フロントエンドでのJWT処理
// frontend/lib/auth.ts
import { jwtDecode } from 'jwt-decode';
import type { JwtPayload } from '../types/api';

export function getUser() {
  const token = localStorage.getItem('token');
  if (!token) return null;
  
  try {
    const payload = jwtDecode<JwtPayload>(token);
    return payload;
  } catch (e) {
    console.error('Invalid token', e);
    return null;
  }
}
```
```

## フルスタック連携のベストプラクティス

### 1. API契約の厳格な管理

- OpenAPIまたはGraphQLスキーマを使用して、APIを厳格に定義する
- 自動的に型定義を生成するツールを統合する（openapi-typescript、graphql-codegenなど）
- API変更時は、フロントエンドとバックエンドの両方のコードを同時に更新する

### 2. 型の共有と同期

Next.jsとNode.js(Hono)の場合：
```typescript
// 共有型を別パッケージとして公開
// または、共有型ディレクトリを作成してシンボリックリンクで連携
```

Next.jsとGolangの場合：
```typescript
// フロントエンド
export interface User {
  id: string;
  name: string;
  email: string;
}

// Golang（コメントでTypeScript型を参照）
// 対応するTypeScript型: frontend/types/api.ts#User
type User struct {
    ID    string `json:"id"`
    Name  string `json:"name"`
    Email string `json:"email"`
}
```

### 3. 一貫したエラー処理

```typescript
// 共有エラー型
export interface ApiError {
  code: string;     // 機械可読エラーコード
  message: string;  // ユーザー向けメッセージ
  details?: any;    // 追加詳細情報
}

// フロントエンドでのエラーハンドリング
async function fetchWithErrorHandling<T>(url: string, options?: RequestInit): Promise<T> {
  const response = await fetch(url, options);
  
  if (!response.ok) {
    const error = await response.json() as ApiError;
    throw new Error(error.message || `API error: ${response.status}`);
  }
  
  return await response.json() as T;
}
```

### 4. モノレポアプローチ

```
/
├── apps/
│   ├── frontend/         # Next.jsアプリケーション
│   └── backend/          # Go/Node.jsバックエンド
├── packages/
│   ├── api-client/       # 型付きAPIクライアント
│   ├── shared-types/     # 共有型定義
│   └── validation/       # 共有バリデーションロジック
├── scripts/
│   └── type-sync.js      # 型定義の同期スクリプト
└── package.json          # ワークスペース設定
```

### 5. BFFパターンの活用

Next.jsのAPIルートをBackend for Frontend (BFF)として使用し、実際のバックエンドAPIを呼び出す方法：

```typescript
// app/api/users/route.ts
import { NextResponse } from 'next/server';
import { fetchBackendApi } from '@/lib/api';
import type { User } from '@/types/api';

export async function GET() {
  try {
    // 実際のバックエンドAPIを呼び出し
    const users = await fetchBackendApi<User[]>('/api/users');
    
    // 必要に応じてデータ変換や加工
    const transformedUsers = users.map(user => ({
      ...user,
      displayName: `${user.name} <${user.email}>`
    }));
    
    return NextResponse.json(transformedUsers);
  } catch (error) {
    console.error('Error fetching users:', error);
    return NextResponse.json(
      { error: 'Failed to fetch users' },
      { status: 500 }
    );
  }
}
```

## まとめ

フルスタック開発環境でのCursor Rulesの効果的な活用は、以下のポイントに焦点を当てることで実現できます：

1. **型定義の共有と同期** - フロントエンドとバックエンド間の一貫した型システム
2. **API契約の厳格な管理** - エンドポイント、リクエスト/レスポンス形式の一貫性
3. **統一されたエラー処理** - エラーの識別と表示の標準化
4. **コード生成の整合性** - AIが生成するコードがエンド・ツー・エンドで動作すること
5. **開発ワークフローの最適化** - 型チェック、検証、テストの統合

これらの原則に従い、適切にCursor Rulesを設定することで、Next.jsとGo/Node.js(Hono)を組み合わせたフルスタック開発の効率と品質を大幅に向上させることができます。
