# Посібник з інтеграції та передачи верстки на Drupal 10/11
## Інженерно-будівельний девелопер «Точев & Co.» (Бургас, Болгарія)

Цей документ містить повні технічні інструкції для Drupal-розробників щодо розгортання та інтеграції готової дизайн-системи та верстки в середовище Drupal 10/11.

---

## 1. Структура теми оформлення (`tochev_theme`)

Тема оформлення розміщена в каталозі `drupal/` та готова для перенесення в `web/themes/custom/tochev_theme/`:

```
web/themes/custom/tochev_theme/
├── tochev_theme.info.yml           # Декларація теми (Drupal 10/11)
├── tochev_theme.libraries.yml      # Реєстрація стилів та скриптів
├── tochev_theme.theme              # PHP-хуки попередньої обробки
├── css/
│   └── theme.css                   # Додаткові стилі та анімації
├── js/
│   ├── theme-toggle.js             # Контролер темної/світлої теми (localStorage)
│   ├── funnel-stream.js            # Інтерактивна 3D воронка об'єктів
│   ├── chessboard.js               # Інтерактивна шахматка поверхів
│   └── mortgage-calc.js            # Іпотечний калькулятор із повзунками
├── templates/
│   ├── html.html.twig              # Базовий HTML-каркас із запобіганням FOUC
│   ├── page.html.twig              # Глобальний лейаут (хедер, навігація, футер)
│   ├── page--front.html.twig       # Головна сторінка з лійкою та метриками
│   ├── node--building-object--full.html.twig   # Повний паспорт об'єкта («Панорама»)
│   ├── node--building-object--teaser.html.twig # Картка об'єкта у воронці та сітці
│   ├── block--standards.html.twig  # Блок 6 європейських будівельних стандартів
│   └── block--calculator.html.twig # Блок інтерактивного іпотечного калькулятора
└── images/
    ├── standards/                  # Оптимізовані рендери стандартів (1.9MB)
    └── projects/                   # 34 оптимізовані рендери об'єктів (8.1MB)
```

---

## 2. Модель даних: Content Types & Fields

### Content Type: `building_object` (Обект / Сграда)

| Поле (Field Machine Name) | Тип даних (Field Type) | Опис / Примітки |
| :--- | :--- | :--- |
| `title` | Text (string) | Назва об'єкта (наприклад, `Сграда «Панорама»`) |
| `field_is_flagship` | Boolean | Прапорець флагманського об'єкта (для відображення в Hero) |
| `field_project_number` | String | Номер у каталозі (`01 / 34`, `02 / 34`) |
| `field_status` | Entity Reference (Taxonomy: `project_status`) | `Акт 14 издаден`, `В строеж`, `Акт 16`, `Проектиране` |
| `field_city` | Entity Reference (Taxonomy: `locations`) | `Бургас`, `София`, `Черноморие` |
| `field_location_text` | String | Текстова адреса (наприклад, `ж.к. Изгрев, бул. Транспортна (до Peugeot)`) |
| `field_short_description` | Text (plain, long) | Короткий опис для карток у каталозі |
| `body` | Text (formatted, long) | Повний опис для сторінки об'єкта |
| `field_price_from` | Decimal / Integer | Стартова ціна за кв.м у EUR (наприклад, `1250`) |
| `field_floors` | String / Integer | Поверховість (`10 надземни етажа`) |
| `field_units_available` | Integer | Кількість вільних квартир (`12`) |
| `field_parking` | String | Тип паркінгу (`Подземен гараж с EV зарядни`) |
| `field_energy_class` | String | Енергоефективність (`Клас "А" (Baumit 10см)`) |
| `field_elevator` | String | Ліфти (`2x Schindler 3300 Inox`) |
| `field_deadline` | String | Строк здачі (`Q4 2026`) |
| `field_main_image` | Image | Головне фото рендеру (мінімум 1280x800px) |
| `field_gallery` | Image (Unlimited) | Додаткові ракурси та фотографії будівництва |

---

## 3. Таксономії (Taxonomies)

1. **`project_status`**:
   - `В строеж` (колір бейджа: Sky blue)
   - `Акт 14 издаден` (колір бейджа: Gold `#C5A880`)
   - `Акт 15`
   - `Акт 16 / Завършен` (колір бейджа: Emerald)
   - `В проектиране`
2. **`locations`**:
   - `Бургас` (райони: Изгрев, Лазур, Сарафово, Меден Рудник, Център)
   - `Крайморие`
   - `София`

---

## 4. Налаштування Views (Представлення)

1. **`views.view.projects_catalog`**:
   - **Display:** Page (`/projects`).
   - **Format:** Unformatted summary with switchable View Modes:
     - **Mode 1 (Воронка):** `node--building-object--teaser.html.twig` всередині горизонтального контейнера `#funnelStream` зі snap-scroll.
     - **Mode 2 (Сітка):** Responsive CSS Grid (`grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6`).
   - **Exposed Filters:** Пошук за назвою, фільтр за статусом (`field_status`), фільтр за локацією (`field_city`).

---

## 5. Оптимізація графічних активів (Web Performance)

Усі 40+ зображень проекту оптимізовані за допомогою прогресивного JPEG стиснення без втрати візуальної чіткості:
- **Загальна вага каталогу зменшена на 50%:** з 26 MB до 13 MB.
- **Роздільна здатність:** 1280px–1600px, що забезпечує 100% різкість на екранах Apple Retina 2x / 3x.
- У верстці для всіх зображень додано атрибут `loading="lazy"`.

---

## 6. Інструкція з активації теми через Drush

```bash
# 1. Скопіювати папку теми
cp -r drupal /path/to/drupal/web/themes/custom/tochev_theme

# 2. Увімкнути тему та встановити як тему за замовчуванням
drush theme:enable tochev_theme
drush config:set system.theme default tochev_theme -y

# 3. Очистити кеш Drupal
drush cr
```
