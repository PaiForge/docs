# コーディング規約 (Code Style & Conventions)

## インポートパス (Import Paths)
**ルール:** `src/` 内では **相対パス** (`../utils` 等) を使用してください。  
**理由:** `tsc` 標準ビルドでの互換性確保（エイリアス解決不要化）と、過度なネスト構造の防止。

## Null vs Undefined
**ルール:** `undefined` を排他的に使用してください（`null` は避ける）。  
**理由:** JSエンジンのデフォルト値との統一。[Reference](https://typescriptbook.jp/reference/values-types-variables/undefined-vs-null)

## Enum 型の実装
**ルール:** `enum` ではなく **const オブジェクト + Union 型** を使用してください。
```typescript
// ✅ 推奨
export const Direction = { Up: 0, Down: 1 } as const;
export type Direction = typeof Direction[keyof typeof Direction];
```
**理由:** バンドルサイズ削減（Tree-shaking有効化）と型安全性の向上。

## 命名規約 (Naming Conventions)
### ドメイン用語のローマ字化
**ルール:** 麻雀用語は **ローマ字** を使用してください（例: `Tehai`, `Shanten`, `Agari`）。  
**理由:** 英語翻訳による意味の曖昧さを排除し、ドメイン概念を正確に表現するため。

### 関数名パターン
**ルール:** `英動詞` + `ローマ字名詞` の形式としてください（例: `calculateAgari`, `decomposeTehai`）。  
**理由:** 操作（英語）と対象（ドメイン語）を明確に区別し、可読性を高めるため。

### ファイル名 (File Naming)
**ルール:** ファイル名は **ケバブケース (kebab-case)** を使用してください（例: `test-helpers.ts`, `user-profile.ts`）。  
**理由:** OSごとのファイルシステム（Case-sensitive vs Case-insensitive）による問題を回避し、npmエコシステムや一般的なTypeScriptプロジェクトとの整合性を保つため。


## ドキュメンテーション (Documentation)
### ユビキタス言語の明記
**ルール:** TSDoc コメント内には、対応する**ユビキタス言語（日本語のドメイン用語）**を必ず明記してください。   
**理由:** `grep "牌種ID"` のように日本語で検索可能にすることで、ドメイン概念の定義場所を特定しやすくし、用語の揺らぎを防ぐため。

### TSDoc の記述ルール
**ルール:**
1. **公開 API (Exported Members):** 原則として TSDoc コメント (`/** ... */`) を記述してください。
2. **型情報の省略:** `@param` や `@return` に型情報 (`{string}` 等) は記述しないでください。TypeScript の型定義と重複するためです。
   - ❌ `* @param {string} name`
   - ✅ `* @param name`

## その他 (Others)
### 型アサーション (Type Assertions)
**ルール:** `as` による型アサーションを原則禁止します (`consistent-type-assertions: never`)。  
**理由:** 型安全性を破壊する可能性があるため。型を特定する必要がある場合は、ユーザー定義型ガード関数 (`isTuple` 等) やバリデーション付きファクトリ関数 (`validateHaiKindId` / `validateHaiId` 等) を使用してください。ただし、テストコード内では例外的に許可しています。

### ラッパーオブジェクト型 (Wrapper Types)
**ルール:** `String`, `Number`, `Boolean`, `Symbol`, `Object` などのラッパー型を使用しないでください。代わりにプリミティブ型 (`string`, `number` 等) を使用してください。  
**理由:** プリミティブ型とラッパーオブジェクトは挙動が異なり、バグの原因となるため。

### 変数のシャドーイング (Shadowing)
**ルール:** 外側のスコープで定義された変数と同名の変数を定義しないでください。  
**理由:** どの変数を参照しているかが曖昧になり、可読性を下げるため。

## 型定義 (Type Definitions)
### Interface vs Type
**ルール:** オブジェクトの型定義には、可能な限り `interface` を使用してください。`type` エイリアスは、Union型、Tuple型、または `interface` よりも著しく簡潔に記述できる場合にのみ使用してください。  
**理由:**
- **パフォーマンス (遅延評価):** TypeScriptコンパイラは `interface` を遅延評価するため、コンパイル速度の向上やエディタの応答性改善に寄与します。対して `type` は宣言時に再帰的に解決・展開されるため、複雑な型でコストが高くなる傾向があります。
- **エラーメッセージ:** 多くの場合、`interface` の方がより簡潔で分かりやすいエラーメッセージを提供します。これは、`type` がコンパイル処理でインライン展開され、エラーメッセージに定義名ではなく構造が表示されてしまうためです。
- **補足 (Declaration Merging):** `interface` は宣言マージが可能（オープン）ですが、`type` は閉じた定義です。本プロジェクトでは主にパフォーマンスとエディタ体験（DX）の観点から `interface` を推奨します。意図しないマージには注意してください。

## 不変性 (Immutability)
### 引数の Readonly
**ルール:** 配列やオブジェクトを引数として受け取る際は、可能な限り `readonly` 修飾子（`readonly T[]` や `Readonly<T>`）を付与してください。  
**理由:** 関数内での意図しないミューテーション（副作用）を防ぐため。Biome などのリンターでは完全な強制が難しいため、規約として定めます。

## アーキテクチャ原則 (Architecture Principles)
### 例外処理の排除 (Railway Oriented Programming)
**ルール:** `throw new Error` などの例外スローを制御フローとして使用せず、代わりに `neverthrow` パッケージの `Result<T, E>` 型を返して型安全にエラーを扱ってください。
**理由:** 例外は関数のシグネチャに現れないため、呼び出し元でのハンドリング漏れを引き起こしやすく、純粋関数の原則（参照透過性）を破壊するためです。

### スマートコンストラクタの徹底 (Parse, don't validate)
**ルール:** `HaiId` や `Tehai13` などのドメイン固有の型（Branded Types）を生成する際は、必ずバリデーションを行い `Result<T, E>` を返すファクトリ関数（スマートコンストラクタ、例: `validateHaiId`, `validateTehai13`）を経由させてください。バリデーションに失敗した場合は `Err` を返します。範囲検証を行わずに `as` で型を付与するだけの関数をファクトリとして扱わないでください。
**理由:** 不正な状態を持つオブジェクトの生成を文法および型のレベルで完全に防ぎ、「一度生成されたオブジェクトは常に正しい状態である」という不変条件を保証するためです。

## エラーハンドリング (Error Handling)
### カスタム例外パターンの廃止と Result 型への移行
**ルール:** 過去のコードで使われていた `ShoushaiError`, `TahaiError` 等のカスタムエラーは、`throw` するのではなく `Result` 型の `E` (エラー型) として用い、`err(new ShoushaiError())` のように返却してください。
**理由:** エラーの発生原因をコード上で明確にするとともに、コンパイラによって全てのパターンマッチング（エラーハンドリング）を強制させるためです。
