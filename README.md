# HighloadCourse2026Chat

Архитектурный проект Telegram-like чата для курса по highload-системам.

## Как делим работу

Работа делится на две команды по принципу **hot path / cold path**.

### Команда A: Realtime Delivery

Фокус: что происходит с сообщением от отправки до получения в реальном времени.

Зона ответственности:
- выбор протокола: WebSocket / SSE / long-polling;
- WebSocket/API Gateway;
- connection/session manager;
- online delivery;
- маршрутизация личных сообщений;
- fanout для групп;
- slow mode;
- backpressure на горячем пути;
- delivery flow diagrams.

Основной файл: [`delivery/todo.md`](delivery/todo.md).

### Команда B: Durable Storage & System Backbone

Фокус: как данные хранятся, масштабируются, читаются из истории и не теряются.

Зона ответственности:
- requirements и capacity расчёты;
- модель данных;
- message storage;
- infinite history;
- edit/delete;
- sharding;
- кэш последних сообщений;
- storage replication и backup/restore;
- offline sync;
- multi-device state.

Основной файл: [`storage/todo.md`](storage/todo.md).

## Совместные зоны

Эти темы нельзя полностью отдать одной команде, потому что они соединяют delivery и storage:

| Тема | Команда A | Команда B |
|---|---|---|
| High-level architecture | runtime-компоненты и путь сообщения | storage-компоненты и зависимости |
| Ordering | доставка в порядке `seq_no` | выдача/хранение `seq_no` |
| Idempotency | дедупликация retry на hot path | идемпотентная запись в storage |
| Multi-device | доставка на online-устройства | per-device cursor / sync state |
| Fanout | online fanout | offline catch-up из истории |
| Reliability | gateway/broker/delivery failures | storage/replica/backup failures |

## Общий контракт между командами

Hot path ожидает от storage:
- `appendMessage(chat_id, sender_id, client_msg_id, payload) -> message_id, seq_no`;
- `getMessages(chat_id, from_seq, limit)`;
- `getMembers(chat_id)`;
- `getUserDevices(user_id)`;
- `editMessage(message_id, new_payload)`;
- `deleteMessage(message_id)`.

Storage ожидает от hot path:
- требования по latency;
- fanout-модель для личных, малых и больших групп;
- нагрузку на чтение последних сообщений;
- retry/idempotency semantics;
- online/offline delivery rules.

## Порядок работы

1. Все вместе фиксируем scope, out of scope и общую блок-схему.
2. Команда B уточняет capacity и storage constraints.
3. Команда A проектирует realtime flow и fanout с учётом этих чисел.
4. Команды синхронизируют интерфейсы: `chat_id`, `message_id`, `seq_no`, cursors, retry.
5. Каждая команда оформляет свою часть в Markdown и диаграммах.
6. Финальная встреча: проверяем, что storage выдерживает hot path, а delivery не нарушает ordering и consistency.

## Входные требования

Базовые требования и допущения зафиксированы в [`requirements.md`](requirements.md).
