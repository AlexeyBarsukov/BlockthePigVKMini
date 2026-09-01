# Деплой фронтенда VK Mini App в Yandex Object Storage

## Требования

- Node.js (v18+)
- npm
- AWS CLI (`brew install awscli` на macOS)
- Сервисный аккаунт Yandex Cloud с ролью `storage.editor`
- Бакет в Yandex Object Storage с включённым хостингом

---

## 1. Настройка Yandex Cloud (один раз)

### 1.1 Создать сервисный аккаунт

1. Открыть [Yandex Cloud Console](https://console.yandex.cloud)
2. Перейти в **IAM** -> **Сервисные аккаунты**
3. Нажать **Создать сервисный аккаунт**
4. Указать имя (например `storage-admin`)
5. Назначить роль **storage.editor** на уровне каталога

### 1.2 Создать статический ключ доступа

1. Открыть созданный сервисный аккаунт
2. Нажать **Создать новый ключ** -> **Создать статический ключ доступа**
3. Сохранить **Идентификатор ключа** (Key ID) и **Секретный ключ** (Secret)
4. Записать оба значения в файл `.env` (переменные `YC_ACCESS_KEY_ID` и `YC_SECRET_ACCESS_KEY`)

### 1.3 Создать бакет

1. Перейти в **Object Storage** -> **Создать бакет**
2. Имя бакета: `merge-ball-vk` (или любое другое)
3. Максимальный размер: без ограничений
4. Доступ на чтение объектов: **Публичный**

### 1.4 Включить хостинг статического сайта

1. Открыть бакет -> вкладка **Веб-сайт**
2. Включить **Хостинг**
3. Главная страница: `index.html`
4. Страница ошибки: `index.html` (для работы SPA-роутинга)
5. Сохранить

После этого сайт будет доступен по адресу:
```
https://<имя-бакета>.website.yandexcloud.net
```

---

## 2. Настройка AWS CLI (один раз)

Выполнить в терминале:

```bash
aws configure set aws_access_key_id <YC_ACCESS_KEY_ID>
aws configure set aws_secret_access_key <YC_SECRET_ACCESS_KEY>
aws configure set default.region ru-central1
aws configure set default.output json
```

Или используя значения из `.env`:

```bash
source .env
aws configure set aws_access_key_id $YC_ACCESS_KEY_ID
aws configure set aws_secret_access_key $YC_SECRET_ACCESS_KEY
aws configure set default.region $YC_REGION
aws configure set default.output json
```

---

## 3. Сборка и деплой

### 3.1 Установить зависимости (если не установлены)

```bash
npm install
```

### 3.2 Собрать проект

```bash
npm run build
```

Собранные файлы появятся в папке `dist/`.

### 3.3 Загрузить в бакет

```bash
aws s3 sync dist/ s3://merge-ball-vk/ \
  --endpoint-url=https://storage.yandexcloud.net \
  --delete \
  --exclude ".DS_Store"
```

Флаги:
- `--delete` — удаляет из бакета файлы, которых нет в `dist/` (очищает старые версии)
- `--exclude ".DS_Store"` — исключает служебные файлы macOS

### 3.4 Проверить

Открыть в браузере:
```
https://merge-ball-vk.website.yandexcloud.net
```

---

## 4. Быстрый деплой (одна команда)

```bash
npm run build && aws s3 sync dist/ s3://merge-ball-vk/ --endpoint-url=https://storage.yandexcloud.net --delete --exclude ".DS_Store"
```

---

## 5. Настройка VK Mini App

1. Перейти в [VK для разработчиков](https://dev.vk.com)
2. Открыть или создать мини-приложение
3. В настройках указать URL: `https://merge-ball-vk.website.yandexcloud.net`
4. Сохранить

---

## Структура файлов после деплоя

```
s3://merge-ball-vk/
  index.html          — точка входа
  favicon.svg         — иконка
  icons.svg           — спрайт иконок
  assets/
    index-*.css       — стили
    index-*.js        — бандл приложения
  fonts/
    roboto-bold.ttf   — шрифт
  music/
    *.mp3             — звуковые эффекты
```
