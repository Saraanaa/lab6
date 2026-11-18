# Лаборатори 6 — Тест сонголтын аргууд

**Оюутны нэр:** `[НЭРЭЭ ОРУУЛНА УУ]`  
**Оюутны код:** `[КОДОО ОРУУЛНА УУ]`

Энэ лаборатори нь код хэрэгжүүлэх биш, банкны шилжүүлгийн функцийн тестийн дизайн хийх ажил юм. PICT-ийн эх кодыг лабын репогоос гадуур build хийсэн бөгөөд build-ийн нотолгоо [results/pict-build.txt](results/pict-build.txt)-д байна.

## А. Category-partition гараар

API: `POST /transfer { "from": ID, "to": ID, "amount": NUM, "currency": CODE }`.

Олон алдаа зэрэг тохиолдвол энэ тайланд `from → to → amount` гэсэн давуу эрэмбэ хэрэглэв. Ингэснээр эхний алдаа дараагийн алдааг дарна.

### A1–A2. Сонголт, төлөөлөх утга ба хязгаарлалт

| Параметр/орчин | Сонголт ба төлөөлөх утга | Тэмдэглэгээ | Үндэслэл |
|---|---|---|---|
| from | Valid `A-1001`; Frozen `A-1002`; Missing `A-9999` | Frozen/Missing `[ERROR]`; Valid `[SINGLE]` | Байхгүй болон царцаасан эх данс тусдаа алдаа өгнө. |
| to | Valid `B-2001`; Missing `B-9999`; SameAsFrom `A-1001` | Missing/SameAsFrom `[ERROR]`; Valid `[SINGLE]` | Байхгүй эсвэл эхтэй ижил хүлээн авагчийг ялгана. |
| amount | Normal `100`; Zero `0`; OverBalance `1500`; OverLimit `6000` | Буруу гурвынх `[ERROR]`; Normal `[SINGLE]` | Эерэг, үлдэгдэл, өдрийн лимитийн гурван өөр нөхцөл. |
| currency | Same `MNT`; Different `USD` | `[IF]` | Өөр валют бол хөрвүүлэлт хийнэ, өөрөө алдаа биш. |
| далд орчин | Balance `1000 MNT`; daily used `0`; daily limit `5000` | `[IF/property]` | OverBalance ба OverLimit-ийг тусгаарлахын тулд орчин хэрэгтэй. |
| ханш | `1 USD = 3500 MNT`, rate available | `[IF/property]` | Шалгалтыг хөрвүүлсний дараах дүнгээр хийнэ. |

Хязгаарлалт:

```text
IF from = Missing THEN amount = Normal
IF to = Missing THEN amount = Normal
```

Эдгээр IF нь Missing алдааг amount-ийн алдаатай хольж error masking үүсгэхгүй. `Valid/Valid/Normal` нь happy path-ийн нэг representative тул `[SINGLE]`; Frozen, Missing болон amount-ийн буруу утгууд нь тусдаа `[ERROR]` representative юм.

Хязгаарлалтгүй олонлог `3 × 3 × 4 × 2 = 72`. IF-үүдийн дараа `6 + 6 - 2 + 32 = 42`, тиймээс category-partition-ийн тооцоо **72 → 42**.

### A3. Спецификацийн хүснэгт

| № | from | to | amount | currency | Хүлээгдэх үр дүн |
|---:|---|---|---:|---|---|
| 1 | Valid `A-1001` | Valid `B-2001` | 100 | MNT | `201 OK` |
| 2 | Frozen `A-1002` | Valid `B-2001` | 100 | MNT | `200 ERROR_FROZEN` |
| 3 | Missing `A-9999` | Valid `B-2001` | 100 | MNT | `200 ERROR_NO_FROM` |
| 4 | Valid `A-1001` | Missing `B-9999` | 100 | MNT | `200 ERROR_NO_TO` |
| 5 | Valid `A-1001` | SameAsFrom `A-1001` | 100 | MNT | `200 ERROR_SAME_ACCOUNT` |
| 6 | Valid `A-1001` | Valid `B-2001` | 0 | MNT | `200 ERROR_BAD_AMOUNT` |
| 7 | Valid `A-1001` | Valid `B-2001` | 1500 | MNT | `200 ERROR_INSUFFICIENT` |
| 8 | Valid `A-1001` | Valid `B-2001` | 6000 | MNT | `200 ERROR_LIMIT` |
| 9 | Valid `A-1001` | Valid `B-2001` | 100 | USD | `201 OK` after conversion |
| 10 | Missing | Missing | 0 | MNT | `200 ERROR_NO_FROM` — from давуу |
| 11 | Valid | Missing | 0 | MNT | `200 ERROR_NO_TO` — to давуу |
| 12 | Frozen | SameAsFrom | 0 | MNT | `200 ERROR_FROZEN` — from давуу |
| 13 | Valid | Valid | 100 | USD | `201 OK`, conversion happy path |
| 14 | Valid | Valid | 1500 | USD | `200 ERROR_INSUFFICIENT` after conversion |

## B. PICT pairwise covering array

### B1. Browser

[browser.model](browser.model) нь зургаан параметртэй лекцийн загвар. `/r` ашиглаагүй.

| Хамрах хүрээ | Үр дүн | Мөрийн тоо (толгойг хассан) |
|---|---|---:|
| 2-way | [results/browser-2way.txt](results/browser-2way.txt) | 10 |
| 3-way | [results/browser-3way.txt](results/browser-3way.txt) | 20 |

Бүрэн олонлог `3×3×2×2×2×2 = 144`. 2-way нь 10 мөр бөгөөд лекцийн гараар хийсэн 9 мөрөөс олон. PICT greedy алгоритм pair бүрийг хамруулдаг боловч глобал хамгийн цөөн мөрийг заавал олдоггүй. 3-way нь 20 мөр болж 144-өөс эрс бага байна.

### B2. Transfer моделиуд

[transfer-nc.model](transfer-nc.model) нь хязгаарлалтгүй, [transfer.model](transfer.model) нь IF-тэй, [transfer-neg.model](transfer-neg.model) нь ERROR утгуудыг `~` negative-ээр тэмдэглэсэн.

| Модель | Мөр | Happy path мөр | Тайлбар |
|---|---:|---:|---|
| transfer-nc | 13 | 1 | Энгийн утга тул олон алдааг нэг мөрт хослуулна |
| transfer | 12 | 0 | IF үйлчилсэн; энэ run-д happy path сонгогдоогүй |
| transfer-neg | 16 | 2 | Нэг мөрт хамгийн ихдээ нэг negative утга байна |

`transfer.model` happy path-ийг хориглоогүй ч pairwise алгоритм заавал сонгохгүй. `~` нь error masking-ийг бууруулдаг боловч Missing үед Normal amount байх ёстой гэх бизнесийн IF-ийг орлохгүй. Тиймээс A2-ын хязгаарлалт шаардлагатай хэвээр.

### Бодит тест кейсүүд

| Кейс | Setup | Transfer body | Oracle |
|---|---|---|---|
| T1 happy | `A-1001` active, 10,000 MNT; `B-2001` active; limit 50,000 | `{from:A-1001,to:B-2001,amount:100,currency:MNT}` | HTTP `201`, `result=OK` |
| T2 negative | `A-1001` active, 1,000 MNT; `B-2001` active | `{from:A-1001,to:B-2001,amount:0,currency:MNT}` | HTTP `200`, `ERROR_BAD_AMOUNT` |
| T3 negative | `A-1002` frozen; `B-2001` active | `{from:A-1002,to:B-2001,amount:100,currency:MNT}` | HTTP `200`, `ERROR_FROZEN` |
| T4 negative | `A-1001` active, 1,000 MNT; `B-2001` active | `{from:A-1001,to:B-2001,amount:1500,currency:MNT}` | HTTP `200`, `ERROR_INSUFFICIENT` |

T2 нь transfer-neg-ийн нэг negative мөрийн бодит тест. Олон алдаатай бол `from → to → amount` oracle эрэмбэ хэрэглэнэ.

## B3. Дүгнэлт

Category-partition нь бизнесийн утга, алдааны семантик, oracle-ийг хүн ойлгомжтой тодорхойлоход сайн. Гараар хийсэн хүснэгтэд бүх алдааны анги болон давхар алдааны эрэмбийг санаатайгаар оруулсан. PICT нь pairwise болон 3-way хослолын хамралтыг системтэй баталгаажуулдаг. Гэхдээ PICT greedy алгоритм хамгийн цөөн мөрийг олох баталгаа өгдөггүй. Мөн happy path зэрэг тусгай мөрийг заавал сонгохгүй. Хязгаарлалтгүй модель олон алдааг нэг мөрт хольж error masking үүсгэж болно. `~` negative утга энэ асуудлыг багасгасан ч бизнесийн IF хязгаарлалтыг орлохгүй. Хамгийн их бодол шаардсан хэсэг нь олон алдааны oracle-ийн давуу эрэмбэ болон валют хөрвүүлэлтийг тодорхойлох байв. Иймээс category-partition нь утга ба oracle-ийг, PICT нь хосын хамралтыг тус тус сайн хангана.

## Файлууд

- [browser.model](browser.model)
- [transfer-nc.model](transfer-nc.model)
- [transfer.model](transfer.model)
- [transfer-neg.model](transfer-neg.model)
- [results/](results)
