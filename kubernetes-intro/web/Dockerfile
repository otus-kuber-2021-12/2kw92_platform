FROM alpine:3.13

ENV NGINX_VERSION 1.18.0-r15

COPY . /opt/
COPY custom.conf /etc/nginx/conf.d/
COPY nginx.conf /etc/nginx/

RUN apk add --no-cache nginx=${NGINX_VERSION} && mkdir -p /run/nginx \
    && apk --no-cache add shadow && usermod -u 1001 nginx \
    && chown -R nginx:nginx /etc/nginx/ /opt/app /run/nginx /var/log/nginx

EXPOSE 8000

USER nginx

CMD ["nginx", "-g", "daemon off;"]
