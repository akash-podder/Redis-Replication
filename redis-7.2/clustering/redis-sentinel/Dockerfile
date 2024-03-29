FROM redis:7.2-alpine

COPY sentinel.conf /etc/redis/

CMD ["redis-sentinel", "/etc/redis/sentinel.conf"]