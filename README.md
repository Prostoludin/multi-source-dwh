## DWH для нескольких источников

### Используемый стек технологий

![Python](https://img.shields.io/badge/Python-F7DF1E?style=for-the-badge&logo=python)
![PostgreSQL](https://img.shields.io/badge/postgresql-316192?style=for-the-badge&color=42aaff&logo=postgresql&logoColor=white)
![Airflow](https://img.shields.io/badge/airflow-316192?style=for-the-badge&logo=apacheairflow&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-316192?style=for-the-badge&logo=docker&logoColor=white)

### Описание проекта
Компания из сферы доставки еды хочет автоматизировать сбор и анализ данных из нескольких источников.

Источники данных:
1. База данных PostgreSQL - система бонусов. Содержит информацию о бонусном ранге каждого клиента, количестве накопленных бонусов, а также информацию об операциях по начислению/списанию бонусов.
2. База данных MongoDB - система заказов. Содержит информацию о ресторане, времени заказа, его статусе.
3. API курьерской службы.

Задачи:
1. Загрузить данные из источников.
2. Необходимо развернуть многослойное DWH по схеме снежинка.
3. Реализовать передачу данных из сервисов в конечную БД путем использования паттерна Transactional Outbox.
4. Построить витрины данных.
