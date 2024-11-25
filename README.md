## DWH для нескольких источников

### Используемый стек технологий

![Python](https://img.shields.io/badge/Python-F7DF1E?style=for-the-badge&logo=python)
![PostgreSQL](https://img.shields.io/badge/postgresql-316192?style=for-the-badge&color=42aaff&logo=postgresql&logoColor=white)
![Airflow](https://img.shields.io/badge/airflow-316192?style=for-the-badge&logo=apacheairflow&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-316192?style=for-the-badge&logo=docker&logoColor=white)

### Описание проекта
Компания из сферы доставки еды хочет автоматизировать сбор и анализ данных из нескольких источников, чтобы автоматизировать взаиморасчеты с контрагентами.

Источники данных:
1. База данных PostgreSQL - система бонусов. Содержит информацию о бонусном ранге каждого клиента, количестве накопленных бонусов, а также информацию об операциях по начислению/списанию бонусов.
2. База данных MongoDB - система заказов. Содержит информацию о ресторане, времени заказа, его статусе.
3. API курьерской службы.

Задачи:
1. Загрузить данные из источников.
2. Развернуть многослойное DWH по схеме снежинка.
3. Реализовать передачу данных из сервисов в конечную БД путем использования паттерна Transactional Outbox.
4. Построить витрины данных.

Описание витрин данных:
## 1. Витрина для расчётов с курьерами.
Необходимо построить витрину, содержащую информацию о выплатах курьерам.

Состав витрины:
 - id — идентификатор записи;
 - courier_id — ID курьера, которому перечисляем;
 - courier_name — Ф. И. О. курьера;
 - settlement_year — год отчёта;
 - settlement_month — месяц отчёта, где 1 — январь и 12 — декабрь;
 - orders_count — количество заказов за период (месяц);
 - orders_total_sum — общая стоимость заказов;
 - rate_avg — средний рейтинг курьера по оценкам пользователей;
 - order_processing_fee — сумма, удержанная компанией за обработку заказов, которая высчитывается как orders_total_sum * 0.25;
 - courier_order_sum — сумма, которую необходимо перечислить курьеру за доставленные им/ей заказы. За каждый доставленный заказ курьер должен получить некоторую сумму в зависимости от рейтинга (см. ниже);
 - courier_tips_sum — сумма, которую пользователи оставили курьеру в качестве чаевых;
 - courier_reward_sum — сумма, которую необходимо перечислить курьеру. Вычисляется как courier_order_sum + courier_tips_sum * 0.95 (5% — комиссия за обработку платежа).

Правила расчёта процента выплаты курьеру в зависимости от рейтинга, где r — это средний рейтинг курьера в расчётном месяце:
 - r < 4 — 5% от заказа, но не менее 100 р.;
 - 4 <= r < 4.5 — 7% от заказа, но не менее 150 р.;
 - 4.5 <= r < 4.9 — 8% от заказа, но не менее 175 р.;
 - 4.9 <= r — 10% от заказа, но не менее 200 р.

## 2. Витрина для расчетов с ресторанами.
Компании нужно организовать взаиморасчёты с ресторанами, которые высчитываются на основе данных из подсистемы обработки заказов и данных по оплатам бонусами.
Бизнес-процесс построен следующим образом:
Компания принимает оплату от клиентов на свой счёт, а в начале следующего месяца, не позднее десяти рабочих дней, переводит необходимые суммы ресторанам-партнёрам. Детализация в витрине должна быть до дня. Пользователь системы может оплатить заказ рублями и частично бонусами. С каждой оплаты компания берёт комиссию 25%.

Состав витрины:
 - id — идентификатор записи;
 - restaurant_id — ID ресторана, которому компания перечисляет необходимые суммы;
 - restaurant_name — наименование ресторана;
 - settlement_date — дата отчёта;
 - orders_count — количество заказов за период (месяц);
 - orders_total_sum — общая стоимость заказов;
 - orders_bonus_payment_sum — сумма оплат бонусами за период;
 - orders_bonus_granted_sum — сумма накопленных бонусов за период;
 - order_processing_fee — сумма, удержанная компанией за обработку заказов, которая высчитывается как orders_total_sum * 0.25;
 - restaurant_reward_sum — сумма, которую необходимо перечислить ресторану, высчитываемая как total_sum - orders_bonus_payment_sum - order_processing_fee.

Для расчёта показателей отбираются только заказы с финальным статусом "CLOSED".

