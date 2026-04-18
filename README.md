# PPS-U4-RA4: Análisis Dinámico APK Pivaa con MobSF y Genymotion

## CREACIÓN Y CONFIGURACIÓN DEL ENTORNO CONTROLADO DE ANÁLISIS DE APLICACIONES MÓVILES

---

## 1. Descarga de la APK

Descargar la aplicación `pivaa.apk`:
```
wget https://github.com/HTBridge/pivaa/blob/master/apks/pivaa.apk
```
O desde el navegador en:
[https://github.com/HTBridge/pivaa/blob/master/apks/pivaa.apk](https://github.com/HTBridge/pivaa/blob/master/apks/pivaa.apk)

---

## 2. Crear máquina virtual en Genymotion

Crear una nueva VM Android:
- Tipo: **Google Nexus 5**
- Android: **7.1** (x86)

Si ya existe, iniciar el dispositivo virtual.

> **Nota:** La versión de Android es importante. En Android 5.1 no funciona correctamente la visualización de pantalla o el botón de lanzador de Activities en MobSF.

---

## 3. Comprobar conexión con el dispositivo virtual

```bash
# Reiniciar el servidor ADB
adb kill-server
adb start-server

# Listar dispositivos
adb devices

# Conectar al emulador (puede tardar unos minutos)
adb connect 127.0.0.1:6555

# Verificar conexión
adb devices
# Debe aparecer: 127.0.0.1:6555  device
```

---

## 4. Instalar la APK en el emulador

```bash
# Instalar pivaa.apk
adb install pivaa.apk

# Desinstalar si es necesario
adb uninstall com.htbridge.pivaa
```

---

## 5. Crear contenedor Docker de MobSF

```bash
# Si usas VirtualBox en vez de QEMU, usar puerto 5555 en vez de 6555
docker run -d \
  --network host \
  --add-host=host.docker.internal:127.0.0.1 \
  -e MOBSF_ANALYZER_IDENTIFIER="host.docker.internal:6555" \
  --name=mobsf \
  opensecurity/mobile-security-framework-mobsf:latest

# Para iniciar el contenedor si ya existe (no se borra con --rm)
docker start mobsf
```

### Explicación de parámetros:

| Parámetro | Descripción |
|-----------|-------------|
| `--network host` | Usa la red del host en vez de Bridge, conectando perfectamente por ADB con Genymotion |
| `--add-host=host.docker.internal:127.0.0.1` | Crea un host para la conexión de MobSF |
| `-e MOBSF_ANALYZER_IDENTIFIER="host.docker.internal:6555"` | Variable de entorno para conexión ADB al host:puerto |
| `--name=mobsf` | Nombre del contenedor |

---

## 6. Comprobar conexión entre Docker MobSF y el emulador

```bash
# Abrir nueva terminal o pestaña

# Acceder al contenedor
docker exec -it mobsf /bin/bash

# Conectar por ADB al dispositivo
adb connect 127.0.0.1:6555
# Debe indicar: already connected to 127.0.0.1:6555

# Listar dispositivos
adb devices
# Debe aparecer: 127.0.0.1:6555  device

# Verificar modelo y versión
adb -s 127.0.0.1:6555 shell getprop ro.product.model
# Debe indicar: Nexus 5

adb -s 127.0.0.1:6555 shell getprop ro.build.version.release
# Debe indicar: 7.1
```

---

## 7. Acceder a MobSF

Abrir navegador y acceder a:
```
http://localhost:8000
```

Autenticarse con:
- Usuario: `mobsf`
- Contraseña: `mobsf`

---

## ANÁLISIS ESTÁTICO

1. Seleccionar **STATIC ANALYZER**
2. Arrastrar y soltar `Pivaa.apk`
3. Analizar los resultados obtenidos (serán la base del análisis dinámico)
4. Identificar problemas para comprobar en el análisis dinámico

---

## LANZAR ENTORNO DE ANÁLISIS DINÁMICO

1. Pulsar en **DYNAMIC ANALYZER**
2. Lanzar **Android Dynamic Analyzer**

### Comprobar conexión MobSF - Android Runtime:

1. Pulsar **MobSFy Android Runtime**
2. Pulsar botón **MobSFy**
3. Verificar mensaje: `Successfully created MobSF Dynamic Analysis environment. MobSF agents and Frida server installed.`
4. Comprobar que aparece `Pivaa` en apps disponibles

> **Si no aparece:** Cambiar `host.docker.internal` por `127.0.0.1` en la configuración de MobSF Android Runtime, quedando `127.0.0.1:6555`. Cerrar configuración y refrescar navegador.

5. Seleccionar `com.htbridge.pivaa` y pulsar **Start Dynamic Analysis**

---

## ENTORNO DE ANÁLISIS DINÁMICO

### Operaciones realizadas por MobSF al iniciar:

- Inicia/instala la APK en el emulador
- Instala Certificado Root CA
- Inicia proxy
- Inicia captura de tráfico de red (PCAP)
- Activa logging completo (logcat)
- Inicia monitorización de archivos y procesos
- Hookea Xposed/Frida modules
- Configura forwarding de puertos ADB

### Operaciones automatizadas de MobSF:

- Clasifica activities, services, receivers, providers
- Detecta certificate pinning bypass
- Enumera bases de datos SQLite
- Extrae certificados SSL
- Analiza malware score de dominios contactados
- Genera reporte de permisos runtime

### Elementos del entorno gráfico:

| Elemento | Descripción |
|----------|-------------|
| **Paquete cargado** | Comunicación con el emulador para operaciones sobre el paquete |
| **Show Screen** | Muestra el emulador (no funcional en Android 5.1) |
| **Remove Root CA** | Limpia el certificado raíz instalado por MobSF/Frida/Burp/mitmproxy |
| **Unset HTTPS Proxy** | Detiene el proxy al finalizar el análisis |
| **TLS/SSL Security Tester** | Detecta y bypassa protecciones SSL/TLS |
| **Get Dependencies** | Lista librerías .so, detecta CVEs, dependencias de terceros |
| **Take a Screenshot** | Captura de pantalla para documentar |
| **Logcat Stream** | Control de adquisición de logs del sistema |
| **Generate Report** | Genera el informe final del análisis |
| **Default Scripts de Frida** | Scripts de instrumentación dinámica avanzada |
| **Frida Code Editor** | Para ejecutar hooks personalizados |
| **Instrumentation** | Spawn & Inject, Inject, Attach |
| **User Interface** | Selección de Activities de la app |
| **Shell Access** | Consola ADB integrada |

---

## FLUJO DE ANÁLISIS DINÁMICO

### Comandos y acciones a realizar:

```
[MobSF] → Lanza APK en emulador
[MobSF] → Inicia captura TLS/SSL Security Tester + Logcat Stream

[TÚ] → Interactuar con la app (15-30 min)
       Credenciales de prueba: test / supersecretpassword

[TÚ] → Get Dependencies
[TÚ] → Exported Activity Tester
[TÚ] → Activity Tester
[TÚ] → Capturas de pantalla cuando sea necesario

[Spawn & Inject] ← PRIMERA VEZ (lanza + inyecta Frida)
       → Interactuar con app (login, acciones)

[Inject] ← PRUEBAS ADICIONALES
       → Seleccionar Auxiliary Frida Scripts → Inject

[Attach] ← ENUMERACIÓN
       → Atacar proceso: PIVAA.com.htbridge.pivaa

[Start Activity] → User Interface → Activities
       → Seleccionar una a una → Start Activity

[Frida Code Editor] → Seleccionar scripts Frida → Inject

[TÚ] → Operaciones sobre Shell si es necesario

[MobSF] → Analiza tráfico, archivos, logs

[TÚ] → Unset Proxy + Remove Root CA

[MobSF] → Generate Report

[TÚ] → Analizar datos del informe
```

---

## INFORME DE ANÁLISIS DINÁMICO

Secciones del informe generado:

| Sección | Descripción |
|---------|-------------|
| **Start HTTP Tools** | Proxy para interceptar y reenviar tráfico, extraer sesiones |
| **HTTP(S) Traffic** | Archivo .txt con el tráfico capturado |
| **Logcat Logs** | Registros ADB: procesos, activities, desencadenadores |
| **Dumpsys Logs** | Estado de servicios: actividades, permisos, conectividad, sensores |
| **Application Data** | Archivos reales de la APK: SQLite, preferencias, tokens, cachés |
| **TLS/SSL Security Tester** | Resultados del test TLS/SSL |
| **Exported Activity Tester / Activity Tester** | Activities probadas con capturas |
| **Screenshots** | Capturas realizadas durante el análisis |
| **Runtime Dependencies** | Dependencias del entorno runtime |
| **Malware Analysis** | Análisis de dominios/IP contra bases de datos de malware |
| **Reconnaissance** | Cortapapeles, URLs, emails, trackers, strings decodificados |
| **File Analysis** | Bases de datos, XML y otros ficheros analizados |

---

## Crédenciales encontradas

En el archivo `lockfile` de Application Data se pueden ver las credenciales introducidas.
