# blog-sample-app-repo1

ECS (Fargate) へのデプロイフローを解説するブログ記事のサンプルアプリケーションです。
「dev / stg は main へのマージで自動デプロイ、prd は [tagpr](https://github.com/Songmu/tagpr) が切ったタグでだけデプロイ」というリリースフローを実装しています。

このリポジトリは **アプリのコード、イメージのビルド、バージョニング** だけを担当します。
デプロイ定義 (ecspresso) とデプロイの実行は [blog-sample-release-repo](https://github.com/gainings/blog-sample-release-repo) にあり、このリポジトリから `repository_dispatch` で起動します。

## リリースフロー

```mermaid
flowchart LR
    PR[Pull Request] -->|CI: test / build| Merge[main にマージ]
    Merge --> Build[build & push<br/>sha-&lt;commit&gt;]
    Build --> Tagpr{tagpr}
    Tagpr -->|通常のマージ| RelPR[リリース PR を作成/更新]
    Tagpr -->|リリース PR のマージ| Tag[タグ vYYYY.MMDD.N を作成]
    RelPR --> Dispatch[repository_dispatch<br/>release_tag なし]
    Tag --> Dispatch2[repository_dispatch<br/>release_tag あり]
    Tag --> Retag[イメージにリリースタグを付与]
    Dispatch --> Rel[blog-sample-release-repo<br/>dev → stg]
    Dispatch2 --> Rel2[blog-sample-release-repo<br/>dev → stg → prd]
```

1. **PR**: `ci.yml` がテストとイメージビルド (push なし) を行う。
2. **main にマージ**: `release.yml` が起動する。
   - イメージを 1 回だけビルドし、`sha-<commit sha>` タグで ECR に push する。
   - tagpr が動く。通常のマージならリリース PR (バージョン更新 + CHANGELOG) を作成/更新する。
   - リリースリポジトリへ `repository_dispatch` を送る (payload: `service`, `image_tag`, `release_tag`)。`release_tag` は空なので **dev → stg** までデプロイされる。
3. **リリース PR をマージ**: 同じ `release.yml` が再び動き、tagpr が CalVer タグ (`v2026.0920.0` のような形式) と GitHub Release を作る。
   - `release_tag` 付きで `repository_dispatch` を送り、**dev → stg → prd** までデプロイされる。イメージは再ビルドせず、同じ `sha-<commit>` を使う。
   - 同じイメージへリリースタグも付与する。
4. **ロールバック**: リリースリポジトリの `rollback.yml` を手動実行する。

### 設計上のポイント

- **ビルドとデプロイの分離**: 全環境で同じイメージを使うので、「stg で確認したものが prd に出る」ことが保証される。
- **リポジトリの分離**: アプリ側は ECR への push 権限しか持たず、ECS への権限はリリースリポジトリ側にだけある。デプロイ定義の変更はアプリの変更と独立にレビューできる。
- **CalVer**: リリースは「いつ出したか」で識別する。tagpr の `calendarVersioning = YYYY.0M0D.MICRO` により `v2026.0920.0` のようなタグになり、同日 2 回目は `v2026.0920.1` になる。
- **GitHub App トークン**: tagpr が `GITHUB_TOKEN` で作った PR には CI が走らず、`GITHUB_TOKEN` では他リポジトリへ `repository_dispatch` も送れない。そのため GitHub App のインストールトークンをワークフロー内で発行して使う。
- **バージョンの埋め込み**: tagpr が更新する `VERSION` を `go:embed` でバイナリに含め、`/` で返す。どのバージョンが動いているかを HTTP で確認できる。

## ディレクトリ構成

```
.
├── main.go / main_test.go     # /health と / を持つ最小の HTTP サーバ
├── VERSION                    # tagpr が更新するバージョンファイル
├── Dockerfile                 # マルチステージビルド → distroless
├── .tagpr                     # tagpr 設定 (CalVer)
└── .github/workflows/
    ├── ci.yml                 # PR: test / build
    └── release.yml            # main push: build → tagpr → dispatch (+ イメージへのリリースタグ付与)
```

## セットアップ

### 1. GitHub App

1. Organization または個人アカウントで GitHub App を作成する。権限は **Contents: Read and write**、**Pull requests: Read and write**。
2. このリポジトリと `blog-sample-release-repo` の両方にインストールする。
3. App ID を repository variable `GH_APP_ID` に、秘密鍵を secret `GH_APP_PRIVATE_KEY` に設定する。

### 2. Repository variables / secrets

| 種別 | 名前 | 例 |
|---|---|---|
| variable | `AWS_REGION` | `ap-northeast-1` |
| variable | `AWS_BUILD_ROLE_ARN` | `arn:aws:iam::111111111111:role/gha-blog-sample-app-build` (ECR push 用 OIDC ロール) |
| variable | `ECR_REGISTRY` | `111111111111.dkr.ecr.ap-northeast-1.amazonaws.com` |
| variable | `ECR_REPOSITORY` | `blog-sample-app` |
| variable | `GH_APP_ID` | `123456` |
| secret | `GH_APP_PRIVATE_KEY` | GitHub App の秘密鍵 (PEM) |

デプロイ先 (ECS) に関する設定はすべてリリースリポジトリ側の `services/blog-sample-app/` にあります。

### 3. バージョンの決まり方

`.tagpr` で CalVer (`YYYY.0M0D.MICRO`) を有効にしているため、バージョンはリリース PR を作成/更新した日付から自動で決まります。同じ日に複数回リリースすると末尾の MICRO が `0`, `1`, `2` と増えます。CalVer モードでは `tagpr:major` / `tagpr:minor` ラベルは無視されます。

`VERSION` は初期値 `0.0.0` で、最初のリリース PR で tagpr が日付形式に書き換えます。

## ローカルでの動作確認

```sh
go test ./...
docker build -t blog-sample-app:local .
docker run --rm -p 8080:8080 -e APP_ENV=local blog-sample-app:local
curl localhost:8080/health   # => ok
curl localhost:8080/         # => {"env":"local","message":"...","version":"0.0.0"}
```
