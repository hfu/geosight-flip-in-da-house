# geosight-flip-in-da-house 🏠🌍
ベアメタル（baremetal）志向で、Docker の中身を OS サービスへ“Flip”するテスト・プロジェクト。まずはフル構成の互換性を維持しつつ、Raspberry Pi 上で軽くて熱い運用を目指します。

- ベース: 本家 [unicef-drp/GeoSight-OS](https://github.com/unicef-drp/GeoSight-OS) を直接クローン
- 構成: PostgreSQL + PostGIS / Redis / Nginx / Django (gunicorn) / Celery / Celery Beat / フロントエンド本番ビルド
- 管理: .env を `/etc/geosight/.env` に配置して一元管理
- ログ: journald に集約、`nginx access_log off`
- 目的: テスト用途（バージョンピンはしない）
- キーワード: baremetal, flip

## 決定事項（方針）
- リポジトリ: geosight-flip-in-da-house
- 取得元: 本家 `unicef-drp/GeoSight-OS` を直接クローン
- 環境変数: `/etc/geosight/.env` 一元管理
- DB設計: 既定のまま（与えられた設計を尊重、細かい調整はしない）
- HTTPS: 後回し（まずは HTTP 2000 番で運用）
- フロントエンド: 初回に `npm run build` を必須化、キャッシュ永続化（2回目以降高速化）
- サービス: `geosight-web` / `geosight-celery` / `geosight-celery-beat` の3本で運用
- ログ: journald に集約、Nginx のアクセスログはオフ
- Django LOGGING: 標準レベルを WARNING 以上に
- アップデート標準手順: `git pull → migrate → collectstatic → 再起動`
- バージョンピン: なし（テストで最新を浴びる）

## 対応環境
- OS: Raspberry Pi OS trixie (Debian 13) 64-bit
- ハード: Raspberry Pi 4B（RAM 4GB 推奨、最低 2GB）
- ストレージ: microSD または USB3 SSD（SSD 推奨）
- PostGIS: apt の ARM64 ネイティブ（`apt install postgresql postgis`）

## クイックスタート
```bash
git clone https://github.com/hfu/geosight-flip-in-da-house.git
cd geosight-flip-in-da-house
sudo ./install.sh
```

セットアップ後、ブラウザで `http://localhost:2000/` を開きます。
- Username: `admin`
- Password: `admin`（すぐに変更推奨）

## インストールで行うこと（概要）
1. apt で必要パッケージ導入
   - `postgresql`, `postgis`, `redis-server`, `nginx`, `python3-venv`, `libpq-dev`, `gdal-bin`, `nodejs`, `npm`
2. GeoSight-OS をクローン（本家）
3. Python venv 作成、依存インストール、Django 用 `.env` 生成（`/etc/geosight/.env`）
4. PostgreSQL 初期化（DB/ユーザ作成、`CREATE EXTENSION postgis;`）
5. `npm run build` によるフロントエンド本番ビルド（キャッシュ永続化）
6. systemd ユニット配置（web / celery / celery-beat）
7. Nginx のリバースプロキシ設定（2000 番）

## gunicorn か uWSGI（短評）
- gunicorn
  - 長所: 軽量、設定が簡単、Pi でも扱いやすい
  - 方針: 初手は gunicorn を採用（まずはスムーズに起動）
- uWSGI
  - 長所: 高機能・微細なチューニング
  - 方針: 必要性が明確になった段階で移行検討（当面は採用しない）

## 運用フロー（標準化）
- 起動/再起動
  - `sudo systemctl restart geosight-web geosight-celery geosight-celery-beat`
- 更新（テスト前提で最新追従）
  - `cd /opt/geosight/GeoSight-OS && git pull`
  - `sudo -u www-data bash -lc ". .venv/bin/activate && python manage.py migrate --noinput"`
  - `sudo -u www-data bash -lc ". .venv/bin/activate && python manage.py collectstatic --noinput"`
  - `sudo systemctl restart geosight-web geosight-celery geosight-celery-beat`
- ログ
  - journald に集約（`journalctl -u geosight-web` など）
  - Nginx アクセスログはオフ（必要ならオンに戻す）

## 既知の注意点
- 初回ビルドは時間がかかる（npm / pip）。2回目以降はキャッシュで高速化
- Redis/Celery/Beat を止める簡約は可能だが、まずはフル構成の互換性を維持（後で段階的に簡約）
- HTTPS は後回し。外部公開時はプロキシやトンネル側で認証・暗号化を検討

## 参考
- GeoSight-OS: https://github.com/unicef-drp/GeoSight-OS
- GeoSight Documentation: https://unicef-drp.github.io/GeoSight-OS-Documentation/
- just: https://github.com/casey/just

## ライセンス
- このリポジトリ（スクリプトと文書）は CC0 (Public Domain)
- GeoSight-OS は AGPLv3 に従う（本家のライセンスに準拠）

---
Made with ❤️ by [hfu](https://github.com/hfu) and GitHub Copilot
