# L04 · RAGAS Diagnostic Report — PayPilot
Набір із ДЗ_2: https://github.com/chaynikova/paypilot-dz2

## Запуск
Покладіть `golden.jsonl` у `~/paypilot/l03/sets/` (на Windows — `C:\l03\sets\`) і виконайте:

```
docker compose run --rm eval --set golden --profile clean
```

`set_hash`: `8fcf774bf7fc`

## Retrieval-кейси
| id | Джерело | Файл і рядок корпусу |
| --- | --- | --- |
| RET-H01 | скарга C-13 | tariffs.md, ## FX spreads by tier, рядок Tier 3 |
| RET-H02 | скарга C-14 | disputes.md, ## Reason codes and chargeback windows, рядок duplicate_charge |
| RET-H03 | вправа 1 лабораторної | tariffs.md, ## FX spreads by tier, рядок Tier 3 |
| RET-H04 | вправа 2 лабораторної | limits.md, ## Daily and monthly limits by tier, рядок Tier 2 |
| RET-H05 | корпус | tariffs.md, ## Transfer fees by type, рядок SEPA |
| RET-H06 | корпус | tariffs.md, ## Transfer fees by type, рядок Card payment |
| RET-H07 | корпус | limits.md, ## Daily and monthly limits by tier, рядок Tier 1 |
| RET-H08 | корпус | limits.md, ## Daily and monthly limits by tier, рядок Tier 3 |
| RET-H09 | корпус | disputes.md, ## Reason codes and chargeback windows, рядок unauthorized_debit |
| RET-H10 | корпус | disputes.md, ## Reason codes and chargeback windows, рядок service_not_rendered |
| RET-H11 | корпус | transfers.md, ## Cut-off times |
| RET-H12 | корпус | fx-operations.md, ## Allowance accounting |

## Прогони
| Прогін | Профіль | Результат | Звіт |
| --- | --- | --- | --- |
| Baseline | `clean` | <N/M> | `reports/…` |
| Профіль заняття | `lesson-04` | <N/M> | `reports/…` |

Прогноз до прогону `lesson-04` (профіль вмикає D16 — kb_broken, нарізка по 160 символів; D17 — top_k 1; D05 — довідник комісій у промпті без пошуку):
- впадуть RET-H01, RET-H03 (`5,000`) і RET-H10 (`90`) — рядок таблиці розрізано між двома чанками: «EUR 5» у `tariffs.md#b3`, «,000» у `#b4`; рядок service_not_rendered — між `disputes.md#b2` і `#b3`;
- найімовірніше впадуть RET-H02, RET-H04, RET-H07, RET-H08, RET-H09, RET-H11 — потрібне число лежить у чанку цілим, але з top_k 1 приходить один чанк, і за збігом слів першим стає інший («Tier», «limit», «transfer» є в багатьох коротких обрізках): для лімітів — `glossary.md#b5` або `tariffs.md#b3` замість `limits.md#b3`, для вікон — `disputes.md#b1` без чисел, як у C-14; повторні пошуки агента можуть випадково влучити;
- не впадуть RET-H05 (`0.35`) і RET-H06 (`1.2%`) — D05 велить відповідати про комісії з довідника в промпті без пошуку, а SEPA 0.35 і картка 1.2 % там правильні; кейси пройдуть, хоча retrieval-шар не працював;
- невпевнено: RET-H12 (`entire`) — потрібний чанк `fx-operations.md#b5` приходить першим лише на запит зі словами на кшталт «allowance boundary portion»; на інші формулювання — ні.
Коміт із прогнозом: `<hash>`
Що впало: <id>. Розбіжність із прогнозом: <чим пояснюється або «немає»>
