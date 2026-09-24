# Карта кода для фичи «пробег ≤ 400 000 км, иначе review»

Артефакт задачи D1 (ветка `d1/1.2.1-1.2.3-pilin-dev`). Код не менялся — только анализ.

## Цепочка «файл → функция → порядок» (путь заявки)

| # | Файл | Функция / место | Что делает |
|---|------|-----------------|-----------|
| 1 | `backend/src/AppFactory.php` | `AppFactory::create()` :25 | Собирает приложение: `require config/rules.php` :27, строит `ApplicationValidator` :31, `LtvCalculator` :36, `DecisionEngine` :37 и вшивает их в `AssessmentService` :30. Роут `POST /api/applications` → `ApplicationController::create` :54 |
| 2 | `backend/src/Http/ApplicationController.php` | `create()` | Принимает JSON, вызывает `AssessmentService::assess()`, отдаёт ответ / HTTP 422 при `ValidationException` |
| 3 | `backend/src/Domain/AssessmentService.php` | `assess()` :28 | Оркестратор: `validate()` :30 → `calculate()` :32 → `decide()` :33 → массив результата :35 |
| 4 | `backend/src/Domain/ApplicationValidator.php` | `validate()` :24 | Валидация полей по `rules`. **Пробег: строки 43–46** |
| 5 | `backend/src/Domain/LtvCalculator.php` | `calculate()` | LTV = сумма / стоимость. Пробег не участвует |
| 6 | `backend/src/Domain/DecisionEngine.php` | `decide($ltv)` :30 | Решение **только по LTV**: `<60 → approve`, `≤85 → review`, иначе `reject`. Пробег не участвует |
| 7 | `backend/src/Repository/ApplicationRepository.php` | `save()` :22 (INSERT :38) | Пишет `mileage` в `vehicles.mileage_km` — уже работает, менять не надо |

## Что уже проверяется про пробег

`backend/src/Domain/ApplicationValidator.php:43-46`:

```php
$mileage = (int) ($payload['mileage'] ?? -1);
if ($mileage < 0 || $mileage > $this->rules['vehicle']['max_mileage_km']) {
    $errors['mileage'] = sprintf('Пробег от 0 до %d км', $this->rules['vehicle']['max_mileage_km']);
}
```

Порог `max_mileage_km = 500000` задан в `backend/config/rules.php:23`.

**Ключевой конфликт с фичей:** сейчас пробег > 500 000 км — ошибка валидации
(`ValidationException` → HTTP 422), а не решение `review`. Фича требует:
400 000 < пробег → решение `review`, то есть поведение меняется с «отклонить
на входе» на «пропустить в оценку с пониженным решением». Это новое правило
на уровне решения, а не правка одного числа.

## Точка вставки правила

**`backend/src/Domain/AssessmentService.php`, `assess()`, между строками 32 и 33** —
после валидации и расчёта LTV, до вызова `DecisionEngine`. Пробег уже есть в
`$input['mileage']`.

Фрагмент (реальные строки 30–35 файла):

```php
        $input = $this->validator->validate($payload);          // :30

        $ltv = $this->ltvCalculator->calculate($input['requested_amount'], $input['market_value']);  // :32
        $decision = $this->decisionEngine->decide($ltv);         // :33  ← точка вставки между :32 и :33

        return [                                                 // :35
```

## Что менять (план, код сегодня не трогаем)

1. `backend/config/rules.php` — добавить порог, например `'review_mileage_km' => 400000` (число не хардкодим).
2. `AssessmentService::assess()` — после `decide($ltv)`: если `$input['mileage'] > 400000` → решение понижается до `DecisionEngine::REVIEW`.
3. Разобраться с существующим капом 500 000 в валидаторе: пробег 400 001–500 000 должен перестать быть ошибкой валидации и стать `review`; > 500 000 — остаётся ошибкой валидации (или тоже `review`, уточнить по спеке).
4. Тесты: `tests/Unit/ApplicationValidatorTest.php` и `AssessmentServiceTest.php` — добавить кейсы на границу 400 000.
