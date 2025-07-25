# SCRUM-261: Реализация контрактов на фронтенде

## Задача для Jira

**Тип:** Epic  
**Приоритет:** High  
**Спринт:** Frontend Sprint 1  
**Оценка:** 8 story points  
**Ответственный:** Frontend Developer  

## Описание

Реализовать модуль контрактов на фронтенде согласно дизайну Figma. Контракты - это договоры между компаниями о массовых перевозках.

## Что нужно реализовать

### 1. Список контрактов
- Показ всех контрактов пользователя
- Фильтры по статусу
- Поиск по номеру/названию
- Создание нового контракта

### 2. Создание контракта
- Форма с полями: тип груза, дата, страны, отсрочка платежа
- Валидация полей
- Сохранение черновика
- Предпросмотр

### 3. Детальный просмотр
- Полная информация о контракте
- Статистика по заявкам
- Список исполнителей
- Действия: расторжение, редактирование

### 4. Для исполнителей
- Список доступных контрактов
- Принятие контракта
- Подписание ЭЦП
- QR-код для мобильного подписания

## API эндпоинты

### Основные
```
GET /api/v1/customer/agreements          // Список контрактов заказчика
GET /api/v1/executor/agreements          // Список контрактов исполнителя
POST /api/v1/customer/agreements         // Создание контракта
GET /api/v1/customer/agreements/{id}     // Детали контракта
POST /api/v1/executor/agreements/{id}/accept  // Принятие
POST /api/v1/executor/agreements/{id}/sign    // Подписание
POST /api/v1/customer/agreements/{id}/terminate // Расторжение
```

### Дополнительные
```
GET /api/v1/qr/agreement/{id}/sign       // QR для подписания
GET /api/v1/dictionaries/cargo-types     // Справочник типов груза
GET /api/v1/dictionaries/countries       // Справочник стран
```

## Компоненты Vue.js

```
- ContractList.vue          // Список контрактов
- ContractCard.vue          // Карточка контракта
- ContractForm.vue          // Форма создания
- ContractDetails.vue       // Детальный просмотр
- ContractSigning.vue       // Подписание ЭЦП
- ContractFilters.vue       // Фильтры и поиск
```

## Критерии приемки

- [ ] Все экраны из дизайна Figma реализованы
- [ ] Работает создание контрактов
- [ ] Работает просмотр деталей
- [ ] Работает принятие исполнителями
- [ ] Работает подписание ЭЦП
- [ ] Работает расторжение контрактов
- [ ] Адаптивность на всех устройствах
- [ ] TypeScript типизация

## Зависимости

- [ ] SCRUM-260: Детальный просмотр контракта (бэкенд)
- [ ] COUBE-002: Расторжение контрактов (бэкенд)

## Ссылки

- [Дизайн Figma](https://www.figma.com/design/N66LiWC21wu8gI0rvSpGaT/Coube?node-id=8053-24164)
- [Документация контрактов](../contract.md) 