FROM python:3.14.7-alpine3.24

ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1

WORKDIR app

RUN addgroup -S app && \
    adduser -S -G app app

COPY requirements.txt .
RUN apk add --no-cache libpq

RUN apk add --no-cache --virtual .build-deps \
        gcc \
        musl-dev \
        postgresql-dev \
        python3-dev && \
    pip install --no-cache-dir -r requirements.txt && \
    apk del .build-deps


COPY --chown=app:app . .

USER app

#CMD ["python", "-m", "app.main"]
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
