# Контракты (contract)

## Таблица applications.contract
- id (PK)
- transportation_id (FK на applications.transportation)
- original_file_id (FK на file.file_meta_info)
- expected_signatures_count (INTEGER)
- signature_with_one_sign, signature_with_two_signs (FK на file.signature)
- created_at, updated_at, created_by, updated_by

## Связи и бизнес-логика
- Контракт связан с заявкой (transportation).
- Может иметь оригинальный файл и файлы подписей.
- Используется для юридического оформления перевозки. 