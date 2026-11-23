# L04 · RAGAS Diagnostic Report — PayPilot
Набір із ДЗ_2: https://github.com/chaynikova/paypilot-dz2

## Запуск
Покладіть `golden.jsonl` у `~/paypilot/l03/sets/` (на Windows — `C:\l03\sets\`) і виконайте:

```
docker compose run --rm eval --set golden --profile clean
```

`set_hash`: `8d746efe4559` (версія 2 набору; версія 1 — `8fcf774bf7fc`)

## Retrieval-кейси
| id | Джерело | Файл і рядок корпусу |
| --- | --- | --- |
| RET-H01 | скарга C-13 | tariffs.md, ## FX spreads by tier, рядок Tier 3 |
| RET-H02 | скарга C-14 | disputes.md, ## Reason codes and chargeback windows, рядок duplicate_charge |
| RET-H03 | вправа 1 лабораторної | tariffs.md, ## FX spreads by tier, рядок Tier 3 |
| RET-H04 | вправа 2 лабораторної | limits.md, ## Daily and monthly limits by tier, рядок Tier 2 |
| RET-H05 | корпус | limits-operations.md, ## Tier changes |
| RET-H06 | корпус | complaints.md, ## Acknowledgement and resolution |
| RET-H07 | корпус | limits.md, ## Daily and monthly limits by tier, рядок Tier 1 |
| RET-H08 | корпус | limits.md, ## Daily and monthly limits by tier, рядок Tier 3 |
| RET-H09 | корпус | cards.md, ## Replacement |
| RET-H10 | корпус | disputes.md, ## Reason codes and chargeback windows, рядок service_not_rendered |
| RET-H11 | корпус | transfers.md, ## Cut-off times |
| RET-H12 | корпус | fx-operations.md, ## Allowance accounting |

## Прогони
| Прогін | Профіль | Результат | Звіт |
| --- | --- | --- | --- |
| Baseline | `clean` | 41/41, код 0 (action 3/3, generation 26/26, retrieval 12/12) | `reports/golden-clean-20261122T131545.json` |
| Профіль заняття | `lesson-04` | 28/41, код 1 (action 3/3, generation 23/26, retrieval 2/12) | `reports/golden-lesson-04-20261122T132058.json` |

Прогноз до прогону `lesson-04` (профіль вмикає D16 — kb_broken, нарізка по 160 символів; D17 — top_k 1; D05 — довідник комісій у промпті без пошуку):
- впадуть RET-H01, RET-H03 (`5,000`) і RET-H10 (`90`) — рядок таблиці розрізано між двома чанками: «EUR 5» у `tariffs.md#b3`, «,000» у `#b4`; рядок service_not_rendered — між `disputes.md#b2` і `#b3`;
- найімовірніше впадуть RET-H02, RET-H04, RET-H07, RET-H08, RET-H09, RET-H11 — потрібне число лежить у чанку цілим, але з top_k 1 приходить один чанк, і за збігом слів першим стає інший («Tier», «limit», «transfer» є в багатьох коротких обрізках): для лімітів — `glossary.md#b5` або `tariffs.md#b3` замість `limits.md#b3`, для вікон — `disputes.md#b1` без чисел, як у C-14; повторні пошуки агента можуть випадково влучити;
- не впадуть RET-H05 (`0.35`) і RET-H06 (`1.2%`) — D05 велить відповідати про комісії з довідника в промпті без пошуку, а SEPA 0.35 і картка 1.2 % там правильні; кейси пройдуть, хоча retrieval-шар не працював;
- невпевнено: RET-H12 (`entire`) — потрібний чанк `fx-operations.md#b5` приходить першим лише на запит зі словами на кшталт «allowance boundary portion»; на інші формулювання — ні.
Коміт із прогнозом: `3f4c0b2` («Prediction before lesson-04 run»)
Що впало (retrieval): RET-H01, RET-H02, RET-H03, RET-H04, RET-H07, RET-H08, RET-H09, RET-H10, RET-H11, RET-H12 — 10 з 12. Пройшли RET-H05 і RET-H06.

Розбіжність із прогнозом: немає. Усі шість кейсів «найімовірніше впадуть» упали, і з тієї причини, яку ми назвали: число лежить у чанку цілим, але з top_k 1 першим приходить інший чанк — для лімітів `tariffs.md#b3` (RET-H04, RET-H07) і `tariffs.md#b1` (RET-H08), для вікон `disputes.md#b1` (RET-H02, RET-H09) і `disputes-process.md#b1` (RET-H10), для відсічки SWIFT — `tariffs.md#b4` з таблицею комісій (RET-H11). Повторні пошуки агента не допомогли: він двічі отримував той самий чанк. Невпевнений RET-H12 упав: першим прийшов `tariffs.md#b2` без правила «entire». RET-H05 і RET-H06 пройшли без жодного виклику пошуку (`tool: none called`): D05 підставив відповідь із довідника в промпті.

Вартість: retrieval-кейси на lesson-04 — 86 654 токени проти 78 786 на clean (+10 %), хоча два з них пройшли зовсім без пошуку; весь набір — 338 924 проти 320 200 (+6 %). Зламаний індекс робить прогін дорожчим: агент перепитує пошук.

Що це означає для набору:
- RET-H05 і RET-H06 на lesson-04 зелені, хоча retrieval-шар не працював: рівень 3 перевіряє лише текст відповіді, а не те, звідки він узявся. Щоб кейс ловив саме retrieval, йому потрібна ще перевірка, що в трейсі є `search_knowledge_base` з потрібним фрагментом, — це рівень перевірки інструментів (L07).
- RET-H04 на clean пройшов із неповною відповіддю (ліміт є, спреду «not found»): кейс перевіряє одну підрядкову половину двочастинної відповіді.
- Поза retrieval-шаром на lesson-04 впали CMP-004 (той самий обірваний `disputes.md#b1`), CMP-001 (D05: «EUR 15.00, no percentage fee») і FX-003-S. FX-003-S — хибне падіння: агент правильно написав «0.90%», а кейс шукає підрядок «0.9%». Це крихкість перевірки рівня 3, а не дефект агента; кейс варто перевести на числове порівняння.

## Версія 2 набору (після рецензії ментора)
За побажанням ментора кейси RET-H05, RET-H06 і RET-H09, які повторювали факти заняття (комісія SEPA, комісія за картку, вікно unauthorized_debit), замінено на вільні факти корпусу, яких на занятті не було:
- RET-H05 — коли набирає чинності зміна тіру: «first calendar day» (`limits-operations.md`, ## Tier changes);
- RET-H06 — строк розгляду скарги: «15 business days» (`complaints.md`, ## Acknowledgement and resolution);
- RET-H09 — строк заміни втраченої картки: «five business days» (`cards.md`, ## Replacement).

Решта 43 кейси не змінювались. Прогони версії 1 (`8fcf774bf7fc`) лишаються в `reports/` як історія.

Прогноз для трьох нових кейсів до прогону `lesson-04` (на kb_broken кожен факт лежить у чанку цілим — `limits-operations.md#b5`, `complaints.md#b4`, `cards.md#b4`, але з top_k 1 приходить один чанк, і за збігом слів першим часто стає інший):
- RET-H05 найімовірніше впаде: на запити зі словами «tier change» першим приходить `glossary.md#b5` (визначення «Tier»), а не `limits-operations.md#b5`;
- RET-H06 найімовірніше впаде: першим приходить `complaints.md#b3` або `#b1` без строку, а «15 business days» лежить у `#b4`;
- RET-H09 — невпевнено: на запит «card replacement days» першим приходить потрібний `cards.md#b4`, на «lost stolen card» — `security-policy.md#b2`;
- на clean усі три пройдуть: kb_clean віддає секції цілими, top_k 4.

Коміт із прогнозом версії 2: `6fdd954` («Set v2: replace RET-H05/H06/H09, prediction, section 6 fix»)

| Прогін | Профіль | Результат | Звіт |
| --- | --- | --- | --- |
| Baseline v2 | `clean` | 40/41, код 1 (retrieval 11/12; упав RET-H06) | `reports/golden-clean-20261123T104524.json` |
| Профіль заняття v2 | `lesson-04` | 29/41, код 1 (action 3/3, generation 25/26, retrieval 1/12) | `reports/golden-lesson-04-20261123T104953.json` |

Результат версії 2 проти прогнозу:
- RET-H05 упав на lesson-04, як і прогнозували, але першим приходив не `glossary.md#b5`, а `tariffs.md#b1` (огляд тірів) — двічі, на обидва формулювання агента; механізм той самий: з top_k 1 приходить один чанк, і за збігом слів «tier» виграє інший.
- RET-H06 упав на lesson-04, як прогнозували: агент тричі шукав, двічі отримав `complaints.md#b3`, обірваний перед «15 business days», і чесно сказав, що строк розгляду обрізано. **Розбіжність:** RET-H06 упав і на clean, хоча ми прогнозували зелений. На kb_clean агент шукав «how long does Verta take to resolve a complaint» і «complaint process timeframe response deadline» — і обидва рази секція `complaints.md#s3` («…resolved within 15 business days») не потрапила в четвірку. Причина в пошуку стенду: він лексичний і порівнює слова як є, без зведення до основи, тож «resolve» у запиті не збігається з «resolved» у тексті. Це не дефект профілю і не помилка кейса: факт є в корпусі дослівно, а система його не знаходить. Кейс лишаємо як справжню знахідку — він ловить клас збою пошуку, якого немає серед D16 і D17; див. `ragas-diagnostic-report.md`, розділ 6, залишковий ризик.
- RET-H09 (невпевнено) упав: на обидва формулювання агента першими прийшли `security-policy.md#b2` і `#b3` (блокування картки), а не `cards.md#b4`.

Нестабільність між прогонами версій 1 і 2 на lesson-04: RET-H07 цього разу пройшов (третій пошук агента влучив у `limits.md#b3`), CMP-004 теж пройшов (другий пошук приніс `disputes.md#b2` із «60»), FX-003-S пройшов («0.9%»). Це той самий імовірнісний характер, про який говорить правило п'яти прогонів: при top_k 1 результат залежить від того, скільки разів і якими словами агент перепитає пошук.
