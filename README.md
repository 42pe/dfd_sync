<div align="center">

# dfd_sync

**Respaldo automático de tus carpetas a Google Drive, configurado por tu agente de IA.**

Claude Code · ChatGPT · cualquier agente con terminal

</div>

---

## 🚀 Empieza aquí

> [!TIP]
> **Copia este prompt, pégalo en tu agente de IA y empieza a usarlo:**
>
> ```text
> Revisa https://github.com/42pe/dfd_sync y ayúdame a configurar el skill de backups automáticos en Google Drive.
> ```
>
> El agente instala el skill, te explica cómo funciona en palabras simples, revisa los permisos y deja tu primera carpeta respaldándose sola.

> [!WARNING]
> **Sin garantías.** Es una herramienta personal y gratuita. Hace lo posible por copiar tus archivos, pero puede fallar sin avisarte. **No reemplaza un backup profesional**: revisa de vez en cuando que tus archivos estén en Google Drive.

---

📖 **La historia de cómo se hizo y cómo funciona, en palabras simples (en inglés):** [STORY.md](STORY.md) · [blog post](https://diegoferreyra.com/writing/a-backup-skill-your-ai-agent-installs)

## ✨ Qué hace

| | |
|---|---|
| ☁️ **Copia en Drive** | Sube cada carpeta configurada a `Backups/<carpeta>` en tu Google Drive. La copia va en una sola dirección, de tu Mac a Drive, y los archivos se abren desde la web o el celular. |
| ⏰ **Automático** | Una tarea de tu Mac se despierta cada hora. Cada carpeta decide si le toca según su frecuencia (por defecto cada 12 horas, como mínimo cada hora). |
| 🕓 **Historial** | Cada archivo que cambia o se borra se guarda en `Backups/_archive/<carpeta>/<fecha-hora>/` antes de subir la versión nueva. |
| 🧹 **Se limpia solo** | Borra el historial de más de 90 días y, si pasa de 1 GB, borra empezando por lo más viejo. La versión más reciente nunca se borra por tamaño. |

## 💬 Uso diario

Pídele a tu agente que use el skill desde la carpeta que quieres respaldar. En Claude Code basta con escribir:

```text
/dfd-bkp
```

| Situación | Qué hace el skill |
|---|---|
| **Primera vez** | Te avisa que no hay garantías, te explica el sistema, revisa los permisos (rclone, Google Drive, la tarea automática), te ayuda a elegir en qué zonas de tu Mac buscar y configura la carpeta actual. |
| **Carpeta nueva** | Crea sus ajustes, te pregunta cada cuánto copiarla (sugiere 12 horas) y te propone qué carpetas no subir. |
| **Carpeta ya configurada** | Te muestra el link a Drive, la última y la próxima copia, si está pausada y cuántas carpetas excluye. Puedes pausarla, cambiar los ajustes o fijar una hora específica. |

## ⚙️ Configuración

Hay dos archivos de texto, con una línea `CLAVE=valor` por ajuste. El agente los crea y los edita por ti.

<details>
<summary><b><code>~/.dfd_bkp_sync</code></b>: global (zonas donde buscar, valores por defecto y última corrida)</summary>

```ini
FOLDER=~/Proyectos              # una línea por carpeta madre
ARCHIVE_RETENTION=90d
ARCHIVE_MAX_SIZE=1.00GB         # hasta 2 decimales; none = sin límite
ARCHIVE_OVERFLOW=keep_one       # keep_one | skip
LAST_CRON_RUN=2026-11-08 21:00:01   # la escribe la tarea automática
LAST_CRON_RESULT=ok
```

</details>

<details>
<summary><b><code>&lt;carpeta&gt;/.dfd_bkp</code></b>: uno por cada carpeta respaldada</summary>

```ini
ENABLED=true                    # false = pausado
DESTINATION=gdrive:Backups/mi-carpeta
FREQUENCY=12h                   # Nh o Nd, mínimo 1h
ARCHIVE_DESTINATION=gdrive:Backups/_archive/mi-carpeta
EXCLUDE=/code/**                # / inicial = desde la raíz de la carpeta
EXCLUDE=node_modules/**         # sin / = en cualquier nivel
EXCLUDE=.DS_Store
```

Las claves `ARCHIVE_*` de este archivo sobrescriben las globales.

</details>

<details>
<summary><b><code>ARCHIVE_OVERFLOW</code></b>: qué pasa si una sola copia supera el límite del historial</summary>

| Valor | Resultado |
|---|---|
| `keep_one` *(por defecto)* | Guarda esa versión y borra todo el historial anterior. |
| `skip` | No guarda historial de esa copia y deja intacto el anterior. |

En los dos casos la copia se hace igual.

</details>

<details>
<summary><b>Carpetas anidadas</b>: una carpeta respaldada dentro de otra</summary>

Si una carpeta respaldada tiene adentro otra que también tiene su propio `.dfd_bkp`, esa carpeta se sube dos veces. El skill te avisa y te deja elegir:
- **Unirla:** se borra su `.dfd_bkp` y queda incluida en la de arriba. Puedes borrar su copia vieja en Drive o conservarla.
- **Separarla:** se excluye de la de arriba y sigue con su propio respaldo.

</details>

## 📋 Requisitos

- macOS. **Por ahora solo funciona en Mac.** En Windows, tu agente probablemente pueda adaptarlo si mantiene los mismos principios: un programa que copia (rclone también funciona en Windows), una tarea programada (el *Programador de tareas* en vez de cron) y un archivo de ajustes por carpeta.
- Un agente de IA que pueda ejecutar comandos en tu Mac: [Claude Code](https://claude.com/claude-code), ChatGPT/Codex u otro.
- [rclone](https://rclone.org), el programa que copia los archivos (`brew install rclone`). El agente te ayuda a instalarlo y a conectarlo con tu Google Drive.

## 🗂️ Estructura

```text
skill/dfd-bkp/
├── SKILL.md            instrucciones para el agente
└── scripts/dfd-bkp     script bash: run, list, status, preview, nested, update, doctor…
```

## 👋 Créditos

Creado por **[diegoferreyra.com](https://diegoferreyra.com)**. Sígueme en Instagram en **[@diegoferreyra.com_](https://instagram.com/diegoferreyra.com_)** para más consejos, talleres y servicios de desarrollo y procesos.

---

## 🤖 Instrucciones para el agente

Si un usuario te pasó este repo para configurar los backups:

1. **Elige dónde guardar el skill.**
   - Claude Code: `~/.claude/skills/dfd-bkp`.
   - Otro agente: la carpeta donde ese agente guarda sus skills o instrucciones reutilizables. Si no tiene, usa `~/.dfd-bkp`.

2. **Revisa si ya está instalado** antes de descargar nada:
   ```bash
   ls -la ~/.claude/skills/dfd-bkp 2>/dev/null; readlink ~/.claude/skills/dfd-bkp
   ```
   - Es un symlink a un clon git de este repo → no lo toques. Actualízalo con `git -C <ese clon> pull` y salta al paso 4.
   - Es una carpeta con `scripts/dfd-bkp` → actualízalo con `<carpeta>/scripts/dfd-bkp update` y salta al paso 4.
   - Existe pero es otra cosa → no lo toques y pregúntale al usuario.
   - No existe → sigue con el paso 3.

3. **Descarga los dos archivos.** No hace falta clonar el repo:
   ```bash
   D=~/.claude/skills/dfd-bkp   # o la carpeta elegida en el paso 1
   R=https://raw.githubusercontent.com/42pe/dfd_sync/main/skill/dfd-bkp
   mkdir -p "$D/scripts"
   curl -fsSL "$R/SKILL.md" -o "$D/SKILL.md"
   curl -fsSL "$R/scripts/dfd-bkp" -o "$D/scripts/dfd-bkp"
   chmod +x "$D/scripts/dfd-bkp"
   ```
   Si `curl` devuelve 404, el repo todavía es privado. Usa `gh api -H "Accept: application/vnd.github.raw" repos/42pe/dfd_sync/contents/skill/dfd-bkp/<archivo>` con una cuenta que tenga acceso.

4. **Lee `SKILL.md` y síguelo en esta misma conversación** desde "Pick the mode". No esperes a que el skill aparezca como comando. Si `~/.dfd_bkp_sync` no existe, es la primera vez (Mode A), y el aviso de "sin garantías" va primero.

5. **Al terminar**, dile al usuario cómo volver a usarlo. En Claude Code, escribiendo `/dfd-bkp` en una sesión nueva. En otros agentes, pidiendo "usa el skill dfd-bkp" o como ese agente invoque las instrucciones guardadas.
