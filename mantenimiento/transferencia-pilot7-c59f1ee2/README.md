# Entrega para Codex Windows 1: transferencia de inicio pilot.2 → pilot.7

**PREPARADA, NO EJECUTADA. Esta entrega no autoriza intervenir en Windows 1.**
Esperar autorización expresa antes de ejecutar cualquiera de los bloques allí.
No volver a ejecutar el arnés ni compilar herramientas o RADAR.

## Archivos e identidad

Entrega pública de mantenimiento: descargar desde la carpeta GitHub
`mantenimiento/transferencia-pilot7-c59f1ee2` estos cuatro archivos:

- `TransferirPropietarioRadar.exe`: herramienta final generada en la VM, sin
  simulación de fallos. No reconstruida para esta entrega.
- `resultado-transferencia-propietario.json`: resultado original, sin reescribir.
- `README.md`: este procedimiento, sin datos personales ni rutas privadas.
- `SHA256SUMS.txt`: huellas de los tres archivos anteriores.

No es un instalador, paquete de actualización ni un nuevo Release de RADAR.
El EXE y JSON mantienen exactamente los SHA-256 de la aceptación sintética.

| Archivo | SHA-256 autorizado de esta entrega |
|---|---|
| `TransferirPropietarioRadar.exe` | `06fd959918246c176b8897aa51d9b9db32d86190d2022b8907a24f0dff4b3fc9` |
| `resultado-transferencia-propietario.json` | `31b4e8814aecb8e1522c808bc7a45ec0cb03a8f5e06cc139e897dbd620de95f6` |

Procedencia: código probado `c59f1ee278a24e21a7ea850adddc3a31bbef3bab`, rama
`codex/transferencia-propietario-pilot7`; evidencia publicada en commit documental
`202514548c8ee8d88ea3988946ee485050144c88`. Fuentes C# sin cambios desde 285559fa.

Ensayo del 16/10/2026: una ejecución en PowerShell 5.1, siete escenarios correctos,
salida 0, stderr vacío, limpieza sintética y políticas permanentes conservadas.
Se ejecutó únicamente la variante con tres constantes de Registro sintéticas.
El EXE final conserva las constantes normales y se compiló sin la opción de
simulación; su SHA se verificó independientemente. **No se ha ejecutado contra
el Registro real.** No se afirma firma Authenticode ni se publica un Release.

## Antes de la futura intervención autorizada

- Misma cuenta Windows de Windows 1 que conserva su perfil y credenciales;
  PowerShell normal, sin elevar ni cambiar usuarios o permisos.
- RADAR completamente cerrado, incluidos bandeja/motor. La herramienta comprueba
  los procesos propios y mutex y no los detiene por la fuerza.
- Pilot.7 ya colocado íntegro en su directorio limpio; no se instala desde este
  entrega de mantenimiento. Conservar todos los programas pilot.2/pilot.5 y respaldos.
- Revalidar el respaldo SQLite 18 más reciente antes de cambiar propietario. Esta
  herramienta no toca la base ni sustituye el procedimiento de respaldo.
- Conservar `config.toml`, `licencia-premium-v1.json`, OAuth y Credential Manager;
  no abrir/extraer secretos, reactivar licencias, conectar correo ni borrar entradas.
- No cambiar inicio automático ni vigilancia manualmente: se conservan sus valores
  anteriores. Si estaban desactivados, seguirán desactivados.

Las rutas se derivan del usuario actual y las huellas son las documentadas para Windows 1.
Se deben verificar otra vez localmente; ante cualquier diferencia, detenerse.
No sustituir las huellas esperadas por las encontradas para hacer pasar el control.

## 1. Preparación y verificación local — solo tras autorización

Colocar los cuatro archivos en `Downloads\RADAR-Transferencia-pilot7-c59f1ee2`
del usuario actual. La descarga no necesita una cuenta GitHub. En PowerShell
normal de la cuenta habitual, y solo tras autorización, ejecutar:

```powershell
$ErrorActionPreference = 'Stop'
$Kit = Join-Path $env:USERPROFILE 'Downloads\RADAR-Transferencia-pilot7-c59f1ee2'
$Herramienta = Join-Path $Kit 'TransferirPropietarioRadar.exe'
$Resultado = Join-Path $Kit 'resultado-transferencia-propietario.json'
$Origen = Join-Path $env:LOCALAPPDATA 'CorreoBlindado\actualizaciones\laboratorio\versiones\d83f6f7e07294326c0a2c739d571248fe25b671b6a0f27dee4aca33e08caeecf\programa\Radar-Antifraude.exe'
$Destino = Join-Path $env:LOCALAPPDATA 'Programs\RADAR-0.6.3-pilot.7-limpio\Radar-Antifraude.exe'
$Perfil = Join-Path $env:APPDATA 'CorreoBlindado'
$DirectorioEstado = Join-Path $env:LOCALAPPDATA 'RADAR-Transferencia-Propietario-pilot7'
$Estado = Join-Path $DirectorioEstado 'estado-transferencia-pilot7.json'
$ShaHerramienta = '06fd959918246c176b8897aa51d9b9db32d86190d2022b8907a24f0dff4b3fc9'
$ShaResultado = '31b4e8814aecb8e1522c808bc7a45ec0cb03a8f5e06cc139e897dbd620de95f6'
$ShaOrigen = 'bde19dbc2235d25dc5a19dfcccc7b580629ab7aa6114fe42cecd7ce52f023a89'
$ShaDestino = '033198ab5d668e3df5aa0d9e45d306294ebe75644c9accded9ef661ee1f833e7'

if ((Get-FileHash -LiteralPath $Herramienta -Algorithm SHA256).Hash -ine $ShaHerramienta) { throw 'HUELLA_HERRAMIENTA_NO_COINCIDE' }
if ((Get-FileHash -LiteralPath $Resultado -Algorithm SHA256).Hash -ine $ShaResultado) { throw 'HUELLA_JSON_NO_COINCIDE' }
if ((Get-FileHash -LiteralPath $Origen -Algorithm SHA256).Hash -ine $ShaOrigen) { throw 'HUELLA_PILOT2_NO_COINCIDE' }
if ((Get-FileHash -LiteralPath $Destino -Algorithm SHA256).Hash -ine $ShaDestino) { throw 'HUELLA_PILOT7_NO_COINCIDE' }
if (-not (Test-Path -LiteralPath (Join-Path $Perfil 'config.toml') -PathType Leaf)) { throw 'CONFIGURACION_AUSENTE' }
if (-not (Test-Path -LiteralPath (Join-Path $Perfil 'licencia-premium-v1.json') -PathType Leaf)) { throw 'ARCHIVO_LICENCIA_AUSENTE' }
if (Test-Path -LiteralPath $DirectorioEstado) { throw 'ESTADO_PREVIO_EXISTENTE_NO_SOBRESCRIBIR' }
```

Estas verificaciones solo calculan hashes y comprueban existencia: no imprimen el
contenido del perfil/licencia. No son una nueva suite ni un ensayo de correo.

## 2. Transferir pilot.2 → pilot.7 y comprobar

En la misma sesión y solo si los requisitos anteriores están confirmados:

```powershell
New-Item -ItemType Directory -Path $DirectorioEstado -ErrorAction Stop | Out-Null
& $Herramienta transferir `
  --origen $Origen --destino $Destino `
  --sha256-origen $ShaOrigen --sha256-destino $ShaDestino `
  --estado $Estado --perfil $Perfil
if ($LASTEXITCODE -ne 0) { throw 'TRANSFERENCIA_REVERTIDA_O_BLOQUEADA: conservar estado y detenerse' }

& $Herramienta verificar --estado $Estado --esperado destino
if ($LASTEXITCODE -ne 0) { throw 'PROPIETARIO_DESTINO_NO_VERIFICADO: detenerse' }
```

Salidas que deben aparecer, ambas con código 0:

```text
TRANSFERIDA_Y_VERIFICADA
ESTADO_VERIFICADO=DESTINO
```

La herramienta conserva preferencias y escribe antes de cambiar el Registro un
estado durable que permite volver al propietario original. No inicia RADAR.
Los valores afectados pertenecen solo al usuario: `Ejecutable`, `Habilitado` y
`VigilanciaActiva` bajo `Software\CorreoBlindado\RadarAntifraude\Inicio`, y la
entrada `CorreoBlindado.RadarAntifraude` de su clave `CurrentVersion\Run`.
No borrar `$Estado`, no editarlo y no volver a ejecutar `transferir` sobre él.

Estos bloques son órdenes interactivas a un EXE; no requieren cambiar
ExecutionPolicy ni ejecutar el arnés con Bypass en Windows 1. No se usan
redirecciones `2>&1` ni se ocultan errores. Si una orden falla, no continuar con
la siguiente ni intentar reparaciones de Registro/perfil.

## 3. Rollback — bloque alternativo, no pegar después del anterior automáticamente

Solo si se decide revertir la intervención autorizada y con RADAR cerrado.
Conservar el mismo `$Estado` y ambos programas íntegros. Si se ha cerrado
PowerShell, restablecer únicamente estas variables:

```powershell
$ErrorActionPreference = 'Stop'
$Herramienta = Join-Path $env:USERPROFILE 'Downloads\RADAR-Transferencia-pilot7-c59f1ee2\TransferirPropietarioRadar.exe'
$Estado = Join-Path $env:LOCALAPPDATA 'RADAR-Transferencia-Propietario-pilot7\estado-transferencia-pilot7.json'
if ((Get-FileHash -LiteralPath $Herramienta -Algorithm SHA256).Hash -ine '06fd959918246c176b8897aa51d9b9db32d86190d2022b8907a24f0dff4b3fc9') { throw 'HUELLA_HERRAMIENTA_NO_COINCIDE' }
if (-not (Test-Path -LiteralPath $Estado -PathType Leaf)) { throw 'ESTADO_DE_TRANSFERENCIA_AUSENTE' }

& $Herramienta restaurar --estado $Estado
if ($LASTEXITCODE -ne 0) { throw 'ROLLBACK_NO_VERIFICABLE: conservar evidencias y detenerse' }

& $Herramienta verificar --estado $Estado --esperado origen
if ($LASTEXITCODE -ne 0) { throw 'PROPIETARIO_PILOT2_NO_RESTAURADO: detenerse' }
```

Salidas requeridas, ambas con código 0:

```text
RESTAURADA_Y_VERIFICADA
ESTADO_VERIFICADO=ORIGEN
```

El rollback restaura propietario y preferencias de inicio, no SQLite ni llaveros,
y no ejecuta pilot.2. Requiere que los EXE y los hashes de `config.toml` y
`licencia-premium-v1.json` sigan coincidiendo con lo capturado. Si durante una
apertura posterior cambiaron esos archivos, puede devolver `perfil_modificado`:
**detenerse, sin editar el estado, restaurar archivos a ciegas o forzar el Registro**.
No es una herramienta de restauración de datos; el respaldo SQLite se conserva
por separado. No se retiran programas/accesos antiguos en esta transferencia.

## Resultado que deberá comunicar Codex Windows 1

Cuenta/contexto usados, huellas coincidentes, salidas/códigos de transferir y
verificar, preferencias conservadas y ubicación del estado; si se ejecutó
rollback, sus dos salidas/códigos. Nunca adjuntar contenidos del perfil, licencia,
tokens o llavero. **Ahora solo se entrega el material: ninguna orden de estos
bloques se ha ejecutado en Windows 1.**
