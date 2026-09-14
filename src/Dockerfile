FROM alpine:3.24 as builder

RUN apk add --no-cache gcc \
    musl-dev \
    fcgi-dev

COPY server/server.c server.c

RUN gcc -o server server.c -lfcgi 


FROM nginx:1.31-alpine3.24

RUN apk add --no-cache spawn-fcgi fcgi

WORKDIR /app

RUN addgroup -g 1000 -S appgroup \
    && adduser -u 1000 -S -G appgroup appuser

COPY ./nginx/nginx.conf /etc/nginx/nginx.conf

RUN chown -R appuser:appgroup /etc/nginx/nginx.conf \
    && chown -R appuser:appgroup /var/cache/nginx \
    && touch /var/run/nginx.pid \
    && chown -R appuser:appgroup /var/run/nginx.pid

USER appuser

COPY --from=builder server server

EXPOSE 8080

CMD spawn-fcgi -p 8080 /app/server && nginx -g 'daemon off;'