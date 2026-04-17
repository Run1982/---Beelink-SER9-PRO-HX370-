Хотел бы передать огромный респект Beelink который делает обалденные вещи и без Beelink ничего бы этого не было.

🧠 Genesis AI Core: Полное руководство по развёртыванию автономной ИИ-системы на Beelink SER9 PRO (HX370)
<img width="1080" height="810" alt="Image" src="https://github.com/user-attachments/assets/aee5ee94-e080-4e4a-a557-a7462b2643a1" />
Версия 3.0 – Финальная, с активной памятью, саморазвитием и автооткатом ( но не последняя).

Это руководство описывает создание полностью автономной, самообучающейся и самовосстанавливающейся ИИ‑системы на базе мини‑ПК Beelink SER9 PRO. Система состоит из двух компонентов, управляемых через Telegram, использует локальные LLM‑модели с аппаратным ускорением GPU, облачную поддержку, долговременную память и механизм автоматического отката при сбоях.

---

📌 Оглавление

1. Функциональные возможности системы
2. Архитектура и используемые технологии
3. Необходимые компоненты и токены
4. Этап 1: Установка Ubuntu 24.04 LTS и настройка BIOS
5. Этап 2: Установка драйверов AMD ROCm и системных утилит
6. Этап 3: Установка Docker (опционально)
7. Этап 4: Установка Ollama и локальных LLM‑моделей
8. Этап 5: Установка Redis и Python‑окружения
9. Этап 6: Создание файлов проекта Genesis Core
   · 6.1. Файл .env
   · 6.2. Файл commands.json
   · 6.3. Файл genesis_core.py (полный код)
10. Этап 7: Настройка systemd‑сервиса для Genesis Core
11. Этап 8: Установка Node.js и OpenClaw Gateway
12. Этап 9: Настройка и запуск OpenClaw Gateway
13. Этап 10: Финальная настройка автозагрузки и отказоустойчивости
14. Этап 11: Проверка работы и мониторинг
15. Список команд Telegram для быстрого доступа
16. Размещение инструкции в GitHub‑репозитории
17. Заключение и дальнейшее развитие

---

🚀 Функциональные возможности системы

🤖 Бот №1: Genesis Core

Основной ИИ‑ассистент, управляющий мини‑ПК.

· Выполняет системные команды по запросу с обязательным подтверждением (через Telegram).
· Ведёт осмысленный диалог с использованием локальных LLM‑моделей.
· Ежедневно (в 09:00) анализирует тренды GitHub, статьи Habr и наличие обновлений Ubuntu / Ollama / OpenClaw, после чего предлагает конкретные улучшения.
· Запоминает все диалоги и предложения в долговременной памяти (MemPalace).
· Безопасно обновляет систему и себя: перед критическими изменениями автоматически создаёт Git‑коммит. При сбое или таймауте автоматически откатывается к последней рабочей версии. Вы также можете запросить откат вручную.
· Поддерживает команды (подробный список см. в разделе Список команд).

🦞 Бот №2: OpenClaw Gateway

Мультиагентная платформа для сложных задач.

· Хаб для AI‑агентов (Planner, Developer, Reviewer).
· Пишет и редактирует код по вашему запросу (при наличии прав).
· Поддерживает различные LLM (локальные и облачные).
· Имеет веб‑интерфейс для управления и мониторинга (http://<IP‑адрес>:18789).
· Может самостоятельно размещать файлы в GitHub‑репозиториях (при наличии токена).

---

🏗️ Архитектура и используемые технологии

Компонент Назначение Версия / Модель
ОС Базовая операционная система Ubuntu 24.04 LTS
Драйвер GPU Аппаратное ускорение вычислений AMD ROCm 7.2.1
Локальный LLM‑сервер Запуск и управление моделями Ollama
Основная LLM Мощная модель для сложных задач gemma4:26b-a4b-it-q4_K_M
Кодовая LLM Быстрая модель для анализа и диалогов deepseek-coder:6.7b
Кэш и очередь Временное хранение данных Redis
Долговременная память Хранение мыслей и истории MemPalace
Язык Genesis Core Основной язык ассистента Python 3.11+
Управление Genesis Core Автоматический запуск и контроль systemd
Платформа агентов Создание и управление AI‑агентами OpenClaw Gateway
Среда для OpenClaw Среда выполнения платформы Node.js 24
Управление OpenClaw Автоматический запуск systemd (user)

---

🔑 Необходимые компоненты и токены

Перед началом убедитесь, что у вас есть все токены и ключи API. Их нужно будет вставить в конфигурационные файлы.

Назначение Где взять Пример формата
Токен бота Genesis Core Создать через @BotFather 718232455565:AAHl3oKjgT7GGG0y6amOlWrFwtF9V3ltaLhPY
Ваш Telegram ID Узнать у @userinfobot 32324633453098
Токен бота OpenClaw Создать второго бота через @BotFather 8755693755275:AAFE5HdhfdsTDjBgzxIq-3UQPOdUdvGXtXLTEo
DeepSeek API Key platform.deepseek.com sk-...
OpenRouter API Key openrouter.ai/keys sk-or-v1-...
GitHub Token github.com/settings/tokens (права repo, workflow) github_pat_...

Внимание! Никогда не публикуйте реальные токены в открытом доступе. В примерах этого руководства используются уже невалидные ключи. Замените их на свои собственные.

---

📝 Этап 1: Установка Ubuntu 24.04 LTS и настройка BIOS

1. Создайте загрузочную флешку с Ubuntu 24.04 LTS.
2. Настройте BIOS (клавиша Del при загрузке):
   · Boot → Установите USB‑накопитель первым в списке загрузки.
   · Advanced → Power Management → Restore on AC Power Loss = [Power On]. Это гарантирует, что мини‑ПК включится автоматически после сбоя питания.
   · Security → Secure Boot = [Disabled].
3. Установите ОС:
   · Выберите "Normal installation".
   · Обязательно отметьте "Install third-party software for graphics and Wi‑Fi hardware...".
   · На этапе разметки диска выберите "Erase disk and install Ubuntu".
   · Создайте пользователя (например, run) и задайте пароль.

---

⚙️ Этап 2: Установка драйверов AMD ROCm и системных утилит

После загрузки системы откройте терминал (Ctrl+Alt+T) и выполните следующие команды:

```bash
# Обновление системы и установка базовых утилит
sudo apt update && sudo apt upgrade -y
sudo apt install -y git curl wget python3-pip python3-venv build-essential nano lm-sensors speedtest-cli htop
sudo sensors-detect --auto

# Установка ROCm 7.2.1 для поддержки GPU Radeon 890M
wget https://repo.radeon.com/amdgpu-install/7.2/ubuntu/noble/amdgpu-install_7.2.70200-1_all.deb
sudo apt install ./amdgpu-install_7.2.70200-1_all.deb
sudo amdgpu-install --usecase=graphics,rocm -y

# Добавление пользователя в группы для доступа к GPU
sudo usermod -a -G video,render $USER
echo 'export HSA_OVERRIDE_GFX_VERSION=11.5.0' >> ~/.bashrc
source ~/.bashrc

# Перезагрузка
sudo reboot
```

После перезагрузки проверьте, что GPU определился, командой rocm-smi. Вы должны увидеть свою видеокарту Radeon 890M.

---

🐳 Этап 3: Установка Docker (опционально)

Этот шаг можно пропустить, если вы не планируете использовать Docker для изоляции задач.

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker $USER
newgrp docker
```

---

🤖 Этап 4: Установка Ollama и локальных LLM‑моделей

```bash
# Установка Ollama
curl -fsSL https://ollama.com/install.sh | sh
sudo systemctl enable --now ollama

# Загрузка моделей (займёт некоторое время)
ollama pull gemma4:26b-a4b-it-q4_K_M
ollama pull deepseek-coder:6.7b
```

Проверьте, что модель отвечает и использует GPU: ollama run deepseek-coder:6.7b "ping" --verbose | grep -i roc. Вы должны увидеть строку с rocBLAS.

---

🐍 Этап 5: Установка Redis и Python‑окружения

```bash
# Установка и запуск Redis
sudo apt install -y redis-server
sudo systemctl enable --now redis-server

# Создание рабочей директории и виртуального окружения Python
mkdir -p ~/genesis_agent_src && cd ~/genesis_agent_src
python3 -m venv venv
source venv/bin/activate

# Установка необходимых Python‑пакетов
pip install --upgrade pip setuptools wheel
pip install mempalace redis requests python-telegram-bot[job-queue] GitPython python-dotenv feedparser PyGithub
```

---

📄 Этап 6: Создание файлов проекта Genesis Core

Все файлы создаются в директории ~/genesis_agent_src.

6.1. Файл переменных окружения .env

Создайте файл .env и обязательно укажите в нём свои реальные токены.

```bash
nano .env
```

Содержимое файла (замените <...> на свои значения):

```ini
TELEGRAM_BOT_TOKEN=<токен_бота_Genesis_Core>
TELEGRAM_CHAT_ID=<ваш_Telegram_ID>
DEEPSEEK_API_KEY=<ваш_DeepSeek_API_Key>
OPENROUTER_API_KEY=<ваш_OpenRouter_API_Key>
GITHUB_TOKEN=<ваш_GitHub_Token>
```

Установите безопасные права на файл:

```bash
chmod 600 .env
```

6.2. Файл с белым списком команд commands.json

```bash
nano commands.json
```

```json
{
  "ping": {
    "description": "Проверить доступность интернета (ping 8.8.8.8)",
    "command": "ping -c 4 8.8.8.8"
  },
  "sensors": {
    "description": "Узнать температуру процессора",
    "command": "sensors"
  },
  "disk": {
    "description": "Показать свободное место на дисках",
    "command": "df -h"
  },
  "update": {
    "description": "Обновить список пакетов",
    "command": "sudo /usr/bin/apt update"
  },
  "upgrade": {
    "description": "Установить обновления системы",
    "command": "sudo /usr/bin/apt upgrade -y"
  },
  "logs": {
    "description": "Показать логи Genesis Core за последний час",
    "command": "sudo /usr/bin/journalctl -u genesis-core.service --since '1 hour ago' --no-pager"
  },
  "speedtest": {
    "description": "Измерить скорость интернета (speedtest-cli)",
    "command": "speedtest-cli --simple"
  },
  "restart": {
    "description": "Перезапустить сервис Genesis Core",
    "command": "sudo /usr/bin/systemctl restart genesis-core.service"
  },
  "rollback": {
    "description": "Откатить последнее изменение кода (Git)",
    "command": "git reset --hard HEAD~1"
  }
}
```

6.3. Главный исполняемый файл genesis_core.py

Создайте файл с приведённым ниже кодом. Это полная версия со всеми функциями: память MemPalace, ежедневный анализ, предложение обновлений, автооткат через Git, подтверждение через Telegram.

```bash
nano genesis_core.py
```

Полный код (скопируйте целиком):

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

import os
import logging
import re
import json
import subprocess
import time
import shutil
from datetime import datetime, timedelta
from threading import Thread
from dotenv import load_dotenv
import redis
import telegram
from telegram.ext import Application, CommandHandler, MessageHandler, filters, ContextTypes
import requests
import feedparser
from github import Github, RateLimitExceededException
from functools import wraps
from mempalace import MemPalace

# Загрузка переменных окружения из .env
load_dotenv()

# --- Конфигурация ---
TELEGRAM_TOKEN = os.getenv("TELEGRAM_BOT_TOKEN")
TELEGRAM_CHAT_ID = os.getenv("TELEGRAM_CHAT_ID")
DEEPSEEK_API_KEY = os.getenv("DEEPSEEK_API_KEY")
OPENROUTER_API_KEY = os.getenv("OPENROUTER_API_KEY")
GITHUB_TOKEN = os.getenv("GITHUB_TOKEN")

# --- Логирование ---
LOG_FILE = "genesis.log"
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    handlers=[logging.FileHandler(LOG_FILE), logging.StreamHandler()]
)
logger = logging.getLogger("GenesisAI")

# --- Глобальные компоненты ---
redis_client = None
mempalace = None
ALLOWED_COMMANDS = {}
GIT_REPO_PATH = os.getcwd()  # предполагаем, что скрипт запускается из корня Git-репозитория

# Список запрещенных команд (защита от RCE)
FORBIDDEN_PATTERNS = [
    r'rm\s+-rf\s+/',
    r'>\s*/dev/sd',
    r'mkfs',
    r'dd\s+if=',
    r'curl.*\|.*sh',
    r'wget.*\|.*sh',
    r'chmod\s+777',
    r':(){ :|:& };:',
]

def normalize_command(cmd: str) -> str:
    return ' '.join(cmd.split())

def is_command_allowed(cmd: str) -> bool:
    norm_cmd = normalize_command(cmd)
    for pattern in FORBIDDEN_PATTERNS:
        if re.search(pattern, norm_cmd, re.IGNORECASE):
            return False
    return True

def find_command_key(cmd: str):
    norm_cmd = normalize_command(cmd)
    for key, info in ALLOWED_COMMANDS.items():
        if normalize_command(info["command"]) == norm_cmd:
            return key
    return None

def init_components():
    global redis_client, mempalace, ALLOWED_COMMANDS

    try:
        redis_client = redis.Redis(host='localhost', port=6379, db=0, decode_responses=True)
        redis_client.ping()
        logger.info("Redis connected")
    except Exception as e:
        logger.warning(f"Redis unavailable: {e}")

    # --- АКТИВИРУЕМ MemPalace ---
    try:
        mempalace = MemPalace(namespace="genesis_core")
        mempalace.load()
        logger.info("MemPalace initialized")
    except Exception as e:
        logger.warning(f"MemPalace unavailable: {e}")
        mempalace = None

    try:
        with open("commands.json", "r") as f:
            ALLOWED_COMMANDS = json.load(f)
    except FileNotFoundError:
        logger.error("commands.json not found!")
        ALLOWED_COMMANDS = {}

# --- Retry декоратор ---
def retry(max_attempts=3, delay=1):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(max_attempts):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if attempt == max_attempts - 1:
                        raise
                    logger.warning(f"Retry {attempt+1}/{max_attempts} for {func.__name__}: {e}")
                    time.sleep(delay)
            return None
        return wrapper
    return decorator

# --- LLM вызовы ---
@retry(max_attempts=3, delay=2)
def call_deepseek(prompt: str) -> str:
    headers = {"Authorization": f"Bearer {DEEPSEEK_API_KEY}"}
    payload = {"model": "deepseek-chat", "messages": [{"role": "user", "content": prompt}], "temperature": 0.7}
    r = requests.post("https://api.deepseek.com/v1/chat/completions", headers=headers, json=payload, timeout=60)
    r.raise_for_status()
    return r.json()["choices"][0]["message"]["content"]

@retry(max_attempts=3, delay=2)
def call_openrouter(prompt: str) -> str:
    headers = {"Authorization": f"Bearer {OPENROUTER_API_KEY}"}
    payload = {"model": "deepseek/deepseek-chat-v3.1:free", "messages": [{"role": "user", "content": prompt}], "temperature": 0.7}
    r = requests.post("https://openrouter.ai/api/v1/chat/completions", headers=headers, json=payload, timeout=60)
    r.raise_for_status()
    return r.json()["choices"][0]["message"]["content"]

@retry(max_attempts=3, delay=2)
def call_ollama(prompt: str, model: str = "deepseek-coder:6.7b") -> str:
    payload = {
        "model": model,
        "messages": [{"role": "user", "content": prompt}],
        "stream": False,
        "options": {"temperature": 0.7}
    }
    r = requests.post("http://localhost:11434/api/chat", json=payload, timeout=90)
    r.raise_for_status()
    return r.json()["message"]["content"]

def think(prompt: str, use_cloud: bool = False, timeout: int = 60) -> str:
    result = [None]
    exception = [None]
    def target():
        try:
            if use_cloud:
                result[0] = call_deepseek(prompt)
            else:
                result[0] = call_ollama(prompt)
        except Exception as e:
            exception[0] = e

    thread = Thread(target=target, daemon=True)
    thread.start()
    thread.join(timeout)
    if thread.is_alive():
        logger.error("LLM call timed out")
        return "Превышено время ожидания ответа от LLM."
    if exception[0]:
        logger.error(f"LLM error: {exception[0]}")
        return f"Ошибка LLM: {exception[0]}"
    return result[0] or "Пустой ответ от LLM."

# --- Самообучение: анализ трендов GitHub, Habr и официальных источников ---
_github_trending_cache = {"data": [], "expires": datetime.min}

def fetch_github_trending():
    global _github_trending_cache
    if datetime.now() < _github_trending_cache["expires"]:
        return _github_trending_cache["data"]

    try:
        g = Github(GITHUB_TOKEN)
        repos = g.search_repositories(query="stars:>100", sort="stars", order="desc")[:5]
        data = [f"{repo.full_name} - {repo.description}" for repo in repos]
        _github_trending_cache = {"data": data, "expires": datetime.now() + timedelta(hours=1)}
        return data
    except RateLimitExceededException:
        logger.warning("GitHub rate limit exceeded, using cache.")
        return _github_trending_cache["data"]
    except Exception as e:
        logger.error(f"GitHub trending error: {e}")
        return []

def fetch_habr():
    try:
        feed = feedparser.parse("https://habr.com/ru/rss/best/daily/")
        return [f"{e.title} - {e.link}" for e in feed.entries[:5]]
    except Exception as e:
        logger.error(f"Habr error: {e}")
        return []

def fetch_official_updates():
    """Проверка обновлений Ubuntu, Ollama, OpenClaw."""
    updates = []
    try:
        # Ubuntu (через apt list)
        proc = subprocess.run(["apt", "list", "--upgradable"], capture_output=True, text=True, timeout=30)
        if proc.returncode == 0:
            lines = proc.stdout.strip().split('\n')
            if len(lines) > 1:
                updates.append(f"Ubuntu: {len(lines)-1} пакетов можно обновить")
    except:
        pass

    try:
        # Ollama (через GitHub API)
        r = requests.get("https://api.github.com/repos/ollama/ollama/releases/latest", timeout=30)
        if r.status_code == 200:
            latest = r.json()["tag_name"]
            current = subprocess.run(["ollama", "--version"], capture_output=True, text=True).stdout.strip()
            if latest not in current:
                updates.append(f"Ollama: доступна новая версия {latest}")
    except:
        pass

    try:
        # OpenClaw (через npm)
        proc = subprocess.run(["npm", "outdated", "-g", "openclaw"], capture_output=True, text=True, timeout=30)
        if proc.returncode != 0:
            updates.append("OpenClaw: можно обновить")
    except:
        pass

    return updates

def self_improvement_cycle():
    github = fetch_github_trending()
    habr = fetch_habr()
    official = fetch_official_updates()
    prompt = f"""Тренды GitHub: {github}
Статьи Habr: {habr}
Официальные обновления: {official}

На основе этих данных предложи **конкретные** улучшения для кода ИИ-агента Genesis Core, а также для самого мини-ПК (оптимизация, новые инструменты, обновления). Предложения должны быть практичными и реализуемыми. Если предлагаешь изменить код, напиши, что именно и зачем."""
    analysis = think(prompt, use_cloud=False)
    if mempalace:
        mempalace.store(analysis, metadata={"type": "self_improvement", "timestamp": datetime.now().isoformat()})
    return analysis

# --- Telegram handlers ---
async def start(update: telegram.Update, context: ContextTypes.DEFAULT_TYPE):
    await update.message.reply_text(
        "🤖 Genesis AI Core. Команды:\n"
        "/status - статус системы\n"
        "/improve - запустить анализ трендов\n"
        "/memory - показать последние записи памяти\n"
        "/updates - проверить обновления ОС и компонентов\n"
        "/upgrade - обновить систему (требует подтверждения)\n"
        "/rollback - откатить последнее изменение кода\n"
        "Также я понимаю фразы: \"обнови систему\", \"какая температура\", \"проверь интернет\" и др."
    )

async def status(update: telegram.Update, context: ContextTypes.DEFAULT_TYPE):
    ollama = "✅" if requests.get("http://localhost:11434/api/tags", timeout=5).status_code == 200 else "❌"
    redis_ok = "✅" if redis_client and redis_client.ping() else "❌"
    mem_ok = "✅" if mempalace else "❌"
    await update.message.reply_text(f"Ollama: {ollama}\nRedis: {redis_ok}\nMemPalace: {mem_ok}")

async def improve(update: telegram.Update, context: ContextTypes.DEFAULT_TYPE):
    await update.message.reply_text("🔍 Анализирую тренды GitHub, Habr и официальные обновления...")
    analysis = self_improvement_cycle()
    await update.message.reply_text(analysis[:4000])

async def memory(update: telegram.Update, context: ContextTypes.DEFAULT_TYPE):
    if mempalace:
        mems = mempalace.retrieve("", limit=5)
        if mems:
            text = "\n\n".join([f"📌 {m['content'][:300]}..." for m in mems])
        else:
            text = "Память пуста."
        await update.message.reply_text(f"🧠 Последние записи:\n{text[:4000]}")
    else:
        await update.message.reply_text("Память недоступна")

async def updates(update: telegram.Update, context: ContextTypes.DEFAULT_TYPE):
    off = fetch_official_updates()
    if off:
        await update.message.reply_text("Доступны обновления:\n" + "\n".join(off))
    else:
        await update.message.reply_text("Все компоненты актуальны.")

async def upgrade(update: telegram.Update, context: ContextTypes.DEFAULT_TYPE):
    user_id = str(update.effective_user.id)
    cmd = "sudo apt update && sudo apt upgrade -y && ollama pull gemma4:26b-a4b-it-q4_K_M && ollama pull deepseek-coder:6.7b && sudo npm update -g openclaw"
    if redis_client:
        redis_client.setex(f"pending_cmd:{user_id}", 120, cmd)
    await update.message.reply_text(
        f"⚠️ Будут выполнены обновления системы, моделей Ollama и OpenClaw.\nКоманда: `{cmd[:100]}...`\n\nОтветьте \"да\" для подтверждения.",
        parse_mode='Markdown'
    )

async def rollback(update: telegram.Update, context: ContextTypes.DEFAULT_TYPE):
    """Откат последнего изменения кода через Git."""
    try:
        subprocess.run(["git", "reset", "--hard", "HEAD~1"], cwd=GIT_REPO_PATH, check=True)
        await update.message.reply_text("✅ Код откачен к предыдущей версии. Перезапустите сервис вручную.")
    except Exception as e:
        await update.message.reply_text(f"❌ Ошибка отката: {e}")

# --- Выполнение команды на хосте ---
def execute_host_command(command: str, timeout: int = 120) -> str:
    norm_cmd = normalize_command(command)
    key = find_command_key(command)
    if not key:
        return "❌ Команда не найдена в белом списке."

    if not is_command_allowed(norm_cmd):
        return "❌ Команда содержит запрещённые паттерны."

    # Перед выполнением создаём Git-коммит текущего состояния (если есть изменения)
    if key in ["update", "upgrade"]:
        subprocess.run(["git", "add", "-A"], cwd=GIT_REPO_PATH)
        subprocess.run(["git", "commit", "-m", f"Auto-commit before {key}"], cwd=GIT_REPO_PATH)

    try:
        proc = subprocess.run(norm_cmd, shell=True, capture_output=True, text=True, timeout=timeout)
        output = (proc.stdout + proc.stderr)[:3000]
        # Если команда выполнилась успешно, фиксируем новое состояние
        if proc.returncode == 0 and key in ["update", "upgrade"]:
            subprocess.run(["git", "add", "-A"], cwd=GIT_REPO_PATH)
            subprocess.run(["git", "commit", "-m", f"Auto-commit after {key}"], cwd=GIT_REPO_PATH)
        return f"✅ Выполнено:\n```\n{output}\n```"
    except subprocess.TimeoutExpired:
        # При таймауте откатываем
        subprocess.run(["git", "reset", "--hard", "HEAD~1"], cwd=GIT_REPO_PATH)
        return "❌ Команда выполнялась слишком долго. Состояние откачено."
    except Exception as e:
        subprocess.run(["git", "reset", "--hard", "HEAD~1"], cwd=GIT_REPO_PATH)
        return f"❌ Ошибка: {e}. Состояние откачено."

# --- Обработчик подтверждения ---
async def handle_confirmation(update: telegram.Update, context: ContextTypes.DEFAULT_TYPE):
    user_id = str(update.effective_user.id)
    text = update.message.text.lower().strip()
    if text in ['да', 'yes', 'y', 'ок', 'подтверждаю']:
        pending = redis_client.get(f"pending_cmd:{user_id}") if redis_client else None
        if pending:
            redis_client.delete(f"pending_cmd:{user_id}")
            result = execute_host_command(pending)
            await update.message.reply_text(result, parse_mode='Markdown')

# --- Шаблоны быстрого распознавания ---
INTENT_PATTERNS = {
    r"(обнови|апдейт).*систем": "update",
    r"установи.*обновлен": "upgrade",
    r"(покажи|выведи).*лог": "logs",
    r"(перезапусти|рестарт).*сервис": "restart",
    r"температур": "sensors",
    r"интернет": "ping",
    r"скорость": "speedtest",
    r"диск": "disk",
    r"откат": "rollback",
}

def match_intent(text: str):
    text_lower = text.lower()
    for pattern, key in INTENT_PATTERNS.items():
        if re.search(pattern, text_lower):
            if key in ALLOWED_COMMANDS:
                return ALLOWED_COMMANDS[key]["command"], ALLOWED_COMMANDS[key]["description"]
    return None, None

# --- Основной обработчик сообщений ---
async def handle_message(update: telegram.Update, context: ContextTypes.DEFAULT_TYPE):
    user_msg = update.message.text
    user_id = str(update.effective_user.id)
    chat_id = str(update.effective_chat.id)

    if chat_id != TELEGRAM_CHAT_ID:
        return

    if user_msg.lower().strip() in ['да', 'yes', 'y', 'ок', 'подтверждаю']:
        await handle_confirmation(update, context)
        return

    cmd, desc = match_intent(user_msg)
    if cmd:
        if redis_client:
            redis_client.setex(f"pending_cmd:{user_id}", 120, cmd)
        await update.message.reply_text(
            f"⚠️ {desc}\nКоманда: `{cmd}`\n\nОтветьте \"да\" для подтверждения.",
            parse_mode='Markdown'
        )
        return

    await context.bot.send_chat_action(chat_id=update.effective_chat.id, action=telegram.constants.ChatAction.TYPING)

    # ИИ-анализ для сложных запросов
    commands_context = json.dumps({k: v["description"] for k, v in ALLOWED_COMMANDS.items()}, ensure_ascii=False)
    analysis_prompt = f"""Пользователь написал: "{user_msg}".
Доступные команды: {commands_context}
Выбери одну команду из списка, которая лучше всего подходит, или ответь "none". Верни только ключ команды."""
    try:
        llm_decision = call_ollama(analysis_prompt, model="deepseek-coder:6.7b").strip().lower()
        logger.info(f"LLM decision: {llm_decision}")
        if llm_decision in ALLOWED_COMMANDS:
            cmd_info = ALLOWED_COMMANDS[llm_decision]
            if redis_client:
                redis_client.setex(f"pending_cmd:{user_id}", 120, cmd_info["command"])
            await update.message.reply_text(
                f"⚠️ {cmd_info['description']}\nКоманда: `{cmd_info['command']}`\n\nОтветьте \"да\" для подтверждения.",
                parse_mode='Markdown'
            )
            return
    except Exception as e:
        logger.error(f"LLM analysis failed: {e}")

    response = think(user_msg, use_cloud=False)
    if mempalace:
        mempalace.store(f"User: {user_msg}\nAI: {response[:200]}", metadata={"type": "conversation"})
    await update.message.reply_text(response[:4000])

# --- Ежедневный цикл саморазвития (в 09:00) ---
def daily_cycle_thread(bot_token, chat_id):
    bot = telegram.Bot(token=bot_token)
    while True:
        now = datetime.now()
        next_run = now.replace(hour=9, minute=0, second=0, microsecond=0)
        if now >= next_run:
            next_run += timedelta(days=1)
        time.sleep((next_run - now).total_seconds())
        try:
            analysis = self_improvement_cycle()
            # Предлагаем улучшения с кнопкой подтверждения (пока просто текст)
            bot.send_message(chat_id=chat_id, text=f"📅 Ежедневный отчёт и предложения по улучшению:\n{analysis[:4000]}")
            if redis_client:
                redis_client.set("last_cycle", datetime.now().isoformat())
        except Exception as e:
            logger.error(f"Daily cycle error: {e}")

# --- Основная функция ---
def main():
    init_components()

    # Создаём Git-репозиторий, если его нет
    if not os.path.exists(os.path.join(GIT_REPO_PATH, ".git")):
        subprocess.run(["git", "init"], cwd=GIT_REPO_PATH)
        subprocess.run(["git", "add", "-A"], cwd=GIT_REPO_PATH)
        subprocess.run(["git", "commit", "-m", "Initial commit"], cwd=GIT_REPO_PATH)

    if redis_client:
        last_start = redis_client.get("last_start_time")
        now_iso = datetime.now().isoformat()
        if not last_start or (datetime.fromisoformat(last_start) + timedelta(hours=1) < datetime.now()):
            try:
                bot_temp = telegram.Bot(token=TELEGRAM_TOKEN)
                bot_temp.send_message(chat_id=TELEGRAM_CHAT_ID, text="🟢 Genesis AI Core запущен с активной памятью и автооткатом.")
                redis_client.set("last_start_time", now_iso)
            except:
                pass

    Thread(target=daily_cycle_thread, args=(TELEGRAM_TOKEN, TELEGRAM_CHAT_ID), daemon=True).start()

    application = Application.builder().token(TELEGRAM_TOKEN).build()
    application.add_handler(CommandHandler("start", start))
    application.add_handler(CommandHandler("status", status))
    application.add_handler(CommandHandler("improve", improve))
    application.add_handler(CommandHandler("memory", memory))
    application.add_handler(CommandHandler("updates", updates))
    application.add_handler(CommandHandler("upgrade", upgrade))
    application.add_handler(CommandHandler("rollback", rollback))
    application.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, handle_message))

    logger.info("Bot polling started")
    application.run_polling()

if __name__ == "__main__":
    main()
```

Сохраните файл (Ctrl+O, Enter, Ctrl+X).

---

⚙️ Этап 7: Настройка systemd‑сервиса для Genesis Core

Создайте файл сервиса, чтобы Genesis Core автоматически запускался при старте системы и перезапускался в случае сбоя.

```bash
sudo nano /etc/systemd/system/genesis-core.service
```

Содержимое файла (замените run на ваше имя пользователя):

```ini
[Unit]
Description=Genesis AI Core
After=network.target ollama.service docker.service
Wants=redis-server.service

[Service]
Type=simple
User=run
WorkingDirectory=/home/run/genesis_agent_src
Environment="PATH=/home/run/genesis_agent_src/venv/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
EnvironmentFile=/home/run/genesis_agent_src/.env
ExecStart=/home/run/genesis_agent_src/venv/bin/python /home/run/genesis_agent_src/genesis_core.py
Restart=on-failure
RestartSec=30
KillSignal=SIGINT
TimeoutStopSec=10

[Install]
WantedBy=multi-user.target
```

Активируйте и запустите сервис:

```bash
sudo systemctl daemon-reload
sudo systemctl enable genesis-core.service
sudo systemctl start genesis-core.service
```

Проверьте статус: sudo systemctl status genesis-core.service.

---

🧩 Этап 8: Установка Node.js и OpenClaw Gateway

Установите Node.js 24 (рекомендуемая версия) и сам OpenClaw.

```bash
curl -fsSL https://deb.nodesource.com/setup_24.x | sudo -E bash -
sudo apt install -y nodejs
sudo npm install -g openclaw@latest
```

---

🦞 Этап 9: Настройка и запуск OpenClaw Gateway

9.1. Создание конфигурационного файла openclaw.json

Создайте файл конфигурации ~/.openclaw/openclaw.json. Важно: Укажите в нём токен вашего второго бота и ваш локальный IP‑адрес для доступа к веб‑интерфейсу.

```bash
mkdir -p ~/.openclaw
nano ~/.openclaw/openclaw.json
```

Содержимое файла (замените <OPENCLAW_BOT_TOKEN> на реальный токен и <ВАШ_IP> на IP вашего мини‑ПК, например 192.168.8.110):

```json
{
  "gateway": {
    "mode": "local",
    "port": 18789,
    "controlUi": {
      "allowedOrigins": [
        "http://localhost:18789",
        "http://127.0.0.1:18789",
        "http://<ВАШ_IP>:18789"
      ],
      "bind": "0.0.0.0"
    },
    "auth": {
      "mode": "token",
      "token": "<OPENCLAW_BOT_TOKEN>"
    }
  },
  "channels": {
    "telegram": {
      "enabled": true
    }
  },
  "agents": {
    "defaults": {
      "model": {
        "primary": "ollama/gemma4:26b-a4b-it-q4_K_M"
      },
      "tools": {
        "enabled": false
      }
    }
  }
}
```

9.2. Установка и запуск systemd‑сервиса

```bash
# Включение автоматического входа для пользовательских сервисов
sudo loginctl enable-linger $USER

# Установка демона OpenClaw
openclaw onboard --install-daemon

# Включение и запуск сервиса
systemctl --user enable openclaw-gateway
systemctl --user start openclaw-gateway
```

Проверьте статус: systemctl --user status openclaw-gateway.

9.3. Сопряжение с вашим Telegram‑аккаунтом

После первого запуска отправьте любое сообщение боту OpenClaw. Он пришлёт код подтверждения. Выполните в терминале команду, заменив КОД на ваш код:

```bash
openclaw pairing approve telegram КОД
```

Теперь бот OpenClaw будет отвечать на ваши сообщения.

---

🔧 Этап 10: Финальная настройка автозагрузки и отказоустойчивости

Для обеспечения максимальной автономности выполните следующие шаги:

```bash
# Настройка GRUB для мгновенной загрузки
sudo nano /etc/default/grub
# Установите GRUB_TIMEOUT=0, GRUB_TIMEOUT_STYLE=hidden, GRUB_RECORDFAIL_TIMEOUT=0
sudo update-grub

# Настройка автоматического входа в консоль (для экстренного доступа)
sudo mkdir -p /etc/systemd/system/getty@tty1.service.d
sudo nano /etc/systemd/system/getty@tty1.service.d/override.conf
# Вставьте:
# [Service]
# ExecStart=
# ExecStart=-/sbin/agetty --autologin run --noclear %I $TERM

# Настройка Watchdog для перезагрузки при зависании
sudo apt install -y watchdog
sudo nano /etc/watchdog.conf
# Раскомментируйте: watchdog-device = /dev/watchdog, max-load-1 = 24, interval = 10
sudo systemctl enable --now watchdog

# Настройка sudo для выполнения команд без пароля
sudo visudo -f /etc/sudoers.d/genesis
# Вставьте: run ALL=(ALL) NOPASSWD: /usr/bin/apt update, /usr/bin/apt upgrade, /usr/bin/systemctl restart genesis-core.service, /usr/bin/journalctl
sudo chmod 0440 /etc/sudoers.d/genesis

# Лог‑ротация для Genesis Core
sudo nano /etc/logrotate.d/genesis
# Вставьте:
# /home/run/genesis_agent_src/genesis.log {
#     daily
#     rotate 7
#     compress
#     missingok
#     notifempty
#     create 0640 run run
# }
```

---

✅ Этап 11: Проверка работы и мониторинг

Проверка ботов в Telegram

Бот Команда Ожидаемый результат
Genesis Core (MyCameraHomeAssistant_bot) /start Приветствие и список команд
 /status Статус Ollama, Redis, MemPalace
 /improve Анализ трендов (от локальной модели)
 /memory Последние записи из памяти
 /updates Информация о доступных обновлениях
 /upgrade Запрос подтверждения, затем полное обновление
 обнови систему Запрос подтверждения, затем apt update
OpenClaw (OpenClaw_AI_R_bot) ping Ответ от модели

Доступ к веб‑интерфейсу OpenClaw

· С самого мини‑ПК: http://127.0.0.1:18789
· С другого устройства в локальной сети: http://<IP_вашего_ПК>:18789

Токен для входа можно найти в логах:

```bash
journalctl --user -u openclaw-gateway -f | grep token
```

Мониторинг состояния сервисов

```bash
sudo systemctl status genesis-core
systemctl --user status openclaw-gateway
```

Логи Genesis Core: sudo journalctl -u genesis-core -f
Логи OpenClaw: journalctl --user -u openclaw-gateway -f

---

📋 Список команд Telegram для быстрого доступа

Команда / Фраза Действие
/start Показать список команд
/status Состояние Ollama, Redis, MemPalace
/improve Запустить анализ трендов (GitHub + Habr + обновления) и получить предложения
/memory Показать последние 5 записей из долговременной памяти
/updates Проверить наличие обновлений Ubuntu, Ollama, OpenClaw
/upgrade Обновить систему, модели Ollama и OpenClaw (требует подтверждения)
/rollback Откатить последнее изменение кода через Git (ручной откат)
обнови систему Запустить sudo apt update (после подтверждения)
какая температура Показать температуру процессора
проверь интернет Выполнить ping 8.8.8.8
покажи логи Показать логи Genesis Core за час
перезапусти сервис Перезапустить Genesis Core
сколько места на диске Выполнить df -h
откати изменения То же, что и /rollback

---

📖 Размещение инструкции в GitHub‑репозитории

Вы можете разместить это руководство в своём GitHub‑репозитории, чтобы другие могли повторить ваш путь.

Способ 1: Вручную через терминал

```bash
cd ~/genesis_agent_src
nano README.md
# Скопируйте всё содержимое этого руководства и вставьте в редактор
git add README.md
git commit -m "Добавлено полное руководство по установке"
git push origin main
```

Способ 2: Через агента Developer в OpenClaw

Отправьте боту OpenClaw команду:

```
@developer создай файл README.md в репозитории genesis_agent_src с содержимым:
[полный текст руководства]
```

---

💎 Заключение и дальнейшее развитие

Поздравляем! Вы создали полностью автономную, отказоустойчивую ИИ‑систему. Ваша система умеет:

· Поддерживать диалог и выполнять системные команды.
· Анализировать тренды разработки и предлагать улучшения.
· Запоминать все взаимодействия в долговременной памяти.
· Безопасно обновлять себя и всё окружение с автоматическим откатом при сбоях.
· Работать 24/7 и автоматически восстанавливаться после отключений питания.
<img width="792" height="1693" alt="Image" src="https://github.com/user-attachments/assets/64b71151-21dd-4119-8dd0-83a2c2c46f33" />
<img width="796" height="1693" alt="Image" src="https://github.com/user-attachments/assets/62156765-4abd-4da1-8320-7270a7e439df" />
<img width="797" height="1693" alt="Image" src="https://github.com/user-attachments/assets/60034eb5-27b9-4881-b9ab-3481f8590384" />
<img width="797" height="1693" alt="Image" src="https://github.com/user-attachments/assets/759e7bbe-be52-498b-86d6-c04d91214422" />
<img width="792" height="1693" alt="Image" src="https://github.com/user-attachments/assets/389696c2-8ea0-4fe0-aa03-1369ff155658" />
🚀 Идеи для дальнейшего развития
· Дома уже давно работает Frigate с Coral, подумают докрутить два доступ к нему анализ видео и обновление кода, с откатом.
· Можно попробовать более рутинные задачи предоставить...идей очень много...от написания кода, для .... попробовать дать доступ к моему не основному роутеру на OpenWrt Flint 2 пусть покопается и в нем, напоминания, и богально погода. 
· Настройте автоматическое применение предложенных улучшений (с вашим подтверждением).
· Интегрируйте ботов: настройте Genesis Core так, чтобы он делегировал задачи по написанию кода агенту Developer в OpenClaw.
· Добавьте новые команды: расширьте файл commands.json и словарь INTENT_PATTERNS.
· Подключите облачных провайдеров: настройте OpenClaw на использование платных и более мощных моделей через API.

Ваш Beelink SER9 PRO теперь не просто мини‑ПК, а персональная ИИ‑лаборатория, способная учиться, развиваться и помогать вам в решении самых разных задач. Удачи в экспериментах и разработке!

В коде не совершенно и он будет постепенно развиваться дальше. 
P.S. токены другие уже)
