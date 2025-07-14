# Счета, акты, реестры (invoices, acts, registries)

## Таблица applications.invoices
- id (PK)
- invoice_number (TEXT, уникальный)
- document_date (DATE)
- customer_organization_id (FK на users.organization)
- executor_organization_id (FK на users.organization)
- суммы (без НДС, НДС, с НДС)
- файлы (excel_file_id, pdf_file_id)
- статус, подписанты, signed_date
- created_at, updated_at, created_by, updated_by

## Таблица applications.acts
- id (PK)
- act_number (TEXT, уникальный)
- document_date (DATE)
- customer_organization_id, executor_organization_id (оба FK на users.organization)
- суммы, файлы, статус, invoice_id (FK на invoices)
- created_at, updated_at, created_by, updated_by

## Таблица applications.registries
- id (PK)
- registry_number (TEXT, уникальный)
- date_from, date_to (DATE)
- act_id (FK на acts)
- customer_organization_id, executor_organization_id (оба FK на users.organization)
- суммы, файлы, статус
- created_at, updated_at, created_by, updated_by

## Связи и бизнес-логика
- Счета, акты и реестры связаны с организациями-заказчиком и исполнителем.
- Все документы могут иметь файлы (excel, pdf).
- Используются для финансового и юридического оформления перевозок.