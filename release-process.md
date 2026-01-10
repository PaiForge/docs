# リリースプロセス (Release Process)

本プロジェクトでは、npm パッケージのリリースにおいて以下の Git Flow ベースの運用を採用しています。

## ブランチ運用 (Branching Model)

- **main**: リリース用ブランチ。常にリリース可能な状態を保ちます。
- **develop**: 開発用ブランチ。トピックブランチはここから分岐し、ここにマージします。
- **release/x.y.z**: リリース準備用ブランチ。`develop` から分岐し、最終確認を経て `main` へマージします。

### リリースの流れ
1. `develop` ブランチから `release/x.y.z` ブランチを作成します。
2. `release` ブランチで必要な確認・更新を行います。
3. `release` ブランチを `main` ブランチにマージします。
4. `main` ブランチのマージコミットに対してタグ（`vX.Y.Z`）を付与し、npm へ公開します。

## リリース準備 (`release/` ブランチでの作業)

`release/x.y.z` ブランチを作成後、以下の手順を実行します。

1. **テスト最終確認**: 全テストがパスすることを確認します。
2. **バージョン更新**: `package.json` の `version` を更新します。
3. **CHANGELOG 更新**: 手動で `CHANGELOG.md` を更新します。
   - `GitHub Release` ノートには、この CHANGELOG の該当バージョンへのリンクまたは内容を転記します。
4. **ビルド成果物の確認**:
   - `npm run build` が成功すること。
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

`release` ブランチを `main` にマージした後、以下の手順で公開します。

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
   
   # node_modules のクリーンインストール
   npm ci
   
   # ビルド
   npm run build
   
   # 最終確認
   npm pack --dry-run
   
   # 公開
   npm publish
   ```
