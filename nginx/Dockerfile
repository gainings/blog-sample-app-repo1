# app コンテナの前段に置くリバースプロキシ (サイドカー)。ECS awsvpc では同一タスク内のコンテナが localhost を共有する。
FROM nginx:1.27-alpine
COPY default.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
