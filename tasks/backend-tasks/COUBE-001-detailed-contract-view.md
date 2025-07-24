# SCRUM-260: Детальный просмотр контракта

## Информация о задаче

**Тип:** Story  
**Приоритет:** High  
**Спринт:** Backend Sprint 1  
**Оценка:** 8 story points  
**Ответственный:** Backend Developer  

## Описание

Реализовать детальный просмотр контракта (массовые перевозки) с полной информацией согласно дизайну Figma. В дизайне есть экраны детального просмотра контракта, но текущий API возвращает только базовую информацию.

## Требования

### Функциональные требования

1. **Новые эндпоинты:**
   - `GET /api/v1/customer/agreements/{id}/details`
   - `GET /api/v1/executor/agreements/{id}/details`

2. **Расширенный AgreementResponse:**
   ```java
   public record AgreementDetailsResponse(
       // Базовые поля
       Long id,
       LocalDateTime deadlineDate,
       Boolean withoutDeadline,
       TransportationType transportationType,
       AgreementStatus status,
       LocalDateTime agreementEndDate,
       String cargoName,
       DictionaryResponse cargoType,
       DictionaryResponse vehicleBodyType,
       Integer paymentDelay,
       Boolean isSignedEds,
       UUID agreementFileId,
       OrganizationResponse customer,
       Boolean useFactoring,
       Boolean useInsurance,
       List<DictionaryResponse> countries,
       
       // Новые поля для детального просмотра
       AgreementStatistics statistics,
       List<AgreementExecutorDetails> executors,
       List<AgreementHistoryItem> history,
       AgreementFileInfo fileInfo
   ) {}
   ```

3. **Статистика контракта:**
   ```java
   public record AgreementStatistics(
       Integer totalApplications,
       Integer activeApplications,
       Integer completedApplications,
       BigDecimal totalAmount,
       LocalDateTime lastActivity
   ) {}
   ```

4. **Детальная информация об исполнителях:**
   ```java
   public record AgreementExecutorDetails(
       Long id,
       OrganizationResponse organization,
       AgreementExecutor.Status status,
       LocalDateTime acceptedAt,
       LocalDateTime signedAt,
       EmployeeResponse signer,
       Integer applicationsCount,
       BigDecimal totalAmount
   ) {}
   ```

5. **История изменений:**
   ```java
   public record AgreementHistoryItem(
       Long id,
       String eventType,
       String description,
       String oldValue,
       String newValue,
       String changedBy,
       LocalDateTime createdAt
   ) {}
   ```

### Технические требования

1. **Производительность:**
   - Время ответа < 200ms
   - Использовать JOIN запросы для оптимизации
   - Добавить индексы для часто используемых полей

2. **Безопасность:**
   - Проверка прав доступа к контракту
   - Валидация входных параметров
   - Аудит запросов к детальной информации

3. **Документация:**
   - Swagger документация для новых эндпоинтов
   - Примеры запросов и ответов
   - Описание всех полей

## Критерии приемки

### Функциональные критерии
- [ ] API возвращает полную информацию о контракте
- [ ] Включена статистика по заявкам (количество, суммы, активность)
- [ ] Включена детальная информация об исполнителях
- [ ] Включена история изменений контракта
- [ ] Разные уровни детализации для заказчика и исполнителя
- [ ] Соответствует дизайну Figma

### Технические критерии
- [ ] Время ответа API < 200ms
- [ ] Покрытие тестами > 80%
- [ ] Swagger документация на 100%
- [ ] Нет N+1 запросов к БД
- [ ] Правильная обработка ошибок

### Качество кода
- [ ] Код соответствует стандартам проекта
- [ ] Добавлены unit тесты
- [ ] Добавлены integration тесты
- [ ] Код прошел code review

## Техническая реализация

### 1. Создать новые DTO классы

```java
// AgreementDetailsResponse.java
@Builder
public record AgreementDetailsResponse(
    // ... поля
) {}

// AgreementStatistics.java
@Builder
public record AgreementStatistics(
    // ... поля
) {}

// AgreementExecutorDetails.java
@Builder
public record AgreementExecutorDetails(
    // ... поля
) {}
```

### 2. Расширить AgreementService

```java
@Service
public class AgreementService {
    
    @Transactional(readOnly = true)
    public AgreementDetailsResponse getAgreementDetails(Long id, boolean isCustomer) {
        Agreement agreement = findAgreementById(id);
        
        // Проверка прав доступа
        validateAccess(agreement, isCustomer);
        
        // Получение статистики
        AgreementStatistics statistics = getAgreementStatistics(agreement);
        
        // Получение детальной информации об исполнителях
        List<AgreementExecutorDetails> executors = getExecutorDetails(agreement);
        
        // Получение истории изменений
        List<AgreementHistoryItem> history = getAgreementHistory(agreement);
        
        return AgreementDetailsResponse.builder()
            // ... сборка ответа
            .build();
    }
}
```

### 3. Добавить новые эндпоинты в контроллеры

```java
@RestController
@RequestMapping("/api/v1/customer/agreements")
public class CustomerAgreementController {
    
    @GetMapping("/{id}/details")
    @Operation(summary = "Детальный просмотр контракта для заказчика")
    public ResponseEntity<AgreementDetailsResponse> getAgreementDetails(
            @PathVariable Long id) {
        return ResponseEntity.ok(agreementService.getAgreementDetails(id, true));
    }
}
```

### 4. Создать репозитории для статистики

```java
@Repository
public interface AgreementStatisticsRepository {
    
    @Query("""
        SELECT COUNT(t) FROM Transportation t 
        WHERE t.agreement.id = :agreementId
        """)
    Integer countApplicationsByAgreement(@Param("agreementId") Long agreementId);
    
    @Query("""
        SELECT SUM(t.transportationCost) FROM Transportation t 
        WHERE t.agreement.id = :agreementId AND t.status = 'COMPLETED'
        """)
    BigDecimal getTotalAmountByAgreement(@Param("agreementId") Long agreementId);
}
```

## Тестирование

### Unit тесты
- [ ] AgreementService.getAgreementDetails()
- [ ] AgreementStatisticsRepository
- [ ] Валидация прав доступа
- [ ] Обработка ошибок

### Integration тесты
- [ ] GET /api/v1/customer/agreements/{id}/details
- [ ] GET /api/v1/executor/agreements/{id}/details
- [ ] Проверка производительности
- [ ] Проверка безопасности

### Тестовые данные
- Создать тестовые контракты с разными статусами
- Создать тестовых исполнителей
- Создать тестовые заявки
- Создать историю изменений

## Риски и митигация

### Риски
1. **Производительность**: Большое количество JOIN запросов
   - **Митигация**: Оптимизировать запросы, добавить индексы

2. **Безопасность**: Утечка информации между заказчиком и исполнителем
   - **Митигация**: Строгая проверка прав доступа

3. **Сложность**: Большой объем данных в одном запросе
   - **Митигация**: Разбить на несколько эндпоинтов при необходимости

## Зависимости

- [ ] COUBE-004: История изменений контракта (массовые перевозки) (для истории)
- [ ] Существующая система авторизации
- [ ] Существующая система уведомлений

## Ссылки

- [Дизайн Figma](https://www.figma.com/design/N66LiWC21wu8gI0rvSpGaT/Coube?node-id=8053-24164)
- [Документация контрактов](../contract.md)
- [Анализ несоответствий](../analysis/contract-design-mismatch.md) 