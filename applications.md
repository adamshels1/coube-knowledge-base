# Заявки (applications)

## Основные таблицы и связи

### applications.transportation (Заявка)
- id (PK)
- organization_id (FK на users.organization) — заказчик
- executor_organization_id (FK на users.organization) — исполнитель
- contact_employee_id (FK на users.employee) — контактное лицо
- transport_id (FK на applications.transport) — транспорт
- contract_id (FK на applications.contract)
- invoice_id (FK на applications.invoices)
- agreement_number (TEXT)
- status, transportation_type, filling_step (ENUM)
- cargo_name, cargo_type_id, tare_type_id, vehicle_body_type_id, capacity_value_id
- cargo_price, cargo_weight, cargo_volume, vehicle_capacity
- currency, deadline_date, additional_info
- created_at, updated_at, created_by, updated_by

### applications.transportation_cost
- id (PK)
- transportation_id (FK на transportation)
- executor_organization_id (FK на users.organization)
- тарифы, валюта, суммы, статус, тип, factoring, NDS и др.

### applications.cargo_loading
- id (PK)
- transportation_id (FK на transportation)
- loading_method_id, loading_operation_id
- адрес, время, контакт, вес, объём и др.

### applications.contract
- id (PK)
- transportation_id (FK на transportation)
- original_file_id, expected_signatures_count, подписи, created_at, updated_at

### applications.invoices
- id (PK)
- invoice_number, document_date
- customer_organization_id (FK на users.organization)
- executor_organization_id (FK на users.organization)
- суммы, файлы, статус, подписанты, created_at, updated_at

### applications.acts
- id (PK)
- act_number, document_date
- customer_organization_id, executor_organization_id (оба FK на users.organization)
- суммы, файлы, статус, invoice_id (FK на invoices), created_at, updated_at

### applications.registries
- id (PK)
- registry_number, date_from, date_to
- act_id (FK на acts)
- customer_organization_id, executor_organization_id (оба FK на users.organization)
- суммы, файлы, статус, created_at, updated_at

### applications.history
- id (PK)
- transportation_id (FK на transportation)
- current_cargo_loading (FK на gis.cargo_loading)
- event_type, created_at, updated_at, created_by, updated_by

## Основные бизнес-связи
- Заявка (transportation) всегда связана с заказчиком (organization_id) и может быть связана с исполнителем (executor_organization_id).
- Контактное лицо — сотрудник заказчика (contact_employee_id).
- К заявке могут быть привязаны: транспорт, контракт, счёт, акт, реестр.
- Все суммы, статусы, документы и история изменений хранятся в соответствующих таблицах.

## Кратко о бизнес-логике
- Заявка создаётся заказчиком, далее может быть выбрана исполнительная организация.
- В процессе заявки формируются связанные сущности: транспорт, контракт, счета, акты, реестры.
- Все действия и статусы заявки отражаются в таблице history.

---

Если нужно подробнее по любой таблице или процессу — дополни этот файл или задай вопрос! 