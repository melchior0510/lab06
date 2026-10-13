# Лаборатори №6: Category-Partition ба PICT

**Оюутан:** Munh-Orgil · **Код:** B242270130
**Хичээл:** F.CSA313 — Программ хангамжийн чанарын баталгаа ба тест (2026)

---

## Хэсэг A: Category-Partition гараар — Шилжүүлгийн функц

### Олон алдааны давуу эрэмбэ (oracle-ийн үндэс)

Нэг хүсэлтэд хэд хэдэн алдаа зэрэг байвал **from → to → amount** дарааллаар шалгана гэж тогтоов. Amount-ийн дотор: `BAD_AMOUNT → INSUFFICIENT → LIMIT`. Үндэслэл: тодорхойлолтын алдааны жагсаалт энэ дарааллаар бичигдсэн бөгөөд "эх үүсвэр данс → хүлээн авагч данс → дүн" гэж шалгах нь шилжүүлгийн бодит урсгалд (эхлээд данс байгаа эсэхийг, дараа нь дүнг шалгах) тохирно. Дүнг шалгахдаа эхлээд утга өөрөө хүчинтэй эсэх (>0), дараа нь үлдэгдэл, хамгийн сүүлд өдрийн лимитээр шалгана.

### A1 — Параметр, сонголт, төлөөлөх утгууд

| Параметр | Сонголт (choice) | Төлөөлөх утга | Тайлбар |
|---|---|---|---|
| **from** | Данс байгаа эсэх / төлөв | `Valid` — байгаа, царцаагдаагүй | Үлдэгдэл 1000 |
| | | `Frozen` — байгаа, царцаагдсан | |
| | | `Missing` — системд байхгүй ID | |
| **to** | Данс байгаа эсэх / from-той харьцаа | `Valid` — байгаа, from-оос өөр | |
| | | `Missing` — байхгүй ID | |
| | | `SameAsFrom` — from-той ижил ID | |
| **amount** | Дүн ба үлдэгдэл/лимитийн харьцаа | `Normal` — 0 < дүн < үлдэгдэл, лимитийн дотор | |
| | | `ExactBalance` — дүн = үлдэгдэл яг тэнцүү | Хязгаарын утга ("хэтрэхгүй" гэдэгт тэнцүү зөвшөөрөгдөнө) |
| | | `Zero` — дүн ≤ 0 (0, -5 нэг анги) | |
| | | `OverBalance` — дүн > үлдэгдэл | |
| | | `OverLimit` — дүн ≤ үлдэгдэл боловч өдрийн лимитээс хэтэрсэн | |
| **currency** | Дансны валюттай харьцаа | `Same` — дансны валютаар | |
| | | `Different` — өөр валют → ханшийн хөрвүүлэлт | |

**Далд орчны сонголтууд** (хүсэлтэд харагдахгүй, setup-д бэлдэх зүйлс):

- **Өдрийн лимит хэр дүүрсэн байсан бэ** — `OverLimit`-ийг үүсгэхийн тулд лимитийг үлдэгдлээс бага (эсвэл өдрийн өмнөх шилжүүлгээр дүүрсэн) болгож бэлдэнэ. Ийм учраас лимитийн төлөв нь тусдаа параметр биш, `amount`-ийн `OverLimit` утгын setup болгон шингээв.
- **Ханшийн хүснэгт / дансны валют** — `Different` үед шалгалтыг хөрвүүлсний дараах дүнгээр хийнэ. Тиймээс `OverBalance + Different` хослолд дүн тоон утгаараа үлдэгдлээс бага боловч хөрвүүлсний дараа хэтэрдэг тохиолдол орно.
- **Дансны үлдэгдэл** (1000) ба **өдрийн лимит** (жишээ нь 5000; `OverLimit`-д 300 болгож бууруулна).

### A2 — Хязгаарлалтууд

| Утга | Тэмдэглэгээ | Нэг өгүүлбэр үндэслэл |
|---|---|---|
| from=`Frozen` | `[ERROR]` | Царцаагдсан данснаас шилжүүлэх нь `ERROR_FROZEN` буцаадаг алдааны анги; to/amount алдаатай хослуулбал үүнийг нуух (masking) тул хослуулахгүй. |
| from=`Missing` | `[ERROR]` | `ERROR_NO_FROM` — эхний шалгалтаар зогсдог тул to/amount-ийн утгыг шалгах боломжгүй, зөвхөн хүчинтэй to + Normal-тай хослуулна. |
| to=`Missing` | `[ERROR]` | `ERROR_NO_TO` — from хүчинтэй үед л хүрнэ; amount алдаа нь to-ийн алдаагаар далдлагдана. |
| to=`SameAsFrom` | `[ERROR]` `[IF from=Valid]` | `ERROR_SAME_ACCOUNT`; from Frozen/Missing үед "from-той ижил" гэдэг утга учиргүй (from-ийн алдаа түрүүлнэ) тул from=Valid байх үед л. |
| amount=`Zero` | `[ERROR]` `[IF from=Valid, to=Valid]` | `ERROR_BAD_AMOUNT`; from/to-ийн алдаа түрүүлдэг тул хоёр данс хүчинтэй үед л хүрнэ. |
| amount=`OverBalance` | `[ERROR]` `[IF from=Valid, to=Valid]` | `ERROR_INSUFFICIENT`; дансны шалгалт давсны дараа л хүрнэ. |
| amount=`OverLimit` | `[ERROR]` `[IF from=Valid, to=Valid]` | `ERROR_LIMIT`; хамгийн сүүлийн шалгалт. |
| amount=`ExactBalance` | `[SINGLE]` `[IF currency=Same]` | Хязгаарын утга нэг удаа шалгахад хангалттай (бүх сонголттой нь үржүүлэх шаардлагагүй), хөрвүүлэлтийн нөлөөг холихгүйн тулд `Same` валюттай. |
| currency=`Different` | — | Алдаа биш, `OK` үр дүнтэй; `ExactBalance`-аас бусад бүх хүчинтэй хослолтой хослуулна. |

Хязгаарлалтыг PICT хэлээр бичвэл:

```
IF [From] <> "Valid" THEN [To] = "Valid" AND [Amount] = "Normal";
IF [To]   <> "Valid" THEN [Amount] = "Normal";
IF [Amount] = "ExactBalance" THEN [Currency] = "Same";
```

**"N → M" тооцоо (бүрэн олонлог → хязгаарлалтын дараа):**

Бүрэн олонлог = 3 (from) × 3 (to) × 5 (amount) × 2 (currency) = **90**.

Хязгаарлалтын дараа:

- from=`Valid`, to=`Valid`: amount 5 × currency 2 = 10, гэхдээ `ExactBalance`-ийн `Different` хувилбар хасагдана (SINGLE) → **9**
- from=`Valid`, to=`Missing`: amount=`Normal` × currency 2 = **2**
- from=`Valid`, to=`SameAsFrom`: amount=`Normal` × currency 2 = **2**
- from=`Frozen`, to=`Valid`, amount=`Normal`: currency 2 = **2**
- from=`Missing`, to=`Valid`, amount=`Normal`: currency 2 = **2**

Нийт 9 + 2 + 2 + 2 + 2 = **90 → 17**. (Python-оор бүх 90-г шүүж 17 гэж давхар шалгав.)

### A3 — Спецификацийн хүснэгт

Эхний 3 мөр нь happy path, `[SINGLE]` мөр орсон. Мөр 11–13 давхар нөхцөл.

| # | from | to | amount | currency | Setup / тайлбар | Хүлээгдэх үр дүн |
|---|---|---|---|---|---|---|
| 1 | Valid | Valid | Normal | Same | Үлдэгдэл 1000, дүн 100 | **201** `OK` |
| 2 | Valid | Valid | Normal | Different | Дансны валют MNT, хүсэлт USD, хөрвүүлсний дараа үлдэгдэл, лимитийн дотор | **201** `OK` |
| 3 | Valid | Valid | ExactBalance | Same | Дүн = 1000 = үлдэгдэл, лимит ≥ 1000 `[SINGLE]` | **201** `OK` |
| 4 | Frozen | Valid | Normal | Same | from данс царцаасан | **200** `ERROR_FROZEN` |
| 5 | Missing | Valid | Normal | Same | from ID системд байхгүй | **200** `ERROR_NO_FROM` |
| 6 | Valid | Missing | Normal | Same | to ID байхгүй | **200** `ERROR_NO_TO` |
| 7 | Valid | SameAsFrom | Normal | Same | to == from | **200** `ERROR_SAME_ACCOUNT` |
| 8 | Valid | Valid | Zero | Same | amount = 0 (ба -5 ижил анги) | **200** `ERROR_BAD_AMOUNT` |
| 9 | Valid | Valid | OverBalance | Same | Дүн 1500 > үлдэгдэл 1000 | **200** `ERROR_INSUFFICIENT` |
| 10 | Valid | Valid | OverLimit | Same | Өдрийн лимит 300, дүн 500 ≤ үлдэгдэл | **200** `ERROR_LIMIT` |
| 11 | Valid | Valid | OverBalance | Different | **Давхар нөхцөл:** тоон дүн 800 < үлдэгдэл 1000, харин хөрвүүлсний дараа 1200 > үлдэгдэл | **200** `ERROR_INSUFFICIENT` |
| 12 | Frozen | Missing | Normal | Same | **Давхар алдаа:** from царцаасан + to байхгүй. Давуу эрэмбээр from түрүүлнэ | **200** `ERROR_FROZEN` |
| 13 | Valid | SameAsFrom | Zero | Same | **Давхар алдаа:** to==from + дүн 0. to нь amount-аас түрүүлнэ | **200** `ERROR_SAME_ACCOUNT` |
| 14 | Valid | Valid | OverBalance+OverLimit | Same | **Давхар алдаа:** дүн үлдэгдэл ба лимит хоёулангаас хэтэрсэн. INSUFFICIENT нь LIMIT-ээс түрүүлнэ | **200** `ERROR_INSUFFICIENT` |
