# Анализ несоответствий: Контракты (Дизайн vs Бэкенд)

## Обзор

Анализ несоответствий между дизайном Figma (node-id 8053:24164) и текущим состоянием бэкенда для модуля контрактов (массовые перевозки).

## ✅ Что соответствует

### API эндпоинты
- ✅ Создание контрактов (`POST /api/v1/customer/agreements`)
- ✅ Черновики контрактов (`POST /api/v1/customer/agreements/draft`)
- ✅ Принятие контрактов исполнителями (`POST /api/v1/executor/agreements/{id}/accept`)
- ✅ Подписание ЭЦП (`POST /api/v1/executor/agreements/{id}/sign`)
- ✅ QR-коды для мобильного подписания (`/api/v1/qr/*`)
- ✅ Список контрактов для заказчика (`GET /api/v1/customer/agreements`)
- ✅ Список контрактов для исполнителя (`GET /api/v1/executor/agreements`)

### Структура данных
- ✅ Таблица `applications.agreement` (основные контракты)
- ✅ Таблица `applications.agreement_executor` (исполнители)
- ✅ Таблица `applications.contract` (договоры-заявки)
- ✅ Статусы контрактов (`DRAFT`, `ACTIVE`, `EXPIRED`, `TERMINATED`)
- ✅ Статусы исполнителей (`PENDING`, `EXECUTOR_ACCEPTED`, `SIGNED`, `REJECTED`)

### Бизнес-логика
- ✅ Создание контрактов заказчиком
- ✅ Принятие контрактов перевозчиками
- ✅ Подписание ЭЦП
- ✅ Связь с заявками на перевозку

## ❌ Что отсутствует

### 1. Детальный просмотр контракта
**Проблема**: В дизайне есть экраны детального просмотра контракта с полной информацией, но API возвращает только базовую информацию.

**Что нужно:**
- Расширенный `AgreementResponse` с полной информацией
- Статистика по заявкам
- Детальная информация об исполнителях
- История изменений

### 2. Расторжение контрактов
**Проблема**: В дизайне есть экраны расторжения контрактов, но API отсутствует.

**Что нужно:**
- Эндпоинт `POST /api/v1/customer/agreements/{id}/terminate`
- Эндпоинт `POST /api/v1/executor/agreements/{id}/terminate`
- Статус `TERMINATED` в `AgreementStatus`
- Поля `terminatedAt` и `terminatedBy` в `Agreement`

### 3. История изменений
**Проблема**: В дизайне предполагается история изменений, но в API нет.

**Что нужно:**
- Таблица `applications.agreement_history`
- Эндпоинт `GET /api/v1/agreements/{id}/history`
- Логирование всех изменений контракта

### 4. Уведомления о статусах
**Проблема**: В дизайне есть уведомления, но система уведомлений для контрактов не реализована.

**Что нужно:**
- Интеграция с существующей системой уведомлений
- Уведомления для всех ключевых событий контрактов
- WhatsApp и Push-уведомления

### 5. Массовые операции
**Проблема**: В дизайне предполагаются массовые операции, но API отсутствует.

**Что нужно:**
- `POST /api/v1/executor/agreements/bulk-accept`
- `POST /api/v1/executor/agreements/bulk-reject`
- `POST /api/v1/customer/agreements/bulk-terminate`

### 6. Расширенная фильтрация
**Проблема**: В дизайне есть расширенные фильтры, но API ограничен.

**Что нужно:**
- Фильтры по дате создания
- Фильтры по типу груза
- Фильтры по статусу исполнителей
- Поиск по номеру заявки

### 7. Статистика контрактов
**Проблема**: В дизайне есть статистика, но API отсутствует.

**Что нужно:**
- Эндпоинт `GET /api/v1/agreements/{id}/statistics`
- Количество заявок по контракту
- Сумма перевозок по контракту
- Статистика исполнителей

## ⚠️ Что требует доработки

### 1. Валидация данных
- Улучшить валидацию входных данных
- Добавить проверки бизнес-правил
- Улучшить сообщения об ошибках

### 2. Производительность
- Оптимизировать запросы к БД
- Добавить индексы для фильтрации
- Кэширование часто используемых данных

### 3. Документация API
- Полная документация всех эндпоинтов
- Примеры запросов и ответов
- Описание ошибок

### 4. Безопасность
- Проверка прав доступа
- Аудит операций
- Валидация подписей

## 📊 Приоритеты

### Высокий приоритет
1. **SCRUM-260**: Детальный просмотр контракта
2. **COUBE-002**: Расторжение контрактов

### Средний приоритет
3. **COUBE-003**: Уведомления о статусах
4. **COUBE-004**: История изменений
5. **COUBE-008**: Улучшение API контрактов

### Низкий приоритет
6. **COUBE-005**: Массовые операции
7. **COUBE-006**: Расширенная фильтрация
8. **COUBE-007**: Статистика контрактов

## 🔄 Рекомендации

### Немедленные действия
1. Создать задачи в Jira с указанными приоритетами
2. Начать с задач высокого приоритета
3. Параллельно работать над улучшением существующего API

### Долгосрочные улучшения
1. Внедрить систему мониторинга API
2. Добавить автоматические тесты
3. Реализовать кэширование
4. Улучшить документацию

## 📈 Метрики успеха

- [ ] Все экраны из дизайна поддерживаются API
- [ ] Время ответа API < 200ms
- [ ] Покрытие тестами > 80%
- [ ] Документация API на 100%
- [ ] Нет критических ошибок в продакшене 