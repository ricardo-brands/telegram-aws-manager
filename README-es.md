# Súper Bot DevOps: Gestión de AWS y Servidores vía Telegram con n8n + Gemini IA

Un flujo avanzado construido en **n8n** que transforma Telegram en un centro de mando completo para operaciones de DevOps. 

Este proyecto combina el control estricto y determinista de la infraestructura en la nube (AWS EC2) con la flexibilidad de la Inteligencia Artificial (Google Gemini) para la gestión interna del sistema operativo Linux vía SSH, utilizando lenguaje natural.

---

## 🎯 ¿Qué hace este bot?

El flujo opera en dos frentes principales:

1. **Gestión de Infraestructura AWS (Programada y Segura):**
   Comandos exactos que llaman a la API de AWS para gestionar instancias EC2. No hay margen para alucinaciones de IA aquí.
   - Encender, Apagar y Reiniciar instancias.
   - Verificar Estado (IP, Estado actual, Hardware).
   - Cambiar el tamaño de la máquina (Ej: `t2.micro` a `t3.medium`) de forma automatizada con la máquina detenida.

2. **Gestión de SO vía SSH (Potenciado por IA):**
   Acceso seguro al terminal Linux donde el usuario pide lo que quiere en *lenguaje natural*. La IA lo traduce a un comando seguro, n8n lo ejecuta vía SSH, y la IA "mastica" el log técnico, devolviendo una respuesta amigable en Telegram.
   - Ej: *"¿Cómo está el consumo de memoria?"* -> Ejecuta `free -m`.
   - Ej: *"¿Qué sitios están configurados en Apache?"* -> Lista el directorio `sites-enabled`.
   - Ej: *"Reinicia el contenedor de la base de datos"* -> Ejecuta `docker-compose restart`.

---

## ⚙️ Arquitectura y Lógica de Nodos (n8n)

El flujo fue diseñado con el concepto de **Enrutamiento Maestro**, separando comandos rígidos de peticiones de IA.

### 🛡️ 1. Capa de Entrada y Seguridad
* **Telegram Trigger:** Escucha todas las notas enviadas al bot.
* **Verificar Usuarios (If):** Un filtro de seguridad vital. El flujo solo avanza si el `chat.id` del remitente está en la lista de IDs autorizados. Las peticiones de extraños son ignoradas.

### 🧠 2. El Cerebro Enrutador
* **Cerebro Lógico (Code Node):** El maestro de la orquesta. Analiza el primer término del mensaje. Si es un comando de infraestructura (ej: `/ligar`, `/mudar`), enruta hacia AWS. Si es `/ajuda`, muestra el menú. Si es cualquier texto en lenguaje natural, lo clasifica como `ssh` y lo envía a la Inteligencia Artificial.
* **Enrutador Maestro (Switch):** Distribuye físicamente el flujo a los 6 caminos posibles basándose en la decisión del Cerebro Lógico.

### ☁️ 3. Brazo de Infraestructura (API de AWS EC2)
* **Nodos HTTP Request (AWS Ligar, Desligar, etc.):** Autenticados vía AWS IAM, envían peticiones seguras vía `POST` (`form-urlencoded`) a la API de EC2.
* **Respuesta: Estado / Acción Ejecutada (Telegram):** Extrae datos del XML devuelto por AWS (usando Regex) y devuelve el estado real o la confirmación de éxito al usuario.

### 🤖 4. Brazo de Sistema Operativo (SSH + Gemini IA)
* **IA - Entiende el Pedido (Chain LLM):** Recibe texto libre del usuario y lo evalúa bajo un prompt de *Seguridad Máxima*. Intenta mapear la intención a una lista estricta de comandos Linux predefinidos. Si la intención es destructiva o fuera de alcance, devuelve "ERROR".
* **¿Comando Permitido? (If):** Verifica si la IA generó un comando válido o bloqueó la ejecución.
* **SSH - Ejecutar:** Accede al servidor vía clave privada (autenticación segura) y ejecuta el terminal.
* **IA - Explica el Resultado:** Toma el `stdout` (log técnico de Linux) y usa Gemini nuevamente para traducir la jerga técnica en un resumen claro, en español, formateado para Telegram (sin usar asteriscos de lista para no romper la API del mensajero).
* **Enviar Respuesta IA (Telegram):** Entrega el diagnóstico final al usuario.

---

## 🛠️ Requisitos previos

Para importar y ejecutar este flujo en tu instancia n8n, necesitarás:

1. **Token del Bot de Telegram:** Creado vía *BotFather*.
2. **Credenciales AWS IAM:** Un usuario de AWS con políticas restringidas (Least Privilege) para `ec2:DescribeInstances`, `ec2:StartInstances`, `ec2:StopInstances`, `ec2:RebootInstances`, y `ec2:ModifyInstanceAttribute`.
3. **Clave Privada SSH:** Para acceso al servidor objetivo.
4. **Google Gemini API Key:** Para el procesamiento de lenguaje natural (El plan gratuito de Google AI Studio es suficiente).
5. (Opcional) **n8n v1.0+**: El flujo utiliza los nodos actualizados `Switch` y `LangChain`.

---

## 💡 Cómo Configurar

1. Importa el archivo `workflow.json` en tu n8n.
2. Añade tus credenciales en todos los nodos correspondientes (Telegram, AWS, SSH, Google Gemini).
3. En el nodo **Verificar Usuarios**, reemplaza el valor `YOUR_CHAT_ID_HERE` con tu Telegram Chat ID (y añade los IDs de tu equipo).
4. En el nodo **Cerebro Lógico**, cambia la variable `instance_id` al ID de tu máquina EC2 y configura tu `regiao` (ej: `sa-east-1`).
5. En el nodo **IA - Entiende el Pedido**, edita el Prompt para incluir las rutas reales de tus contenedores Docker, servicios o scripts que la IA tiene permitido ejecutar.

---

## 💬 Ejemplos de Uso en Telegram

**Para Infraestructura:**
> `/status` -> Devuelve Estado, IP y Tipo de Hardware.  
> `/desligar` -> Detiene la máquina para mantenimiento.  
> `/mudar t3.medium` -> Actualiza el tamaño del servidor.

**Para el Sistema Operativo:**
> "¿Cómo está el espacio en disco?" -> *La IA ingresa, ejecuta `df -h` y responde amigablemente.* > "¿Qué dominios están corriendo en Apache?" -> *La IA ejecuta la revisión de vhosts y lista los sitios.* > "Reinicia el contenedor del ERP por favor" -> *La IA encuentra la ruta correcta, ejecuta docker-compose restart y avisa cuando vuelve.*

---

## 🤝 Contribuir
Siéntete libre de hacer un fork del proyecto, crear pull requests con nuevos comandos, o mejorar los prompts de protección de la IA.

Desarrollado para simplificar la vida de SysAdmins e Ingenieros DevOps. ¡A por todas! 💀
