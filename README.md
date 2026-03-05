# FastAPI интернет‑магазин

Проект backend‑API интернет‑магазина с ролями покупателей и продавцов, корзиной, заказами и интеграцией оплаты через YooKassa.

## Ключевые возможности
- Регистрация пользователей с ролями `buyer` и `seller`.
- JWT‑аутентификация: access + refresh токены.
- CRUD категорий (в т.ч. вложенные категории) с мягким удалением.
- CRUD товаров с загрузкой изображений и мягким удалением.
- Поиск и фильтрация товаров: категории, цена, наличие, продавец, полнотекстовый поиск (PostgreSQL `tsvector` + GIN индекс).
- Корзина: добавить/обновить/удалить позиции, подсчёт total.
- Оформление заказа: списание остатков, создание заказа, инициирование оплаты.
- Интеграция YooKassa + обработка вебхуков с проверкой IP.

## Технологии
- **FastAPI**, **Uvicorn**
- **PostgreSQL**, **SQLAlchemy 2.0 (async)**, **Alembic**
- **JWT** (access + refresh), **bcrypt**
- **YooKassa SDK**
- **Pydantic v2**, **python‑multipart**

## Архитектура проекта
- `app/main.py` — инициализация приложения и роутинг
- `app/routers/` — ручки API (users, products, categories, cart, orders, payments)
- `app/models/` — ORM‑модели
- `app/schemas.py` — Pydantic‑схемы
- `app/auth.py` — хеширование паролей и JWT
- `app/payments.py` — интеграция с YooKassa
- `app/migrations/` — миграции Alembic
- `media/` — загруженные изображения товаров

## Быстрый старт (локально)
1. Поднять PostgreSQL (docker):

```bash
docker compose up -d
```

2. Создать `.env` в корне проекта:

```env
SECRET_KEY=your_secret_key
YOOKASSA_SHOP_ID=your_shop_id
YOOKASSA_SECRET_KEY=your_shop_secret
YOOKASSA_RETURN_URL=http://localhost:8000/
```

3. Установить зависимости и запустить миграции:

```bash
poetry install
poetry run alembic upgrade head
```

4. Запуск приложения:

```bash
poetry run uvicorn app.main:app --reload
```

Swagger будет доступен на `http://localhost:8000/docs`.

## Пример API‑сценария
1. Создать пользователя‑продавца (`/users`).
2. Получить токен (`/users/token`).
3. Создать категорию (`/categories`).
4. Создать товар с картинкой (`/products` с `multipart/form-data`).
5. Покупатель добавляет товар в корзину (`/cart/items`).
6. Оформляет заказ (`/orders/checkout`) — возвращается `confirmation_url` для оплаты.
7. YooKassa отправляет вебхук в `/payments/yookassa/webhook`.

## Основные эндпоинты
- `POST /users` — регистрация
- `POST /users/token` — логин
- `POST /users/refresh-token` — обновление access‑токена

- `GET /categories` — список категорий
- `POST /categories` — создать категорию
- `PUT /categories/{id}` — обновить
- `DELETE /categories/{id}` — soft delete

- `GET /products` — список товаров + фильтры
- `POST /products` — создать товар (seller only)
- `GET /products/{id}` — товар по id
- `PUT /products/{id}` — обновить (seller only)
- `DELETE /products/{id}` — soft delete

- `GET /cart` — корзина пользователя
- `POST /cart/items` — добавить в корзину
- `PUT /cart/items/{product_id}` — изменить количество
- `DELETE /cart/items/{product_id}` — удалить позицию
- `DELETE /cart` — очистить корзину

- `POST /orders/checkout` — создать заказ + инициировать оплату
- `GET /orders` — список заказов пользователя
- `GET /orders/{id}` — детали заказа

- `POST /payments/yookassa/webhook` — вебхук оплаты

## Особенности реализации
- Полнотекстовый поиск через `tsvector` и GIN‑индекс в PostgreSQL.
- Защита продавцов: создавать/редактировать товары могут только `seller`.
- Заказы проверяют остатки, списывают сток и очищают корзину транзакционно.
- Вебхук проверяет IP адреса YooKassa.