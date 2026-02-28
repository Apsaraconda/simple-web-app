FROM python:3.13-alpine AS builder

RUN addgroup appgroup && adduser --disabled-password appuser && adduser appuser appgroup

#  Поскольку финальный образ основан на alpine, то компиляцию ведем на образе alpine:
#  используем apk add, build-base и musl-dev, чтобы бинарник правильно определился
RUN apk add --no-cache \
    build-base \
    musl-dev \
    libffi-dev \
    && pip install -qq pyinstaller==6.13.0

WORKDIR /app

COPY app.py .

RUN pyinstaller --onefile --log-level ERROR --name app app.py

RUN chown -R appuser:appgroup /app

# Финальная стадия
FROM alpine:3.23

WORKDIR /app

RUN addgroup appgroup && adduser --disabled-password appuser && adduser appuser appgroup

COPY --from=builder --chown=appuser:appgroup /app/dist/app /app

RUN chown -R appuser:appgroup /app

RUN chmod +x app

USER appuser

EXPOSE 8080

HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD wget --no-verbose --tries=1 --spider http://127.0.0.1:8080 || exit 1

CMD ["sh", "-c", "echo 'Container starting...' && ./app || echo 'app failed with code $?'"]