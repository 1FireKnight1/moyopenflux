# 🚀 OpenFlux GitHub Actions

> Бесплатный способ запустить **OpenFlux Exit Node** через GitHub Actions без отдельного VPS.

Этот репозиторий позволяет использовать GitHub-hosted runner в качестве временного Exit Node для приложения **OpenFlux** на iPhone.

## ✨ Что это даёт

* 🆓 Не нужен отдельный VPS
* 💳 Не нужна банковская карта
* 🐧 Используется Linux runner от GitHub
* 📱 Можно подключить OpenFlux на iPhone
* 🔐 Ссылка Yandex Docs хранится в GitHub Secrets
* ⚡ Запуск выполняется одной кнопкой
* 🛠 OpenFlux автоматически скачивается и собирается из официального репозитория

---

## 🧩 Как это работает

```text
📱 iPhone
    │
    │ OpenFlux
    ▼
📝 Yandex Docs transport
    │
    ▼
🐙 GitHub Actions
    │
    │ Ubuntu
    ▼
🚀 OpenFlux Exit Node
    │
    ▼
🌐 Internet
```

GitHub Actions создаёт временную Linux-машину, на которой автоматически запускается OpenFlux.

---

# 📋 Установка

## 1. Сделайте Fork

Сначала нажмите **Fork** в правом верхнем углу этого репозитория.

После этого у вас появится **своя копия**:

```text
https://github.com/ВАШ_АККАУНТ/moyopenflux
```

> Не изменяйте оригинальный репозиторий. Работайте со своей копией через **Fork**.

---

## 2. Создайте Secret

В своей копии репозитория откройте:

**Settings → Secrets and variables → Actions → New repository secret**

Создайте секрет:

```text
YANDEX_DOC_URL
```

В значение вставьте **свою ссылку на Yandex Docs**, которую используете в OpenFlux.

### 🔒 Важно

Не вставляйте ссылку напрямую в `openflux.yml`.

Правильно:

```text
GitHub Secrets
    ↓
YANDEX_DOC_URL
    ↓
OpenFlux
```

Неправильно:

```yaml
--url "https://..."
```

в исходном коде.

---

# ▶️ Запуск

После создания Secret:

1. Откройте вкладку **Actions**.
2. Выберите **OpenFlux Exit Node**.
3. Нажмите **Run workflow**.
4. Дождитесь запуска Exit Node.
5. Откройте OpenFlux на iPhone.
6. Запустите тест/туннель.

При успешном запуске в Actions появится примерно:

```text
Starting OpenFlux
Mode: proxy
Transport: yandex
URL: configured (hidden)
Debug: disabled

Mode: EXIT NODE
Transport: yandex
Exit mode: proxy
Running as EXIT NODE (proxy mode)
```

В приложении OpenFlux после подключения должно появиться:

```text
Test OK
```

После этого начинает идти передача данных.

---

# 🔐 Безопасность

`YANDEX_DOC_URL` используется как GitHub Secret и не должен находиться в открытом коде.

Workflow дополнительно маскирует значение:

```bash
echo "::add-mask::$YANDEX_DOC_URL"
```

Также `--debug` не используется, чтобы не создавать лишний подробный вывод.

### Никогда не публикуйте:

* Yandex Docs URL;
* содержимое GitHub Secrets;
* личные токены;
* приватные ключи.

---

# ⚙️ Что происходит автоматически

После запуска workflow:

### 1. Загружается OpenFlux

Используется официальный репозиторий:

https://github.com/p1neappleXpress/OpenFlux

```bash
git clone --depth 1 https://github.com/p1neappleXpress/OpenFlux.git
```

### 2. Устанавливается Go

Версия Go соответствует требованиям текущего OpenFlux.

### 3. Выполняется сборка

```bash
go mod tidy
go build -o universal-bypass-tool .
```

### 4. Настраивается `iptables`

```bash
sudo iptables -A OUTPUT \
  -p tcp \
  --tcp-flags RST RST \
  -j DROP
```

### 5. Проверяется raw socket

Runner проверяется на возможность создания raw socket.

### 6. Запускается Exit Node

```bash
sudo ./universal-bypass-tool \
  --exit-node \
  --transport yandex \
  --url "$YANDEX_DOC_URL"
```

---

# 📱 Использование

После запуска workflow просто включите туннель в приложении OpenFlux на iPhone.

Схема подключения:

```text
iPhone
  ↓
OpenFlux
  ↓
Yandex Docs
  ↓
GitHub Actions
  ↓
OpenFlux Exit Node
  ↓
Internet
```

---

# ⚠️ Ограничения

## GitHub Actions — не постоянный VPS

Runner временный.

Когда workflow заканчивается:

* виртуальная машина уничтожается;
* Exit Node прекращает работу;
* IP может измениться;
* туннель потребуется запустить заново.

Поэтому этот способ подходит прежде всего для **временного использования и тестирования**.

## Ограничение времени

Один GitHub Actions job имеет ограниченное время выполнения.

Если runner остановился, просто запустите workflow заново.

---

# 🔄 Как запустить снова

Когда предыдущий runner завершился:

**Actions → OpenFlux Exit Node → Run workflow**

И снова включите OpenFlux на iPhone.

---

# ❓ Частые проблемы

### `Test Failed`

Проверьте:

* запущен ли workflow;
* появился ли в логах `Running as EXIT NODE`;
* правильно ли создан `YANDEX_DOC_URL`;
* используется ли именно ваша ссылка Yandex Docs.

### `YANDEX_DOC_URL is not configured`

Secret не создан или называется неправильно.

Имя должно быть строго:

```text
YANDEX_DOC_URL
```

### В логах нет URL

Это нормально.

URL специально скрывается из логов.

### Runner работает, но Exit Node не подключается

Убедитесь, что workflow дошёл до:

```text
Running as EXIT NODE (proxy mode)
```

---

# ⭐ Благодарность

Проект использует:

**OpenFlux**
https://github.com/p1neappleXpress/OpenFlux

Оригинальный проект создан **p1neappleXpress**.

Этот репозиторий содержит удобную конфигурацию GitHub Actions для запуска OpenFlux без отдельного VPS.

---

## 📌 Быстрый старт

```text
Fork репозитория
      ↓
Создать YANDEX_DOC_URL
      ↓
Settings → Secrets → Actions
      ↓
Actions
      ↓
OpenFlux Exit Node
      ↓
Run workflow
      ↓
Открыть OpenFlux на iPhone
      ↓
Test OK ✅
```
