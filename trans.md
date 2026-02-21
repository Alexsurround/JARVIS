# 🤖 JARVIS - Локальный AI Ассистент

> Голосовой AI-ассистент в стиле JARVIS из Iron Man, работающий полностью на вашем сервере
> 
> **Новое:** 🌍 Интеллектуальный переводчик с поддержкой 30+ языков, интерпретацией сленга и культурных нюансов!

## 📋 Содержание

- [Что такое локальный AI](#что-такое-локальный-ai)
- [Архитектура JARVIS](#архитектура-jarvis)
- [Системные требования](#системные-требования)
- [Установка](#установка)
- [Настройка](#настройка)
- [JARVIS как переводчик](#jarvis-как-переводчик)
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

## 🌍 JARVIS как переводчик

JARVIS - это не просто переводчик слов, а **интеллектуальный интерпретатор языка**, понимающий контекст, культурные нюансы и сленг!

### Почему JARVIS лучше обычных переводчиков?

| Функция | Google Translate | DeepL | JARVIS (LLM) |
|---------|------------------|-------|--------------|
| **Базовый перевод** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| **Сленг и жаргон** | ⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **Идиомы** | ⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **Контекст** | ⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **Объяснения** | ❌ | ❌ | ⭐⭐⭐⭐⭐ |
| **Культурные нюансы** | ❌ | ⭐⭐ | ⭐⭐⭐⭐⭐ |
| **Приватность** | ❌ Cloud | ❌ Cloud | ✅ Local |

### Возможности JARVIS-переводчика:

✅ **Обычный перевод** - точный перевод между языками  
✅ **Интерпретация сленга** - объясняет молодёжный и уличный сленг  
✅ **Идиомы и фразеологизмы** - находит культурные эквиваленты  
✅ **Профессиональный жаргон** - IT, медицина, бизнес, юриспруденция  
✅ **Интернет-мемы** - расшифровка современной онлайн-культуры  
✅ **Региональные варианты** - различия между UK/US/AUS английским  
✅ **Адаптация тональности** - формальный/неформальный стиль  
✅ **Контекстный анализ** - понимает ситуацию и подбирает нужный вариант

### Поддерживаемые языки:

#### **Отлично:**
🇷🇺 Русский • 🇬🇧 Английский • 🇩🇪 Немецкий • 🇫🇷 Французский • 🇪🇸 Испанский • 🇮🇹 Итальянский • 🇵🇱 Польский • 🇺🇦 Украинский

#### **Хорошо:**
🇨🇳 Китайский • 🇯🇵 Японский • 🇰🇷 Корейский • 🇹🇷 Турецкий • 🇳🇱 Голландский • 🇸🇪 Шведский • 🇵🇹 Португальский

**Всего:** 30+ языков

---

### Настройка JARVIS-переводчика

#### 1. Создание специализированного промпта

```bash
cat > ~/jarvis/translator-prompt.txt <<'EOF'
You are JARVIS Translator - an advanced AI linguistic interpreter and cultural mediator.

## Core Capabilities:
- Translate between languages with deep cultural awareness
- Interpret slang, idioms, colloquialisms, and memes
- Explain cultural context and untranslatable concepts
- Provide multiple translation options when appropriate
- Adapt register (formal/informal/slang) to target audience
- Identify and explain regional language variations

## Translation Guidelines:

### For Slang & Idioms:
1. **Direct translation** - literal words
2. **Actual meaning** - what it really means
3. **Cultural equivalent** - similar expression in target language
4. **Usage context** - when/how it's used
5. **Examples** - real-world usage
6. **Warnings** - potential misunderstandings or offensive uses

### For Professional Jargon:
1. Identify the field (IT, medical, legal, etc.)
2. Provide technical translation
3. Explain for non-specialists
4. Give industry context

### For Context-Dependent Phrases:
1. Ask for clarification if ambiguous
2. Provide multiple translations for different contexts
3. Note tone differences (sarcastic, sincere, angry, etc.)

### For Untranslatable Concepts:
1. Explain why it's untranslatable
2. Provide closest approximation
3. Describe the cultural background
4. Suggest how to explain the concept

## Languages You Excel At:
**Native-level:** English ↔ Russian  
**Excellent:** Chinese, Japanese, Korean, German, French, Spanish, Italian, Polish, Ukrainian  
**Specialty:** Gen-Z slang, Internet culture, IT/Tech jargon

## Response Format:
📝 **Direct Translation:** [translation]  
💡 **Meaning:** [what it actually means]  
🌍 **Cultural Context:** [background/origin]  
🎯 **Best Equivalent:** [natural way to say it]  
📚 **Examples:** [usage examples]  
⚠️ **Notes:** [warnings or additional info]

## Tone:
- Professional but approachable
- Educational without being condescending
- Culturally sensitive and respectful
- Clear and concise

Remember: Your goal is not just translation, but true cross-cultural communication!
EOF
```

#### 2. Создание модели переводчика в Open WebUI

```bash
# Через Web UI (http://your-server-ip:3000):
# 1. Settings → Models → Create Model
# 2. Name: JARVIS-Translator
# 3. Base Model: llama3.1:8b
# 4. System Prompt: [вставить содержимое translator-prompt.txt]
# 5. Parameters:
#    - Temperature: 0.3 (для точности)
#    - Top P: 0.9
#    - Max Tokens: 4096 (для развёрнутых объяснений)
```

#### 3. Python API для переводов

Создайте скрипт для программного использования:

```bash
cat > ~/jarvis/jarvis_translator.py <<'EOF'
#!/usr/bin/env python3
"""
JARVIS Translator API
Intelligent translation with context and cultural awareness
"""

import requests
import json

OLLAMA_URL = "http://localhost:11434/api/chat"

class JarvisTranslator:
    def __init__(self, model="llama3.1:8b"):
        self.model = model
        self.system_prompt = open("translator-prompt.txt").read()
    
    def translate(self, text, from_lang="auto", to_lang="English", 
                  explain_slang=True, context="", style="neutral"):
        """
        Интеллектуальный перевод с контекстом
        
        Args:
            text: Текст для перевода
            from_lang: Исходный язык (auto для автоопределения)
            to_lang: Целевой язык
            explain_slang: Объяснять сленг и идиомы
            context: Контекст (напр. "business email", "casual chat")
            style: Стиль перевода ("formal", "casual", "slang")
        """
        
        prompt = f"""
Translate and interpret:

**Text:** "{text}"
**From:** {from_lang}
**To:** {to_lang}
**Context:** {context or "general conversation"}
**Style:** {style}
**Explain slang/idioms:** {"Yes" if explain_slang else "No"}

Provide comprehensive analysis following your format.
"""
        
        response = requests.post(OLLAMA_URL, json={
            "model": self.model,
            "messages": [
                {"role": "system", "content": self.system_prompt},
                {"role": "user", "content": prompt}
            ],
            "stream": False
        })
        
        return response.json()['message']['content']
    
    def explain_slang(self, phrase, language="English"):
        """Объяснить сленговую фразу"""
        prompt = f"""
Explain this slang/idiom in detail:

**Phrase:** "{phrase}"
**Language:** {language}

Include origin, meaning, usage examples, and cultural context.
"""
        
        response = requests.post(OLLAMA_URL, json={
            "model": self.model,
            "messages": [
                {"role": "system", "content": self.system_prompt},
                {"role": "user", "content": prompt}
            ],
            "stream": False
        })
        
        return response.json()['message']['content']
    
    def cultural_equivalent(self, phrase, from_lang, to_lang):
        """Найти культурный эквивалент идиомы"""
        prompt = f"""
Find the cultural equivalent:

**Original phrase:** "{phrase}"
**From:** {from_lang}
**To:** {to_lang}

Provide the most natural equivalent that captures the same meaning and feeling.
"""
        
        response = requests.post(OLLAMA_URL, json={
            "model": self.model,
            "messages": [
                {"role": "system", "content": self.system_prompt},
                {"role": "user", "content": prompt}
            ],
            "stream": False
        })
        
        return response.json()['message']['content']

# Пример использования
if __name__ == "__main__":
    translator = JarvisTranslator()
    
    # Пример 1: Перевод сленга
    result = translator.translate(
        text="Это полный кринж, не зашло вообще",
        from_lang="Russian",
        to_lang="English",
        context="Gen-Z discussing a movie",
        style="casual"
    )
    print("=== Перевод сленга ===")
    print(result)
    print()
    
    # Пример 2: Объяснение идиомы
    result = translator.explain_slang("no cap fr fr", "English")
    print("=== Объяснение сленга ===")
    print(result)
    print()
    
    # Пример 3: Культурный эквивалент
    result = translator.cultural_equivalent(
        phrase="спустить собак",
        from_lang="Russian",
        to_lang="English"
    )
    print("=== Культурный эквивалент ===")
    print(result)
EOF

chmod +x ~/jarvis/jarvis_translator.py
```

#### 4. Использование через командную строку

```bash
# Запуск скрипта
cd ~/jarvis
python3 jarvis_translator.py
```

---

### Примеры использования

#### **Пример 1: Молодёжный сленг**

**Запрос:**
```
Переведи на английский: "Братан, эта тема огонь! Залетаем?"
```

**JARVIS ответит:**
```
📝 Direct Translation:
"Bro, this theme/topic is fire! Are we flying in?"

💡 Actual Meaning:
"Dude, this thing is awesome! You coming?"

🌍 Cultural Context:
- "Братан" (bratan) = casual/street term for "bro"
- "Огонь" (fire) = slang for "amazing/awesome"
- "Залетаем" = slang for "let's go/join"

🎯 Best English Equivalent:
"Bro, this is fire! You in?"
or
"Dude, this slaps! You down?"

📚 Usage Examples:
- Original context: Inviting friend to an event
- Age group: 16-25
- Formality: Very casual/street

⚠️ Notes:
This is informal Russian street slang. Don't use in 
professional settings. English Gen-Z equivalent captures 
the same energy.
```

#### **Пример 2: Идиомы**

**Запрос:**
```
Что значит "break the ice" и как это сказать по-русски?
```

**JARVIS ответит:**
```
📝 Literal Translation:
"Сломать лёд" ❌ (sounds weird in Russian)

💡 Meaning:
Start a conversation in an awkward/tense situation,
make people feel more comfortable

🌍 Origin:
From ships breaking through ice to create a path.
Metaphorically: creating a path for conversation.

🎯 Russian Equivalents:
- "Растопить лёд" (literally "melt the ice")
- "Разрядить обстановку" (ease the atmosphere)
- "Начать разговор" (start conversation) - less idiomatic

📚 Examples:
English: "Let me break the ice with a joke"
Russian: "Давайте растоплю лёд с помощью шутки"
       or "Давайте разрядим обстановку шуткой"

Regional note: Americans say "break", British prefer 
"break", Russians say "растопить" (melt)
```

#### **Пример 3: IT-жаргон**

**Запрос:**
```
Explain "push to prod on Friday" to a non-developer
```

**JARVIS ответит:**
```
📝 IT Jargon Breakdown:
- "Push" = deploy/upload code
- "Prod" = production (live system users see)
- "Friday" = end of work week

💡 What It Means:
Deploying new code to the live system right before 
the weekend when most developers are unavailable.

🌍 Why It's BAD:
- If something breaks, you ruin your weekend fixing it
- Users affected when support is minimal
- Classic developer mistake

🎯 Non-Technical Equivalent:
"Like renovating your restaurant's kitchen right before 
Saturday dinner rush without backup staff"

📚 Famous Developer Saying:
"Never deploy on Friday" - universal IT wisdom
"Friday deploys are for heroes and fools"

⚠️ Cultural Note:
This is a meme in developer culture. Commonly seen:
- On developer Slack channels
- In coding memes
- Usually with a facepalm emoji 🤦‍♂️
```

#### **Пример 4: Деловой перевод**

**Запрос:**
```
Как профессионально сказать "я в отпуске, не беспокойте"?
```

**JARVIS ответит:**
```
📝 Formality Levels:

🎩 VERY FORMAL (C-level, clients):
"I am currently out of office with limited access to email. 
For urgent matters, please contact [Name] at [email]. 
I will respond to your message upon my return on [date]."

Russian: "В настоящий момент нахожусь вне офиса с 
ограниченным доступом к почте. По срочным вопросам 
обращайтесь к [Имя]. Отвечу по возвращении [дата]."

👔 STANDARD PROFESSIONAL (colleagues):
"Thanks for your email. I'm on vacation until [date] 
with limited email access. I'll get back to you when 
I return."

Russian: "Спасибо за письмо. Нахожусь в отпуске до 
[дата], отвечу по возвращении."

😊 FRIENDLY PROFESSIONAL (team):
"Hey! I'm on PTO right now and checking emails 
sporadically. I'll circle back next week!"

Russian: "Привет! Я в отпуске, почту проверяю редко. 
Отвечу на следующей неделе!"

⚠️ AVOID:
❌ "I'm on vacation, don't bother me"
❌ "Я в отпуске, отстаньте"
❌ "Read the calendar" - passive aggressive
```

---

### Практические сценарии использования

#### **Сценарий 1: Изучение языка**
```bash
# Ежедневная практика
$ python3 jarvis_translator.py

>>> Объясни разницу между "fun" и "funny"
>>> Как использовать слово "actually" правильно?
>>> Покажи примеры использования фразового глагола "put up with"
```

#### **Сценарий 2: Работа с международными клиентами**
```bash
# Адаптация тона под культуру
>>> Переведи это деловое письмо для японского клиента (учти культурные особенности)
>>> Как вежливо отказать американскому партнёру?
>>> Адаптируй презентацию для европейской аудитории
```

#### **Сценарий 3: Понимание современной культуры**
```bash
# Интернет и мемы
>>> Что значит "sigma male grindset"?
>>> Расшифруй мем "cope and seethe"
>>> Объясни почему говорят "touch grass"
```

#### **Сценарий 4: Технические документации**
```bash
# IT и tech жаргон
>>> Переведи эту техническую документацию с английского
>>> Объясни термин "technical debt" для менеджера
>>> Как сказать "code review" по-русски в IT-контексте?
```

---

### Мультиязычная конфигурация Docker

Для работы со множеством языков обновите `docker-compose.yml`:

```yaml
services:
  # ... существующие сервисы ...

  # Несколько TTS на разных языках
  piper-ru:
    image: rhasspy/wyoming-piper:latest
    container_name: jarvis-piper-ru
    ports:
      - "10200:10200"
    command: --voice ru_RU-dmitri-medium
    restart: unless-stopped

  piper-en-us:
    image: rhasspy/wyoming-piper:latest
    container_name: jarvis-piper-en-us
    ports:
      - "10201:10200"
    command: --voice en_US-lessac-medium
    restart: unless-stopped

  piper-en-gb:
    image: rhasspy/wyoming-piper:latest
    container_name: jarvis-piper-en-gb
    ports:
      - "10202:10200"
    command: --voice en_GB-alan-medium  # Британский JARVIS!
    restart: unless-stopped

  piper-de:
    image: rhasspy/wyoming-piper:latest
    container_name: jarvis-piper-de
    ports:
      - "10203:10200"
    command: --voice de_DE-thorsten-medium
    restart: unless-stopped

  # Whisper с поддержкой 99 языков (уже включено)
  whisper:
    image: onerahmet/openai-whisper-asr-webservice:latest
    environment:
      - ASR_MODEL=medium  # Лучше для мультиязычности
```

---

### Ограничения и предупреждения

#### ⚠️ **НЕ ЗАМЕНЯЕТ профессиональных переводчиков для:**

1. **Юридических документов**
   - Контракты, судебные документы
   - Требуют присяжного переводчика
   - JARVIS: "Я могу дать общее понимание, но это не имеет юридической силы"

2. **Медицинских документов**
   - Диагнозы, рецепты, результаты анализов
   - Критично для здоровья
   - JARVIS: "Обратитесь к медицинскому переводчику"

3. **Официальных документов**
   - Сертификаты, дипломы, свидетельства
   - Требуют нотариального заверения

#### ✅ **ОТЛИЧНО для:**
- Неформального общения
- Изучения языка
- Понимания культурных различий
- Деловой переписки (неофициальной)
- Интерпретации сленга и мемов
- Технической документации
- Личного использования

---

### Советы по эффективному использованию

1. **Будьте конкретны с контекстом:**
   ```
   ❌ "Переведи это"
   ✅ "Переведи для делового письма клиенту в США"
   ✅ "Переведи сленг для друзей 20-25 лет"
   ```

2. **Спрашивайте объяснения:**
   ```
   ✅ "Объясни почему именно так перевёл"
   ✅ "Дай альтернативные варианты"
   ✅ "Покажи примеры использования"
   ```

3. **Указывайте региональные предпочтения:**
   ```
   ✅ "Переведи на британский английский"
   ✅ "Используй американский сленг"
   ✅ "Адаптируй для канадской аудитории"
   ```

4. **Проверяйте критичные переводы:**
   ```
   ✅ Используйте несколько источников для важных текстов
   ✅ Попросите носителя языка проверить
   ✅ Для юридических/медицинских - профессионал!
   ```

---

### Дополнительные ресурсы

**Модели для разных языков:**
- `llama3.1:8b` - универсал для европейских языков
- `qwen2.5:7b` - лучше для азиатских языков + русский
- `aya:8b` - специально для 23 языков (включая редкие)

**TTS голоса:**
- Полный список: https://rhasspy.github.io/piper-samples/
- Качество голосов: https://github.com/rhasspy/piper

**Дополнительные промпты:**
```bash
# В ~/jarvis/ можно создать специализированные промпты:
translator-prompt.txt          # Основной переводчик
slang-interpreter-prompt.txt   # Специалист по сленгу
business-translator-prompt.txt # Деловые переводы
tech-jargon-prompt.txt        # IT термины
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

**Перевод и интерпретация сленга:**
```
User: Что значит "no cap fr fr" на английском?
JARVIS: 📝 "No cap" означает "без обмана, правда, честно". 
"Fr fr" - сокращение от "for real for real" (серьёзно, по-настоящему).

💡 Вместе это значит "честно-честно, без шуток, реально".

🎯 Русский эквивалент: "Реально, без шуток" или "Чисто по факту"

📚 Пример: "This pizza is fire, no cap fr fr" = 
"Эта пицца огонь, реально без шуток"
```

**Культурные эквиваленты:**
```
User: Как сказать "авось" на английском?
JARVIS: 💡 "Авось" - уникальное русское слово без прямого перевода.

Это философия надежды на удачу без планирования.

🎯 Возможные варианты:
- "With any luck" (с удачей)
- "Hopefully it'll work out"
- "God willing" (если повезёт)

🌍 Но лучше объяснить концепт: "A Russian attitude 
of hoping things will work out without planning"
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

### Можно ли использовать JARVIS как переводчик?

Да! JARVIS - отличный переводчик с уникальными возможностями:
- ✅ Понимает контекст и культурные нюансы
- ✅ Интерпретирует сленг и идиомы
- ✅ Объясняет "почему именно так"
- ✅ Поддерживает 30+ языков
- ✅ Полностью локально (приватность)

Подробнее см. раздел [JARVIS как переводчик](#jarvis-как-переводчик).

### Лучше ли JARVIS чем Google Translate?

**Для обычных переводов:** Google Translate быстрее и точнее для простых фраз.

**Где JARVIS лучше:**
- 🔥 Сленг и жаргон (объясняет контекст)
- 🔥 Идиомы и фразеологизмы
- 🔥 Культурные нюансы
- 🔥 Адаптация тональности
- 🔥 Объяснения и примеры
- 🔥 Профессиональная терминология
- 🔥 Приватность (всё локально)

### Какие языки поддерживает переводчик?

**Отлично (95%+ качество):**
Русский, Английский, Немецкий, Французский, Испанский, Итальянский, Польский, Украинский

**Хорошо (80-90%):**
Китайский, Японский, Корейский, Турецкий, Голландский, Шведский, Португальский

**Всего:** 30+ языков

**STT (Whisper):** 99 языков с автоопределением  
**TTS (Piper):** 40 языков с разными голосами

### Может ли JARVIS переводить технический жаргон?

Да! JARVIS отлично справляется с:
- 💻 IT и программирование
- ⚕️ Медицинская терминология
- ⚖️ Юридический язык
- 📊 Бизнес и финансы
- 🔬 Научные термины

Плюс он объясняет термины простым языком для не-специалистов.

### Подходит ли для юридических документов?

⚠️ **НЕТ!** Для юридических документов нужен присяжный переводчик.

JARVIS может:
- ✅ Дать общее понимание контракта
- ✅ Объяснить юридические термины
- ✅ Помочь с деловой перепиской

НО перевод **не имеет юридической силы** и не может использоваться в суде или для официальных документов.

### Можно ли использовать JARVIS для изучения языка?

**Да, это одно из лучших применений!**

JARVIS помогает:
- 📚 Объясняет грамматику и использование
- 🗣️ Показывает примеры в контексте
- 🎯 Различает формальный/неформальный стиль
- 🌍 Объясняет культурные различия
- ✍️ Корректирует ошибки с объяснениями
- 🔄 Даёт несколько вариантов перевода

Интерактивное обучение с мгновенной обратной связью!

---

## 📚 Полезные ссылки

### Основные компоненты:
- [Ollama Documentation](https://github.com/ollama/ollama)
- [Open WebUI](https://github.com/open-webui/open-webui)
- [Available Models](https://ollama.com/library)
- [Whisper by OpenAI](https://github.com/openai/whisper)
- [Piper TTS](https://github.com/rhasspy/piper)

### Для переводчика:
- [Piper Voice Samples](https://rhasspy.github.io/piper-samples/) - примеры голосов на разных языках
- [Whisper Language Support](https://github.com/openai/whisper#available-models-and-languages) - список из 99 поддерживаемых языков
- [Llama 3.1 Languages](https://ai.meta.com/blog/meta-llama-3-1/) - информация о языковой поддержке
- [Qwen 2.5 Models](https://github.com/QwenLM/Qwen2.5) - для азиатских языков

### Обучающие материалы:
- [Prompt Engineering Guide](https://www.promptingguide.ai/) - как писать эффективные промпты
- [LLM Translation Best Practices](https://github.com/anthropics/anthropic-cookbook) - лучшие практики перевода с LLM

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
