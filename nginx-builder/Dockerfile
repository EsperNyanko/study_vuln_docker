# -----------------------------------------------------------
# ステージ1: ビルドステージ（Ubuntu 22.04 LTS を使用）
# -----------------------------------------------------------
FROM ubuntu:22.04 AS builder

# ビルドに必要なパッケージをインストール
RUN apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
    build-essential \
    libpcre3-dev \
    zlib1g-dev \
    libssl-dev \
    wget \
    ca-certificates && \
    rm -rf /var/lib/apt/lists/*

ENV NGINX_VERSION 1.25.3
ENV NGINX_FILENAME nginx-$NGINX_VERSION
ENV NGINX_TARBALL $NGINX_FILENAME.tar.gz

# WORKDIR を Nginxソースのルートがある /usr/src に設定
WORKDIR /usr/src/

# ダウンロード、展開、configure/make/make install を単一のRUNブロックで実行
RUN wget http://nginx.org/download/$NGINX_TARBALL && \
    tar -xzf $NGINX_TARBALL && \
    cd $NGINX_FILENAME && \
    ./configure \
      --prefix=/usr/local/nginx \
      --sbin-path=/usr/local/nginx/sbin/nginx \
      --conf-path=/usr/local/nginx/conf/nginx.conf \
      --error-log-path=/var/log/nginx/error.log \
      --http-log-path=/var/log/nginx/access.log \
      --http-client-body-temp-path=/tmp \
      --with-pcre \
      --with-http_ssl_module \
      --with-http_stub_status_module && \
    make && \
    make install

# -----------------------------------------------------------
# ステージ2: 実行環境ステージ（軽量な Ubuntu イメージを使用）
# -----------------------------------------------------------

FROM ubuntu:22.04

# Nginx実行に必要なユーザーとディレクトリを作成
RUN groupadd -r nginx && useradd -r -g nginx -s /sbin/nologin -d /var/cache/nginx nginx
RUN mkdir -p /var/cache/nginx /var/log/nginx /usr/local/nginx/conf/conf.d 
#RUN mkdir -p /usr/local/nginx/proxy_temp && chown nginx:nginx /usr/local/nginx/proxy_temp

# ビルドステージからビルド済みのNginxファイルをコピー
COPY --from=builder /usr/local/nginx /usr/local/nginx
COPY nginx.conf /usr/local/nginx/conf/nginx.conf

# コンテンツルートディレクトリを作成
RUN mkdir -p /usr/local/nginx/html

# Nginxの実行に必要なディレクトリの権限を設定 (ただし、これは docker-compose の entrypointで上書きされます)
RUN chown -R nginx:nginx /var/log/nginx /var/cache/nginx /usr/local/nginx/html

# Nginxの実行ユーザーを設定
USER nginx

# 80番ポートを公開
EXPOSE 80

# CMD は docker-compose.yml の entrypoint に上書きされます
CMD /usr/local/nginx/sbin/nginx -g 'daemon off;'
#CMD ["/usr/local/nginx/sbin/nginx", "-g", "daemon off;"]
#ENTRYPOINT ["tail", "-F", "/dev/null"]
