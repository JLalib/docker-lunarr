# Lunarr: Servidor de streaming de medios autohospedado en Docker

Plataforma moderna para gestionar y reproducir películas y series de televisión. Incluye monitoreo de biblioteca, búsqueda en indexers, transcodificación HLS, integración con TMDB, reproducción directa, frontend SvelteKit + backend Go. Ligero, rápido y completamente autohospedado.

## ¿Qué es Lunarr?

Lunarr es un servidor de streaming multimedia moderno y minimalista diseñado como alternativa a Plex, permitiéndote organizar, monitorear y reproducir tu colección de películas y programas de TV de forma completamente autohospedada. A diferencia de Plex (que recopila datos de usuario), Jellyfin (más consumo de recursos) o Emby (requiere suscripción para ciertas funciones), Lunarr es ligero, de código abierto y enfocado en la privacidad y el rendimiento.

### Características principales

- **Interfaz de usuario moderna**: Diseño responsivo basado en SvelteKit para una experiencia fluida en cualquier dispositivo.
- **Gestión de biblioteca inteligente**: Organiza tu contenido por películas, series, estado (visto/no visto), calidad y más.
- **Metadatos TMDB automáticos**: Obtiene pósters, información de reparto, equipo, valoraciones y recomendaciones de The Movie Database.
- **Reproducción versátil**: Soporta *direct play* (sin transcodificación) cuando es posible y transcodificación HLS con FFmpeg para compatibilidad universal.
- **Múltiples fuentes de medios**: Accede a tu contenido mediante sistema de archivos local, SFTP o WebDAV (solo lectura para mayor seguridad).
- **Actualizaciones automáticas**: Configura *watchers* para escanear periódicamente nuevas incorporaciones en tus bibliotecas.
- **Integración con indexers**: Conéctate a buscadores de torrents/Usenet para descarga automática de contenido (opcional).
- **API REST completa**: Permite integraciones personalizadas, webhooks y automatizaciones mediante herramientas externas.
- **Soporte multi-usuario**: Crea cuentas individuales con roles (administrador, usuario estándar) y perfiles personalizados.
- **Cero telemetría**: 100% privado – ningún dato se envía a servidores externos; todo se almacena localmente.
- **Licencia MIT**: Código abierto totalmente modificable y redistribuible.

## Requisitos del sistema

- Docker y Docker Compose instalados y funcionando.
- Memoria RAM: 1-2 GB (suficiente para la mayoría de usos domésticos; ideal para Raspberry Pi o VPS baratos).
- Espacio en disco: 10 GB o más, dependiendo del tamaño de tu colección multimedia y la caché de transcodificación.
- Puerto TCP: 3000 (personalizable mediante el mapeo de puertos en Docker Compose).
- Base de datos: SQLite (embebido en la imagen oficial de Docker, no requiere configuración adicional).
- FFmpeg: Incluido en la imagen de Docker para manejar la transcodificación cuando sea necesario.
- Variable de entorno : Cadena segura de al menos 32 bytes en formato hexadecimal (generable con c7fc0b4f2a51d8ee2730c57aa02aea2840a5a69e5ffb443e987eda9132f9aa71).

## Instalación con Docker Compose

Sigue estos pasos para poner en marcha tu instancia de Lunarr en cuestión de minutos:

### Paso 1: Preparar el archivo de configuración
Crea un archivo llamado  en un directorio vacío y copia el siguiente contenido:

```yaml
version: '3.8'

services:
  lunarr:
    image: sayem314/lunarr:latest
    container_name: lunarr
    restart: unless-stopped
    ports:
      - "3000:3000"
    volumes:
      - lunarr-data:/data
      - /mnt/media:/media:ro
    environment:
      - AUTH_SECRET=***  # ¡IMPORTANTE: reemplaza con un valor seguro!
      - ORIGIN=http://localhost:3000                # Ajustar si usas un dominio personalizado (ej: https://media.tudominio.com)

volumes:
  lunarr-data
```

> **Nota de seguridad**: El valor de `AUTH_SECRET` debe ser único y secreto. Généralo con:
> ```bash
> openssl rand -hex 32
> ```
> Nunca lo compartas ni lo subas a repositorios públicos si contiene tu valor real (este ejemplo usa un placeholder).

### Paso 2: Ajustar los volúmenes según tu infraestructura
- `/mnt/media:/media:ro` → Reemplaza `/mnt/media` por la ruta absoluta donde tengas almacenada tu colección de películas y series (debe ser accesible en modo lectura seule para mayor seguridad).
- Si utilizas SFTP o WebDAV en lugar de un montaje directo, ajusta este volumen según corresponda (consultar la documentación oficial de Lunarr para detalles).

### Paso 3: Iniciar los contenedores
Desde el directorio que contiene `docker-compose.yml`, ejecuta:
```bash
docker compose up -d
```
Docker descargará la imagen necesaria y lanzará el servicio en segundo plano.

### Paso 4: Acceder a la interfaz web
Abre tu navegador y navega a:
- `http://localhost:3000` (si estás en el mismo equipo)
o la IP/puerto correspondiente si accedes remotamente.

La primera cuenta de usuario que crees se convertirá automáticamente en administrador.

## Primeros pasos tras el inicio

1. **Crear tu cuenta de administrador**
   - Al acceder por primera vez, serás invitado a registrar un usuario.
   - Completa el formulario con tu correo electrónico y una contraseña segura.
   - Este usuario tendrá privilegios de administración completa.

2. **Agregar tus bibliotecas de medios**
   - Ve a la sección ""Bibliotecas"" en el menú lateral.
   - Haz clic en ""Añadir biblioteca"".
   - Selecciona el tipo: **Local** (si usaste el montaje `/mnt/media`), **SFTP** o **WebDAV**.
   - Asigna un nombre descriptivo (ej: "Películas 4K", "Series Infantiles").
   - Especifica la ruta exacta dentro del contenedor (por defecto `/media` si seguiste el ejemplo arriba).
   - Guarda los cambios.

3. **Ejecutar el escaneo inicial**
   - Dentro de la lista de bibliotecas, selecciona la que acabas de crear.
   - Haz clic en el botón ""Escanear"".
   - Lunar comenzará a indexar todos los archivos de medios, obteniendo metadatos de TMDB y preparando tu biblioteca para navegación y reproducción.
   - Este proceso puede tardar desde unos minutos hasta varias horas, dependiendo del tamaño de tu colección y el rendimiento de tu disco.

4. **Explorar y reproducir tu contenido**
   - Una vez finalizado el escaneo, ve a ""Explorar"" o ""Biblioteca"" en el menú principal.
   - Navega por tu colección usando las vistas de cuadrícula o lista.
   - Aplica filtros por género, año, estado (visto/no visto), calidad, etc.
   - Haz clic en cualquier título para ver sus detalles detallados (elenco, trama, etc.).
   - Para reproducir, simplemente pulsa el botón ""Reproducir"".
   - Lunar determinará automáticamente si usar reproducción directa o transcodificación HLS según tus capacidades de red y dispositivo.

5. **Configurar actualizaciones automáticas (opcional pero recomendado)**
   - En la configuración de cada biblioteca, activa los ""Guardianes automáticos"" (Watchers).
   - Establece una frecuencia de escaneo (ej: cada 6 horas, diario) para que Lunar detecte y incorpore nuevos archivos sin intervención manual.
   - Ajusta la hora de ejecución para evitar períodos de alto uso si lo deseas.

## Mantenimiento y administración

### Ver los registros en tiempo real
```bash
docker logs -f lunarr
```
Útil para depurar problemas de conexión, escaneo o reproducción.

### Actualizar a la última versión
Para beneficiarte de las últimas mejoras y parches de seguridad:
```bash
# Descarga la imagen más reciente
docker compose pull

# Vuelve a crear los contenedores con la nueva imagen
docker compose up -d
```

### Crear copias de seguridad de tu configuración y metadatos
Los datos de la aplicación (base de datos SQLite, configuración, caché de portadas, etc.) se almacenan en el volumen `lunarr-data`:
```bash
# Detener el contenedor antes de hacer backup (recomendado para consistencia)
docker compose down

# Copiar los datos a tu máquina host
docker cp lunarr:/data ./lunarr-backup-20260702

# Volver a levantar el servicio
docker compose up -d
```
Para restaurar, simplemente invierte el proceso (asegurándote de que el contenedor esté detenido durante la copia).

### Monitorear el consumo de recursos
```bash
docker stats lunarr
```
Muestra en tiempo real el uso de CPU, memoria, E/S de disco y red del contenedor.

### Reiniciar el servicio
```bash
docker compose restart
```
Útil después de cambiar variables de entorno o configuraciones que requieren un reinicio completo.

## Comparativa con alternativas populares

| Característica            | Lunarr                          | Plex                            | Jellyfin                      | Emby                          |
|---------------------------|----------------------------------|----------------------------------|-------------------------------|-------------------------------|
| **Modelo de licencia**    | Código abierto (MIT)             | Freemium (funciones avanzadas de pago) | Código abierto (GPLv2)       | Freemium (características premium de pago) |
| **Telemetría / privacidad** | **Ninguna** – 100% local        | Recopila datos de uso por defecto | Ninguna                      | Opcional (dependiendo del plan) |
| **Consumo de recursos**   | Muy bajo (~100-200 MB RAM en reposo) | Medio-alto                      | Medio                        | Medio-alto                    |
| **Facilidad de instalación** | Alta (Docker Compose sencillo)  | Alta (instalador guiado)        | Media (requiere más configuración) | Media                        |
| **Ecosistema de apps**    | Limitado (navegador web + apps básicas) | Extenso (apps oficiales para TV, móviles, consolas) | Moderado (apps comunitarias) | Bueno (apps oficiales + algunas de terceros) |
| **Funciones avanzadas**   | Esenciales bien implementadas    | Amplio (TV en vivo, DVR, música, fotos) | Muy completo (similar a Plex) | Completo + opciones de pago   |
| **Ideal para**            | Usuarios que priorizan ligereza, privacidad y simplicidad | Quien quiere una solución todo-en-uno con soporte multiplataforma | Aquellos que buscan máximo control sin costo | Entornos que necesitan características empresariales o de pago específico |

### Veredicto
Elige **Lunarr** si:
- Valorás la privacidad absoluta y el control total sobre tus datos.
- Prefieres una interfaz moderna y limpia sin sobrecarga innecesaria.
- Tienes hardware limitado (Raspberry Pi, VPS económico) y necesitas eficiencia de recursos.
- Quieres evitar suscripciones o modelos freemium que bloqueen funcionalidades básicas.

Considera **Plex** si:
- Necesitas acceso remoto fluido sin configurar puertos o DNS complejos.
- Dependés de aplicaciones móviles/TV de alta calidad y constantemente actualizadas.
- No te importa compartir datos de uso a cambio de comodidad y características avanzadas.

Considera **Jellyfin** si:
- Quieres una alternativa completamente gratuita y de código abierto con un conjunto de características muy completo.
- No te importa una interfaz algo menos moderna pero altamente funcional.

Considera **Emby** si:
- Requerís funcionalidades específicas de nivel empresarial o características premium dispuestas a pagar por ellas.
- Valoras el soporte técnico oficial y la integración con ciertos ecosistemas de pago.

## Licencia

Este proyecto se distribuye bajo la Licencia MIT. Consulta el archivo [LICENSE](LICENSE) para obtener los detalles completos.