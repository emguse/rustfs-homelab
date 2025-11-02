# RustFS Homelab Environment

この構成は、Docker Compose によって **RustFS** を運用するための最小構成です。  
HTTPS 逆プロキシとして **Caddy** を利用し、LAN 内での S3 互換ストレージ提供を目的としています。  
本番運用は意図しておらず、利用者の責任において個人または教育用途または実験用途を目的として提供されます。  
RustFS および Caddy は各ライセンス条項に従ってご利用ください。  

---

## 構成概要

| サービス | 役割 |
|-----------|------|
| **rustfs** | S3 互換ストレージサーバ |
| **caddy** | HTTPS 逆プロキシ、TLS 終端 |

---

## 前提条件

- Docker および Docker Compose v2 以降
- 有効なローカル証明書 (`mkcert` などで生成)
- ローカル DNS または `/etc/hosts` による名前解決設定

---

## ディレクトリ構成

```
rustfs-homelab/
├── docker-compose.yml
├── docker-compose.override.yml.sample
├── Caddyfile
├── certs/
│   ├── server.crt
│   └── server.key
├── .env.sample
├── .gitignore
└── README.md
```

---

## セットアップ手順

### 1. `.env` の準備

`.env.sample` をコピーして `.env` を作成します。

```bash
cp .env.sample .env
```

`.env` 内には認証情報やホスト名など、各環境固有の値を設定します。

> `.env` は Git 管理下に含めません。

---

### 2. `docker-compose.override.yml` の作成

環境ごとのオーバーライド設定を行います。

```bash
cp docker-compose.override.yml.sample docker-compose.override.yml
```

Caddy の環境変数やポート設定を調整してください。

---

### 3. 証明書の配置

`certs/` ディレクトリにローカル証明書ファイルを配置します。

例（mkcert 使用時）：

```bash
mkcert -install
mkcert example.local
mv example.local.pem certs/server.crt
mv example.local-key.pem certs/server.key
```

---

### 4. 起動

```bash
docker compose up -d
```

正常に起動すると、RustFS が HTTPS 経由で利用可能になります。

---

## 動作確認

### ブラウザで確認

```
https://<your-domain>/console/
```

### AWS CLI で確認

```bash
AWS_ACCESS_KEY_ID=<access-key> \
AWS_SECRET_ACCESS_KEY=<secret-key> \
AWS_EC2_METADATA_DISABLED=true \
aws --endpoint-url https://<your-domain> --no-verify-ssl s3 ls
```

> 証明書が信頼されていない場合は `--no-verify-ssl` オプションを付与します。

---

## データ永続化

RustFS のデータは Docker volume `rustfs-data` に永続化されます。

バックアップ例：

```bash
docker run --rm -v rustfs-data:/data -v $(pwd):/backup alpine \
  tar czf /backup/rustfs-data.tar.gz /data
```

---

## クリーンアップ

```bash
docker compose down
docker volume rm rustfs-homelab_rustfs-data
```

---

## ファイル管理規約

| ファイル名 | Git 管理 | 内容 |
|-------------|-----------|------|
| `.env.sample` | ✅ | 環境変数の雛形 |
| `.env` | ❌ | 実際の環境変数（機密値を含む） |
| `docker-compose.override.yml.sample` | ✅ | 環境ごとの構成例 |
| `docker-compose.override.yml` | ❌ | 実際のローカル設定 |
| `certs/` | ❌ | 実際の証明書ファイル |

---

## 推奨設定

### `/etc/hosts` の例

```
192.168.x.x example.local
```

### ブラウザ証明書の信頼化（macOS）

```bash
open ~/Library/Application\ Support/mkcert/rootCA.pem
```

システム設定にて「常に信頼」として登録します。

---

## 付記

- RustFS の管理者認証情報は `.env` にて指定します。
- Caddy の設定は環境変数で上書き可能です。
- HTTPS のみで運用することを推奨します。

---

## License

This repository (configuration files and documentation) is licensed under the **MIT License**.  
RustFS and Caddy are licensed separately under the **Apache License 2.0**.

See [LICENSE](./LICENSE) for details.