## Матрица мастер-систем

| Сущность | Источники As-Is | Мастер-система на 3 месяца | Целевая мастер-система (12 мес) | Связь с BIAN | Обоснование |
|----------|-----------------|----------------------------|----------------------------------|--------------|-------------|
| **Клиент** | FU CRM, RB CRM, частично Core обеих сторон | **MDM Customer Hub** | **MDM** | Party Data Management | Нужна золотая запись, match & merge, единый customer_id. MDM — единственный источник правды. |
| **Договор** | FU Core, RB Core, CRM обеих сторон | Dual: FU Core (для FU-клиентов) + RB Core (для RB-клиентов) + синхронизация в MDM/интеграционный слой | **Целевой Core Banking** (или Agreement Hub) | Agreement / Product Agreement | На переходном этапе сохраняем мастер в исходных Core. В целевом — единый Agreement domain. |
| **Продукт** | FU Core/CRM, RB Core/CRM, Cards, Loans | **Справочник в MDM** (временный Product Catalog) | **Product Directory (MDM / Product Hub)** | Product Directory | Критично унифицировать коды продуктов уже на 3-м месяце. |
| **Счёт** | FU Core Banking, RB Core Banking | Dual-master: FU Core / RB Core | **Целевой Core Banking** | Current Account / Deposit Account | Счета остаются в Core. MDM не может быть мастером финансовых остатков. |
| **Карта** | Карты FinUnion, Карты RetailBank | Dual-master: FU Cards / RB Cards | **Целевой Card Management** | Card Management | Карточный процессинг остаётся мастером статусов и событий карты. |
| **Кредит / Заём** | FU Loans, RB Loans | Dual-master: FU Loans / RB Loans | **Целевой Loan System** | Loan / Credit Facility | Кредитные данные критичны и остаются в профильных системах. |
| **Транзакция** | Core, Cards, Loans обеих сторон | Источник = система-создатель операции | **Целевые OLTP-системы** (через CDC в Lakehouse/DWH) | Account Transaction / Payment Execution | Транзакции иммутабельны после фиксации. Мастер = система, которая создала проводку. |
| **Платёж** | Core, платёжные шлюзы, CRM | Dual / система-инициатор | **Payment Hub / Core** | Payment Order / Payment Execution | Разделяем «инициирование» (CRM/каналы) и «исполнение» (Core). |
| **Обращение** | FU CRM, RB CRM | Dual: FU CRM / RB CRM + единый customer_id через MDM | **Единый CRM / Case Management** | Customer Contact / Case Management | На 3 месяца — синхронизация через MDM + временные потоки. |
| **Справочник** | Все системы (разрозненно) | **MDM Reference Data** | **MDM Reference Data** | Reference Data Management | Один из самых важных доменов. Централизация обязательна. |
| **Организация** | Core, CRM обеих сторон | **MDM** (как часть Party) | **MDM** | Party Data Management (Legal Entity) | Юрлица ведутся в том же Party-домене, что и физлица. |
| **Событие / Документ** | Все системы + файловые хранилища | Источник = система-создатель + Object Storage | **Event Store / Document Management + Lakehouse** | Document Management / Event Log | Метаданные и lineage — в каталоге, сами файлы — в объектном хранилище. |

## Важные замечания

- **MDM не назначается** мастером для счетов, транзакций, кредитов и карт.
- На 3-м месяце допускается **dual-master** с чёткими правилами приоритета (например, «если клиент уже есть в FU — приоритет FU, иначе RB + создание в MDM»).
- BIAN-домены помогают чётко провести границы: Party → MDM, Account/Transaction → Core, Card → Card Processing, Loan → Loan System.