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
* поддерживает выбор транспорта: **Yandex Docs** или **Mail.ru Docs**.

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

Создайте секрет(ы) для нужного транспорта:

| Транспорт | Workflow | Secret |
|---|---|---|
| **Yandex Docs** | OpenFlux Exit Node (Yandex) | `YANDEX_DOC_URL` |
| **Mail.ru Docs** | OpenFlux Exit Node (Mail.ru) | `MAILRU_DOC_URL` |

В значение вставьте свою ссылку на нужный документ (Yandex Docs или Mail.ru Docs), которую вы используете для OpenFlux.

### 🔐 Важно

Не вставляйте ссылку непосредственно в YAML.

Она передаётся в workflow через GitHub Secrets:

```yaml
${{ secrets.YANDEX_DOC_URL }}
${{ secrets.MAILRU_DOC_URL }}
```

И дополнительно маскируется в логах.

---

## 3. Запустите workflow

Откройте:

**Actions** → выберите workflow по нужному транспорту:

* **OpenFlux Exit Node (Yandex)** — transport `yandex`, секрет `YANDEX_DOC_URL`;
* **OpenFlux Exit Node (Mail.ru)** — transport `mailru`, секрет `MAILRU_DOC_URL`.

Затем нажмите **Run workflow**.

После запуска GitHub автоматически подготовит сервер и запустит Exit Node.

В конце в логах должно появиться примерно:

```text
Starting OpenFlux
Mode: l4 (proxy)
Transport: yandex
URL: configured (hidden)
Debug: disabled

=== Universal Bypass Tool ===
Role: exit
Transport: yandex
Exit mode: l4
Codec: batched (zstd + coalescing)
Running as EXIT NODE (mode=l4)
```

> Для Mail.ru в логах транспорт покажется как `mailru`: `Transport: mailru`.

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

Транспорт и ссылка на документ передаются параметрами:

```bash
sudo ./universal-bypass-tool \
  --role=exit \
  --mode=l4 \
  --transport "$TRANSPORT" \
  --url "$DOC_URL"
```

* `TRANSPORT` — `yandex` или `mailru`;
* `DOC_URL` — берётся из соответствующего GitHub Secret (`YANDEX_DOC_URL` / `MAILRU_DOC_URL`).

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

`YANDEX_DOC_URL` и `MAILRU_DOC_URL` не хранятся в открытом виде в репозитории.

Workflow использует GitHub Secrets:

```text
YANDEX_DOC_URL
MAILRU_DOC_URL
```

Значение подставляется только через `env` и маскируется перед записью в логи:

```bash
echo "::add-mask::$DOC_URL"
```

Подробный режим:

```text
--debug
```

намеренно не включён.

### Никогда не публикуйте

* Yandex Docs URL;
* Mail.ru Docs URL;
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
│       ├── openflux.yml          (Yandex Docs)
│       └── openflux-mailru.yml   (Mail.ru Docs)
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
Секрет транспорта → GitHub Secrets
  ↓
Actions
  ↓
Run workflow (Yandex или Mail.ru)
  ↓
OpenFlux Exit Node запускается
  ↓
Подключение клиента OpenFlux
  ↓
✅ Готово
```

