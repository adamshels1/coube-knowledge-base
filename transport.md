# Транспорт (transport)

## Таблица applications.transport
- id (PK)
- vehicle_id (FK на applications.vehicle)
- status (ENUM)
- created_at, updated_at, created_by, updated_by

## Таблица applications.transport_employee
- transport_id (FK на applications.transport)
- employee_id (FK на users.employee)
- PK: (transport_id, employee_id)

## Связи и бизнес-логика
- Транспорт связывает транспортное средство и сотрудников (водителей).
- Используется для назначения водителей на конкретные перевозки.
- Может быть связан с заявкой через transport_id. 