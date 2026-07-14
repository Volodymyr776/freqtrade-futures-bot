# Freqtrade Futures Bot — Project Instructions

## Project
- Root: D:\projects\freqtrade\
- Exchange: Binance Futures (isolated margin)
- Strategy: NostalgiaForInfinityX7
- Mode: VPS = LIVE (с 13.07.2026) | Локально = DRY-RUN (остаётся тестовым, нет статического IP)
- Python: 3.12.10 (venv)
- Freqtrade: 2026.6
- FreqUI: http://localhost:8080 (freqtrade / <пароль в .env>)

## Confirmed Baseline
Локальная dry-run база копится с нуля с 09.07.2026
VPS: LIVE с 13.07.2026, реальный баланс на Futures-кошельке ~400 USDT (увеличен 13.07.2026, было ~200)
Backtest 2024 (10 пар, дефолт, подтверждено 09.07.2026): +95.15% годовых, 65 сделок, 65W/0L
Backtest 01.04-04.07.2026 (10 пар, дефолт): +7.64% (+76.423 USDT), 6 сделок, 5W/1L - выборка мала, ненадежна для выводов
Активные short-условия: 501, 502, 542, 661 — все True (с 12.07.2026)
Telegram-уведомления активны с 14.07.2026 (локально и на VPS)

## Key Files
- user_data/config.json — основной конфиг (VPS: dry_run=false LIVE; локально: dry_run=true — НЕ синхронизировать это поле через git; файл в .gitignore целиком)
- user_data/config_backtest.json — конфиг для бэктеста/hyperopt (статический список пар)
- user_data/strategies/NostalgiaForInfinityX7.py — стратегия
- user_data/strategies/NostalgiaForInfinityX7.json — файл параметров Hyperopt (НЕ должен существовать сейчас — удалён, стратегия на дефолте)
- user_data/logs/freqtrade.log — лог файл
- .env — секреты (jwt_secret_key, ws_token, FreqUI password, Binance API key/secret, Telegram token/chat_id) — НЕ хранить в config.json
- load_env.ps1 — загружает .env в сессию PowerShell перед запуском бота (локально)
- run_bot.sh — обёртка авто-рестарта на VPS (использует set -a; source .env; set +a для экспорта переменных)

## Launch Commands (локально Windows)
cd D:\projects\freqtrade
venv\Scripts\activate
.\load_env.ps1
chcp 65001; $env:PYTHONIOENCODING="utf-8"; python -m freqtrade trade --config user_data/config.json --strategy NostalgiaForInfinityX7

## VPS (Contabo, vamp@62.84.182.36)
- Путь: ~/freqtrade-futures-bot
- FreqUI порт: 8081 (8080 занят чужим ботом root/home/ubuntu/NFI — не трогать)
- Запуск: nohup ./run_bot.sh >> logs_wrapper.log 2>&1 &
- Автозапуск: crontab @reboot -> run_bot.sh
- Внешний доступ к FreqUI: через SSH-туннель
- ВАЖНО: перед запуском/остановкой бота всегда проверять pgrep -af freqtrade — были случаи дублирующихся процессов run_bot.sh/freqtrade, убивать только PID из freqtrade-futures-bot, никогда не трогать процесс из /home/ubuntu/NFI
- ВАЖНО: при изменении .env на VPS недостаточно убить процесс freqtrade — run_bot.sh делает source .env только один раз до цикла while true. Нужно убить САМ процесс run_bot.sh (pgrep -af run_bot.sh), затем заново nohup ./run_bot.sh >> logs_wrapper.log 2>&1 &

## Roadmap
- STEP 1: Установка и настройка — DONE (13.06.2026)
- STEP 2: Наблюдение dry_run — DONE
- STEP 3: Бэктест на исторических данных — DONE (04.07.2026, +95.15% за 2024)
- STEP 4: Оптимизация (Hyperopt) — ЗАВЕРШЕНО, ВЫВОД: дефолтные параметры лучше (3 попытки, все хуже дефолта)
- STEP 5: Реальная торговля — ЗАПУЩЕНО 13.07.2026 на VPS
  - API-ключ пересоздан как "binance_vamp_futures" (старый ключ не поддерживал фьючерсы - был создан до активации Futures-аккаунта)
  - Пройден тест по фьючерсам Binance, Futures-аккаунт активирован, $200 переведены на Futures-кошелёк
  - Исправлен "Multi-Asset Mode is not supported" -> переключено на "Режим одного актива" (USDT-M)
  - Подтверждён "Односторонний режим" в Position Mode
  - dry_run=false установлен НАПРЯМУЮ на VPS (не через git push - осознанное исключение из workflow)
  - Бот запущен, state RUNNING, heartbeat подтверждён
  - Осталось: удалить старый ключ binance_vamp, сменить jwt_secret_key (засветился в консоли), следить за ордерами напрямую на Binance первые дни
- STEP 6: Масштабирование — ЗАВЕРШЕНО 14.07.2026
  - Депозит увеличен пользователем до ~400 USDT на VPS (подхватилось автоматически, unlimited stake)
  - Telegram настроен: бот @vVamPv_bot, enabled=true, notification_settings все on (entry/exit/status/warning/startup/protection_trigger)
  - Секреты FREQTRADE__TELEGRAM__TOKEN / FREQTRADE__TELEGRAM__CHAT_ID в .env локально и на VPS
  - Обнаружена особенность run_bot.sh (source .env один раз до цикла) — задокументирована в разделе VPS выше
  - Опционально отложено: авто-сводка баланс/прибыль по расписанию, Binance Futures Testnet
- STEP 7: FreqAI — дальняя цель

## Log Workflow
Перед отправкой логов всегда фильтровать:
Get-Content "D:\projects\freqtrade\user_data\logs\freqtrade.log" | Select-Object -Last 50

## Session Protocol
Перед любыми изменениями подтвердить:
1. Цель — что улучшаем (WR / PnL / стабильность)
2. Текущий результат dry_run / live
3. Файл который меняем + критерий успеха

## Snap Format
По команде "snap" выдать:
---SNAP---
Goal: [что решали]
Status: [done / in progress / blocked]
Baseline: trades=X | WR=X% | Profit=X USDT | Balance=X USDT
Last change: [файл + что изменили]
Next step: [что делать дальше]
---END SNAP---

## Delivery Standard
- Файлы выдавать ПОЛНОСТЬЮ одним блоком PowerShell 7.6.2 (Set-Content -Path с -Encoding utf8NoBOM и here-string в одинарных кавычках)
- Никогда не выдавать частичные куски файла
- Команды давать по одной, ждать результата
- Файлы держать компактными (до 150 строк)
- Если файл > 150 строк — предложить рефакторинг

## Coding Rules
- Никакого хардкода — все параметры выносить в config.json
- Никаких магических чисел и строк прямо в коде
- Не добавлять новый код если можно улучшить существующий
- Трогать только то что нужно
- ASCII-only внутри кода (без кириллицы в комментариях)

## Secrets Management
- Все секреты (JWT secret, WS token, FreqUI password, Binance API key/secret, Telegram token/chat_id) — ТОЛЬКО в .env
- config.json содержит пустые плейсхолдеры, заполняемые из env-переменных в рантайме
- Переменные для Binance API: FREQTRADE__EXCHANGE__KEY / FREQTRADE__EXCHANGE__SECRET (нестандартные имена типа BINANCE_API_KEY не работают - Freqtrade их не распознаёт)
- Переменные для Telegram: FREQTRADE__TELEGRAM__TOKEN / FREQTRADE__TELEGRAM__CHAT_ID
- НИКОГДА не присылать значения секретов в чат, не присылать скриншоты с открытыми значениями .env
- Перед запуском бота всегда выполнять .\load_env.ps1 (локально) / run_bot.sh грузит .env сам (VPS)

## Known Issues
- UnicodeEncodeError в консоли — стратегия генерирует emoji в strategy_msg. Не влияет на торговлю. Игнорировать.
- aiodns/pycares переустанавливаются как побочный эффект pip install -U freqtrade или pip install "freqtrade[hyperopt]" — после обновления/установки всегда делать pip uninstall aiodns pycares -y
- --job-workers -1 в hyperopt на Windows с большим датасетом вызывает WinError 1450 — использовать --job-workers 4 (или меньше)
- Запуск бота без .env вызывает ошибку config_validation: '' is too short (jwt_secret_key). Локально грузить load_env.ps1, на VPS run_bot.sh должен иметь set -a/source .env/set +a
- VPS: порт 8080 занят чужим ботом (root/home/ubuntu/NFI) — использовать 8081, чужой процесс не трогать (нет sudo)
- API-ключ Binance, созданный ДО активации Futures-аккаунта, не может получить разрешение "Включить фьючерсы" — нужно создавать новый ключ после активации
- "Multi-Asset Mode is not supported by freqtrade" — на Binance Futures USDT-M должен быть выбран "Режим одного актива", не "Режим мульти-активов"
- nano: при сохранении .env всегда проверять имя файла в строке "File Name to Write" перед Enter — были случаи сохранения в файл с опечаткой вместо .env
- run_bot.sh на VPS делает source .env только ОДИН РАЗ до цикла while true — простой kill процесса freqtrade НЕ подхватывает изменения .env, нужно убивать сам run_bot.sh и запускать заново

## Communication Rules
- Язык: русский
- Уровень: новичок — объяснять каждый шаг подробно
- При ошибках разбирать детально

## Attach to Every Chat
- Логи если есть ошибки
- config.json / config_backtest.json если есть изменения
- Прогресс Hyperopt (если запущен)
