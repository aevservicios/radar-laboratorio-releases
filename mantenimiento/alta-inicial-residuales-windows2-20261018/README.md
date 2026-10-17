# Alta inicial con preferencias residuales · Antigravity Windows 2

Herramienta de mantenimiento para el ZIP **1.0.0-pilot.1** ya descargado y
verificado. No es un nuevo instalador ni una nueva versión de RADAR. No cambia
el ZIP, configura cuentas, activa licencias ni inicia el motor.

## Antes de ejecutar en Windows 2

Esta entrega no ha intervenido en Windows 2. Los comandos siguientes se ejecutarán
allí únicamente por Antigravity en la cuenta habitual, con PowerShell normal,
sin elevar ni cambiar ExecutionPolicy. No utilizar `transferir` para este caso.

1. Conservar fuera del perfil nuevo el respaldo verificado de programa,
   configuración, OAuth y datos de imap.13. No borrar ni exportar Credential
   Manager. No mezclar SQLite/configuración antigua con el perfil nuevo.
2. Cerrar RADAR normalmente y confirmar ausencia de procesos. Si no cierra,
   detenerse, sin forzar procesos. La herramienta vuelve a comprobar los mutex
   y procesos del usuario antes de escribir.
3. El nuevo `%APPDATA%\CorreoBlindado` y `%LOCALAPPDATA%\CorreoBlindado` deben
   estar vacíos y sin enlaces/reparse points. Si contienen datos anteriores,
   detenerse hasta que su conservación y retirada del perfil activo estén
   verificadas; esta herramienta no mueve, elimina ni copia datos.
4. Debe estar ausente la entrada propia `CorreoBlindado.RadarAntifraude` de
   `HKCU\Software\Microsoft\Windows\CurrentVersion\Run` y la clave propia
   `HKCU\Software\CorreoBlindado\RadarAntifraude\Inicio` puede estar ausente,
   vacía o contener exclusivamente `Habilitado` y/o `VigilanciaActiva`, ambos
   REG_DWORD con valor 0 o 1. Se admite el estado 1/1 dejado por la desinstalación
   oficial de imap.13. Cualquier propietario (incluso vacío), subclave, tipo o
   valor inesperado supone conflicto: detenerse. No editar ni borrar el Registro.
5. Mantener el programa 1.0.0-pilot.1 íntegro en su carpeta definitiva nueva,
   fuera de esos perfiles. El lanzador raíz es `Radar-Antifraude.exe`;
   nunca registrar ni ejecutar directamente `runtime\RadarMotor.exe`.

## Descarga y verificación del mantenimiento

Descargar de la carpeta pública de esta entrega `TransferirPropietarioRadar.exe`,
`SHA256SUMS.txt`, `integridad-alta.json` y este documento. Reutilizar la política
`alta-inicial-windows.json` anterior, sin editarla. Si falta, descargar la
[política original](https://raw.githubusercontent.com/aevservicios/radar-laboratorio-releases/e3d094db6235e6c18f9ceac812375356f25c798f/mantenimiento/alta-inicial-windows2-20261018/alta-inicial-windows.json).

Guardar estos archivos en `Downloads\RADAR-Alta-Residuales-Windows2` de la cuenta
habitual. La política mantiene su SHA-256 original, comprobado por el bloque.

El EXE es .NET Framework x64 para el Windows ya ensayado; no requiere Python,
pip ni instalar otro setup. No tiene firma Authenticode: comprobar las huellas
con las recibidas por el canal de entrega. La política se entrega aparte del ZIP
y solo se usa para comprobar su identidad; no se instala dentro del producto.

Si el intento con la herramienta anterior quedó bloqueado y no generó estado,
conservar su carpeta sin borrarla. Este procedimiento usa otra carpeta de trabajo.
Si ya hay un estado durable de un alta real, detenerse y revisarlo antes de crear
otra operación; no editar ni reutilizar estados para forzar el alta.

## Alta inicial — un único bloque, sin iniciar RADAR

Si el ZIP se extrajo en otra carpeta, sustituir únicamente `$Instalacion` por su
ruta absoluta real comprobada. No cambiar las huellas ni redirigir APPDATA o
LOCALAPPDATA. Las rutas de perfil de este procedimiento son las habituales que
utilizará el motor; no indicar una carpeta vacía alternativa para pasar el control.

```powershell
$ErrorActionPreference = 'Stop'
$Kit = Join-Path $env:USERPROFILE 'Downloads\RADAR-Alta-Residuales-Windows2'
$Herramienta = Join-Path $Kit 'TransferirPropietarioRadar.exe'
$Politica = Join-Path $Kit 'alta-inicial-windows.json'
$Instalacion = Join-Path $env:LOCALAPPDATA 'Programs\RADAR-1.0.0-pilot.1-limpio'
$Lanzador = Join-Path $Instalacion 'Radar-Antifraude.exe'
$Perfil = Join-Path $env:APPDATA 'CorreoBlindado'
$Datos = Join-Path $env:LOCALAPPDATA 'CorreoBlindado'
$Trabajo = Join-Path $env:LOCALAPPDATA 'RADAR-Alta-Residuales-Windows2'
$Estado = Join-Path $Trabajo 'estado-alta.json'
$ShaHerramienta = '67f573c7814aa8a8697ee41e45186389c738d1cc761cbeabe9fe7077c16cdbc6'
$ShaPolitica = '50e02c6ab4260a28604d8aaa0980ef7c845385af24d006134b8f4e989899ce8a'

if ((Get-FileHash -LiteralPath $Herramienta -Algorithm SHA256).Hash -ine $ShaHerramienta) { throw 'HERRAMIENTA_NO_COINCIDE' }
if ((Get-FileHash -LiteralPath $Politica -Algorithm SHA256).Hash -ine $ShaPolitica) { throw 'POLITICA_NO_COINCIDE' }
if ((Get-FileHash -LiteralPath $Lanzador -Algorithm SHA256).Hash -ine 'fa6b8feb9622019d497d4b4fb65325e4021feb3cb706c0c2459c5c1fa9b7538a') { throw 'LANZADOR_NO_COINCIDE' }
if (Get-Process -Name 'Radar-Antifraude','RadarMotor' -ErrorAction SilentlyContinue) { throw 'RADAR_NO_ESTA_DETENIDO' }
if (Test-Path -LiteralPath $Trabajo) { throw 'TRABAJO_EXISTENTE_NO_REUTILIZAR' }
foreach ($Carpeta in @($Perfil,$Datos)) {
    if (-not (Test-Path -LiteralPath $Carpeta)) { New-Item -ItemType Directory -Path $Carpeta | Out-Null }
    if (@(Get-ChildItem -LiteralPath $Carpeta -Force).Count -ne 0) { throw 'PERFIL_NUEVO_NO_VACIO' }
}
New-Item -ItemType Directory -Path $Trabajo | Out-Null

& $Herramienta alta-inicial --lanzador $Lanzador --politica $Politica --sha256-politica $ShaPolitica --perfil $Perfil --datos $Datos --estado $Estado --iniciar-windows no
if ($LASTEXITCODE -ne 0) { throw 'ALTA_BLOQUEADA_O_REVERTIDA_CONSERVAR_ESTADO' }
& $Herramienta verificar-alta --estado $Estado --esperado registrada
if ($LASTEXITCODE -ne 0) { throw 'ALTA_NO_VERIFICADA_DETENERSE' }
```

Salidas requeridas con código 0: `ALTA_INICIAL_REGISTRADA_Y_VERIFICADA` y
`ALTA_VERIFICADA=REGISTRADA`. Guardar el estado durable localmente; contiene rutas
y el SID del usuario, no se debe subir a GitHub ni editar.

Este bloque elige explícitamente **no** iniciar al entrar en Windows: registra
el propietario, `Habilitado=0`, `VigilanciaActiva=0` y mantiene ausente Run. En
el visor, propietario será true, iniciar_windows false y registrado false
(este último indica la entrada Run, no la titularidad). La opción de inicio
podrá activarse después por decisión del usuario desde Configuración. Esta
herramienta corregida rechaza `--iniciar-windows si` para el alta inicial.

El alta permite un perfil Estándar sin `licencia-premium-v1.json`. No crea ese
archivo, concede Premium, consulta OAuth ni modifica el almacén de credenciales.
El estado de derechos posterior lo determina el licenciamiento existente.

Tras estas dos salidas, Antigravity puede comunicar el alta correcta y detenerse.
La primera apertura/configuración de correo es una operación separada: no iniciar
Gmail, sincronizaciones o activaciones como parte del alta de propietario.

## Revertir exclusivamente el alta

Bloque alternativo, no ejecutar automáticamente después del alta correcta.
Misma cuenta, RADAR cerrado y mismo lanzador intacto. No necesita que el perfil
siga vacío ni restaura datos/configuración/licencias anteriores. Restablecer
`$Herramienta` y `$Estado` con las mismas rutas si se cerró PowerShell; revalidar
la huella del EXE con el valor anterior antes de ejecutar.

```powershell
& $Herramienta restaurar-alta --estado $Estado
if ($LASTEXITCODE -ne 0) { throw 'ROLLBACK_BLOQUEADO_NO_FORZAR' }
& $Herramienta verificar-alta --estado $Estado --esperado ausente
if ($LASTEXITCODE -ne 0) { throw 'AUSENCIA_NO_VERIFICADA_DETENERSE' }
```

Salidas requeridas con código 0: `ALTA_INICIAL_REVERTIDA_Y_VERIFICADA` y
`ALTA_VERIFICADA=AUSENTE`. AUSENTE se refiere al propietario y su entrada Run:
las preferencias históricas se recuperan exactamente, con su tipo REG_DWORD,
valor y presencia/ausencia. Si antes eran 1/1, vuelven a ser 1/1 sin propietario
ni Run; no se ejecuta el programa. Solo se retiran las claves vacías creadas por
el alta. Se conservan claves y entradas ajenas y todos los datos actuales. Una reversión ya completada puede verificarse o
repetirse sin borrar datos. Los estados `preparada` y `restaurando` permiten
recuperar una interrupción utilizando la misma orden.

Si cambió el propietario, Run, las preferencias del alta o la identidad del
lanzador, la herramienta rechaza la reversión sin sustituir esos valores ajenos.
Conservar estado e informar el código; no modificarlo para forzarla. `restaurar`
y `verificar` corresponden al modo antiguo de transferencia y no se utilizan aquí.
