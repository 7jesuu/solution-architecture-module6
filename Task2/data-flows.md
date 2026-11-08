# 3. Основные потоки данных (промежуточное состояние — 3 месяца)

Ниже описаны ключевые потоки, необходимые для минимально работоспособного объединённого ландшафта.

| № | Источник | Приёмник | Сущности | Действие | Режим обмена | Обоснование |
|---|----------|----------|----------|----------|--------------|-------------|
| 1 | RB CRM | MDM Customer Hub | Клиент | Create / Update / Deduplicate | Async (near real-time) | Загрузка клиентов RetailBank в единый хаб, построение golden record |
| 2 | FU CRM | MDM Customer Hub | Клиент | Create / Update / Deduplicate | Async (near real-time) | То же для клиентов FinUnion |
| 3 | MDM Customer Hub | FU CRM, RB CRM | Клиент (golden + customer_id) | Enrich / Update | Async | Обратная синхронизация единого ID и золотых атрибутов |
| 4 | MDM Customer Hub | DWH | Клиент | Read / Aggregate | Batch + CDC | Единый клиентский слой для отчётности |
| 5 | FU Core Banking | DWH | Счёт, Договор, Транзакция | Read / Aggregate | CDC / Batch | Построение управленческой отчётности |
| 6 | RB Core Banking | DWH | Счёт, Договор, Транзакция | Read / Aggregate | CDC / Batch | То же для RetailBank |
| 7 | FU Cards | DWH | Карта, Карточные операции | Read / Aggregate | CDC / Batch | Карточная отчётность |
| 8 | RB Cards | DWH | Карта, Карточные операции | Read / Aggregate | CDC / Batch | То же |
| 9 | FU Loans | DWH | Кредит, Кредитный портфель | Read / Aggregate | CDC / Batch | Кредитная отчётность |
| 10 | RB Loans | DWH | Кредит, Кредитный портфель | Read / Aggregate | CDC / Batch | То же |
| 11 | MDM + DWH | BI | Клиент, Счёт, Продукт, Операции | Read / Aggregate | Batch | Единые витрины управленческой отчётности |
| 12 | DWH | Регулятор | Отчётность | Aggregate / Export | Batch | Регуляторная отчётность (внешний потребитель) |
| 13 | FU CRM / RB CRM | MDM + DWH | Обращение | Create / Read | Async / Batch | Единая история обращений через customer_id |
| 14 | Все системы | Data Catalog / DataHub | Метаданные, Lineage | Create / Update | Async | Каталог данных и трассируемость |
| 15 | MDM Reference Data | Все потребительские системы | Справочники | Read / Enrich | Async / Cache | Централизованные справочники |
| 16 | Integration Layer | Все | Любые | Route / Validate / Transform | — | Единая точка управления потоками (вместо point-to-point) |

## Краткие принципы потоков на 3-м месяце

- **Клиент** — все изменения проходят через MDM.
- **Финансовые данные** — только извлечение (CDC/batch) в DWH, без изменения мастер-систем.
- **Point-to-point** постепенно заменяется на **Integration Layer**.
- Регулятор получает данные **только из DWH**, а не напрямую из Core.