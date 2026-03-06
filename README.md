# StreamerMonitor

Мониторинг стримеров на платформе Trovo с уведомлениями в Telegram.

## 📋 Описание

Streamer-Monitor - это легковесное приложение на Go, которое отслеживает статус стримеров на платформе Trovo и отправляет уведомления в Telegram, когда стримеры выходят в эфир.

## ✨ Возможности

- 🔍 Мониторинг статуса стримеров Trovo в реальном времени
- 📱 Уведомления в Telegram при выходе стримера в эфир
- 💾 Сохранение состояния стримеров в JSON файле
- ⚡ Оптимизированные HTTP клиенты с пулом соединений
- 🛡️ Надежная обработка ошибок
- 🏗️ Модульная архитектура с интерфейсами

## 🏗️ Архитектура

```
StreamerMonitor/
├── cmd/
│   └── app.go                 # Главный файл приложения
├── internal/
│   ├── interfaces/
│   │   └── interfaces.go      # Интерфейсы для клиентов
│   ├── services/
│   │   └── streamers.go       # Модели данных стримеров
│   ├── streamers/
│   │   ├── streamers.go       # Функции работы с файлами
│   │   └── streamers.json     # База данных стримеров
│   ├── telegram/
│   │   ├── client.go          # Telegram API клиент
│   │   └── types.go           # Типы данных Telegram
│   └── trovo/
│       ├── client.go          # Trovo API клиент
│       └── types.go           # Типы данных Trovo
├── go.mod                     # Зависимости Go
└── README.md                  # Документация
```

## 🚀 Установка и запуск

### Предварительные требования

- Go 1.24 или выше
- Аккаунт Trovo с API доступом
- Telegram Bot Token

### 1. Клонирование репозитория

```bash
git clone <repository-url>
cd Streamer-Monitor
```

### 2. Настройка переменных окружения

Создайте файл `.env` или установите переменные окружения:

```bash
# Trovo API
export TROVO_CLIENT_ID="your_trovo_client_id"
export TROVO_ACCESS_TOKEN="your_trovo_access_token"
export TROVO_REFRESH_TOKEN="your_trovo_refresh_token"
export TROVO_CLIENT_SECRET="your_trovo_client_secret"

# Telegram (опционально, можно настроить в коде)
export TELEGRAM_BOT_TOKEN="your_telegram_bot_token"
export TELEGRAM_CHAT_ID="your_telegram_chat_id"
```

### 3. Настройка стримеров

Отредактируйте файл `internal/streamers/streamers.json`:

```json
[
  {
    "username": "Nickname_1",
    "isLive": false,
    "lastCheck": "2024-01-01T00:00:00Z",
    "previews": [
      {
        "game": "Kenshi",
        "url": "https://example.com/i/preview_1"
      }
    ]
  }
]
```

### 4. Настройка списка каналов

В файле `cmd/app.go` измените список отслеживаемых каналов:

```go
channels := []string{
    "Nickname_1",
    "Nickname_2",
    "Nickname_3",
    "Nickname_4",
}
```

### 5. Запуск приложения

```bash
go run cmd/app.go
```

## 🔧 Конфигурация

### Telegram настройки

В файле `internal/telegram/client.go`:

```go
const (
    TelegramToken       = "your_bot_token"
    TelegramChatID      = "your_chat_id"
    TelegramSendPhoto   = "https://api.telegram.org/bot{token}/sendPhoto"
)
```

### Trovo API настройки

В файле `internal/trovo/client.go`:

```go
const (
    URL               = "https://open.trovo.live/page/login.html"
    URL_TOKEN_REFRESH = "https://open-api.trovo.live/openplatform/refreshtoken"
    URL_CHANNEL_INFO  = "https://open-api.trovo.live/openplatform/channels/id"
)
```

## 📊 Структура данных

### Streamer

```go
type Streamer struct {
    Username     string    `json:"username"`
    IsLive       bool      `json:"isLive"`
    LastCheck    time.Time `json:"lastCheck"`
    GamePreviews []Preview `json:"previews,omitempty"`
}
```

### Preview

```go
type Preview struct {
    Game string `json:"game"`
    URL  string `json:"url"`
}
```

## 🔄 Логика работы

1. **Инициализация**: Приложение загружает список стримеров из JSON файла
2. **Мониторинг**: Для каждого канала запрашивается текущий статус через Trovo API
3. **Сравнение**: Сравнивается текущий статус с сохраненным
4. **Уведомление**: При переходе офлайн → онлайн отправляется уведомление в Telegram
5. **Сохранение**: Обновленное состояние сохраняется в JSON файл

## 🛠️ Разработка

### Добавление нового стримера

1. Добавьте имя в массив `channels` в `cmd/app.go`
2. Добавьте запись в `internal/streamers/streamers.json`

### Добавление новых функций

1. Создайте интерфейс в `internal/interfaces/interfaces.go`
2. Реализуйте интерфейс в соответствующем клиенте
3. Обновите `main()` для использования новой функциональности

### Тестирование

```bash
# Проверка синтаксиса
go build ./cmd/app.go

# Запуск с отладкой
go run cmd/app.go
```

## 📝 API Reference

### TrovoClient

```go
type TrovoClient interface {
    ChannelByUsername(username string) (*trovo.ChannelInfo, error)
    ChannelByID(channelID string) (*trovo.ChannelInfo, error)
    RefreshAccessToken(accessToken, refreshToken string) (string, string, error)
    ConfigureAuthorizationURL(ClientID, ResponseType string, Scope []string, RedirectURL string) (string, error)
}
```

### TelegramClient

```go
type TelegramClient interface {
    SendMessageWithPhoto(chatId, caption, imageUrl string) error
    SendTextMessage(text string) error
    SendMessageWithError(chatId string, err error) error
}
```

## 🚨 Обработка ошибок

Приложение включает комплексную обработку ошибок:

- HTTP таймауты и сетевые ошибки
- Ошибки парсинга JSON
- Ошибки файловых операций
- Ошибки API Trovo и Telegram

## ⚡ Производительность

- **HTTP клиенты**: Настроен пул соединений (100 соединений, 10 на хост)
- **Алгоритмы**: O(n) сложность для поиска стримеров
- **Таймауты**: 10 секунд для HTTP запросов, 5 секунд для соединений
- **Keep-Alive**: Переиспользование соединений для повышения производительности

## 🔒 Безопасность

⚠️ **Важно**: В текущей версии токены хранятся в коде. Для продакшена рекомендуется:

1. Вынести все токены в переменные окружения
2. Использовать файлы конфигурации
3. Добавить шифрование чувствительных данных

## 📈 Мониторинг

Приложение выводит логи в консоль:

```
Notification sent: Nickname_1 is now online
Streamer Nickname_1 is now offline
Error getting channel info for Nickname_1: connection timeout
```

## 🤝 Вклад в проект

1. Fork репозитория
2. Создайте feature branch (`git checkout -b feature/amazing-feature`)
3. Commit изменения (`git commit -m 'Add amazing feature'`)
4. Push в branch (`git push origin feature/amazing-feature`)
5. Откройте Pull Request

## 📄 Лицензия

Этот проект распространяется под лицензией MIT. См. файл `LICENSE` для подробностей.

## 🆘 Поддержка

Если у вас возникли проблемы:

1. Проверьте переменные окружения
2. Убедитесь, что файл `streamers.json` существует
3. Проверьте доступность API Trovo и Telegram
4. Создайте issue в репозитории

## 🔄 История изменений

### v1.1.0 (Текущая версия)
- ✅ Рефакторинг архитектуры
- ✅ Добавлены интерфейсы для тестируемости
- ✅ Оптимизированы HTTP клиенты
- ✅ Улучшена обработка ошибок
- ✅ Исправлены алгоритмы производительности

### v1.0.0
- 🎉 Первоначальный релиз
- 📱 Базовый мониторинг стримеров
- 📨 Уведомления в Telegram

---

**Создано с ❤️ на Go**
