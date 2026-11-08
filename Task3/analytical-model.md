# 5. Аналитическая модель

Выбрана схема **«звезда»** для области **«Транзакции и клиентская активность»**.

## Почему звезда

- Простые и быстрые OLAP-запросы (фильтры по измерениям + агрегаты по факту).
- Понятна бизнес-пользователям и BI-инструментам.
- Хорошо ложится на регламентированную отчётность и управленческие дашборды.
- Легче поддерживать, чем снежинку, на этапе объединения двух банков.

(Снежинка может быть использована позже внутри отдельных измерений, если появится сложная иерархия продуктов или организационной структуры.)

## Модель

| Тип таблицы | Имя | Описание | Ключевые поля |
|-------------|-----|----------|---------------|
| **Fact** | `fact_transactions` | Финансовые операции | transaction_id, customer_sk, account_sk, product_sk, date_sk, amount, currency, direction, channel_sk, status |
| **Fact** | `fact_account_balances` | Ежедневные остатки по счетам (snapshot) | account_sk, customer_sk, date_sk, balance, currency |
| **Fact** | `fact_loan_portfolio` | Срез кредитного портфеля | loan_sk, customer_sk, date_sk, outstanding, rate, status_sk |
| **Dimension** | `dim_customer` | Клиент (из MDM golden) | customer_sk, customer_id, full_name, inn, segment, status, open_date |
| **Dimension** | `dim_account` | Счёт | account_sk, account_id, account_number, currency, type, status, open_date |
| **Dimension** | `dim_product` | Продукт | product_sk, product_code, product_name, product_type, line |
| **Dimension** | `dim_channel` | Канал операции | channel_sk, channel_code, channel_name |
| **Dimension** | `dim_date` | Календарь | date_sk, date, year, quarter, month, week, day_of_week, is_holiday |
| **Dimension** | `dim_status` | Унифицированные статусы | status_sk, status_code, status_name, entity_type |
| **Dimension** | `dim_organization` | Организация / контрагент | org_sk, org_id, name, inn |

## Связи (упрощённо)

fact_transactions
├── dim_customer
├── dim_account
├── dim_product
├── dim_channel
├── dim_date
└── dim_statusfact_account_balances
├── dim_customer
├── dim_account
└── dim_datefact_loan_portfolio
├── dim_customer
├── dim_date
└── dim_status

## Почему модель подходит для OLAP

- Факты содержат числовые меры (`amount`, `balance`, `outstanding`) — удобно агрегировать.
- Все срезы идут через измерения — типичные запросы «по клиентам / продуктам / каналам / периодам» выполняются эффективно.
- `dim_customer` строится из MDM, поэтому в аналитике сразу используется единый клиент.
- Статусы и продукты нормализованы через справочники — исчезает проблема разных кодировок двух банков.