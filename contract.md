# Контракты (Contracts)

## Обзор

**Контракты** в системе Coube — это **договоры между компаниями о перевозках в пачке** (рамковые соглашения). Это не отдельные договоры на каждую перевозку, а долгосрочные соглашения, которые определяют условия сотрудничества между заказчиком и перевозчиками.

### Как работают контракты:

1. **Компания-заказчик** заключает **контракт** с **перевозчиками**
2. Контракт определяет условия сотрудничества на определенный период
3. Когда заказчик создает **заявку**, он указывает **контракт**
4. Эту заявку видят **только те перевозчики**, у которых есть этот контракт
5. Перевозчики могут брать заявки только по своим контрактам

### Пример:
- Компания "Логистик-А" заключает контракт с перевозчиками "Транспорт-1", "Транспорт-2", "Транспорт-3"
- Когда "Логистик-А" создает заявку и указывает этот контракт
- Заявку видят только "Транспорт-1", "Транспорт-2", "Транспорт-3"
- Другие перевозчики эту заявку не видят

## Типы контрактов в системе

### 1. **applications.agreement** - Основные контракты (Agreement)
Это и есть **контракты** в понимании системы - договоры между заказчиком и перевозчиками.

### 2. **applications.contract** - Договоры-заявки (Contract)
Это **договоры-заявки** на конкретную перевозку, которые генерируются автоматически при создании заявки.

### 3. **factoring.contract** - Факторинг контракты
Финансовые договоры для факторинга.

## Структура данных

### Таблица `applications.agreement` (Контракты)

```sql
CREATE TABLE applications.agreement (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    organization_id BIGINT,                    -- Заказчик (организация)
    deadline_date TIMESTAMP,                   -- Дата окончания контракта
    without_deadline BOOLEAN,                  -- Без срока действия
    transportation_type TEXT,                  -- Тип перевозки
    status TEXT,                              -- Статус контракта
    agreement_end_date TIMESTAMP,              -- Дата окончания
    cargo_name TEXT,                          -- Наименование груза
    cargo_type_id BIGINT,                     -- Тип груза
    vehicle_body_type_id BIGINT,              -- Тип кузова
    payment_delay INTEGER,                     -- Отсрочка платежа (дни)
    is_signed_eds BOOLEAN,                    -- Подписан ЭЦП
    use_factoring BOOLEAN DEFAULT FALSE,      -- Использовать факторинг
    use_insurance BOOLEAN DEFAULT FALSE,      -- Использовать страхование
    agreement_file_id UUID UNIQUE,            -- Файл контракта
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    created_by TEXT,
    updated_by TEXT
);
```

### Таблица `applications.agreement_executor` (Исполнители контракта)

```sql
CREATE TABLE applications.agreement_executor (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    agreement_id BIGINT NOT NULL,             -- ID контракта
    organization_id BIGINT,                   -- Перевозчик (организация)
    status TEXT,                             -- Статус исполнителя
    contract_id BIGINT UNIQUE,               -- Связанный договор-заявка
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    created_by TEXT,
    updated_by TEXT
);
```

### Таблица `applications.contract` (Договоры-заявки)

```sql
CREATE TABLE applications.contract (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    transportation_id BIGINT,                 -- Связанная заявка
    agreement_executor_id BIGINT,             -- Связанный исполнитель контракта
    original_file_id BIGINT,                 -- Оригинальный файл
    expected_signatures_count INTEGER,        -- Ожидаемое количество подписей
    signature_with_one_sign BIGINT,          -- Подпись с одной подписью
    signature_with_two_signs BIGINT,         -- Подпись с двумя подписями
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    created_by TEXT,
    updated_by TEXT
);
```

## API для работы с контрактами

### Для заказчика (`/api/v1/customer/agreements`)

#### Создание контракта
```http
POST /api/v1/customer/agreements/draft
Content-Type: application/json

{
  "deadlineDate": "2024-12-31T23:59:59",
  "withoutDeadline": false,
  "transportationType": "INTERNATIONAL",
  "agreementEndDate": "2024-12-31T23:59:59",
  "cargoName": "Строительные материалы",
  "cargoType": {"id": 1},
  "vehicleBodyType": {"id": 2},
  "paymentDelay": 30,
  "isSignedEds": true,
  "useFactoring": false,
  "useInsurance": true,
  "countries": [{"id": 1}, {"id": 2}]
}
```

#### Получение списка контрактов
```http
GET /api/v1/customer/agreements?page=0&size=20&status=ACTIVE
```

#### Создание контракта
```http
POST /api/v1/customer/agreements
Content-Type: application/json

{
  // те же поля что и в draft
}
```

### Для перевозчика (`/api/v1/executor/agreements`)

#### Получение доступных контрактов
```http
GET /api/v1/executor/agreements?page=0&size=20
```

#### Принятие контракта
```http
POST /api/v1/executor/agreements/{agreementId}/accept
Content-Type: application/json

{
  "executorOrganizationId": 123,
  "executorSignerId": 456
}
```

#### Подписание контракта
```http
POST /api/v1/executor/agreements/{agreementId}/sign
Content-Type: application/json

{
  "signedCmsBase64": "base64_encoded_signature",
  "originalDataBase64": "base64_encoded_document"
}
```

## Анализ дизайна из Figma

### Основные экраны контрактов:

#### 1. **Список контрактов (Заказчик)**
- Отображение всех контрактов компании
- Фильтрация по статусу, дате, типу груза
- Поиск по номеру заявки
- Возможность создания нового контракта

#### 2. **Создание контракта (Заказчик)**
- Форма создания контракта с полями:
  - Тип перевозки
  - Наименование груза
  - Тип груза
  - Тип кузова
  - Отсрочка платежа
  - Использование факторинга/страхования
  - Страны перевозки
  - Срок действия контракта

#### 3. **Просмотр контракта (Заказчик)**
- Детальная информация о контракте
- Список исполнителей (перевозчиков)
- Статусы подписания
- Возможность расторжения

#### 4. **Контракты перевозчика**
- Список доступных контрактов
- Возможность принятия контракта
- Подписание контракта
- Просмотр заявок по контракту

#### 5. **Подписание контракта**
- ЭЦП подписание
- QR-код для мобильного подписания
- Статусы подписания

### Соответствие дизайна и API:

#### ✅ **Что есть в API и дизайне:**
- Создание контрактов (Agreement)
- Принятие контрактов исполнителями
- Подписание ЭЦП
- Статусы контрактов
- Фильтрация и поиск
- QR-код для мобильного подписания

#### ⚠️ **Что может потребовать доработки:**
- Детальный просмотр контракта с полной информацией
- Расторжение контрактов
- История изменений контракта
- Уведомления о статусах
- Массовые операции с контрактами

## Бизнес-логика

### Жизненный цикл контракта:

1. **Создание** - Заказчик создает контракт
2. **Размещение** - Контракт становится доступным перевозчикам
3. **Принятие** - Перевозчики принимают контракт
4. **Подписание** - Стороны подписывают контракт ЭЦП
5. **Активный** - Контракт действует, по нему создаются заявки
6. **Расторжение** - Контракт может быть расторгнут

### Связь с заявками:

- При создании заявки заказчик указывает контракт
- Заявка видна только перевозчикам с этим контрактом
- Автоматически генерируется договор-заявка (Contract)
- Подписание договора-заявки происходит отдельно

## Статусы контрактов

### AgreementStatus:
- `DRAFT` - Черновик
- `ACTIVE` - Активный
- `EXPIRED` - Истекший
- `TERMINATED` - Расторгнутый

### AgreementExecutor.Status:
- `PENDING` - Ожидает принятия
- `EXECUTOR_ACCEPTED` - Принят исполнителем
- `SIGNED` - Подписан
- `REJECTED` - Отклонен

## Архитектурные компоненты

### Сервисы:
- `AgreementService` - Основная логика контрактов
- `AgreementExecutorService` - Логика исполнителей
- `ContractService` - Логика договоров-заявок
- `QrService` - QR-коды для подписания

### Контроллеры:
- `CustomerAgreementController` - API для заказчика
- `ExecutorAgreementController` - API для перевозчика
- `QrController` - QR-коды

### Репозитории:
- `AgreementRepository`
- `AgreementExecutorRepository`
- `ContractRepository`

## Рекомендации по развитию

### Приоритетные доработки:
1. **Детальный просмотр контракта** - полная информация о контракте
2. **Расторжение контрактов** - функционал расторжения
3. **Уведомления** - уведомления о статусах контрактов
4. **История изменений** - логирование изменений контракта
5. **Массовые операции** - массовое принятие/отклонение контрактов

### Долгосрочные улучшения:
1. **Шаблоны контрактов** - предустановленные шаблоны
2. **Условия контрактов** - гибкие условия и правила
3. **Аналитика контрактов** - статистика и отчеты
4. **Интеграция с ЭЦП** - расширенная интеграция с ЭЦП
5. **Мобильное приложение** - полноценное мобильное приложение 