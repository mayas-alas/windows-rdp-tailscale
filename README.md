# windows-rdp-tailscale
El workflow crea un Windows Server 2025 efímero, descarga tu repositorio, instala Tailscale, crea el usuario administrador ghrdp, habilita RDP con NLA/TLS y permite el puerto 3389 únicamente desde rangos de Tailscale. Las acciones están fijadas por SHA para evitar cambios inesperados. Validé la estructura YAML y los parámetros contra las acciones actuales de Tailscale y actions/checkout.

## 1. Configura el tag en Tailscale

En Tailscale Admin Console → Access controls, integra estas entradas en tu política actual. No reemplaces todo el archivo si ya tienes reglas:
---
{
  "tagOwners": {
    "tag:github-rdp": []
  },

  "grants": [
    {
      "src": ["TU_EMAIL_DE_TAILSCALE@example.com"],
      "dst": ["tag:github-rdp"],
      "ip": ["tcp:3389", "udp:3389"]
    }
  ]
}
---
Reemplaza el correo con el que utilizas para iniciar sesión en Tailscale. La política concede RDP solamente a tus dispositivos y solamente hacia la VM etiquetada como tag:github-rdp. Tailscale recomienda usar grants para configuraciones nuevas y admite restricciones por protocolo y puerto.

## 2. Crea las credenciales OAuth de Tailscale

En:

Tailscale Admin Console
→ Trust credentials
→ Create OAuth client

Configura:

Scope: auth_keys
Tag:   tag:github-rdp

Copia:

Client ID
Client secret

El scope auth_keys y el tag del cliente deben coincidir con tag:github-rdp. El nodo creado por la acción es efímero y Tailscale lo elimina al terminar el job.

## 3. Crea tres Secrets en GitHub

En el repositorio:

Settings
→ Secrets and variables
→ Actions
→ New repository secret

Crea exactamente:

TS_OAUTH_CLIENT_ID
TS_OAUTH_SECRET
RDP_PASSWORD

Valores:

TS_OAUTH_CLIENT_ID = Client ID de Tailscale
TS_OAUTH_SECRET    = Client secret de Tailscale
RDP_PASSWORD       = contraseña fuerte para Windows

Puedes cargarlos con GitHub CLI:

gh secret set TS_OAUTH_CLIENT_ID
gh secret set TS_OAUTH_SECRET
gh secret set RDP_PASSWORD

Para generar una contraseña robusta desde PowerShell:

$bytes = [byte[]]::new(24)
[Security.Cryptography.RandomNumberGenerator]::Fill($bytes)
[Convert]::ToBase64String($bytes) + "!Aa1"

No recomiendo hardcodearla en el YAML: quedaría guardada en el historial Git y potencialmente visible para personas con acceso al repositorio. Tailscale también recomienda mantener OAuth secrets fuera del código fuente.

## 4. Ejecútalo

En GitHub:

Actions
→ Windows RDP via Tailscale
→ Run workflow

Selecciona una duración entre 60 y 330 minutos.

Cuando llegue al paso Publish connection details, abre el Job summary. Ahí aparecerán:

Tailscale IPv4
MagicDNS hostname
Username: ghrdp
Port: 3389
Workspace del repositorio

Desde tu Windows local, conectado al mismo tailnet:

mstsc.exe

Conecta usando:

Equipo:   IP de Tailscale o nombre MagicDNS
Usuario:  ghrdp
Password: contenido de RDP_PASSWORD

Al terminar, cancela el workflow desde GitHub Actions. El runner hospedado tiene un límite máximo de seis horas; el archivo deja margen de inicialización y permite sesiones de hasta 330 minutos.
