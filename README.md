# Powershell
#Aportaciones a mi trabajo, dia a dia o meramente pruebas.
#ESQUELETO DE CARPETAS, CONDICIONES DE DUPLICADO Y TEXTO EN EL BLOCK DE NOTAS.
# Definir la ruta de la carpeta base
$rutaBase = "D:\LACOSA\2.2024\1. DL"

# Definir la ruta del archivo que contiene los números de carpeta registrados
$rutaArchivo = "D:\LACOSA\0.DS\R\RCARPETAS.txt"

# Función para obtener el último número asignado
function ObtenerUltimoNumero {
    $ultimoNumero = 0
    $carpetas = Get-ChildItem $rutaBase -Directory
    foreach ($carpeta in $carpetas) {
        if ($carpeta.Name -match '^(\d+)\.L') {
            $numero = [int]$Matches[1]
            if ($numero -gt $ultimoNumero) {
                $ultimoNumero = $numero
            }
        }
    }
    return $ultimoNumero
}

# Función para leer los números de carpeta registrados desde el archivo
function LeerNumerosRegistrados {
    if (Test-Path $rutaArchivo) {
        Get-Content $rutaArchivo
    } else {
        @()
    }
}

# Función para agregar un número al archivo
function AgregarNumeroAlArchivo($numero) {
    Add-Content $rutaArchivo $numero
}

# Función para procesar una carpeta
function ProcesarCarpeta($carpeta) {
    $nombreOriginal = $carpeta.Name
    if ($nombreOriginal -match '^\d+$') {
        $numero = [int]$nombreOriginal
        $ultimoNumero = ObtenerUltimoNumero
        $numerosRegistrados = LeerNumerosRegistrados
        if ($numero -gt $ultimoNumero -and $numero -notin $numerosRegistrados) {
            $nuevoNumero = $ultimoNumero + 1
            $nuevoNombre = "$nuevoNumero.L $nombreOriginal"
            $nuevaRuta = Join-Path $rutaBase $nuevoNombre

            Rename-Item -Path $carpeta.FullName -NewName $nuevoNombre
            $fechaHora = Get-Date
            Write-Host "Se ha modificado la carpeta principal a $($nuevoNombre) - Fecha y hora: $($fechaHora)"

            $carpetasProcesadas += $nombreOriginal

            CrearSubcarpetas $nuevaRuta $nombreOriginal
            CrearAccesoDirecto $nuevaRuta
            AgregarNumeroAlArchivo $nombreoriginal
            # Llamar a la función para crear el archivo de texto y agregar el contenido solo una vez
            CrearArchivoTexto $nuevaRuta
        }
    }
}

# Función para crear las subcarpetas
function CrearSubcarpetas($ruta, $nombreOriginal) {
    $subcarpetas = @(".1 ASF NORMAL", ".2 ASF MODIFICADO", ".3 RIEGO LIGA", ".4 RIEGO IMPREGNACION", ".5 TIRADO CARPETA", ".6 MEZCLA ASF", ".7 MEZCLA ASF MOD", ".8 CARPETA ASF", ".9 AGREGADOS ASF", ".10 SPT", ".11 PCA", ".12 BASE", ".13 SUBBASE", ".14 SUBRASANTE", ".15 TERRAPLEN", ".16 GRAVA", ".17 ARENA", ".18 PSVM", ".19 CILINDROS", ".20 VIGAS", ".21 MORTEROS", ".22 ESCLEROMETRO", ".23 TROMPA ELEFANTE", ".24 CONO Y ARENA", ".25 EQUIPO N", ".26 ACERO", ".27 BLOCK", ".28 PLANICIDADES")

    foreach ($subcarpetaNombre in $subcarpetas) {
        $subcarpetaNuevoNombre = "$nombreOriginal$subcarpetaNombre"
        $subcarpetaPath = Join-Path $ruta $subcarpetaNuevoNombre

        if (-not (Test-Path -Path $subcarpetaPath)) {
            New-Item -ItemType Directory -Path $subcarpetaPath | Out-Null
        }
    }
}

# Función para crear el acceso directo
function CrearAccesoDirecto($ruta) {
    $formatosPath = "D:\LACOSA\1.SGC\FORMATOS"
    $shortcutName = "00.SGC-FORMATOS.lnk"
    $shortcutPath = Join-Path $ruta $shortcutName

    if (-not (Test-Path -Path $shortcutPath)) {
        $WshShell = New-Object -ComObject WScript.Shell
        $Shortcut = $WshShell.CreateShortcut($shortcutPath)
        $Shortcut.TargetPath = $formatosPath
        $Shortcut.Save()
    }
}

# Función para crear el archivo de texto y agregar el contenido solo una vez
function CrearArchivoTexto($ruta) {
    $documentoNombre = "$nombreOriginal.txt"
    $documentoPath = Join-Path $ruta $documentoNombre
    
    if (-not (Test-Path $documentoPath)) {
        New-Item -ItemType File -Path $documentoPath | Out-Null
        $contenido = @"
NIVEL DE PRUEBAS
SOLO SERÁN LAS PRUEBAS CLASIFICADAS POR ÁREA (ASFALTO, GEOTECNIA, CONCRETOS, COMPACTACIONES, PRUEBAS ESPECIALES) 

1.ASFALTO:
    1-CALIDADES DE ASFALTO NORMAL
    2-CALIDADES DE ASFALTO MODIFICADO
    3-RIEGO DE LIGA
    4-RIEGO DE IMPREGNACIÓN
    5-CONTROL DE TIRADO DE CARPETA
    6-CALIDADES MEZCLA ASFALTICA NORMAL
    7-CALIDADES MEZCLA ASFALTICA MODIFICADO
    8-COMPACTACIÓN DE CARPETA ASF.
    9-AGREGADOS ASF.

2.GEOTECNIA:
    10-SPT
    11-PCA

3.TERRACERIAS:
    12-CALIDADES DE BASE
    13-CALIDADES SUBBASE
    14-CALIDADES DE SUBRASANTE
    15-CALIDAD DE TERRAPLEN
    16-CALIDAD DE GRAVA
    17-CALIDAD DE ARENA
    18-PSVM

4.CONCRETOS:
    19-CILINDROS
    20-VIGAS
    21-MORTEROS
    22-ESCLERÓMETRO

5.COMPACTACIONES:
    23-TROMPA DE ELEFANTE
    24-CONO Y ARENA
    25-EQUIPO N

6.PRUEBAS ESPECIALES:
    26-ACERO
    27-BLOCK
    28-PLANICIDADES
"@
        Add-Content -Path $documentoPath -Value $contenido
    }
}

# Crear el archivo RCARPETAS.txt si no existe y agregar números si es necesario
if (-not (Test-Path $rutaArchivo)) {
    New-Item -Path $rutaArchivo -ItemType File | Out-Null
}

# Bucle infinito para monitorear nuevas carpetas
while ($true) {
    $carpetas = Get-ChildItem $rutaBase -Directory
    if ($carpetas.Count -gt 0) {
        foreach ($carpeta in $carpetas) {
            ProcesarCarpeta $carpeta
        }
    }
    Start-Sleep -Seconds 2
}




#En resumen hace esto:
#Se definen las rutas base y del archivo RCARPETAS.txt.
#Se verifica si el archivo RCARPETAS.txt existe.
#Si no existe, se crea.
#Se inicia un bucle infinito para monitorear nuevas carpetas.
#Se obtiene una lista de carpetas en la ruta base.
#Si no hay carpetas, se espera 2 segundos y se vuelve a obtener la lista.
#Si hay carpetas, se procede con cada carpeta.
#Para cada carpeta:
#Se verifica si el nombre de la carpeta es un número.
#Si no lo es, se ignora la carpeta.
#Se obtiene el último número asignado.
#Se leen los números de carpeta registrados desde el archivo.
#Se verifica si el número de la carpeta es mayor que el último número asignado y no está en la lista de números registrados.
#Si no cumple la condición, se ignora la carpeta.
#Se incrementa el último número asignado en 1.
#Se renombra la carpeta con el nuevo número asignado.
#Se muestra un mensaje de modificación de carpeta.
#Se crean las subcarpetas.
#Se crea un acceso directo.
#Se agrega el número al archivo RCARPETAS.txt.
#Si el archivo de texto no existe, se crea y se agrega contenido.
#Se espera 2 segundos.
#Fin del bucle para cada carpeta.
#Fin del programa.(No alcanza, fin del programa unicamente forzado)

