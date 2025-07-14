# Транспортные средства (vehicle)

## Таблица applications.vehicle
- id (PK)
- transportation_type (ENUM)
- vehicle_body_type_id (FK на dictionaries.vehicle_body_type)
- image_id (FK на file.file_meta_info)
- executor_organization_id (FK на users.organization)
- brand, model, registration_plate, issue_year, color
- semi_trailer_model, semi_trailer_registration_plate, semi_trailer_issue_year
- power, capacity_value, cargo_volume
- registration_certificate_id (FK на file.file_meta_info)
- created_at, updated_at, created_by, updated_by

## Связи и бизнес-логика
- Каждое транспортное средство может быть связано с организацией-исполнителем.
- Используется для формирования заявок, контрактов, транспортных операций.
- Может иметь связанные файлы (фото, документы). 