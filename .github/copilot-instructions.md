# Copilot Instructions for geosight-flip-in-da-house 🤖🎧

このリポジトリは「Dockerの中身を OS サービスへ Flip」するテスト・プロジェクト。キーワードは baremetal。安全第一、でもちょっと遊び心を忘れずに。

## ゴール
- フル構成互換のベアメタル起動（PostgreSQL+PostGIS / Redis / Nginx / Django(gunicorn) / Celery / Celery Beat / Frontend build）
- `.env` は `/etc/geosight/.env` で一元管理
- 初回 `npm run build` 必須、キャッシュ永続化で 2 回目以降高速化
- journald にログ集約、Nginx は `access_log off`

## 原則（テストの流儀）
- バージョンピンはしない（最新を浴びる）
- 本家 `unicef-drp/GeoSight-OS` をそのまま使う（直接クローン）
- HTTPS は後回し（まずは 2000 番で現場を通す）

## 期待するふるまい
- 変更提案は「壊さない」を最優先（フル構成維持）
- gunicorn で始め、必要になったら uWSGI を検討
- 設定値は `.env` を唯一の真実源に
- アップデートは「git pull → migrate → collectstatic → 再起動」の型に乗せる

## Do/Don't
- Do: systemd ユニットの依存関係（After/Requires）を丁寧に
- Do: journald 前提でログ設計（標準出力へ、Nginx は access_log off）
- Do: npm/pip キャッシュ永続化の提案（2回目以降を速く）
- Don't: いきなり Redis/Celery を削る（後で段階的に）
- Don't: バージョンを固定する（今回はテスト）

## 遊び心のルール（Flipの精神）
- コードコメントに短いライムを入れてよし
  - 例: `# flip the beat: cache heats the street`
- デバッグメッセージは軽やかに
  - 例: `"Booting geosight-web: keep it lean, keep it clean"`
- ただし、冗長すぎるノリは NG（読みやすさ最優先）

## 成果物イメージ
- `install.sh`: 1コマンドで「apt → clone → venv → migrate → build → nginx/systemd」まで
- `/etc/systemd/system/`:
  - `geosight-web.service`
  - `geosight-celery.service`
  - `geosight-celery-beat.service`
- `/etc/nginx/sites-available/geosight`（`access_log off;`）
- `/etc/geosight/.env`（権限 600）

## トラブル時の合言葉
- 「まずは Flip せず、基盤を見直す」
  - ネットワーク/権限/依存順序/環境変数を確認
- 「ビルドは一度、キャッシュは次へ」
  - 初回重いのは仕様。2回目以降で速くする

## 小さな願い
- Piの鼓動に合わせて、軽く熱く。Flipした基盤で地図が踊る瞬間を一緒に作ろう。

— Copilotより、ビートに忠実に。Keep it lean, keep it clean. 🎶
