# Freqtrade Futures Bot — Project Instructions

## Project
- Root: D:\projects\freqtrade\
- Exchange: Binance Futures (isolated margin)
- Strategy: NostalgiaForInfinityX7
- Mode: DRY-RUN -> Real money
- Python: 3.12.10 (venv)
- Freqtrade: 2026.6
- FreqUI: http://localhost:8080 (freqtrade / <пароль в .env>)

## Confirmed Baseline
Локальная dry-run база СБРОШЕНА 09.07.2026 (после обновления Freqtrade 2026.6) - статистика копится заново с нуля
Start (до сброса): 22:00 13.06.2026 | Blacklist: BEAT/USDT, ESPORTS/USDT
Backtest 2024 (10 пар, дефолт, подтверждено 09.07.2026): +95.15% годовых, 65 сделок, 65W/0L
Backtest 01.04-04.07.2026 (10 пар, дефолт): +7.64% (+76.423 USDT), 6 сделок, 5W/1L - выборка мала, ненадежна для выводов

## Key Files
- user_data/config.json — основной конфиг (dry-run/live)
- user_data/config_backtest.json — конфиг для бэктеста/hyperopt (статический список пар)
- user_data/strategies/NostalgiaForInfinityX7.py — стратегия
- user_data/strategies/NostalgiaForInfinityX7.json — файл параметров Hyperopt (НЕ должен существовать сейчас — удалён, стратегия на дефолтах)
- user_data/logs/freqtrade.log — лог файл
- .env — секреты (jwt_secret_key, ws_token, FreqUI password, Binance API key/secret) — НЕ хранить в config.json
- load_env.ps1 — загружает .env в сессию PowerShell перед запуском бота (локально)
- run_bot.sh — обёртка авто-рестарта на VPS (использует set -a; source .env; set +a для экспорта переменных)

## Launch Commands (локально Windows)
cd D:\projects\freqtrade
venv\Scripts\activate
.\load_env.ps1
chcp 65001; $env:PYTHONIOENCODING="utf-8"; python -m freqtrade trade --config user_data/config.json --strategy NostalgiaForInfinityX7

## VPS (Contabo, <user>@<vps_ip>)
- Путь: ~/freqtrade-futures-bot
- FreqUI порт: 8081 (8080 занят чужим ботом root/home/ubuntu/NFI — не трогать)
- Запуск: nohup ./run_bot.sh >> logs_wrapper.log 2>&1 &
- Автозапуск: crontab @reboot -> run_bot.sh
- Внешний доступ к FreqUI: через SSH-туннель

## Roadmap
- STEP 1: Установка и настройка — DONE (13.06.2026)
- STEP 2: Наблюдение dry_run — DONE (43 сделки, 43W/3L)
- STEP 3: Бэктест на исторических данных — DONE (04.07.2026, +96.24% за 2024)
- STEP 4: Оптимизация (Hyperopt) — ЗАВЕРШЕНО, ВЫВОД: дефолтные параметры лучше
  - Попытка 1: SharpeHyperOptLoss, 500 эпох, roi/stoploss/trailing, train на 4 мес (март-июль 2026) -> переобучение (5 сделок), провал на валидации 2024 (66.04% vs 96.24%)
  - Попытка 2: SharpeHyperOptLoss, 552/1000 эпох (прервано), roi/stoploss/trailing, train на 2024 -> хуже дефолта даже на трейне (66.12% vs 96.24%), stoploss -21.1% слишком широкий
  - buy/sell spaces НЕ поддерживаются стратегией (нет IntParameter/DecimalParameter)
  - Windows: --job-workers -1 вызывает WinError 1450 на больших датасетах, использовать --job-workers 4
  - Решение: оставить дефолтные параметры стратегии, не применять Hyperopt-результаты
- STEP 5: Реальная торговля (API ключи + dry_run=false) — В ПРОЦЕССЕ
  - Freqtrade обновлён 2026.5.1 -> 2026.6 (06.07.2026, локально и VPS)
  - VPS dry-run восстановлен: порт 8081, run_bot.sh чинен (set -a/set +a для .env)
  - Binance API ключи созданы, скачаны, пока НЕ вписаны в .env
- STEP 6: Масштабирование + Telegram + VPS
- STEP 7: FreqAI — дальняя цель

## Log Workflow
Перед отправкой логов всегда фильтровать:
Get-Content "D:\projects\freqtrade\user_data\logs\freqtrade.log" | Select-Object -Last 50

## Session Protocol
Перед любыми изменениями подтвердить:
1. Цель — что улучшаем (WR / PnL / стабильность)
2. Текущий результат dry_run
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
- Все секреты (JWT secret, WS token, FreqUI password, Binance API key/secret) — ТОЛЬКО в .env
- config.json содержит пустые плейсхолдеры, заполняемые из env-переменных в рантайме
- НИКОГДА не присылать значения секретов в чат
- Перед запуском бота всегда выполнять .\load_env.ps1 (локально) / run_bot.sh грузит .env сам (VPS)

## Known Issues
- UnicodeEncodeError в консоли — стратегия генерирует emoji в strategy_msg. Не влияет на торговлю. Игнорировать.
- aiodns/pycares переустанавливаются как побочный эффект pip install -U freqtrade или pip install "freqtrade[hyperopt]" — после обновления/установки всегда делать pip uninstall aiodns pycares -y
- --job-workers -1 в hyperopt на Windows с большим датасетом вызывает WinError 1450 — использовать --job-workers 4 (или меньше)
- Запуск бота без .env вызывает ошибку config_validation: '' is too short (jwt_secret_key). Локально грузить load_env.ps1, на VPS run_bot.sh должен иметь set -a/source .env/set +a
- VPS: порт 8080 занят чужим ботом (root/home/ubuntu/NFI) — использовать 8081, чужой процесс не трогать (нет sudo)

## Communication Rules
- Язык: русский
- Уровень: новичок — объяснять каждый шаг подробно
- При ошибках разбирать детально

## Attach to Every Chat
- Логи если есть ошибки
- config.json / config_backtest.json если есть изменения
- Прогресс Hyperopt (если запущен)
