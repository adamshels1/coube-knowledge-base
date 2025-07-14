# Структура БД (актуальная)

## Таблица users.organization
- id (PK)
- business_type
- iin_bin
- organization_status
- organization_name
- legal_address
- actual_address
- vat
- ...

## Таблица users.employee
- id (PK)
- organization_id (FK на users.organization)
- employee_status
- email
- phone
- iin
- first_name, last_name, middle_name
- is_kz_residence
- telegram_acc
- ...

## Таблица users.employees_roles
- employee_id (FK на users.employee)
- role (TEXT)
- PK: (employee_id, role) 