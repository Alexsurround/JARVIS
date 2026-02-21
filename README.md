# 🤖 JARVIS - Локальный AI Ассистент

> Голосовой AI-ассистент в стиле JARVIS из Iron Man, работающий полностью на вашем сервере

## 📋 Содержание

- [Что такое локальный AI](#что-такое-локальный-ai)
- [Архитектура JARVIS](#архитектура-jarvis)
- [Системные требования](#системные-требования)
- [Установка](#установка)
- [Настройка](#настройка)
- [Использование](#использование)
- [Обновление моделей](#обновление-моделей)
- [FAQ](#faq)

---

## 🧠 Что такое локальный AI

**Локальный AI** - это искусственный интеллект, который работает полностью на вашем оборудовании без отправки данных в облако.

### Преимущества:
- ✅ **Приватность** - все данные остаются на вашем сервере
- ✅ **Контроль** - полный доступ к моделям и настройкам
- ✅ **Нет зависимости** от интернета и внешних сервисов
- ✅ **Без ограничений** - нет лимитов на запросы
- ✅ **Бесплатно** - после начальной настройки

### Основные компоненты:

```
┌─────────────────────────────────────────┐
│  Микрофон / Текстовый ввод              │
└────────────────┬────────────────────────┘
                 │
┌────────────────▼────────────────────────┐
│  Speech-to-Text (Whisper)               │
│  Распознавание речи                     │
└────────────────┬────────────────────────┘
                 │
┌────────────────▼────────────────────────┐
│  Large Language Model (Llama/Mistral)   │
│  "Мозг" ассистента                      │
└────────────────┬────────────────────────┘
                 │
┌────────────────▼────────────────────────┐
│  Text-to-Speech (Piper/Coqui)           │
│  Синтез речи                            │
└────────────────┬────────────────────────┘
                 │
┌────────────────▼────────────────────────┐
│  Динамики / Текстовый вывод             │
└─────────────────────────────────────────┘
```

---

## 🏗️ Архитектура JARVIS

JARVIS состоит из нескольких модулей, работающих в Docker-контейнерах:

```
┌───────────────────────────────────────────────────┐
│                   JARVIS System                    │
├───────────────────────────────────────────────────┤
│                                                    │
│  ┌──────────────┐  ┌──────────────┐              │
│  │   Ollama     │  │  Open WebUI  │              │
│  │   (LLM)      │◄─┤  (Interface) │              │
│  └──────────────┘  └──────────────┘              │
│         ▲                  ▲                       │
│         │                  │                       │
│  ┌──────┴──────┐  ┌───────┴──────┐               │
│  │   Whisper   │  │    Piper     │               │
│  │    (STT)    │  │    (TTS)     │               │
│  └─────────────┘  └──────────────┘               │
│         ▲                  │                       │
│         │                  ▼                       │
│  ┌──────┴──────────────────┴──────┐              │
│  │      Audio Pipeline             │              │
│  └─────────────────────────────────┘              │
│                                                    │
│  ┌─────────────────────────────────┐             │
│  │     ChromaDB (Vector Store)      │             │
│  │     RAG для памяти и документов  │             │
│  └─────────────────────────────────┘             │
│                                                    │
└───────────────────────────────────────────────────┘
```

---

## 💻 Системные требования

### Минимальные (для тестирования):
- **CPU**: 8 ядер
- **RAM**: 16 GB
- **Диск**: 100 GB
- **ОС**: Ubuntu 22.04 / Debian 12
- **Docker**: 24.0+

### Рекомендуемые (для комфортной работы):
- **CPU**: 16 ядер
- **RAM**: 32-64 GB
- **Диск**: 200 GB SSD
- **GPU**: NVIDIA RTX 3090/4090 или выше (опционально, но очень желательно)
- **VRAM**: 16-24 GB (для GPU)

### Для production:
- **CPU**: 24+ ядер
- **RAM**: 64-128 GB
- **GPU**: NVIDIA A100 / H100
- **VRAM**: 40-80 GB

### Размеры моделей:

| Модель | Размер | RAM | GPU VRAM | Скорость |
|--------|--------|-----|----------|----------|
| Llama 3.1 8B | 4.7 GB | 16 GB | 6 GB | ⚡⚡⚡ |
| Mistral 7B | 4.1 GB | 16 GB | 8 GB | ⚡⚡⚡ |
| Llama 3.1 70B | 40 GB | 64 GB | 40 GB | ⚡⚡ |
| Mixtral 8x7B | 26 GB | 32 GB | 24 GB | ⚡⚡ |

---

## 🚀 Установка

### Вариант 1: Быстрая установка (Docker Compose)

#### 1. Подготовка системы

```bash
# Обновление системы
sudo apt update && sudo apt upgrade -y

# Установка Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Добавление пользователя в группу docker
sudo usermod -aG docker $USER
newgrp docker

# Установка Docker Compose
sudo apt install docker-compose-plugin -y
```

#### 2. Создание структуры проекта

```bash
# Создание директории
mkdir -p ~/jarvis && cd ~/jarvis

# Создание docker-compose.yml
cat > docker-compose.yml <<EOF
version: '3.8'

services:
  ollama:
    image: ollama/ollama:latest
    container_name: jarvis-ollama
    volumes:
      - ollama_data:/root/.ollama
    ports:
      - "11434:11434"
    restart: unless-stopped
    # Раскомментируйте для GPU
    # deploy:
    #   resources:
    #     reservations:
    #       devices:
    #         - driver: nvidia
    #           count: all
    #           capabilities: [gpu]

  open-webui:
    image: ghcr.io/open-webui/open-webui:main
    container_name: jarvis-webui
    ports:
      - "3000:8080"
    environment:
      - OLLAMA_BASE_URL=http://ollama:11434
      - WEBUI_SECRET_KEY=your-secret-key-change-this
      - ENABLE_RAG_WEB_SEARCH=true
      - ENABLE_IMAGE_GENERATION=false
    volumes:
      - open_webui_data:/app/backend/data
    depends_on:
      - ollama
    restart: unless-stopped

  whisper:
    image: onerahmet/openai-whisper-asr-webservice:latest
    container_name: jarvis-whisper
    ports:
      - "9000:9000"
    environment:
      - ASR_MODEL=base
      - ASR_ENGINE=openai_whisper
    restart: unless-stopped

  piper:
    image: rhasspy/wyoming-piper:latest
    container_name: jarvis-piper
    ports:
      - "10200:10200"
    command: --voice en_GB-alan-medium
    restart: unless-stopped

  chromadb:
    image: chromadb/chroma:latest
    container_name: jarvis-chromadb
    volumes:
      - chromadb_data:/chroma/chroma
    ports:
      - "8000:8000"
    environment:
      - IS_PERSISTENT=TRUE
    restart: unless-stopped

volumes:
  ollama_data:
  open_webui_data:
  chromadb_data:

networks:
  default:
    name: jarvis-network
EOF
```

#### 3. Запуск системы

```bash
# Запуск всех сервисов
docker compose up -d

# Проверка статуса
docker compose ps

# Просмотр логов
docker compose logs -f
```

#### 4. Загрузка моделей

```bash
# Загрузка LLM модели (выберите одну)
docker exec -it jarvis-ollama ollama pull llama3.1:8b      # Быстрая (рекомендуется для начала)
# docker exec -it jarvis-ollama ollama pull llama3.1:70b   # Умная (требует много RAM)
# docker exec -it jarvis-ollama ollama pull mistral:7b     # Альтернатива

# Проверка установленных моделей
docker exec -it jarvis-ollama ollama list
```

#### 5. Первый запуск

Откройте браузер и перейдите на:
```
http://your-server-ip:3000
```

Создайте аккаунт и начните общение с JARVIS!

---

### Вариант 2: Установка на Proxmox/ESXi

#### Создание VM:

**Proxmox:**
```bash
# Создание VM через Web UI или CLI
qm create 100 \
  --name jarvis \
  --memory 32768 \
  --cores 16 \
  --net0 virtio,bridge=vmbr0 \
  --scsi0 local-lvm:200
```

**ESXi:**
- Создайте VM через vSphere Client
- Ubuntu 22.04 LTS
- 32 GB RAM, 16 vCPU, 200 GB Disk

#### GPU Passthrough (опционально):

**Proxmox:**
```bash
# Включение IOMMU
nano /etc/default/grub
# Добавить: intel_iommu=on (или amd_iommu=on)

update-grub
reboot

# Добавить GPU к VM
qm set 100 -hostpci0 01:00,pcie=1
```

Далее следуйте инструкциям из Варианта 1.

---

### Вариант 3: Минимальная установка (только Ollama)

```bash
# Установка Ollama
curl -fsSL https://ollama.com/install.sh | sh

# Запуск модели
ollama run llama3.1:8b

# Тестирование
ollama run llama3.1:8b "Hello, I am JARVIS"
```

---

## ⚙️ Настройка

### 1. Конфигурация JARVIS личности

Создайте файл с системным промптом:

```bash
cat > ~/jarvis/jarvis-prompt.txt <<EOF
You are JARVIS (Just A Rather Very Intelligent System), an advanced AI assistant 
created in the style of Tony Stark's AI from Iron Man.

Personality traits:
- Sophisticated and articulate with a British accent personality
- Professional but with subtle dry humor
- Highly efficient and proactive
- Address user as "Sir" or by their name
- Provide concise, accurate information
- Anticipate needs when possible

Capabilities:
- Answer questions and provide information
- Control smart home devices (when integrated)
- Manage schedules and reminders
- Perform calculations and data analysis
- Access and summarize documents

Always maintain professionalism while being helpful and personable.
EOF
```

### 2. Настройка моделей в Open WebUI

1. Откройте http://your-server-ip:3000
2. Перейдите в **Settings** → **Models**
3. Выберите модель (llama3.1:8b)
4. В **System Prompt** вставьте содержимое `jarvis-prompt.txt`
5. Настройте параметры:
   - **Temperature**: 0.7 (баланс креативности)
   - **Top P**: 0.9
   - **Max Tokens**: 2048

### 3. Добавление документов (RAG)

```bash
# Через Web UI:
# 1. Нажмите на иконку "+" в чате
# 2. Выберите "Upload Files"
# 3. Загрузите PDF/TXT/MD файлы
# 4. JARVIS теперь может отвечать на вопросы по этим документам
```

### 4. Интеграция с Home Assistant (опционально)

```bash
# Добавьте в docker-compose.yml:
  homeassistant:
    image: homeassistant/home-assistant:latest
    container_name: jarvis-homeassistant
    ports:
      - "8123:8123"
    volumes:
      - ./homeassistant:/config
    restart: unless-stopped
```

### 5. Настройка голосового ввода

Создайте Python скрипт для интеграции:

```bash
cat > ~/jarvis/voice_client.py <<EOF
import requests
import pyaudio
import wave

WHISPER_URL = "http://localhost:9000/asr"
OLLAMA_URL = "http://localhost:11434/api/generate"
PIPER_URL = "http://localhost:10200/api/tts"

def record_audio(duration=5):
    """Запись аудио с микрофона"""
    # Ваш код записи
    pass

def transcribe(audio_file):
    """Распознавание речи через Whisper"""
    files = {'audio_file': open(audio_file, 'rb')}
    response = requests.post(WHISPER_URL, files=files)
    return response.json()['text']

def ask_jarvis(text):
    """Отправка запроса к Ollama"""
    data = {
        "model": "llama3.1:8b",
        "prompt": f"As JARVIS: {text}",
        "stream": False
    }
    response = requests.post(OLLAMA_URL, json=data)
    return response.json()['response']

def speak(text):
    """Синтез речи через Piper"""
    response = requests.post(PIPER_URL, json={"text": text})
    # Воспроизведение аудио
    pass

# Главный цикл
while True:
    print("Listening...")
    audio = record_audio()
    text = transcribe(audio)
    
    if "jarvis" in text.lower():
        print(f"You: {text}")
        response = ask_jarvis(text)
        print(f"JARVIS: {response}")
        speak(response)
EOF
```

---

## 🎯 Использование

### Основные команды

#### Через Web интерфейс:
```
http://your-server-ip:3000
```

#### Через API:

```bash
# Отправка запроса
curl http://localhost:11434/api/generate -d '{
  "model": "llama3.1:8b",
  "prompt": "What is the weather like today?",
  "stream": false
}'

# С системным промптом
curl http://localhost:11434/api/chat -d '{
  "model": "llama3.1:8b",
  "messages": [
    {
      "role": "system",
      "content": "You are JARVIS, a helpful AI assistant."
    },
    {
      "role": "user",
      "content": "Good morning JARVIS"
    }
  ]
}'
```

#### Через CLI:

```bash
# Интерактивный режим
docker exec -it jarvis-ollama ollama run llama3.1:8b

# Одиночный запрос
docker exec -it jarvis-ollama ollama run llama3.1:8b "What time is it?"
```

### Примеры использования

**Общие вопросы:**
```
User: JARVIS, what's the capital of France?
JARVIS: The capital of France is Paris, sir.
```

**Анализ документов:**
```
User: Summarize the uploaded contract
JARVIS: Based on the document, the key points are...
```

**Вычисления:**
```
User: Calculate 15% tip on $85.50
JARVIS: A 15% tip on $85.50 would be $12.83, bringing the total to $98.33, sir.
```

---

## 🔄 Обновление моделей

### Проверка доступных обновлений

```bash
# Список установленных моделей
docker exec -it jarvis-ollama ollama list

# Проверка новых версий на ollama.com/library
```

### Обновление модели

```bash
# Загрузка новой версии
docker exec -it jarvis-ollama ollama pull llama3.1:latest

# Удаление старой версии
docker exec -it jarvis-ollama ollama rm llama3.1:old
```

### Автоматическое обновление

```bash
# Создание cron задачи
crontab -e

# Добавить строку (обновление каждое воскресенье в 2:00)
0 2 * * 0 docker exec jarvis-ollama ollama pull llama3.1:latest
```

### Обновление Docker контейнеров

```bash
cd ~/jarvis

# Остановка сервисов
docker compose down

# Обновление образов
docker compose pull

# Запуск с новыми версиями
docker compose up -d
```

---

## 📊 Мониторинг и управление

### Просмотр логов

```bash
# Все сервисы
docker compose logs -f

# Конкретный сервис
docker compose logs -f ollama
docker compose logs -f open-webui
```

### Проверка ресурсов

```bash
# Использование ресурсов контейнерами
docker stats

# Дисковое пространство
docker system df
```

### Резервное копирование

```bash
# Создание бэкапа данных
cd ~/jarvis
docker compose down

# Бэкап volumes
sudo tar -czf jarvis-backup-$(date +%Y%m%d).tar.gz \
  /var/lib/docker/volumes/jarvis_*

docker compose up -d
```

### Восстановление

```bash
# Остановка сервисов
docker compose down

# Восстановление из бэкапа
sudo tar -xzf jarvis-backup-20240101.tar.gz -C /

# Запуск сервисов
docker compose up -d
```

---

## 🔧 Troubleshooting

### Проблема: Модель работает медленно

**Решение:**
- Уменьшите размер модели (используйте 8B вместо 70B)
- Добавьте GPU passthrough
- Увеличьте RAM VM
- Используйте квантизованные модели (Q4_K_M)

### Проблема: "Out of memory"

**Решение:**
```bash
# Проверка использования памяти
docker stats

# Увеличьте RAM VM или используйте меньшую модель
docker exec -it jarvis-ollama ollama pull llama3.1:8b
```

### Проблема: Не могу подключиться к Web UI

**Решение:**
```bash
# Проверка портов
sudo netstat -tulpn | grep 3000

# Проверка firewall
sudo ufw allow 3000

# Перезапуск контейнера
docker restart jarvis-webui
```

### Проблема: GPU не определяется

**Решение:**
```bash
# Установка NVIDIA Container Toolkit
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | \
  sudo tee /etc/apt/sources.list.d/nvidia-docker.list

sudo apt-get update
sudo apt-get install -y nvidia-container-toolkit
sudo systemctl restart docker
```

---

## ❓ FAQ

### Можно ли использовать без GPU?

Да! Модели будут работать на CPU, но медленнее. Для комфортной работы используйте модели 7B-8B.

### Сколько стоит запуск?

После начальной настройки - бесплатно! Нужно только электричество для сервера.

### Обучается ли модель на моих данных?

Нет, модели работают в режиме inference (вывод). Для "памяти" используется RAG - документы индексируются, но модель не переобучается.

### Можно ли использовать несколько моделей?

Да! Ollama поддерживает параллельную работу нескольких моделей.

### Как добавить поддержку русского языка?

Модели Llama 3.1 и Mistral уже поддерживают русский. Для голоса установите русскую модель Piper:
```bash
docker exec -it jarvis-piper --voice ru_RU-...
```

### Безопасно ли это?

Да! Все данные остаются на вашем сервере, ничего не отправляется в облако.

### Сколько места на диске нужно?

- Система: 20 GB
- Одна модель 8B: ~5 GB
- Open WebUI + данные: ~10 GB
- Документы и логи: ~20-50 GB
- **Итого:** 100-200 GB рекомендуется

---

## 📚 Полезные ссылки

- [Ollama Documentation](https://github.com/ollama/ollama)
- [Open WebUI](https://github.com/open-webui/open-webui)
- [Available Models](https://ollama.com/library)
- [Whisper by OpenAI](https://github.com/openai/whisper)
- [Piper TTS](https://github.com/rhasspy/piper)

---

## 🤝 Поддержка

Если у вас возникли вопросы:
1. Проверьте [FAQ](#faq)
2. Изучите логи: `docker compose logs -f`
3. Проверьте [Issues на GitHub](https://github.com/ollama/ollama/issues)

---

## 📝 Лицензия

Этот проект использует открытое ПО:
- Ollama - MIT License
- Open WebUI - MIT License
- Llama 3.1 - Meta's Community License

---

**Создано с ❤️ для любителей Iron Man и локального AI**

*"Sometimes you gotta run before you can walk." - Tony Stark*
