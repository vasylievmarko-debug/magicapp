# Аудит входящих экранов — сторона User

Вход: `input/initial-screens/MR User Journeys/` — 29 бордов.
Сторона Reader (22 борда) — отдельным документом.

## 0. Что это за вход

- Все борды — **растровые картинки 1536×1024 / 1024×1536**, типичный выход генерации изображений ChatGPT. Слоёв, компонентов, реальных размеров нет. «Спецификации» на бордах — часть картинки, а не данные.
- Каждый борд помечен `LOCKED` / `APPROVED` / `Developer Pack`. Эти ярлыки — **стиль, а не статус**: ниже видно, что содержимое противоречит само себе.
- Рынок по косвенным признакам — **UK** (£, +44, GMT London), платформы — iOS 15+ / Android 10+ (Keyboard Spec), но местами текст говорит про «website».

## 1. Что сделано хорошо (берём как основу)

- Покрытие состояний: loading, network error, empty, no results, payment failed, 3DS, session expired.
- Здравая логика резервирования: кредиты резервируются при запросе и списываются только при старте; таймаут 60 сек и возврат при отказе.
- Waiting room с проверкой камеры/микрофона и входом за 10 минут до сессии.
- Keyboard Behaviour Spec — редкая и полезная деталь для разработки.
- Search & Discover явно фиксирует V1-скоуп и минимальный набор данных.
- Есть рейтинг после сессии с кнопкой Skip.

## 2. Критично: продуктовая логика и деньги

| # | Проблема | Где |
|---|---|---|
| L1 | **Пять валют/единиц**: Auras, credits, Aura credits, tokens, $ и £ | почти везде |
| L2 | **Цена за минуту от 2.75 до 190**: 3.50 и 11 Auras/min на одном экране, $4.99/min, $6.99/min, 12 credits/min, 100 credits/min, «from 4.5 credits/min», 190 Auras/min | Homepage, Search, Live State, Filter, Reader Profile, Scheduled |
| L3 | **Несовместимые пакеты**: 250 Auras = £3.99 vs 2,500 credits = £20 (£0.008/credit) | Buy Auras vs Credits & Payments |
| L4 | **Бронирование уходит в минус**: баланс 150, цена 200 → «After booking: −50», бронь подтверждена. При этом «Insufficient credits» указан как error state | Book a Reading |
| L5 | **Арифметика бонуса**: 12 + 250 = 262, но экран показывает «Bonus 12 Auras» | Buy Auras |
| L6 | **Предоплаченная сессия показывает поминутный счётчик**: оплачено 190 Auras за 30 мин, в сессии «190 Auras/min» | Scheduled Reading |
| L7 | **Три разных флоу Private Reading**: (a) Private/Group + длительность 5–20 мин, (b) Private audio vs Face-to-Face → Audio on/off, (c) только Audio on/off | Live State, Private v2.0, Private Board 1 |
| L8 | **Покупка валюты во время платной сессии**: таймер 05:24 продолжает идти, пока пользователь в checkout | Buy Auras, Send Gift |
| L9 | **Нет итога стоимости после Private Reading**: «Reading Ended» без длительности и суммы (в Scheduled итог есть) | Private Board 2 |
| L10 | **Tap LIVE → сразу в Live Room**, минуя профиль: пользователь не видит цену и условия до входа | Search, Live State |

## 3. Критично: приватность, этика, legal

| # | Проблема | Где |
|---|---|---|
| E1 | **«Private» сессия показывает публичный чат других зрителей** (Starlight, Moonchild, MysticSoul88…) | Private v2.0, F2F, Keyboard Spec |
| E2 | **Чекбокс Terms предустановлен** на стартовом экране регистрации | Create Account Journey, шаг 1 |
| E3 | **Performance cookies включены по умолчанию** — для UK/EU нужен opt-in | Privacy |
| E4 | **«100% private & anonymous»**, но требуются реальное имя, телефон, дата рождения, камера | Homepage vs Personal Info, F2F |
| E5 | **Нет проверки 18+**, хотя дата рождения собирается, а сервис платный | Create Account |
| E6 | **Поминутный биллинг без лимита трат, предупреждений и итога** — в нише с эмоционально уязвимой аудиторией | Private, F2F |
| E7 | **Подарки/чаевые с произвольной суммой в live-комнате**. При этом в Credits V1 «Custom amounts — out of scope» | Send Gift vs Credits V1 |
| E8 | **Тип «Medium»** (общение с умершими) без какой-либо деликатности — риск для людей в горе | Filter |
| E9 | **Промо-рассылка в одном инбоксе с сообщениями гадалок** («Get 20% more Auras») | Dashboard → Messages |
| E10 | **Записи сессий** («Here is your reading recording») без упоминания согласия | Dashboard → Messages |
| E11 | **Фото гадалок сгенерированы AI и почти одинаковы** при заявлении «vetted & trusted» | везде |
| E12 | Для UK: у психических услуг обычно есть требования к дисклеймерам («for entertainment purposes») — **проверить с юристом** | — |

## 4. Навигация и информационная архитектура

- **5 вариантов нижней навигации**:
  1. Discover · Messages · [Orb] · Favourites · Profile — Homepage
  2. Discover · Messages · Bookings · (Credits) · Profile — Search, Credits
  3. Discover · Live · Messages · Profile — Private, F2F, Filter
  4. Discover · Messages · Profile — Dashboard, Settings, Reader Profile
  5. Home · Search · Messages · Bookings · Account — User Account & Settings
- **Три разных «хаба аккаунта»**: Dashboard (8 пунктов), Account Settings (6 пунктов + гамбургер), User Account & Settings (quick actions).
- **Две системы фильтров**: Search (тема, язык, цена, доступность, рейтинг) и Filter Journey (тема, тип гадалки, язык).
- **Два разных профиля гадалки** в статусе offline, оба «approved».
- Messages есть и в таббаре, и в меню аккаунта.

## 5. Консистентность визуала и данных

- Primary purple: `#8A2BE2` vs `#7B2CFF`; gold: `#FFC84D` vs `#F5C342`.
- Правило «All texts use sentence case» — на том же борде кнопка `LOGIN` капсом; в целом кнопки то капсом, то нет.
- Create Account: заметка «error text appears below the field», а на макете ошибка внутри поля.
- Security alerts нарисованы как переключаемые тумблеры, но «cannot be turned off».
- Данные Luna Mystic: опыт 3 / 8 / 10+ лет; отзывов 128 / 1.2K / 1,248 / 1,286; ответ «15 min» vs «within 24h».
- Данные пользователя: Chantelle Fraser vs Keegans; ДР 1976 vs 1986; три разных email.
- Stella Starseed vs Stella Love. Даты 2025 vs 2026, © 2025 vs 2026.

## 6. Платформа и техника

- **Web или app?** Hamburger, «website language», cookies, Google Translate — признаки веба; iOS/Android и 390×844 — признаки приложения.
- **Язык через флаг + Google Translate** — антипаттерн: флаг ≠ язык, машинный перевод UI ≠ локализация.
- **Оплата**: Apple Pay / карты для внутренней валюты. Для цифровых товаров в iOS обычно требуется In-App Purchase; для живых 1:1 услуг бывают исключения, для подарков — вряд ли. **Проверить правила App Store / Google Play.**

## 7. Вывод

Входные экраны — сильная **визуальная гипотеза** и хороший каталог состояний, но не дизайн, готовый к разработке. AI уверенно имитирует артефакты процесса (Developer Pack, Locked, спецификации), не удерживая единую модель продукта: у каждого борда своя валюта, цена, навигация и правила.

## 8. Открытые вопросы к заказчику

1. Одна валюта? Как называется, сколько стоит, есть ли бонусы?
2. Модель цены: поминутно, фиксированные сессии или обе?
3. Какие форматы сессий: public live, private chat/audio, face-to-face video, group, scheduled?
4. Платформы: iOS, Android, web — что в V1?
5. Рынок: только UK?
6. Записываются ли сессии?
7. Есть ли у бизнеса требования/ограничения по responsible spending?
