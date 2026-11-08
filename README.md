# NEXO IA — Asistente de analítica web con IA

🥈 **Segundo lugar — Reto de Inteligencia Artificial, Hackathon Manizales (marzo de 2026)**, organizada por el Ministerio TIC (programa Talento Tech) y la Unión Temporal IU Training (Universidad de Antioquia, Universidad de Caldas y Ubicua Technology).

NEXO IA es un asistente de inteligencia de negocios que interpreta métricas de comportamiento web (rage clicks, dead clicks, engagement, duración de sesión, país y dispositivo) y responde en lenguaje natural para apoyar decisiones de marketing y UX.

## Cómo funciona

1. **Respuestas fijas primero.** Una base de conocimiento local responde al instante las preguntas frecuentes, sin llamar a ningún modelo.
2. **Datos en vivo como contexto.** Para el resto de preguntas, el backend consulta las tablas de métricas en Supabase (con caché de 2 minutos) e incorpora esos datos y la fecha actual al contexto del modelo.
3. **Dos modelos en paralelo.** La pregunta se envía al mismo tiempo a GPT (OpenAI) y a Gemini; se devuelve la primera respuesta válida. Si un proveedor falla o tarda, el otro responde.
4. **Voz y Telegram.** El usuario puede dictar la pregunta (transcripción con Whisper) o consultar desde un bot de Telegram con acceso restringido.

## Stack

| Capa | Tecnología |
| --- | --- |
| Backend | Python, Flask, Flask-Limiter, Flask-CORS |
| Modelos | OpenAI (GPT y Whisper), Google Gemini |
| Datos | Supabase (API REST) |
| Canales | Web (HTML, CSS y JavaScript) y bot de Telegram (pyTelegramBotAPI) |

## Estructura

```
app.py             # API Flask: login, /chat, /transcribe y consulta a Supabase
ai_clients.py      # Clientes de OpenAI y Gemini
knowledge_base.py  # Respuestas fijas (capa 1)
telegram_bot.py    # Bot de Telegram que reutiliza la misma lógica
templates/         # Vistas de login y chat
static/            # Estilos y scripts del frontend
test_keys.py       # Prueba de conexión con los proveedores de IA
```

## Ejecución local

```bash
python -m venv .venv
# Windows: .\.venv\Scripts\Activate.ps1   |   Linux/macOS: source .venv/bin/activate
pip install -r requirements.txt
python app.py
```

La aplicación queda disponible en `http://localhost:5000`.

### Variables de entorno (`.env`, nunca se sube al repositorio)

| Variable | Uso |
| --- | --- |
| `OPENAI_API_KEY` | GPT y transcripción con Whisper |
| `GEMINI_API_KEY` | Gemini |
| `SUPABASE_URL`, `SUPABASE_KEY` | Fuente de métricas |
| `ADMIN_USER`, `ADMIN_PASS` | Acceso a la interfaz web (`ADMIN_PASS` es obligatoria) |
| `SECRET_KEY` | Firma de la sesión de Flask (obligatoria) |
| `TELEGRAM_TOKEN` | Token del bot de Telegram |
| `TELEGRAM_BOT_PASSWORD`, `TELEGRAM_ALLOWED_USERS` | Control de acceso al bot (sin contraseña, el bot no se inicia) |
| `RACE_TIMEOUT`, `PORT`, `FLASK_DEBUG` | Opcionales: tiempo de espera de los modelos, puerto y modo debug (`1` solo en desarrollo) |

## Endpoints

| Método | Ruta | Descripción |
| --- | --- | --- |
| GET/POST | `/login` | Inicio de sesión (máximo 3 sesiones activas) |
| GET | `/` | Interfaz de chat |
| POST | `/chat` | Recibe `{ "message": "..." }` y responde `{ "source", "response", "time_taken" }` |
| POST | `/transcribe` | Recibe un audio y devuelve el texto transcrito |

## Seguridad

- Las credenciales se leen solo desde variables de entorno, sin valores por defecto: la aplicación no arranca sin `ADMIN_PASS` ni `SECRET_KEY`, y el bot no se inicia sin contraseña.
- El modo debug está desactivado salvo que se active explícitamente con `FLASK_DEBUG=1`.
- Límite de peticiones por IP (200 por día y 50 por hora) y tamaño máximo de carga de 5 MB.
- CORS restringido a dominios permitidos.

## Autor

Jhoann Esteban Reyes Higuera — [github.com/LDking777](https://github.com/LDking777)
