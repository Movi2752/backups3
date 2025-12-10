Минимальная платформа бэкапов (5 контейнеров, только Docker Hub)

Состав
- postgres — тестовая БД для бэкапа.
- minio — S3-хранилище бэкапов (порт 9000 API, 9001 консоль).
- minio-init — одноразово создаёт бакеты `backups` и `file-backups` через mc.
- pg-backup — cron-дампы Postgres в бакет `backups`.
- restic-backup — cron-бэкап каталога `/data` в бакет `file-backups`.

Быстрый старт
1) Скопируй и подправь переменные:
   ```
   cp env.example .env
   ```
   Проверь пароли и cron.
2) Запуск:
   ```
   docker compose pull
   docker compose up -d
   ```
3) Доступ:
   - MinIO console: http://localhost:9001 (логин/пароль из `.env`), S3 API: http://localhost:9000

Проверка бэкапов
- Ручной запуск Postgres-бэкапа: `docker compose run --rm pg-backup /backup.sh`
- Ручной запуск restic: `docker compose run --rm restic-backup backup`
- Посмотреть снимки restic: `docker compose run --rm restic-backup snapshots`

Настройка путей для файлового бэкапа
- По умолчанию бэкапится том `file-data`. Чтобы бэкапить локальную папку, смонтируй её:
  ```
  restic-backup:
    volumes:
      - ./data_to_backup:/data
  ```

Заметки
- `minio-init` может завершиться после создания бакетов — это нормально.
- Все образы с Docker Hub: `postgres`, `minio/minio:latest`, `minio/mc:latest`, `schickling/postgres-backup-s3`, `mazzolino/restic:1.7.1`.
- При необходимости зафиксируй версии MinIO/mc на конкретный тег.

