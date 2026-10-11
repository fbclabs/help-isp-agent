# Crear el bot de Telegram (5 min)

1. En Telegram, busca **@BotFather** (con la palomita azul) y abre el chat.
2. Envía `/newbot`.
3. Nombre visible: `Fibra Soporte` (el que quieras).
4. Usuario del bot: debe terminar en `bot`, por ejemplo `fibra_help_isp_bot`.
5. BotFather responde con un **token** como `123456789:AAH...`. Cópialo.
6. Pégalo en `.env` (nunca en el repo):
   ```
   TELEGRAM_BOT_TOKEN=123456789:AAH...
   ```
7. Ejecuta desde la raíz: `boxlang src/telegram.bxs`
8. En Telegram abre tu bot, pulsa **Iniciar** (`/start`) y escribe tu problema.

- Las aprobaciones humanas aparecen en la consola donde corre `telegram.bxs` (consola del operador).
- `/nuevo` reinicia la conversación de ese chat.
- Solo puede correr **una** instancia del bot a la vez (si no, Telegram responde error 409).
