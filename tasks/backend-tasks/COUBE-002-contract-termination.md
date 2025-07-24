# COUBE-002: Расторжение контрактов

## Информация о задаче

**Тип:** Story  
**Приоритет:** High  
**Спринт:** Backend Sprint 1  
**Оценка:** 5 story points  
**Ответственный:** Backend Developer  

## Описание

Реализовать функционал расторжения контрактов согласно дизайну Figma. В дизайне есть экраны расторжения контрактов, но API отсутствует.

## Требования

### Функциональные требования

1. **Новые эндпоинты:**
   - `POST /api/v1/customer/agreements/{id}/terminate`
   - `POST /api/v1/executor/agreements/{id}/terminate`

2. **Расширение модели Agreement:**
   ```java
   @Entity
   public class Agreement extends BaseIdEntity {
       // ... существующие поля
       
       @Enumerated(EnumType.STRING)
       private AgreementStatus status;
       
       @Column(name = "terminated_at")
       private LocalDateTime terminatedAt;
       
       @Column(name = "terminated_by")
       private String terminatedBy;
       
       @Column(name = "termination_reason")
       private String terminationReason;
   }
   ```

3. **Новый статус в AgreementStatus:**
   ```java
   public enum AgreementStatus {
       DRAFT,
       ACTIVE,
       EXPIRED,
       TERMINATED  // новый статус
   }
   ```

4. **DTO для расторжения:**
   ```java
   public record AgreementTerminationRequest(
       @NotBlank(message = "Причина расторжения обязательна")
       @Size(max = 1000, message = "Причина не более 1000 символов")
       String reason,
       
       @NotNull(message = "Подтверждение расторжения обязательно")
       Boolean confirmed
   ) {}
   ```

5. **ЭЦП подписание для расторжения:**
   - Подписание расторжения ЭЦП
   - Валидация подписи
   - Сохранение подписанного документа

### Технические требования

1. **Безопасность:**
   - Проверка прав доступа к контракту
   - Валидация возможности расторжения
   - Аудит операции расторжения

2. **Бизнес-логика:**
   - Проверка статуса контракта (только ACTIVE можно расторгнуть)
   - Проверка наличия активных заявок
   - Уведомления участникам контракта

3. **Производительность:**
   - Время ответа < 500ms
   - Асинхронная отправка уведомлений
   - Оптимизированные запросы к БД

## Критерии приемки

### Функциональные критерии
- [ ] API для расторжения контрактов работает
- [ ] Подтверждение расторжения ЭЦП
- [ ] Уведомления отправляются участникам
- [ ] Статус контракта изменяется на TERMINATED
- [ ] Сохраняется причина и дата расторжения
- [ ] Проверка прав доступа работает корректно
- [ ] Соответствует дизайну Figma

### Технические критерии
- [ ] Время ответа API < 500ms
- [ ] Покрытие тестами > 80%
- [ ] Swagger документация на 100%
- [ ] Правильная обработка ошибок
- [ ] Логирование всех операций

### Качество кода
- [ ] Код соответствует стандартам проекта
- [ ] Добавлены unit тесты
- [ ] Добавлены integration тесты
- [ ] Код прошел code review

## Техническая реализация

### 1. Миграция БД

```sql
-- Добавить поля для расторжения в таблицу agreement
ALTER TABLE applications.agreement 
ADD COLUMN terminated_at TIMESTAMP,
ADD COLUMN terminated_by TEXT,
ADD COLUMN termination_reason TEXT;

-- Добавить индекс для поиска по статусу
CREATE INDEX idx_agreement_status ON applications.agreement (status);

-- Добавить индекс для поиска по дате расторжения
CREATE INDEX idx_agreement_terminated_at ON applications.agreement (terminated_at);
```

### 2. Расширить AgreementService

```java
@Service
public class AgreementService {
    
    @Transactional
    public AgreementResponse terminateAgreement(Long id, AgreementTerminationRequest request, boolean isCustomer) {
        Agreement agreement = findAgreementById(id);
        
        // Проверка прав доступа
        validateTerminationAccess(agreement, isCustomer);
        
        // Проверка возможности расторжения
        validateTerminationPossibility(agreement);
        
        // Расторжение контракта
        agreement.setStatus(AgreementStatus.TERMINATED);
        agreement.setTerminatedAt(LocalDateTime.now());
        agreement.setTerminatedBy(currentUserService.get().username());
        agreement.setTerminationReason(request.reason());
        
        agreement = agreementRepository.save(agreement);
        
        // Отправка уведомлений
        notificationService.notifyAgreementTermination(agreement);
        
        // Логирование
        auditService.logAgreementTermination(agreement, request);
        
        return agreementMapper.toResponse(agreement);
    }
    
    private void validateTerminationAccess(Agreement agreement, boolean isCustomer) {
        String currentUserOrg = currentUserService.getOrgIinBin();
        
        if (isCustomer) {
            if (!agreement.getCustomerOrganization().getIinBin().equals(currentUserOrg)) {
                throw new NoAccessException("Только заказчик может расторгнуть контракт");
            }
        } else {
            // Проверка что пользователь является исполнителем контракта
            boolean isExecutor = agreement.getAgreementExecutors().stream()
                .anyMatch(executor -> executor.getExecutorOrganization().getIinBin().equals(currentUserOrg));
            
            if (!isExecutor) {
                throw new NoAccessException("Только исполнитель может расторгнуть контракт");
            }
        }
    }
    
    private void validateTerminationPossibility(Agreement agreement) {
        if (agreement.getStatus() != AgreementStatus.ACTIVE) {
            throw new ResourceConflictException("Можно расторгнуть только активный контракт");
        }
        
        // Проверка активных заявок
        long activeApplications = transportationService.countActiveApplicationsByAgreement(agreement.getId());
        if (activeApplications > 0) {
            throw new ResourceConflictException("Нельзя расторгнуть контракт с активными заявками");
        }
    }
}
```

### 3. Добавить эндпоинты в контроллеры

```java
@RestController
@RequestMapping("/api/v1/customer/agreements")
public class CustomerAgreementController {
    
    @PostMapping("/{id}/terminate")
    @Operation(summary = "Расторжение контракта заказчиком")
    public ResponseEntity<AgreementResponse> terminateAgreement(
            @PathVariable Long id,
            @Valid @RequestBody AgreementTerminationRequest request) {
        return ResponseEntity.ok(agreementService.terminateAgreement(id, request, true));
    }
}

@RestController
@RequestMapping("/api/v1/executor/agreements")
public class ExecutorAgreementController {
    
    @PostMapping("/{id}/terminate")
    @Operation(summary = "Расторжение контракта исполнителем")
    public ResponseEntity<AgreementResponse> terminateAgreement(
            @PathVariable Long id,
            @Valid @RequestBody AgreementTerminationRequest request) {
        return ResponseEntity.ok(agreementService.terminateAgreement(id, request, false));
    }
}
```

### 4. Создать сервис уведомлений

```java
@Service
public class AgreementNotificationService {
    
    @Async
    public void notifyAgreementTermination(Agreement agreement) {
        // Уведомление заказчику
        notifyCustomer(agreement);
        
        // Уведомление исполнителям
        notifyExecutors(agreement);
    }
    
    private void notifyCustomer(Agreement agreement) {
        String message = String.format(
            "Контракт №%d расторгнут. Причина: %s",
            agreement.getId(),
            agreement.getTerminationReason()
        );
        
        notificationService.sendNotification(
            NotificationRequest.builder()
                .title("Расторжение контракта")
                .body(message)
                .eventType("AGREEMENT_TERMINATED")
                .build()
        );
    }
    
    private void notifyExecutors(Agreement agreement) {
        for (AgreementExecutor executor : agreement.getAgreementExecutors()) {
            String message = String.format(
                "Контракт №%d расторгнут. Причина: %s",
                agreement.getId(),
                agreement.getTerminationReason()
            );
            
            notificationService.sendNotification(
                NotificationRequest.builder()
                    .title("Расторжение контракта")
                    .body(message)
                    .eventType("AGREEMENT_TERMINATED")
                    .build()
            );
        }
    }
}
```

### 5. ЭЦП подписание

```java
@Service
public class AgreementSignatureService {
    
    public void signTermination(Agreement agreement, String signedCmsBase64, String originalDataBase64) {
        // Валидация подписи
        validateSignature(signedCmsBase64, originalDataBase64);
        
        // Сохранение подписанного документа
        FileMetaInfo signedFile = fileService.saveSignedDocument(
            agreement.getAgreementFile(),
            signedCmsBase64,
            originalDataBase64
        );
        
        // Обновление контракта
        agreement.setTerminationFile(signedFile);
        agreementRepository.save(agreement);
    }
}
```

## Тестирование

### Unit тесты
- [ ] AgreementService.terminateAgreement()
- [ ] Валидация прав доступа
- [ ] Валидация возможности расторжения
- [ ] Обработка ошибок

### Integration тесты
- [ ] POST /api/v1/customer/agreements/{id}/terminate
- [ ] POST /api/v1/executor/agreements/{id}/terminate
- [ ] Проверка уведомлений
- [ ] Проверка ЭЦП подписания

### Тестовые данные
- Создать тестовые контракты в разных статусах
- Создать тестовых пользователей с разными ролями
- Создать тестовые заявки

## Риски и митигация

### Риски
1. **Безопасность**: Неавторизованное расторжение контрактов
   - **Митигация**: Строгая проверка прав доступа

2. **Бизнес-логика**: Расторжение контракта с активными заявками
   - **Митигация**: Проверка активных заявок перед расторжением

3. **Производительность**: Медленные уведомления
   - **Митигация**: Асинхронная отправка уведомлений

## Зависимости

- [ ] COUBE-003: Уведомления о статусах контрактов
- [ ] Существующая система авторизации
- [ ] Существующая система ЭЦП
- [ ] Существующая система уведомлений

## Ссылки

- [Дизайн Figma](https://www.figma.com/design/N66LiWC21wu8gI0rvSpGaT/Coube?node-id=8053-24164)
- [Документация контрактов](../contract.md)
- [Анализ несоответствий](../analysis/contract-design-mismatch.md) 