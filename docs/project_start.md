# KeePass Menu Bar Reader (macOS) — PRD

## 1. Overview

Лёгкое macOS-приложение в menu bar для быстрого доступа к KeePass (`.kdbx`) базам в режиме read-only.

Формат использования:

- клик по иконке в menu bar
- появляется popover (как у системных приложений)
- поиск → копирование

Ключевая идея:
убрать трение между «нужно достать пароль» и «я его получил».

---

## 2. Goals

- Доступ к паролям за < 3 секунд
- Ноль работы с окнами / Dock
- Поддержка нескольких баз
- Минимум повторных unlock
- Безопасный clipboard (автоочистка)

---

## 3. Non-Goals (v1)

- Редактирование записей
- Синхронизация (iCloud / Dropbox)
- Browser autofill
- TOTP
- Key file

---

## 4. Core UX

### Основной поток

1. Клик по menu bar
2. Открывается popover
3. Ввод поиска
4. Выбор записи
5. Copy Password

Весь сценарий должен занимать 1–2 секунды.

---

## 5. Functional Requirements

### 5.1 Menu Bar App

- Нет Dock-иконки
- Нет отдельного окна
- Только menu bar + popover

Требования:

- popover открывается мгновенно (<200ms)
- popover закрывается при клике вне области
- работает на любом рабочем столе (Spaces)

---

### 5.2 Multi Database

- Поддержка нескольких `.kdbx`
- Добавление через file picker

Хранение:

- secure bookmark на файл

Для каждой базы:

- имя
- путь
- статус (locked / unlocked)
- timestamp последнего unlock

---

### 5.3 Unlock Model

После первого unlock:

- база остаётся расшифрованной в памяти

Поведение:

- не требуется вводить пароль при каждом открытии popover

Автоблокировка:

- sleep
- screen lock
- inactivity timeout (дефолт: 15–30 минут)

---

### 5.4 Search

Глобальный поиск по всем открытым базам.

По полям:

- title
- username
- url

Результаты:

- title
- username
- имя базы (источник)

---

### 5.5 Entry View

Поля:

- title
- username
- password (скрыт)
- notes (опционально)

Действия:

- Copy Username
- Copy Password
- Copy Notes

---

### 5.6 Clipboard Security (Critical)

#### Copy Password

- пароль кладётся в clipboard
- запускается таймер (дефолт: 15 секунд)

#### Очистка clipboard

Через 15 секунд:

- если clipboard == скопированный пароль → очистить
- если clipboard изменился → ничего не делать

#### Дополнительно

- новый copy → сбрасывает предыдущий таймер
- при lock приложения → очистить clipboard

---

### 5.7 Session Model

- базы остаются открытыми в памяти
- popover не влияет на состояние unlock

Поведение:

- открыл popover → сразу работаешь
- нет повторного unlock

---

### 5.8 Lock Triggers

Автоблокировка происходит при:

- system sleep
- screen lock
- inactivity timeout
- manual lock (опционально)

---

## 6. Settings

Минимальный набор:

- Clipboard clear time:
  
  - 10 / 15 / 30 / 60 сек

- Auto-lock timeout:
  
  - 5 / 15 / 30 / 60 мин

- Toggle:
  
  - lock on sleep (on)
  - lock on screen lock (on)

---

## 7. Tech Design

### UI

- SwiftUI + AppKit
- NSStatusItem (menu bar icon)
- NSPopover (всплывающее окно)

---

### KeePass Layer

- KeePassKit (или аналогичная библиотека)

Требования:

- поддержка KDBX 4

---

### Storage

- file bookmarks (sandbox-safe)
- in-memory decrypted databases

---

## 8. Architecture

### Core Modules

- MenuBarController
- PopoverController
- DatabaseManager
- UnlockManager
- SearchEngine
- ClipboardManager

---

### Data Flow

User click  
→ Popover open  
→ Search input  
→ Query unlocked DBs  
→ Results list  
→ Select entry  
→ Copy password  
→ Clipboard timer → Clear  

---

## 9. Security Model

- мастер-пароль не сохраняется на диск
- базы расшифрованы только в RAM
- clipboard очищается автоматически
- авто-блокировка по системным событиям

---

## 10. Risks

### Технические

- несовместимость с некоторыми KDBX
- баги в сторонней библиотеке
- утечки памяти

### UX

- некорректная очистка clipboard
- путаница между базами

---

## 11. Success Criteria

- пользователь получает пароль < 3 секунд
- нет необходимости повторного unlock при каждом открытии
- clipboard очищается корректно в 100% случаев
- приложение работает одинаково на всех desktop (Spaces)

---

## 12. v2 (Future)

- Touch ID unlock
- Keychain integration
- Key file support
- TOTP
- hotkeys (global shortcut)
- quick paste (auto paste после копирования)
