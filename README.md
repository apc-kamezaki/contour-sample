# Contour ingress controller sample

## Configuration

コンフィグレーションファイルは以下に格納。

 - settings
   - namespace.yaml K8s cluster内のアプリケーションnamespace
   - workloads  アプリケーションデプロイのYaml設定
   - routings　Contour向けアプリケーションルーティングYaml設定

デプロイ手順については [README](settings/README.md) に記載。