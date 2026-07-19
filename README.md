# Windows 11 GNX lab via Dockur

Este repositorio crea un segundo host Windows 11 efímero para validar Quetzalcoatl sin adaptar el producto al runner nativo de GitHub.

El workflow:

1. Usa Ubuntu 24.04 y falla temprano si no obtiene KVM API 12, virtualización anidada, TUN, memoria o disco suficientes.
2. Libera únicamente herramientas preinstaladas del runner efímero que no usa el laboratorio.
3. Inicia la imagen oficial Dockur fijada por digest con 6 vCPU, 12 GiB RAM, disco disperso de 100 GiB y VMX/SVM expuesto al guest.
4. Conecta el borde Linux a Tailscale mediante una auth key y publica TCP 3389 con Tailscale Serve; Windows no queda publicado en Internet.
5. Mantiene vivo el guest durante la sesión y tolera los reinicios de Windows requeridos por el instalador.

## Secretos

Configura sólo:

- RDP_PASSWORD: contraseña de 12 a 127 caracteres para el usuario gnxlab.
- TS_RDP_AUTH_KEY: auth key reutilizable y preautorizada, con únicamente tag:github-rdp.

La auth key de Quetzalcoatl no se guarda en GitHub. Se ingresa directamente en gnx configure dentro del guest y debe tener únicamente tag:quetzalcoatl-node.

## Ejecución

    gh workflow run windows-rdp-tailscale.yml \
      -R mayas-alas/windows-rdp-tailscale \
      --ref codex/dockur-second-host \
      -f session_minutes=240

El summary del job muestra la IP Tailscale, MagicDNS, usuario y puerto. La contraseña permanece en el secret RDP_PASSWORD.

## Criterio de aceptación

RDP listo sólo convierte al guest en candidato. La evidencia de segundo host existe únicamente cuando el mismo QuetzalcoatlSetup.exe pasa HostPreflight, reinicia, converge y reporta gnx status --json sin cambios específicos para Dockur.

GitHub documenta 4 CPU, 16 GiB RAM y 14 GiB SSD para el runner público estándar, y considera experimental la virtualización anidada. Por eso el workflow sobreasigna los 6 vCPU del guest, recupera espacio del runner y conserva gates fail-stop; no emula KVM ni relaja el instalador.
