# リリースプロセス (Release Process)

本プロジェクトでは、npm パッケージのリリースにおいて以下の Git Flow ベースの運用を採用しています。
`main` へのマージ（push）をトリガーに GitHub Actions が自動でビルド・公開まで行うため、
開発者がローカルから `npm publish` を実行する必要はありません。

## ブランチ運用 (Branching Model)

- **main**: リリース用ブランチ。常にリリース可能な状態を保ちます。push されると自動リリースが走ります。
- **develop**: 開発用ブランチ。トピックブランチはここから分岐し、ここにマージします。
- **release/x.y.z**: リリース準備用ブランチ。`develop` から分岐し、最終確認を経て `main` へマージします。

### リリースの流れ
1. `develop` ブランチから `release/x.y.z` ブランチを作成します。
2. `release` ブランチで必要な確認・更新を行います。
3. `release` ブランチを `main` ブランチにマージし、`main` を push します。
4. push をトリガーに GitHub Actions（`.github/workflows/release.yml`）が自動的に
   ビルド・公開・タグ付け・GitHub Release 作成まで行います（詳細は後述）。
5. `release` ブランチを `develop` にもマージし戻し、バージョン更新と CHANGELOG を同期します。

## リリース準備 (`release/` ブランチでの作業)

`release/x.y.z` ブランチを作成後、以下の手順を実行します。

1. **テスト最終確認**: 全テストがパスすることを確認します。
2. **バージョン更新**: `package.json` の `version` を更新します。
   - Semantic Versioning に従い、公開 API に変更のない内部リファクタリングは patch を上げます。
3. **CHANGELOG 更新**: 手動で `CHANGELOG.md` を更新します。
   - GitHub Release ノートはワークフローが `--generate-notes` で自動生成するため、
     CHANGELOG への転記や別途の記述は不要です。
4. **ビルド成果物の確認**:
   - `build` コマンドが成功すること（採用しているパッケージマネージャーの詳細は
     [パッケージマネージャー選定基準](package-manager.md) を参照）。
   - `.d.ts` (型定義ファイル) が正しく生成されており、公開 API と一致していること。
   - `npm pack --dry-run` で梱包されるファイル一覧を確認すること。
   - `npm ls --omit=dev` で不要な `devDependency` が混入していないか確認すること。

   > **推奨**: 別の検証用プロジェクトを作成し、実際にインストールして動作確認を行うこと。
   > ```bash
   > npm pack
   > # 別プロジェクトで実行
   > npm install /path/to/pai-forge-riichi-mahjong-0.2.0.tgz
   > ```

## リリース手順 (`main` マージ後の作業)

`release` ブランチを `main` にマージし、`main` を push すると、GitHub Actions
（`.github/workflows/release.yml`）が以下を自動的に実行します。

1. **品質ゲート**: lint・typecheck・テスト・ビルドを実行します。失敗した場合は公開されません。
2. **公開判定**: `package.json` の `version` が npm レジストリに未公開であれば公開対象、
   既に公開済み（バージョンを上げない push）であれば何もせず終了します。
3. **npm publish**: [npm Trusted Publishing](https://docs.npmjs.com/trusted-publishers)
   （OIDC）を使用して公開します。**アクセストークンの発行・管理は不要**です。
   - 事前準備として、npmjs.com のパッケージ設定 → Trusted Publisher に
     `GitHub Actions / <組織>/<リポジトリ> / release.yml` を登録しておく必要があります。
   - Trusted Publishing 経由の公開には provenance（来歴署名）が自動的に付与されます。
4. **タグ・Release 作成**: `main` の `HEAD` に `vX.Y.Z` タグを打ち、push した上で
   GitHub Release を自動作成します（リリースノートは `--generate-notes` で自動生成）。

### ローカルから手動で公開する場合（フォールバック）

`release.yml` が未導入のリポジトリ、または自動リリースが失敗した場合の代替手段として、
ローカルから手動で公開することもできます。

1. **タグ付け**:
   - `main` の `HEAD` (リリースコミット) にタグを打ちます (例: `v0.2.0`)。
   - `package.json` の `version` とタグが一致していることを確認してください。
2. **リモート反映**:
   ```bash
   git push origin main --tags
   ```
3. **npm publish**:
   ローカル環境から実行する場合、必ずクリーンな状態でビルドしてから公開します。

   ```bash
   git checkout main
   git pull

   # node_modules のクリーンインストール（コマンドはパッケージマネージャーに準拠）
   npm ci

   # ビルド
   npm run build

   # 最終確認
   npm pack --dry-run

   # 公開
   npm publish
   ```
