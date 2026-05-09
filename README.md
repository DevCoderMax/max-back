# Plano de Backend MVP — MAX

Backend em Python para um **super assistente pessoal** com texto, voz, calendário, tarefas, tempo de tela, contexto e sincronização entre celular e PC.

---

## 1. Visão geral

O MAX não deve ser apenas um app de tarefas.  
Ele deve funcionar como um **backend de eventos pessoais**, onde tudo que acontece vira dado para o assistente tomar decisões.

Exemplo:

```txt
Usuário abriu Instagram
Usuário ficou 45 minutos
Existe tarefa pendente
Horário atual é de trabalho
Personalidade atual = Focus
        ↓
Context Engine analisa
        ↓
Decision Engine decide
        ↓
MAX fala ou notifica:
"Você quer ativar o modo foco por 25 minutos?"
```

---

## 2. Stack principal

### Backend

```txt
FastAPI
Uvicorn/Gunicorn
Pydantic
SQLAlchemy 2
Alembic
PostgreSQL
Redis
Celery ou Dramatiq
JWT/Auth
WebSocket
```

### IA

```txt
OpenAI API
Google Gemini API
Anthropic API
pgvector
tiktoken
httpx
```

### Ferramentas auxiliares

```txt
python-dateutil
pendulum
orjson
loguru ou structlog
pytest
pytest-asyncio
ruff
mypy
```

---

## 3. Arquitetura geral

```txt
Flutter Mobile/Desktop
        ↓
FastAPI Gateway
        ↓
PostgreSQL  ← dados principais
Redis       ← cache, sessões, filas leves
Worker IA   ← análise de contexto
Worker Jobs ← lembretes, notificações, resumo diário
WebSocket   ← sincronização em tempo real
```

---

## 4. Estrutura de pastas

```txt
max_backend/
  app/
    main.py

    core/
      config.py
      security.py
      database.py
      redis.py
      logging.py
      exceptions.py

    modules/
      auth/
        router.py
        schemas.py
        models.py
        service.py
        repository.py

      users/
        router.py
        schemas.py
        models.py
        service.py
        repository.py

      devices/
        router.py
        schemas.py
        models.py
        service.py
        repository.py

      assistant/
        router.py
        schemas.py
        models.py
        service.py
        prompts.py

      tasks/
        router.py
        schemas.py
        models.py
        service.py
        repository.py

      calendar/
        router.py
        schemas.py
        models.py
        service.py
        integrations.py

      reminders/
        router.py
        schemas.py
        models.py
        service.py
        scheduler.py

      activity/
        router.py
        schemas.py
        models.py
        service.py
        analyzer.py

      screen_time/
        router.py
        schemas.py
        models.py
        service.py

      notifications/
        router.py
        schemas.py
        models.py
        service.py
        push.py

      voice/
        router.py
        schemas.py
        models.py
        service.py

      memory/
        router.py
        schemas.py
        models.py
        service.py
        vector_store.py

      realtime/
        websocket.py
        manager.py
        events.py

      ai/
        router.py
        schemas.py
        service.py
        context_engine.py
        decision_engine.py
        llm_client.py

    workers/
      celery_app.py
      tasks.py
      reminder_jobs.py
      ai_jobs.py
      notification_jobs.py

    migrations/
    tests/
```

---

## 5. Padrão por módulo

Cada módulo deve seguir uma separação clara:

```txt
router.py      → endpoints da API
schemas.py     → modelos Pydantic
models.py      → modelos SQLAlchemy
service.py     → regra de negócio
repository.py  → acesso ao banco
```

Exemplo:

```txt
tasks/
  router.py
  schemas.py
  models.py
  service.py
  repository.py
```

---

## 6. Banco de dados

Banco principal recomendado:

```txt
PostgreSQL
```

Motivos:

- confiável;
- escalável;
- ótimo para dados relacionais;
- permite usar `pgvector` no futuro;
- bom para calendário, tarefas e eventos.

---

## 7. Tabelas principais

```txt
users
devices
assistant_profiles
assistant_settings
tasks
calendar_events
reminders
activity_events
screen_time_logs
notification_rules
notifications
memory_entries
conversation_sessions
conversation_messages
voice_settings
integration_accounts
audit_logs
```

---

## 8. Módulo de usuários

### Tabela: users

```txt
id
name
email
password_hash
timezone
locale
created_at
updated_at
is_active
```

### Endpoints

```txt
GET /users/me
PATCH /users/me
GET /users/me/settings
PATCH /users/me/settings
```

---

## 9. Autenticação

MVP:

```txt
email + senha
JWT access token
refresh token
```

### Bibliotecas

```txt
python-jose[cryptography]
passlib[bcrypt]
```

### Endpoints

```txt
POST /auth/register
POST /auth/login
POST /auth/refresh
POST /auth/logout
GET /auth/me
```

---

## 10. Módulo de dispositivos

Cada celular, PC ou navegador conectado deve ser registrado como dispositivo.

### Tabela: devices

```txt
id
user_id
name
platform
push_token
last_seen_at
app_version
is_active
created_at
```

### Plataformas possíveis

```txt
android
ios
windows
linux
macos
web
```

### Endpoints

```txt
POST /devices/register
GET /devices
PATCH /devices/{id}
DELETE /devices/{id}
```

---

## 11. Módulo de tarefas

### Tabela: tasks

```txt
id
user_id
title
description
status
priority
due_at
estimated_minutes
source
created_by_ai
created_at
updated_at
```

### Status

```txt
pending
in_progress
completed
cancelled
archived
```

### Prioridades

```txt
low
medium
high
urgent
```

### Endpoints

```txt
POST /tasks
GET /tasks
GET /tasks/today
GET /tasks/{id}
PATCH /tasks/{id}
DELETE /tasks/{id}
POST /tasks/{id}/complete
```

---

## 12. Módulo de calendário

No MVP, criar calendário interno primeiro.  
Depois integrar com Google Calendar, Outlook e calendário local do celular.

### Tabela: calendar_events

```txt
id
user_id
title
description
starts_at
ends_at
location
source
external_id
created_at
updated_at
```

### Endpoints

```txt
POST /calendar/events
GET /calendar/events
GET /calendar/today
GET /calendar/events/{id}
PATCH /calendar/events/{id}
DELETE /calendar/events/{id}
```

---

## 13. Módulo de lembretes

### Tabela: reminders

```txt
id
user_id
task_id
calendar_event_id
title
message
remind_at
status
delivery_type
created_at
```

### Delivery types

```txt
push
voice
email
in_app
```

### Endpoints

```txt
POST /reminders
GET /reminders
GET /reminders/today
PATCH /reminders/{id}
DELETE /reminders/{id}
```

---

## 14. Tempo de tela

Esse é um dos grandes diferenciais do MAX.

### Tabela: screen_time_logs

```txt
id
user_id
device_id
app_name
package_name
started_at
ended_at
duration_seconds
category
created_at
```

### Categorias

```txt
social
work
study
entertainment
game
system
unknown
```

### Endpoints

```txt
POST /screen-time/logs
GET /screen-time/summary
GET /screen-time/today
GET /screen-time/by-app
```

---

## 15. Eventos de atividade

Além do tempo de tela, tudo deve poder virar evento.

### Tabela: activity_events

```txt
id
user_id
device_id
event_type
payload
created_at
```

### Exemplos de event_type

```txt
app_opened
app_closed
screen_unlocked
screen_locked
notification_received
focus_started
focus_ended
task_created
task_completed
voice_command
assistant_message
```

### Endpoint principal

```txt
POST /events/ingest
```

Exemplo de payload:

```json
{
  "device_id": "device_123",
  "event_type": "app_opened",
  "payload": {
    "app_name": "Instagram",
    "package_name": "com.instagram.android"
  }
}
```

---

## 16. Personalidades da assistente

O usuário pode escolher ou personalizar como a assistente se comporta.

### Tabela: assistant_profiles

```txt
id
user_id
name
style
proactivity_level
humor_level
formality_level
interruption_level
voice_id
is_default
created_at
```

### Perfis iniciais

```txt
Focus
Executive
Companion
Ghost
Coach
Custom
```

### Exemplo

```json
{
  "name": "Focus",
  "proactivity_level": 8,
  "humor_level": 1,
  "formality_level": 6,
  "interruption_level": 4
}
```

---

## 17. Configurações da assistente

### Tabela: assistant_settings

```txt
id
user_id
wake_word
speak_enabled
proactive_enabled
quiet_hours_start
quiet_hours_end
max_daily_interventions
created_at
updated_at
```

### Exemplo

```json
{
  "wake_word": "Nexa",
  "speak_enabled": true,
  "proactive_enabled": true,
  "quiet_hours_start": "22:00",
  "quiet_hours_end": "07:00",
  "max_daily_interventions": 12
}
```

---

## 18. Motor de contexto

Arquivos:

```txt
ai/context_engine.py
ai/decision_engine.py
```

O Context Engine recebe:

```txt
tarefas
calendário
tempo de tela
últimos eventos
horário atual
personalidade ativa
preferências
histórico
```

E entrega um contexto pronto para decisão.

---

## 19. Motor de decisão

O Decision Engine decide:

```txt
falar
não falar
notificar silenciosamente
sugerir ação
criar tarefa
reorganizar agenda
ativar modo foco
```

Exemplo de retorno:

```json
{
  "action": "suggest_focus",
  "priority": "medium",
  "message": "Você já passou bastante tempo no Instagram. Quer que eu ative o modo foco por 25 minutos?",
  "delivery": "voice_and_push"
}
```

---

## 20. Memória

No MVP, memória simples em PostgreSQL.  
Depois, memória vetorial com pgvector.

### Tabela: memory_entries

```txt
id
user_id
type
content
importance
source
created_at
updated_at
```

### Tipos

```txt
preference
routine
habit
fact
behavior_pattern
```

### Exemplos

```txt
Usuário costuma estudar melhor pela manhã.
Usuário prefere lembretes por voz.
Usuário não gosta de ser interrompido enquanto joga.
Usuário costuma procrastinar em redes sociais à noite.
```

---

## 21. Conversas

### Tabela: conversation_sessions

```txt
id
user_id
device_id
started_at
ended_at
created_at
```

### Tabela: conversation_messages

```txt
id
session_id
user_id
role
content
created_at
```

### Roles

```txt
user
assistant
system
tool
```

---

## 22. Notificações

### Tabela: notifications

```txt
id
user_id
device_id
title
body
speak_text
priority
status
created_at
sent_at
read_at
```

### Status

```txt
pending
sent
read
failed
cancelled
```

### Endpoints

```txt
GET /notifications
POST /notifications/send-test
PATCH /notifications/{id}/read
DELETE /notifications/{id}
```

---

## 23. Voz

No MVP, o backend envia o texto e o app Flutter usa TTS local.

Fluxo inicial:

```txt
Backend decide:
"Você tem reunião em 10 minutos"

Flutter recebe:
speak_text

Flutter fala usando TTS local.
```

Depois:

```txt
Backend gera áudio com API externa
Flutter apenas toca o arquivo
```

### Tabela: voice_settings

```txt
id
user_id
voice_provider
voice_id
speed
pitch
language
created_at
updated_at
```

---

## 24. WebSocket

Usar WebSocket para sincronização em tempo real entre PC e celular.

### Canais

```txt
/ws/sync
/ws/assistant
/ws/devices
```

### Exemplos de eventos

```json
{
  "type": "task_created",
  "payload": {
    "id": "task_123",
    "title": "Estudar Flutter"
  }
}
```

```json
{
  "type": "assistant_message",
  "payload": {
    "text": "Sua próxima tarefa começa em 10 minutos.",
    "speak": true
  }
}
```

---

## 25. Workers

Usar workers para tarefas que não devem travar a API.

### Responsabilidades

```txt
enviar lembretes
analisar tempo de tela
gerar resumo diário
rodar IA em background
enviar notificações
sincronizar integrações
```

### Arquivos

```txt
workers/
  celery_app.py
  reminder_jobs.py
  ai_jobs.py
  notification_jobs.py
```

---

## 26. Celery ou Dramatiq?

### Celery

Vantagens:

```txt
maduro
muito usado
bom para produção
muitos recursos
bom para tarefas agendadas
```

### Dramatiq

Vantagens:

```txt
mais simples
mais limpo
ótimo para MVP
boa performance
menos burocrático
```

### Recomendação

```txt
MVP rápido: Dramatiq + Redis
Produto robusto: Celery + Redis/RabbitMQ
```

---

## 27. API inicial do MVP

```txt
/auth/register
/auth/login
/auth/me

/users/me
/users/me/settings

/devices/register
/devices
/devices/{id}

/assistant/profile
/assistant/profile/custom
/assistant/chat
/assistant/suggest
/assistant/context

/tasks
/tasks/today
/tasks/{id}/complete

/calendar/events
/calendar/today

/reminders
/reminders/{id}

/events/ingest

/screen-time/logs
/screen-time/summary
/screen-time/today

/notifications
/notifications/send-test

/ws/sync
/ws/assistant
```

---

## 28. Fases do MVP

### Fase 1 — Base

```txt
Auth
Usuários
Dispositivos
Tarefas
Calendário interno
Lembretes
Notificações simples
```

### Fase 2 — Assistente

```txt
Chat com IA
Personalidade
Configuração de tom
Memória simples
Sugestões baseadas em tarefas
```

### Fase 3 — Contexto real

```txt
Tempo de tela
Apps usados
Eventos do celular
Eventos do PC
Análise de hábitos
```

### Fase 4 — Proatividade

```txt
Assistente decide quando falar
Resumo do dia
Modo foco
Sugestão de rotina
Reorganização de agenda
```

### Fase 5 — Ecossistema

```txt
Google Calendar
Microsoft Outlook
Email
WhatsApp futuramente
Integração desktop
Comandos no PC
Automações locais
```

---

## 29. Docker Compose inicial

```yaml
services:
  api:
    build: .
    command: uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
    ports:
      - "8000:8000"
    env_file:
      - .env
    depends_on:
      - db
      - redis

  db:
    image: postgres:16
    environment:
      POSTGRES_DB: maxdb
      POSTGRES_USER: max
      POSTGRES_PASSWORD: maxpass
    ports:
      - "5432:5432"
    volumes:
      - max_postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:7
    ports:
      - "6379:6379"

  worker:
    build: .
    command: celery -A app.workers.celery_app worker --loglevel=info
    env_file:
      - .env
    depends_on:
      - redis
      - db

volumes:
  max_postgres_data:
```

---

## 30. Dependências iniciais

Arquivo `requirements.txt` sugerido:

```txt
fastapi
uvicorn[standard]
gunicorn
pydantic
pydantic-settings
sqlalchemy[asyncio]
asyncpg
alembic
redis
celery
python-jose[cryptography]
passlib[bcrypt]
python-multipart
httpx
orjson
loguru
python-dateutil
pendulum
pytest
pytest-asyncio
ruff
mypy
openai
```

---

## 31. Variáveis de ambiente

Arquivo `.env`:

```env
APP_NAME=MAX
APP_ENV=development
APP_DEBUG=true

DATABASE_URL=postgresql+asyncpg://max:maxpass@db:5432/maxdb
REDIS_URL=redis://redis:6379/0

JWT_SECRET_KEY=change-me
JWT_ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=30
REFRESH_TOKEN_EXPIRE_DAYS=30

OPENAI_API_KEY=
GEMINI_API_KEY=

DEFAULT_TIMEZONE=America/Recife
```

---

## 32. Prioridade de desenvolvimento

Ordem recomendada:

```txt
1. Criar projeto FastAPI
2. Configurar PostgreSQL
3. Configurar SQLAlchemy async
4. Configurar Alembic
5. Criar auth
6. Criar users
7. Criar devices
8. Criar tasks
9. Criar calendar
10. Criar reminders
11. Criar events ingest
12. Criar screen time
13. Criar assistant profiles
14. Criar notifications
15. Criar context engine
16. Criar chat com IA
17. Criar WebSocket sync
18. Criar workers
```

---

## 33. Princípio principal

O backend do MAX deve seguir esta ideia:

```txt
Tudo é evento.
Todo evento gera contexto.
Todo contexto pode gerar uma decisão.
Toda decisão pode virar fala, notificação ou ação.
```

Esse é o ponto que diferencia o MAX de um simples app de calendário.

---

## 34. Frase-guia

> MAX não é uma agenda com IA.  
> MAX é um sistema pessoal de contexto, rotina e ação.
