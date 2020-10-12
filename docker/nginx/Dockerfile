FROM alpine:3.11
MAINTAINER Kargapoltsev Sergey <soburon@yandex.ru>
# Берем версию 1.19.3 с сайта https://nginx.org/

LABEL nginx-version="1.19.3" 

# Перед сборкой версии, скачаем и установим необходимые пакеты
RUN \
  build_pkgs="build-base linux-headers wget zlib-dev openssl-dev pcre-dev" && \ 
  runtime_pkgs="openssl ca-certificates zlib pcre" && \
  apk --update add ${build_pkgs} ${runtime_pkgs} && \
  cd /tmp && \
  wget http://nginx.org/download/nginx-1.19.3.tar.gz && \
  tar xzf nginx-1.19.3.tar.gz && \
  cd /tmp/nginx-1.19.3 && \
    ./configure \
    --prefix=/etc/nginx \
    --sbin-path=/usr/sbin/nginx \
    --conf-path=/etc/nginx/nginx.conf \
    --error-log-path=/var/log/nginx/error.log \
    --http-log-path=/var/log/nginx/access.log \
    --pid-path=/var/run/nginx.pid \
    --lock-path=/var/run/nginx.lock \
    --http-client-body-temp-path=/var/cache/nginx/client_temp \
    --http-proxy-temp-path=/var/cache/nginx/proxy_temp \
    --http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp \
    --http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp \
    --http-scgi-temp-path=/var/cache/nginx/scgi_temp \
    --user=nginx \
    --group=nginx \
    --with-http_ssl_module \
	--with-mail \
    --with-mail_ssl_module \
    --with-stream \
    --with-stream_ssl_module \
	--with-ipv6 \
    --with-threads \
    --with-http_realip_module \
    --with-http_addition_module \
    --with-http_sub_module \
    --with-http_dav_module \
    --with-http_flv_module \
    --with-http_mp4_module \
    --with-http_gunzip_module \
    --with-http_gzip_static_module \
    --with-http_random_index_module \
    --with-http_secure_link_module \
    --with-http_stub_status_module \
    --with-http_auth_request_module \
    --with-file-aio \
    --with-http_slice_module \
    --with-http_v2_module && \
  make && make install && \ 
  adduser -D nginx && \
  rm -rf /tmp/* && \
  apk del ${build_pkgs} && \
  rm -rf /var/cache/apk/*

# Копируем наш nginx конфиг. Там же прописан адрес где мы ожидаем страничку index.html >  /opt/www/
COPY params/nginx.conf /etc/nginx/nginx.conf

VOLUME ["/var/cache/nginx"]

EXPOSE 80
# не обязательно. По умолчанию будет SIGTERM
#STOPSIGNAL SIGTERM

CMD ["nginx", "-g", "daemon off;"]


