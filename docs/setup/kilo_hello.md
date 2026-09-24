# kilo hello — ответ агента на первый запрос

Готов.

1) Сервис — учебная **предварительная оценка заявки на заём под ПТС**: принимает заявку (VIN, год выпуска, пробег, оценочная стоимость, сумма, срок), считает LTV и возвращает решение `approve` / `review` / `reject`. Все данные синтетические, репозиторий публичный (`README.md`).
2) В `Makefile` цели `up` / `down` / `ps` / `logs` / `install` / `test` (PHPUnit) / `lint` (`php -l`) / `seed` / `help`; в `docker-compose.yml` сервис `backend` (PHP 8.3 + Slim, порт 8080) и БД `db` (MySQL 8.0, порт 3306).
3) Решение считается в `backend/src/Domain/DecisionEngine.php::decide(float $ltv)` по порогам из `backend/config/rules.php` (`approve_max = 60.0`, `review_max = 85.0`); вызывается через `backend/src/Domain/AssessmentService.php`.

модель: openrouter/minimax/minimax-m3 (MiniMax M3)
