# 🚀 OpenFlux Exit Node — GitHub Actions

Бесплатный способ запустить **OpenFlux Exit Node** через GitHub Actions без отдельного VPS.

Подходит для использования с клиентами OpenFlux на **разных устройствах и платформах**, при условии что клиент поддерживает подключение к Exit Node.

> ✅ Работоспособность схемы проверена на **iOS / iPhone 12 Pro**.

---

## 💡 Что это такое

Этот репозиторий содержит готовый GitHub Actions workflow, который автоматически:

* запускает Ubuntu Linux runner;
* загружает актуальный OpenFlux;
* устанавливает необходимую версию Go;
* собирает `universal-bypass-tool`;
* настраивает `iptables`;
* запускает OpenFlux в режиме **Exit Node**;
* использует **Yandex Docs** как transport.

Отдельный VPS для этого способа не нужен.

---

## 🔗 Схема

```text
                 ┌──────────────────────┐
                 │      OpenFlux        │
                 │  Client / устройство │
                 └──────────┬───────────┘
                            │
                            │ Yandex Docs
                            ▼
                 ┌──────────────────────┐
                 │   GitHub Actions     │
                 │    Ubuntu Runner     │
                 └──────────┬───────────┘
                            │
                            ▼
                 ┌──────────────────────┐
                 │ OpenFlux Exit Node   │
                 │      proxy mode      │
                 └──────────┬───────────┘
                            │
                            ▼
                         Internet
```

Клиентом может выступать любое поддерживаемое OpenFlux устройство.

---

# ⚡ Быстрый старт

## 1. Сделайте Fork

Нажмите **Fork** в правом верхнем углу этого репозитория.

После этого у вас появится собственная копия:

```text
https://github.com/ВАШ_АККАУНТ/moyopenflux
```

Работать нужно именно со своей копией.

---

## 2. Создайте Secret

В своей копии откройте:

**Settings → Secrets and variables → Actions → New repository secret**

Создайте секрет:

```text
YANDEX_DOC_URL
```

В значение вставьте свою ссылку на **Yandex Docs**, которую вы используете для OpenFlux.

### 🔐 Важно

Не вставляйте ссылку непосредственно в YAML.

Она передаётся в workflow через GitHub Secrets:

```yaml
${{ secrets.YANDEX_DOC_URL }}
```

И дополнительно маскируется в логах.

---

## 3. Запустите workflow

Откройте:

**Actions → OpenFlux Exit Node → Run workflow**

После запуска GitHub автоматически подготовит сервер и запустит Exit Node.

В конце в логах должно появиться примерно:

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

После этого можно подключать клиент OpenFlux.

---

# 🛠️ Что делает workflow

### Загрузка OpenFlux

Используется актуальная версия официального проекта:

```bash
git clone --depth 1 \
  https://github.com/p1neappleXpress/OpenFlux.git
```

### Сборка

```bash
go mod tidy
go build -o universal-bypass-tool .
```

### Настройка `iptables`

```bash
sudo iptables -A OUTPUT \
  -p tcp \
  --tcp-flags RST RST \
  -j DROP
```

### Запуск Exit Node

```bash
sudo ./universal-bypass-tool \
  --exit-node \
  --transport yandex \
  --url "$YANDEX_DOC_URL"
```

---

# 📱 Проверка

Схема была фактически проверена на:

* **iPhone 12 Pro**
* iOS
* Wi-Fi
* мобильной сети 4G

Результат:

```text
Test OK
```

После подключения через Exit Node передача данных работала, а доступ к ресурсам через туннель успешно осуществлялся.

> Проверка на iOS подтверждает работоспособность самой схемы Exit Node + Yandex transport. Другие устройства могут использовать тот же Exit Node, если соответствующий клиент OpenFlux поддерживает это подключение.

---

# 🔒 Безопасность

`YANDEX_DOC_URL` не хранится в открытом виде в репозитории.

Workflow использует GitHub Secret:

```text
YANDEX_DOC_URL
```

и маскирует его:

```bash
echo "::add-mask::$YANDEX_DOC_URL"
```

Подробный режим:

```text
--debug
```

намеренно не включён.

### Никогда не публикуйте

* Yandex Docs URL;
* GitHub Secrets;
* токены;
* приватные ключи;
* другие конфиденциальные данные.

---

# ⚠️ Ограничения

GitHub Actions runner является **временной виртуальной машиной**, а не постоянным VPS.

После завершения job:

* Exit Node остановится;
* виртуальная машина будет удалена;
* IP-адрес может измениться;
* workflow необходимо запустить снова.

Поэтому данный способ лучше всего подходит для:

* тестирования;
* временного использования;
* случаев, когда нет собственного VPS.

---

# 🆓 Почему GitHub Actions?

Не требуется:

* отдельный VPS;
* банковская карта для VPS;
* настройка Linux-сервера вручную;
* установка OpenFlux на сервер вручную.

Достаточно сделать Fork, добавить один Secret и запустить workflow.

---

# 📂 Структура репозитория

```text
moyopenflux/
├── .github/
│   └── workflows/
│       └── openflux.yml
└── README.md
```

OpenFlux не хранится внутри этого репозитория.

При каждом запуске workflow получает его непосредственно из официального репозитория.

---

# 🔗 Полезные ссылки

### OpenFlux

https://github.com/p1neappleXpress/OpenFlux

### Этот репозиторий

https://github.com/versh72/moyopenflux

---

# ⭐ Credits

Основано на проекте **OpenFlux** от [p1neappleXpress](https://github.com/p1neappleXpress).

Этот репозиторий предоставляет готовый способ запуска OpenFlux Exit Node через GitHub Actions.

---

## 🚀 В двух словах

```text
Fork
  ↓
YANDEX_DOC_URL → GitHub Secrets
  ↓
Actions
  ↓
Run workflow
  ↓
OpenFlux Exit Node запускается
  ↓
Подключение клиента OpenFlux
  ↓
✅ Готово
```

