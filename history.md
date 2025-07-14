# История изменений (history)

## Таблица applications.history
- id (PK)
- transportation_id (FK на applications.transportation)
- current_cargo_loading (FK на gis.cargo_loading)
- event_type (TEXT)
- created_at, updated_at, created_by, updated_by

## Связи и бизнес-логика
- Фиксирует все ключевые события по заявке и погрузке.
- Используется для аудита и отслеживания статусов перевозки. 