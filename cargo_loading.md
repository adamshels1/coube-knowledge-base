# Погрузка/разгрузка (cargo_loading)

## Таблица applications.cargo_loading
- id (PK)
- transportation_id (FK на applications.transportation)
- loading_method_id, loading_operation_id (FK на справочники)
- order_num (INT)
- loading_type (ENUM)
- shipper_bin (TEXT)
- loading_datetime (TIMESTAMP)
- address, longitude, latitude
- commentary, weight, weight_unit, volume, loading_time_hours
- contact_number, contact_person_name
- created_at, updated_at, created_by, updated_by

## Связи и бизнес-логика
- Каждая запись связана с заявкой (transportation).
- Описывает этапы и параметры погрузки/разгрузки.
- Используется для планирования и контроля перевозки. 