# **Система учёта пользователей в CRM**

## **1. Описание системы**
Система учёта пользователей предназначена для работы с данными веб-сайта и мобильных приложений. Она идентифицирует как авторизованных, так и неавторизованных пользователей, связывает устройства и обеспечивает возможность рассылки push-уведомлений.

### **Особенности системы:**
1. **Идентификация пользователей**:
   - Обработка как авторизованных, так и неавторизованных пользователей.
   - Связывание пользователей с их устройствами (веб, мобильные приложения).
2. **Сегментация и рекомендации**:
   - Модуль искусственного интеллекта для анализа поведения пользователей.
   - Рекомендации товаров и услуг на основе истории посещений.
3. **Оптимизация уведомлений**:
   - Умные оповещения (push-уведомления) на основе предпочтений пользователей.
4. **Асинхронность**:
   - Обработка данных в реальном времени без нагрузки на фронтенд (веб и приложения).

---

## **2. Архитектура системы**

### **2.1 Общая схема**
'''plaintext
User Data Collection --> CRM Backend --> Database (users, client_ids, push_tokens)
'''

1. **User Data Collection**:
   - Генерация и передача `ClientID` (для неавторизованных) и `PushToken` (для отправки уведомлений).
   - Сбор данных о пользователях и устройствах (веб и мобильные приложения).
2. **CRM Backend**:
   - Обработка данных о пользователях.
   - Взаимодействие с базами данных и модулями для анализа поведения.
3. **Database**:
   - Хранение информации о пользователях (`users`), устройствах (`client_ids`) и push-токенах (`push_tokens`).

---

## **3. Таблицы базы данных**

### **3.1 Таблица `users`**
| Поле          | Тип данных        | Описание                              |
|---------------|-------------------|---------------------------------------|
| `user_id`     | UUID (PK)         | Уникальный идентификатор пользователя |
| `email`       | VARCHAR           | Email пользователя                    |
| `phone`       | VARCHAR           | Телефон пользователя                  |
| `auth_status` | BOOLEAN           | Статус авторизации (0 — не авторизован, 1 — авторизован) |
| `created_at`  | TIMESTAMP         | Дата создания записи                  |
| `updated_at`  | TIMESTAMP         | Дата последнего обновления            |

### **3.2 Таблица `client_ids`**
| Поле          | Тип данных        | Описание                              |
|---------------|-------------------|---------------------------------------|
| `id`          | UUID (PK)         | Уникальный идентификатор записи       |
| `user_id`     | UUID (FK)         | Ссылка на таблицу `users`             |
| `client_id`   | VARCHAR           | Уникальный идентификатор устройства   |
| `device_type` | VARCHAR           | Тип устройства (web, mobile)          |
| `created_at`  | TIMESTAMP         | Дата привязки                         |

### **3.3 Таблица `push_tokens`**
| Поле          | Тип данных        | Описание                              |
|---------------|-------------------|---------------------------------------|
| `id`          | UUID (PK)         | Уникальный идентификатор записи       |
| `user_id`     | UUID (FK)         | Ссылка на таблицу `users`             |
| `push_token`  | VARCHAR           | Уникальный токен для пушей            |
| `device_type` | VARCHAR           | Тип устройства (android, ios, web)    |
| `created_at`  | TIMESTAMP         | Дата привязки токена                  |
| `last_used_at`| TIMESTAMP         | Дата последнего использования         |

---

## **4. Поток обработки данных**

### **4.1 Входящие данные**
Данные пользователей поступают в систему, например, следующие события:
'''json
{
  "client_id": "CID123456",
  "device_type": "web",
  "push_token": "TOKEN123456",
  "event": "page_visit",
  "timestamp": "2024-12-07T10:00:00Z"
}
'''

### **4.2 Обработка события**
1. **Получение данных**:
   - Система получает события от пользователей (например, посещение страницы, регистрация, авторизация).
2. **Обработка в CRM Backend**:
   - На основе полученных данных обновляются записи в базе данных, создаются новые записи в таблицах `users`, `client_ids` и `push_tokens`, если это необходимо.
   - Взаимодействие с системой искусственного интеллекта для анализа поведения и рекомендаций.

---

## **5. Искусственный интеллект**

### **5.1 Модуль сегментации пользователей**
- Сегментация на основе:
  - Типа устройства (`device_type`).
  - Истории посещений (`event`).
  - Геолокации и активности (если доступно).
  
Пример:
- **Группа 1**: Новые пользователи, которые только начали посещать сайт.
- **Группа 2**: Пользователи, активно совершающие покупки.
- **Группа 3**: Пользователи, использующие мобильное приложение.

### **5.2 Модуль рекомендаций**
- Генерация персонализированных рекомендаций товаров или услуг на основе:
  - Посещённых страниц и товаров.
  - Предыдущих покупок (для авторизованных пользователей).
  - Привычек и времени активности пользователей.

### **5.3 Модуль оптимизации уведомлений**
- Определение оптимального времени и канала для отправки push-уведомлений.
  - **Пример**: Если пользователь активен на iOS в вечернее время, пуши отправляются в этот период.
- **Цель**: Увеличить вовлеченность и отклики на уведомления.

---

## **6. Пример сценариев**

### **Сценарий 1: Новый пользователь заходит через веб-сайт**
1. Генерируется новый `ClientID`.
2. Данные о пользователе и устройстве отправляются в CRM, где создаются записи в таблицах `users` (для неавторизованного пользователя) и `client_ids`.
3. Искусственный интеллект может анализировать поведение и сегментировать пользователя, например, как "нового посетителя".

### **Сценарий 2: Пользователь авторизуется через email**
1. Пользователь вводит email и авторизуется.
2. В CRM обновляется запись в таблице `users`, и теперь пользователь считается авторизованным.
3. Обновляется запись в таблице `client_ids`, связывая старый `ClientID` с новым `user_id`.

### **Сценарий 3: Отправка push-уведомления**
1. AI анализирует поведение и сегментацию пользователей.
2. Для группы "активные пользователи" отправляются push-уведомления с рекомендациями или новыми предложениями.
3. Уведомления отправляются через Firebase Cloud Messaging (FCM) или Apple Push Notification Service (APNs).

---

## **7. Преимущества системы**
1. **Гибкость и масштабируемость**:
   - Обработка миллионов запросов без потери производительности.
   - Лёгкая интеграция с другими сервисами (например, AI и пуш-уведомления).
2. **Персонализация**:
   - Сегментация пользователей и рекомендация контента, основанного на поведении.
3. **Высокая доступность**:
   - Система гарантирует непрерывную работу благодаря асинхронной обработке данных.

---

## **8. Технологии**
- **Базы данных**: PostgreSQL (реляционная), Redis (кеширование).
- **Искусственный интеллект**: TensorFlow/PyTorch для рекомендаций.
- **Push-уведомления**: Firebase Cloud Messaging (FCM), Apple Push Notification Service (APNs).

# **Полный список данных для отслеживания пользователя в CRM**

Для полноценного анализа поведения пользователей и управления персонализированным контентом в интернет-магазине, система должна собирать и передавать в CRM множество данных о действиях пользователей. Вот полный список возможных событий и данных, которые могут быть отправлены:

---

## **1. Основные события пользователя**

### **1.1 Вход пользователя**
- **Тип события**: `user_login`  
- **Параметры**:
  - `user_id` — Идентификатор пользователя (если авторизован).
  - `client_id` — Уникальный идентификатор устройства (если не авторизован).
  - `device_type` — Тип устройства (web, mobile).
  - `auth_status` — Статус авторизации (0 — не авторизован, 1 — авторизован).
  - `timestamp` — Время события.

### **1.2 Выход пользователя**
- **Тип события**: `user_logout`  
- **Параметры**:
  - `user_id` — Идентификатор пользователя.
  - `timestamp` — Время события.

---

## **2. Взаимодействие с сайтом / приложением**

### **2.1 Посещение страниц**
- **Тип события**: `page_view`  
- **Параметры**:
  - `user_id` — Идентификатор пользователя.
  - `client_id` — Идентификатор устройства (если не авторизован).
  - `page_url` — URL страницы.
  - `referrer` — Страница, с которой пришёл пользователь.
  - `timestamp` — Время посещения.

### **2.2 Время на странице**
- **Тип события**: `time_on_page`  
- **Параметры**:
  - `user_id` — Идентификатор пользователя.
  - `client_id` — Идентификатор устройства.
  - `page_url` — URL страницы.
  - `time_spent` — Время, проведённое на странице в секундах.
  - `timestamp` — Время события.

---

## **3. Просмотр товаров и добавление в корзину**

### **3.1 Просмотр товара**
- **Тип события**: `product_view`  
- **Параметры**:
  - `user_id` — Идентификатор пользователя.
  - `client_id` — Идентификатор устройства.
  - `product_id` — Идентификатор товара.
  - `product_name` — Название товара.
  - `category` — Категория товара.
  - `price` — Цена товара.
  - `timestamp` — Время события.

### **3.2 Добавление товара в корзину**
- **Тип события**: `add_to_cart`  
- **Параметры**:
  - `user_id` — Идентификатор пользователя.
  - `client_id` — Идентификатор устройства.
  - `product_id` — Идентификатор товара.
  - `product_name` — Название товара.
  - `quantity` — Количество товара.
  - `price` — Цена товара.
  - `timestamp` — Время добавления.

### **3.3 Удаление товара из корзины**
- **Тип события**: `remove_from_cart`  
- **Параметры**:
  - `user_id` — Идентификатор пользователя.
  - `client_id` — Идентификатор устройства.
  - `product_id` — Идентификатор товара.
  - `timestamp` — Время события.

---

## **4. Оформление заказа**

### **4.1 Начало оформления заказа**
- **Тип события**: `checkout_start`  
- **Параметры**:
  - `user_id` — Идентификатор пользователя.
  - `client_id` — Идентификатор устройства.
  - `cart_items` — Список товаров в корзине (включает product_id, quantity, price).
  - `total_price` — Общая стоимость заказа.
  - `timestamp` — Время начала оформления заказа.

### **4.2 Оформление заказа завершено**
- **Тип события**: `order_complete`  
- **Параметры**:
  - `user_id` — Идентификатор пользователя.
  - `client_id` — Идентификатор устройства.
  - `order_id` — Идентификатор заказа.
  - `order_items` — Список товаров в заказе (включает product_id, quantity, price).
  - `total_price` — Общая стоимость заказа.
  - `shipping_address` — Адрес доставки.
  - `payment_method` — Способ оплаты.
  - `timestamp` — Время завершения заказа.

### **4.3 Отказ от оформления заказа**
- **Тип события**: `checkout_abandonment`  
- **Параметры**:
  - `user_id` — Идентификатор пользователя.
  - `client_id` — Идентификатор устройства.
  - `cart_items` — Список товаров в корзине.
  - `timestamp` — Время отказа от заказа.

---

## **5. Оплата**

### **5.1 Успешная оплата**
- **Тип события**: `payment_success`  
- **Параметры**:
  - `user_id` — Идентификатор пользователя.
  - `client_id` — Идентификатор устройства.
  - `order_id` — Идентификатор заказа.
  - `payment_amount` — Сумма оплаты.
  - `payment_method` — Метод оплаты.
  - `timestamp` — Время успешной оплаты.

### **5.2 Ошибка при оплате**
- **Тип события**: `payment_failure`  
- **Параметры**:
  - `user_id` — Идентификатор пользователя.
  - `client_id` — Идентификатор устройства.
  - `order_id` — Идентификатор заказа.
  - `error_message` — Сообщение об ошибке.
  - `timestamp` — Время ошибки.

---

## **6. Взаимодействие с пуш-уведомлениями**

### **6.1 Получение пуш-уведомления**
- **Тип события**: `push_received`  
- **Параметры**:
  - `user_id` — Идентификатор пользователя.
  - `client_id` — Идентификатор устройства.
  - `push_token` — Токен пуш-уведомления.
  - `notification_type` — Тип уведомления.
  - `timestamp` — Время получения уведомления.

### **6.2 Открытие пуш-уведомления**
- **Тип события**: `push_opened`  
- **Параметры**:
  - `user_id` — Идентификатор пользователя.
  - `client_id` — Идентификатор устройства.
  - `push_token` — Токен пуш-уведомления.
  - `notification_type` — Тип уведомления.
  - `timestamp` — Время открытия уведомления.

---

## **7. Поведение на сайте**

### **7.1 Поиск товара**
- **Тип события**: `search`  
- **Параметры**:
  - `user_id` — Идентификатор пользователя.
  - `client_id` — Идентификатор устройства.
  - `search_query` — Запрос поиска.
  - `timestamp` — Время поиска.

### **7.2 Прокрутка страницы**
- **Тип события**: `scroll`  
- **Параметры**:
  - `user_id` — Идентификатор пользователя.
  - `client_id` — Идентификатор устройства.
  - `page_url` — URL страницы.
  - `scroll_depth` — Глубина прокрутки.
  - `timestamp` — Время прокрутки.

### **7.3 Клик по элементу**
- **Тип события**: `click`  
- **Параметры**:
  - `user_id` — Идентификатор пользователя.
  - `client_id` — Идентификатор устройства.
  - `page_url` — URL страницы.
  - `clicked_element` — Элемент, по которому был клик.
  - `timestamp` — Время клика.

---

## **8. Геолокация пользователя**

### **8.1 Геолокация при входе**
- **Тип события**: `geo_location`  
- **Параметры**:
  - `user_id` — Идентификатор пользователя.
  - `client_id` — Идентификатор устройства.
  - `latitude` — Широта.
  - `longitude` — Долгота.
  - `timestamp` — Время события.

---

## **9. История пользователей**

### **9.1 История покупок**
- **Тип события**: `purchase_history`  
- **Параметры**:
  - `user_id` — Идентификатор пользователя.
  - `order_id` — Идентификатор заказа.
  - `order_items` — Список товаров (включает product_id, quantity, price).
  - `total_price` — Общая стоимость заказа.
  - `purchase_date` — Дата покупки.

### **9.2 История посещений**
- **Тип события**: `visit_history`  
- **Параметры**:
  - `user_id` — Идентификатор пользователя.
  - `page_url` — URL посещённой страницы.
  - `visit_count` — Количество посещений.
  - `last_visit` — Дата последнего посещения.

---

## **10. Сегментация и Рекомендации**

### **10.1 Рекомендации пользователю**
- **Тип события**: `user_recommendation`  
- **Параметры**:
  - `user_id` — Идентификатор пользователя.
  - `client_id` — Идентификатор устройства.
  - `recommended_products` — Список рекомендованных товаров (product_id, product_name).
  - `timestamp` — Время генерации рекомендаций.


# Стек технологий для CRM-системы интернет-магазина

Для разработки эффективной CRM-системы для интернет-магазина предлагается следующий стек технологий. Этот набор инструментов и технологий обеспечит стабильность, масштабируемость, безопасность и возможность интеграции с другими системами.

---

## **1. Архитектура**

- **Микросервисная архитектура** — Обеспечивает гибкость и масштабируемость системы, где различные компоненты могут быть разделены и обрабатываться независимо.
  
---

## **2. База данных**

- **PostgreSQL** — Реляционная СУБД, известная своей мощностью и возможностью работы с большими объемами данных. Используется для хранения основной информации о пользователях, заказах и транзакциях.
  
- **Redis** — Система кэширования и хранения данных в памяти для ускорения работы CRM-системы. Используется для кэширования данных, сессий и событий.

---

## **3. Серверная сторона (Backend)**

- **Node.js** — Платформа для создания высокопроизводительных серверных приложений. Подходит для обработки асинхронных операций и поддерживает работу с WebSocket.
  
- **Python** — Используется для создания аналитических сервисов, обработки больших данных и работы с алгоритмами машинного обучения, рекомендательными системами.
  
- **WebSocket (WS)** — Технология для обеспечения двусторонней связи между сервером и клиентом, что позволяет передавать данные в реальном времени без задержек.

- **Express.js** — Web-фреймворк для Node.js, используемый для создания API и маршрутизации.

- **GraphQL** — Современный подход для создания API, позволяющий клиенту запрашивать только необходимые данные. Гибкая альтернатива REST API.

---

## **4. Фронтенд (Frontend)**

- **React.js** — Библиотека для создания динамичных интерфейсов пользователя. Подходит для построения Single Page Application (SPA).
  
- **Redux** — Менеджер состояния для React, который помогает централизованно управлять состоянием приложения.

- **Redux Query** — Инструмент для управления запросами и кэшированием данных, позволяет удобно работать с API, интегрированным с Redux.

- **Service Worker и Caching** — Для работы с кэшированием на клиентской стороне и улучшения производительности путем хранения данных и статических ресурсов в браузере.

- **Nginx** — Веб-сервер и обратный прокси-сервер для распределения нагрузки, защиты системы и ускорения обработки запросов.

- **Proxy (HTTP Proxy Middleware)** — Прокси-сервер для маршрутизации HTTP-запросов и реализации дополнительной безопасности и управления трафиком.

---

## **5. Машинное обучение и искусственный интеллект**

- **TensorFlow / PyTorch** — Популярные фреймворки для машинного обучения, которые можно использовать для разработки рекомендательных систем и обработки данных.
  
- **spaCy** — Библиотека для обработки естественного языка (NLP), полезна для анализа текста, создания чат-ботов и анализа данных пользователей.

---

## **6. Разработка и управление инфраструктурой**

- **Docker** — Инструмент для контейнеризации приложений, позволяющий запускать CRM-систему в изолированных контейнерах с зависимостями, что упрощает развертывание и переносимость.
  
- **Kubernetes / Docker Swarm** — Платформы для оркестрации контейнеров, управления масштабированием и автоматическим развертыванием сервисов в кластерной среде.

- **pgAdmin** — Веб-интерфейс для управления базой данных PostgreSQL, который облегчает администрирование и работу с данными.

---

## **7. Безопасность**

- **OAuth 2.0 / JWT** — Стандарты авторизации и аутентификации, обеспечивающие безопасность API и управления доступом пользователей.
  
- **SSL / TLS** — Протоколы для безопасной передачи данных между клиентом и сервером.

---

Этот стек технологий обеспечивает мощную и гибкую CRM-систему для интернет-магазина с возможностью масштабирования, высокой производительностью и безопасностью. Используя эти инструменты, можно построить систему, которая будет легко интегрироваться с различными сервисами, обеспечивая комплексный анализ и управление данными о пользователях и их действиях.
