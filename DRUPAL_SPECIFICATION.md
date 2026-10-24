# 🛠️ Drupal 11 Developer Handover Guide: Bento Redesign for Tochev & Co.

> **Призначення документа:** Повна технічна інструкція для верстальника та Drupal-розробника щодо впровадження нового дизайну (Концепція 3: High-Tech Bento Hub) на базі Drupal 11.  
> **Створені прототипи сторінок:**
> 1. [index.html](index.html) — Головна сторінка Bento Hub з інтерактивною шахматкою, калькулятором і перемикачем тем.
> 2. [panorama.html](panorama.html) — Розгорнута сторінка окремого об'єкта (`node--building-object--full`).
> 3. [contacts.html](contacts.html) — Офіційна сторінка контактів з відділами та картою (`page--kontakti`).

---

## 1. Типографіка та Токени Дизайну

### 1.1. Оновлена типографіка (100% підтримка Кирилиці)
- **Заголовки (H1-H4):** `Manrope` (вага 600, 700, 800) — сучасний високотехнологічний геометричний шрифт із відмінною геометрією літер болгарської та української кирилиці.
- **Текст та Інтерфейс:** `Plus Jakarta Sans` (вага 400, 500, 600) — бездоганна читабельність технічних параметрів, площ та цін.

Підключення в `tochev_theme.libraries.yml`:
```yaml
global-styling:
  css:
    theme:
      https://fonts.googleapis.com/css2?family=Manrope:wght@500;600;700;800&family=Plus+Jakarta+Sans:ital,wght@0,400;0,500;0,600;0,700;1,400&display=swap: { type: external }
      css/style.css: {}
  js:
    js/theme-switcher.js: {}
    js/chessboard.js: {}
```

---

### 1.2. Перламутрова темна та керамічна світла палітра
```css
/* Перламутровий темний режим (Dark Pearlescent Titanium) */
.dark body {
  background-color: #080C14;
  color: #E2E8F0;
  background-image: 
    radial-gradient(at 15% 15%, rgba(56, 189, 248, 0.08) 0px, transparent 50%),
    radial-gradient(at 85% 10%, rgba(99, 102, 241, 0.07) 0px, transparent 50%),
    radial-gradient(at 50% 50%, rgba(20, 184, 166, 0.05) 0px, transparent 65%);
}

.dark .bento-card {
  background: linear-gradient(135deg, rgba(20, 27, 45, 0.72) 0%, rgba(13, 19, 33, 0.85) 100%);
  border: 1px solid rgba(255, 255, 255, 0.09);
  box-shadow: 0 10px 30px -10px rgba(0, 0, 0, 0.5), inset 0 1px 1px 0 rgba(255, 255, 255, 0.1);
}

/* Світлий керамічно-перламутровий режим (Light Ceramic Pearl) */
.light body {
  background-color: #F8FAFC;
  color: #0F172A;
  background-image: 
    radial-gradient(at 10% 10%, rgba(224, 242, 254, 0.8) 0px, transparent 50%),
    radial-gradient(at 90% 15%, rgba(238, 242, 255, 0.7) 0px, transparent 50%);
}

.light .bento-card {
  background: rgba(255, 255, 255, 0.94);
  border: 1px solid rgba(226, 232, 240, 0.9);
  box-shadow: 0 10px 25px -5px rgba(15, 23, 42, 0.04);
}
```

---

## 2. Реалізація перемикача тем для Drupal (`theme-switcher.js`)

Файл `themes/custom/tochev_theme/js/theme-switcher.js`:
```javascript
(function (Drupal) {
  'use strict';

  function applyTheme(theme) {
    if (theme === 'light') {
      document.documentElement.classList.remove('dark');
      document.documentElement.classList.add('light');
    } else {
      document.documentElement.classList.add('dark');
      document.documentElement.classList.remove('light');
    }
  }

  const saved = localStorage.getItem('tochev_theme') || 'dark';
  applyTheme(saved);

  Drupal.behaviors.tochevThemeToggle = {
    attach: function (context) {
      const btn = context.querySelector('#themeToggleBtn');
      if (btn) {
        btn.addEventListener('click', function () {
          const isDark = document.documentElement.classList.contains('dark');
          const nextTheme = isDark ? 'light' : 'dark';
          applyTheme(nextTheme);
          localStorage.setItem('tochev_theme', nextTheme);
        });
      }
    }
  };
})(Drupal);
```

---

## 3. Шаблон окремого об'єкта (`node--building-object--full.html.twig`)

Розташування: `themes/custom/tochev_theme/templates/content/node--building-object--full.html.twig`:

```twig
<article{{ attributes.addClass('node-building-object-full') }}>
  <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 pt-4 pb-2 text-xs text-slate-500">
    <a href="{{ front_page }}">Начало</a> /
    <a href="{{ path('view.objects_catalog_bento.page_1') }}">Обекти</a> /
    <span class="font-bold text-slate-900 dark:text-white">{{ label }}</span>
  </div>

  <section class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-6">
    <div class="grid grid-cols-1 lg:grid-cols-12 gap-6 items-start">
      <div class="lg:col-span-7 space-y-4">
        <div class="bento-card overflow-hidden relative rounded-2xl h-[420px]">
          {{ content.field_main_image }}
          <div class="absolute top-4 left-4 flex space-x-2">
            <span class="px-3 py-1 rounded-xl badge-act14 text-xs font-bold">
              {{ content.field_act_status }}
            </span>
          </div>
        </div>
        <div class="grid grid-cols-4 gap-3">
          {{ content.field_gallery }}
        </div>
      </div>

      <div class="lg:col-span-5 bento-card p-6 border-sky-500/30">
        <div class="flex justify-between items-center mb-4">
          <span class="text-xs text-slate-500">Паспорт на сградата</span>
          <span class="px-2.5 py-1 rounded-md bg-emerald-500/15 text-emerald-600 font-bold text-xs">
            {{ content.field_available_units_count }} свободни
          </span>
        </div>

        <div class="text-3xl font-extrabold text-slate-900 dark:text-white mb-4">
          от {{ node.field_min_price_sqm.value }} € / м²
        </div>

        <div class="grid grid-cols-2 gap-3 mb-6 text-xs">
          <div class="p-3 rounded-xl bg-slate-100 dark:bg-slate-900">
            <span>Етажност:</span> <strong>{{ node.field_floors_count.value }} етажа</strong>
          </div>
          <div class="p-3 rounded-xl bg-slate-100 dark:bg-slate-900">
            <span>Енергиен клас:</span> <strong>Клас "А"</strong>
          </div>
        </div>

        {{ drupal_entity('webform', 'zapitvane_za_obekt') }}
      </div>
    </div>
  </section>

  <section id="object-chessboard" class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-10 border-t border-slate-200 dark:border-slate-800">
    <h2 class="text-2xl font-bold mb-4">Разпределения по етажи</h2>
    {{ drupal_view('chessboard_apartments', 'block_1', node.id) }}
  </section>
</article>
```

---

## 4. Шаблон сторінки Контакти (`page--kontakti.html.twig`)

Розташування: `themes/custom/tochev_theme/templates/layout/page--kontakti.html.twig`:

```twig
<main class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-8">
  <div class="grid grid-cols-1 lg:grid-cols-12 gap-6 items-start">
    <div class="lg:col-span-7 space-y-6">
      <div class="bento-card p-6">
        <h1 class="text-2xl font-extrabold text-slate-900 dark:text-white mb-2">Централен офис Точев &amp; Co.</h1>
        <p class="text-xs text-slate-500 mb-6">гр. Бургас 8000, ул. Хаджи Димитър 8 - партер</p>

        <div class="grid grid-cols-1 md:grid-cols-3 gap-3">
          <div class="p-3.5 rounded-xl bg-slate-50 dark:bg-slate-900 border border-slate-200 dark:border-slate-800">
            <span class="text-xs font-bold text-emerald-500 block">Отдел Продажби</span>
            <a href="tel:+359899144080" class="text-sm font-bold block">+359 899 144 080</a>
          </div>
          <div class="p-3.5 rounded-xl bg-slate-50 dark:bg-slate-900 border border-slate-200 dark:border-slate-800">
            <span class="text-xs font-bold text-sky-500 block">Счетоводство</span>
            <a href="tel:+359897807388" class="text-sm font-bold block">+359 897 807 388</a>
          </div>
          <div class="p-3.5 rounded-xl bg-slate-50 dark:bg-slate-900 border border-slate-200 dark:border-slate-800">
            <span class="text-xs font-bold text-amber-500 block">Технически отдел</span>
            <a href="tel:+359899144089" class="text-sm font-bold block">+359 899 144 089</a>
          </div>
        </div>
      </div>

      <div class="bento-card p-6">
        {{ page.contacts_map }}
      </div>
    </div>

    <div class="lg:col-span-5 space-y-6">
      <div class="bento-card p-6 border-sky-500/30">
        <h2 class="text-xl font-bold mb-2">Заявете оглед</h2>
        {{ page.contacts_form }}
      </div>
    </div>
  </div>
</main>
```

---
© 2026 Tochev & Co. Redesign Project.
