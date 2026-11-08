# 4. Управление потоками данных

| Источник | Приёмник | Сущность | Действие | Формат | Частота | Тип обмена | SLA | Владелец потока | Контроль качества | Безопасность | Мониторинг |
|----------|----------|----------|----------|--------|---------|------------|-----|-----------------|-------------------|--------------|------------|
| RB CRM | MDM | Клиент | Create/Update/Deduplicate | JSON/Avro | Near real-time | Async (Kafka/ESB) | ≤ 5 мин | Data Office | Проверка дублей, обязательные поля, match confidence | TLS, шифрование ПДн, RBAC | Lag, error rate, duplicate rate |
| FU CRM | MDM | Клиент | Create/Update/Deduplicate | JSON/Avro | Near real-time | Async | ≤ 5 мин | Data Office | То же | TLS, шифрование ПДн | Lag, error rate |
| MDM | FU CRM / RB CRM | Клиент (golden) | Enrich/Update | JSON | Near real-time | Async | ≤ 10 мин | Data Office | Проверка, что customer_id проставлен | TLS, аудит изменений | Success rate, conflict count |
| MDM | DWH | Клиент | Load | Parquet/CSV | Hourly + CDC | Batch + CDC | ≤ 1 час | Data Office | Полнота, актуальность golden record | Шифрование, маскирование в non-prod | Row count, freshness |
| FU Core | DWH | Счёт, Договор, Транзакция | Extract | CDC (Debezium) / Flat | Near real-time / Nightly | CDC + Batch | CDC ≤ 15 мин, Batch ≤ 6:00 | ИТ Core + Data Office | Сверка балансов, контроль суммы транзакций | TLS, отдельный технический пользователь | Lag CDC, reconciliation breaks |
| RB Core | DWH | Счёт, Договор, Транзакция | Extract | CDC / Flat | Near real-time / Nightly | CDC + Batch | То же | ИТ Core RB + Data Office | То же | TLS | Lag, reconciliation |
| FU Cards / RB Cards | DWH | Карта, Операции | Extract | CDC / Batch | Near real-time / Nightly | CDC + Batch | ≤ 15 мин / 6:00 | Card Operations + Data Office | Контроль статусов карт, PCI-поля маскируются | TLS, токенизация PAN | Lag, mask compliance |
| FU Loans / RB Loans | DWH | Кредит | Extract | CDC / Batch | Nightly + intraday | CDC + Batch | ≤ 30 мин / 6:00 | Credit Ops + Data Office | Сверка портфеля, контроль residual | TLS | Portfolio reconciliation |
| DWH | BI | Витрины | Publish | Таблицы/Parquet | Nightly + on-demand | Batch | ≤ 7:00 | Data Office | Data quality rules на витринах | RBAC на витрины | Freshness, test results |
| DWH | Регулятор | Отчётность | Export | XML/CSV (регуляторный) | По календарю | Batch | Согласно регулятору | Финансовый департамент | Полная сверка с источником | Канал защищён, аудит выгрузки | Delivery confirmation |
| Все системы | Data Catalog | Метаданные, Lineage | Register | OpenAPI / Atlas / DataHub events | Continuous | Async | ≤ 15 мин | Data Office | Обязательность schema + owner | RBAC | Coverage % of datasets |
| MDM Ref Data | Потребители | Справочники | Distribute | JSON / API | On change + cache | Async + Cache | ≤ 5 мин после изменения | Data Office | Версионирование, period of validity | TLS, dual-control на изменение | Cache hit, version lag |

## Дополнительные требования к управлению потоками

- Все потоки с ПДн и финансовыми данными проходят через **Integration Layer** (не point-to-point).
- Обязателен **аудит** кто/когда/что изменил в мастер-данных.
- При нарушении SLA — автоматический алерт в Data Office и владельцу домена.
- Dead-letter queue обязателен для всех async-потоков.