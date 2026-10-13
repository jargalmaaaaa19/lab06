Лаборатори 6: Тест тохиолдол боловсруулах — Category-Partition ба PICT

Оюутны код: B232270005
Оюутны нэр: З.Жаргалмаа

from → to → amount, амжилтын дотор BAD_AMOUNT → INSUFFICIENT → LIMIT. Ингэснээр бүтэн эрэмбэ нь:

NO_FROM → FROZEN → NO_TO → SAME_ACCOUNT → BAD_AMOUNT → INSUFFICIENT → LIMIT

Үндэслэл: тодорхойлолтод алдааны кодууд яг энэ дарааллаар жагсаагдсан бөгөөд "эхлээд оролтын объектууд (данс) байгаа эсэхийг, дараа нь дүнг шалгадаг" гэсэн логик утгатай.

### Даалгавар A1 — Сонголт ба төлөөлөх утгууд

| Параметр | Сонголт ба төлөөлөх утгууд | Тайлбар |
| :--- | :--- | :--- |
| **from** (данс) | Valid, Frozen, Missing | Байгаа / царцсан / байхгүй |
| **to** (данс) | Valid, Missing, SameAsFrom | Байгаа / байхгүй / from-той ижил |
| **amount** (дүн) | Normal, ExactBalance, Zero, OverBalance, OverLimit | `ExactBalance` нь хилийн утга: үлдэгдэлтэй яг тэнцүү |
| **currency** (валют) | Same, Different | Данснаас ижил / өөр валют |

### Далд орчны сонголтууд ба тооцооллын тайлбар

Тест дизайн хийхэд дараах далд орчны (environmental) сонголтууд нөлөөлөх бөгөөд үүнийг тест кейс бүрийн setup хэсэгт тодорхой зааж өгнө (жишээ нь: “from данс: 1000 USD, өдрийн лимит: 5000 USD, өнөөдөр зарцуулсан: 0 USD”):

- **Дансны үлдэгдэл:** OverBalance, ExactBalance зэрэг нөхцөлийг тодорхойлоход дансны одоогийн үлдэгдлийн хэмжээ орчин болж оролцоно.
- **Өдрийн лимит хэр дүүрсэн байдал:** Лимитийн төлөв нь хоосон (өмнө нь гүйлгээ хийгээгүй) эсвэл дүүрэх дөхсөн (өдрийн лимитийн дийлэнх хэсгийг ашигласан) байж болно. Иймд жижиг хэмжээний amount хүртэл нийлбэр дүнгээрээ өдрийн лимитээс хэтрэх боломжтойг анхаарна.
- **Валютын ханш:** Different буюу өөр валюттай үед ханшийн хөрвүүлэлт хийгдэх тул ханшийн орчин шаардлагатай бөгөөд үлдэгдэл болон лимитийг хөрвүүлсний дараах дүнгээр шалгана.

> *Тайлбар:* Дээрх далд орчны тоон утга, хэмжигдэхүүнүүд нь тест кейсийн бодит setup-ыг бүрдүүлэх боловч Category-Partition-ий нийт болон хязгаарлалтын дараах бүрэн олонлогийн тооцооллын үржвэрт шууд тусдаа параметр болж тооцогдохгүй (эс бөгөөс боломжит хувилбарын тоо хэтэрч олон болж өсөх тул үндсэн логик параметрээр хязгаарлана).


### Даалгавар A2 — Хязгаарлалтууд ба тэмдэглэгээ

| Параметр / Утга | Тэмдэглэгээ | Үндэслэл |
| :--- | :--- | :--- |
| **from = Frozen** | `[ERROR]` | ERROR_FROZEN буцна, үлдсэн шалгалт хийгдэхгүй. |
| **from = Missing** | `[ERROR]` | ERROR_NO_FROM буцна, бусад утга ач холбогдолгүй болно. |
| **to = Missing** | `[ERROR]` | ERROR_NO_TO буцна. |
| **to = SameAsFrom** | `[ERROR]`, `[IF from байгаа]` | `from` байгаа үед л хооронд нь харьцуулж "ижил" эсэхийг шалгана. |
| **amount = Zero** | `[ERROR]` | amount ≤ 0 тул ERROR_BAD_AMOUNT буцна. |
| **amount = OverBalance** | `[ERROR]` | Дансны үлдэгдлээс хэтэрсэн тул ERROR_INSUFFICIENT буцна. |
| **amount = OverLimit** | `[ERROR]` | Өдрийн лимитээс хэтэрсэн тул ERROR_LIMIT буцна. |
| **amount = ExactBalance** | `[SINGLE]` | Хилийн утга. Нэг удаа шалгахад хангалттай, бусад утгатай хосолж үржүүлэх шаардлагагүй. |
| **currency = Different** | (энгийн) | Happy path болон энгийн урсгалд хоёр утгыг хоёуланг нь шалгана. |

#### Бүрэн олонлог ба хязгаарлалтын дараах тооцоо (N → M)
- **Бүрэн олонлогийн тоо (N):** 3 (from) × 3 (to) × 5 (amount) × 2 (currency) = 90
- **Хязгаарлалт ба тэмдэглэгээний дараах тоо (M):** 10
  - `[ERROR]` тус бүрийг шалгах тестүүд: 2 (from) + 2 (to) + 3 (amount) = 7
  - Алдаагүй утгуудын хослол (Happy path болон бусад): from=Valid × to=Valid × amount=Normal × currency(2) = 2
  - Нийт тест кейс: 7 + 1 + 2 = 10 (Үүнд хилийн утга `SINGLE` нэг удаа орно)
- **Дүгнэлт:** 90 → 10

### Даалгавар A3 — Спецификацийн хүснэгт

| № | from | to | amount | currency | Хүлээгдэх үр дүн |
| :---: | :--- | :--- | :--- | :--- | :--- |
| **1** | Valid | Valid | Normal | Same | 201 OK (happy path) |
| **2** | Valid | Valid | Normal | Different | 201 OK (ханшийн хөрвүүлэлт) |
| **3** | Valid | Valid | ExactBalance | Same | 201 OK (SINGLE) |
| **4** | Missing | Valid | Normal | Same | 200 ERROR_NO_FROM |
| **5** | Frozen | Valid | Normal | Same | 200 ERROR_FROZEN |
| **6** | Valid | Missing | Normal | Same | 200 ERROR_NO_TO |
| **7** | Valid | SameAsFrom | Normal | Same | 200 ERROR_SAME_ACCOUNT |
| **8** | Valid | Valid | Zero | Same | 200 ERROR_BAD_AMOUNT |
| **9** | Valid | Valid | OverBalance | Different | 200 ERROR_INSUFFICIENT (хөрвүүлсний дараа шалгана) |
| **10** | Valid | Valid | OverLimit | Same | 200 ERROR_LIMIT |
| **11** | Frozen | Missing | Zero | Same | 200 ERROR_FROZEN (давхар: from → to → amount) |
| **12** | Valid | Valid | OverBalance & OverLimit | Same | 200 ERROR_INSUFFICIENT (давхар: balance → limit) |

