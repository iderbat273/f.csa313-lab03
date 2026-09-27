# F.CSA313 — Лаборатори №3

## Чанарын сценарио → SLO → k6 threshold

**Оюутны нэр:** Б.Идэрбат
**Оюутны код:** B242270075
**Хичээл:** F.CSA313
**Үйлдлийн систем:** Debian GNU/Linux 13.7

## 1. Орчны мэдээлэл

* Node.js: v20.19.2
* npm: 9.2.0
* Grafana k6: v2.2.0
* Git: 2.47.3
* Express: 5.2.1
* API: `http://localhost:3000`

## 2. Чанарын сценарионууд

### 2.1 Performance scenario — Cart

| Хэсэг              | Тодорхойлолт                                                      |
| ------------------ | ----------------------------------------------------------------- |
| Тойм               | Сагсанд бараа нэмэх хүсэлт хэвийн ачаалалд хурдан хариулах ёстой. |
| Системийн төлөв    | Express API ажиллаж, `POST /cart/add` хүсэлт хүлээн авч байна.    |
| Орчны төлөв        | Debian локал машин, localhost:3000, 20 VU, 2 минут.               |
| Гадаад өдөөлт      | 20 виртуал хэрэглэгч сагсанд бараа нэмэх хүсэлт илгээнэ.          |
| Шаардлагатай хариу | HTTP 200 болон `ok=true` JSON хариу буцаана.                      |
| Хэмжүүр            | `/cart/add` хүсэлтийн p95 хариу хугацаа 200 мс-ээс бага байна.    |

### 2.2 Reliability scenario — Payment

| Хэсэг              | Тодорхойлолт                                                       |
| ------------------ | ------------------------------------------------------------------ |
| Тойм               | Төлбөрийн хүсэлтүүдийг найдвартай боловсруулах.                    |
| Системийн төлөв    | Express API ажиллаж, `POST /pay` endpoint хүсэлт хүлээн авч байна. |
| Орчны төлөв        | Debian локал машин, 20 VU, 2 минут.                                |
| Гадаад өдөөлт      | Виртуал хэрэглэгчид төлбөрийн хүсэлт илгээнэ.                      |
| Шаардлагатай хариу | Амжилттай төлбөрт HTTP 200 болон `paid=true` буцаана.              |
| Хэмжүүр            | `/pay` endpoint-ийн алдааны хувь 8%-аас бага байна.                |

### 2.3 Availability scenario — Server crash

| Хэсэг              | Тодорхойлолт                                                                     |
| ------------------ | -------------------------------------------------------------------------------- |
| Тойм               | Сервер түр зогсож, дахин асах үед үйлчилгээний хүртээмжийг шалгах.               |
| Системийн төлөв    | API эхэндээ ажиллаж байна. Тестийн явцад серверийг 10 секунд зогсооно.           |
| Орчны төлөв        | Debian локал машин, 20 VU, 2 минут.                                              |
| Гадаад өдөөлт      | Тестийн 60 дахь секунд орчимд серверийг зогсоож, 10 секундын дараа дахин асаана. |
| Шаардлагатай хариу | Сервер дахин ассаны дараа хүсэлтүүд дахин амжилттай боловсруулагдана.            |
| Хэмжүүр            | `checks` success rate 90%-аас их байна.                                          |

### 2.4 Additional performance scenario — Report

| Хэсэг              | Тодорхойлолт                                                            |
| ------------------ | ----------------------------------------------------------------------- |
| Тойм               | Тайлангийн endpoint ачааллын үед зөвшөөрөгдөх хугацаанд хариулах ёстой. |
| Системийн төлөв    | Express API ажиллаж, `GET /report` хүсэлт хүлээн авч байна.             |
| Орчны төлөв        | Debian локал машин, 20 VU, 2 минут.                                     |
| Гадаад өдөөлт      | Виртуал хэрэглэгчид тайлангийн хүсэлт илгээнэ.                          |
| Шаардлагатай хариу | HTTP 200 болон `rows=20000` JSON хариу буцаана.                         |
| Хэмжүүр            | `/report` хүсэлтийн p95 хариу хугацаа 450 мс-ээс бага байна.            |

## 3. Сценариогийн 5 шинж

1. **Итгэмжтэй** — локал Express API-ийн бодит зан төлөвт тулгуурласан.
2. **Үнэ цэнэтэй** — гүйцэтгэл, найдвартай байдал, хүртээмжийг шалгана.
3. **Тодорхой** — endpoint, VU, хугацаа болон өдөөлтийг тодорхойлсон.
4. **Нарийн** — p95 latency болон failure/success rate хэмжүүрүүд ашигласан.
5. **Ойлгомжтой** — системийн төлөв, өдөөлт, шаардлагатай хариу болон хэмжүүрийг дарааллаар тодорхойлсон.

## 4. SLO ба k6 thresholds

| Сценарио              | SLI                     | k6 threshold | Нөхцөл                       |
| --------------------- | ----------------------- | ------------ | ---------------------------- |
| Performance — Cart    | `/cart/add` latency p95 | `< 200 ms`   | 20 VU, 2 минут               |
| Reliability — Payment | `/pay` error rate       | `< 8%`       | 20 VU, 2 минут               |
| Availability          | checks success rate     | `> 90%`      | 20 VU, 2 минут, 10 сек crash |
| Performance — Report  | `/report` latency p95   | `< 450 ms`   | 20 VU, 2 минут               |

### Босго сонгосон үндэслэл

* **Cart:** `/cart/add` нь сервер талд нэмэлт delayгүй тул p95 `<200 ms` гэж сонгосон.
* **Payment:** `/pay` endpoint нь зориудаар ойролцоогоор 5% санамсаргүй алдаатай тул `<8%` threshold нь хэвийн variation-ийг зөвшөөрөх боловч алдаа ихсэх үед илрүүлэх зорилготой.
* **Availability:** `>90%` checks success rate нь нийт шалгалтын хамгийн ихдээ 10%-ийг амжилтгүй болохыг зөвшөөрнө.
* **Report:** `/report` endpoint 200–400 ms-ийн санамсаргүй delay-тэй тул p95 `<450 ms` threshold сонгосон.

## 5. PASS тестийн үр дүн

PASS тестийг дараах командаар ажиллуулсан:

```bash
k6 run slo-test.js 2>&1 | tee results/pass.txt
```

Үндсэн үр дүн:

| Metric         |    Үр дүн | Threshold | Үр дүн |
| -------------- | --------: | --------: | ------ |
| Checks         |    98.17% |      >90% | PASS   |
| Cart p95       |   1.89 ms |   <200 ms | PASS   |
| Report p95     | 389.59 ms |   <450 ms | PASS   |
| Pay error rate |     5.48% |       <8% | PASS   |

PASS тестийн бүрэн output:

`results/pass.txt`

Холбогдох screenshot:

`screenshots/06-pass.png`

## 6. Chaos тестийн үр дүн

Chaos тестийн үед 2 минутын тестийн явцад серверийг 10 секунд зогсоож, дараа нь дахин асаасан.

Сервер зогссон үед k6-ийн хүсэлтүүд `connection refused` алдаа авч, checks болон payment error-rate thresholds зөрчигдсөн.

Үндсэн үр дүн:

| Metric                     |    Үр дүн | Threshold | Үр дүн |
| -------------------------- | --------: | --------: | ------ |
| Checks success rate        |    87.70% |      >90% | FAIL   |
| Cart p95                   |   1.68 ms |   <200 ms | PASS   |
| Report p95                 | 389.48 ms |   <450 ms | PASS   |
| Pay error rate             |    15.31% |       <8% | FAIL   |
| Total request failure rate |    12.29% |         — | —      |

Chaos тестийн нийт:

* Нийт checks: 5679
* Амжилттай checks: 4981
* Амжилтгүй checks: 698
* Нийт хүсэлт: 5679
* Нийт failed requests: 698

Request-based availability:

```text
Availability = 1 - failed requests / total requests
             = 1 - 698 / 5679
             = 87.71%
```

Иймээс chaos нөхцөлд request-based availability ойролцоогоор **87.71%** байсан.

Chaos тестийн бүрэн output:

`results/chaos.txt`

Холбогдох screenshot:

`screenshots/07-chaos.png`
## 7. Error Budget

### Request-based Error Budget

Availability SLO нь 90% тул нийт request/check-ийн 10%-ийг failure budget гэж үзнэ.

Chaos test-ийн нийт request/check:

5679

Зөвшөөрөгдөх failure:

5679 × 10% = 567.9 ≈ 568 failures

Бодит failure:

698 failures

Иймээс:

- Allowed failures: ≈568
- Actual failures: 698
- Excess failures: 698 - 568 = 130
- Actual failure rate: 12.29%
- Request-based availability: 87.71%

Request-based тооцоогоор availability SLO хангагдаагүй.

### Time-based Error Budget

2 минутын тестийн нийт хугацаа:

120 секунд

Availability SLO = 90% тул зөвшөөрөгдөх downtime:

120 × 10% = 12 секунд

Chaos test-ийн үед серверийг 10 секунд зориуд зогсоосон.

- Total test time: 120 секунд
- Allowed downtime: 12 секунд
- Actual downtime: 10 секунд
- Remaining time budget: 2 секунд
- Time-based availability:

(120 - 10) / 120 × 100 = 91.67%

Time-based тооцоогоор 90%-ийн availability SLO хангагдсан байна.

### Comparison

| Арга | Нийт хэмжээ | Allowed budget | Actual | Availability |
|---|---:|---:|---:|---:|
| Request-based | 5679 requests/checks | ≈568 failures | 698 failures | 87.71% |
| Time-based | 120 секунд | 12 sec downtime | 10 sec | 91.67% |

Эдгээр хоёр арга өөр үр дүн өгч байна. Time-based арга нь зөвхөн серверийн downtime-ийг хэмждэг бол request-based арга нь тухайн хугацаанд амжилтгүй болсон request/check-үүдийг тооцдог. Chaos test-ийн үед сервер зогссоны улмаас connection refused болсон олон request/check үүссэн тул request-based availability нь time-based availability-ээс бага гарсан.

## 8. Deliberate FAIL тест

Threshold-ийн failure behavior-ийг шалгахын тулд тусдаа `fail-test.js` файл ашигласан.

Энэ тестэд зориудаар:

```text
http_req_duration{name:report}: p(95)<100ms
```

гэсэн бодит нөхцөлд биелэх боломжгүй хатуу threshold тавьсан.

Тестийн үр дүн:

```text
p(95)=388.36ms
```

байсан бөгөөд:

```text
p(95)<100ms
```

threshold зөрчигдсөн.

k6-ийн exit code:

```text
K6_EXIT_CODE=99
```

гарсан нь threshold failure үед k6 non-zero exit code буцааж байгааг харуулсан.

Deliberate FAIL тестийн бүрэн output:

`results/fail.txt`

Холбогдох screenshot:

`screenshots/08-fail.png`

## 9. Тестийн нотолгоо

Лабораторийн явцын screenshot-ууд:

1. `screenshots/03-server-running.png` — Express server ажиллаж байгаа байдал
2. `screenshots/04-api-test.png` — API endpoint-үүдийн curl тест
3. `screenshots/05-scenarios-slo.png` — Scenario болон SLO хүснэгт
4. `screenshots/06-pass.png` — PASS тестийн threshold үр дүн
5. `screenshots/07-chaos.png` — Chaos тестийн threshold failure
6. `screenshots/08-fail.png` — Deliberate FAIL болон non-zero exit code

Бүрэн k6 output-ууд:

* `results/pass.txt`
* `results/chaos.txt`
* `results/fail.txt`

## 10. Дүгнэлт

Энэ лабораториор чанарын шаардлагыг чанарын сценарио хэлбэрээр тодорхойлж, тэдгээрийг SLO болон k6 threshold-тэй холбож туршсан. PASS тестийн үед cart болон report endpoint-ийн latency threshold-үүд, payment-ийн error-rate болон нийт checks threshold бүгд хангагдсан. Cart endpoint-ийн p95 latency 1.89 ms байсан нь 200 ms-ийн босгоос бага байв. Report endpoint-ийн p95 latency 389.59 ms байсан нь 450 ms-ийн босгыг хангаж байв. Payment endpoint-ийн failure rate 5.48% байсан нь 8%-ийн босгоос бага байв. Chaos тестийн үед серверийг 10 секунд зогсоосноор connection refused алдаанууд үүсэж, checks success rate 87.70% болсон. Энэ үед payment error rate мөн 15.31% болж, 8%-ийн SLO-г хангаагүй. Request-based availability 87.71% байсан бөгөөд 90%-ийн SLO-оос доогуур гарсан. Deliberate FAIL тестээр зориуд хатуу threshold тавихад k6 threshold failure илрүүлж, 99 гэсэн non-zero exit code буцаасан. Иймээс k6 threshold нь системийн хэвийн ажиллагаа, fault нөхцөл болон зориудын failure-ийг ялган илрүүлэхэд ашиглагдаж болохыг туршилтаар харуулсан.
