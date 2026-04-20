# 💻Clase 03 - Primeros pasos con Scala

# 🛠️ 1 - Preparación del Entorno de Trabajo para Scala

## Vía 1: IntelliJ IDEA Community Edition (Windows 11 + WSL2)

> 
> 
> 
> **Terminal principal:** PowerShell (Windows 11)
> 
> **Objetivo:** Instalar desde cero WSL2, Ubuntu, Java, sbt e IntelliJ para programar en Scala
> 

---

## 📋 Punto de partida

Esta clase asume que se tiene:

- Windows 11 instalado y actualizado
- **PowerShell** como terminal (no se requiere nada más instalado)
- Conexión a internet estable
- Al menos **8 GB de RAM** y **10 GB de espacio en disco libre**
- Permisos de administrador en el equipo

> ⚠️ **Importante:** WSL2, Ubuntu y todas las herramientas de desarrollo se instalarán desde cero siguiendo esta guía paso a paso.
> 

---

## 🗺️ Visión General del Stack

Al terminar esta clase tendrás este entorno funcionando:

```
Windows 11
│
├── 🪟 LADO WINDOWS
│   ├── Java JDK 17                    (requerido por IntelliJ)
│   ├── sbt                            (para que IntelliJ gestione proyectos)
│   ├── IntelliJ IDEA Community
│   │   └── Plugin: Scala
│   └── Escritorio\Curso-Scala-Copia\  ← copia de seguridad de tus prácticas
│
└── 🐧 LADO WSL2 (Ubuntu 22.04)
    ├── Java JDK 17           (para compilar y ejecutar desde terminal)
    ├── sbt                   (para compilar y ejecutar desde terminal)
    └── ~/Curso-Scala/        ← directorio de trabajo principal
        └── hola-scala/       (primer proyecto de práctica)
```

### ¿Por qué instalar Java y sbt en los dos lados?

| Entorno | Para qué se usa |
| --- | --- |
| **Windows** | IntelliJ necesita su propio JDK para funcionar y compilar proyectos |
| **WSL2** | La terminal de Linux permite gestionar proyectos, compilar y ejecutar sin abrir IntelliJ |
| **Escritorio Windows** | Carpeta `Curso-Scala-Copia` guarda una copia de seguridad de tus prácticas al terminar cada sesión |

Los **archivos de código** viven en WSL (`~/Curso-Scala`) y se copian a Windows al final de cada sesión.

---

## 🔧 Paso 1 — Habilitar WSL2 e instalar Ubuntu desde PowerShell

### 1.1 Abrir PowerShell como Administrador

1. Presiona `Win + X`
2. Selecciona **Terminal de Windows (Administrador)** o **Windows PowerShell (Admin)**
3. Confirma el cuadro de UAC (Control de cuentas de usuario) con **Sí**

Verifica que estás en PowerShell mirando el prompt:

```powershell
# Deberías ver algo como:
PS C:\Windows\System32>
```

### 1.2 Habilitar las características necesarias de Windows

Ejecuta estos dos comandos. Cada uno puede tardar un minuto:

```powershell
# Habilita el subsistema de Linux
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
```

```powershell
# Habilita la plataforma de máquina virtual (requerida por WSL2)
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```

Salida esperada en ambos casos:

```
La operación finalizó correctamente.
```

### 1.3 Reiniciar Windows

```powershell
Restart-Computer
```

> ⚠️ El reinicio es obligatorio para que Windows active las características habilitadas.
> 

### 1.4 Instalar Ubuntu 22.04 desde PowerShell

Abre PowerShell **como Administrador** nuevamente y ejecuta:

```powershell
wsl --install -d Ubuntu-22.04
```

Este comando hace todo automáticamente:

- Descarga e instala el kernel de Linux para WSL2
- Establece WSL2 como versión por defecto
- Descarga e instala Ubuntu 22.04

La descarga de Ubuntu pesa aproximadamente **500 MB**. Al terminar, Ubuntu abrirá una ventana nueva automáticamente.

### 1.5 Crear tu usuario de Ubuntu

Cuando Ubuntu abra por primera vez, te pedirá crear un usuario Linux (independiente de tu usuario Windows):

```bash
Installing, this may take a few minutes...
Please create a default UNIX user account.
Enter new UNIX username: tuNombre
New password: ********
Retype new password: ********
```

> 💡 Elige un nombre en **minúsculas, sin espacios ni tildes**. Por ejemplo: `juan`, `maria`, `dev`.
> 

### 1.6 Actualizar los paquetes de Ubuntu

Ya dentro de la terminal de Ubuntu:

```bash
sudo apt update && sudo apt upgrade -y
```

Ingresa tu contraseña cuando la solicite. Puede tardar **2-5 minutos**.

### 1.7 Verificar WSL2 desde PowerShell

Vuelve a PowerShell y ejecuta:

```powershell
wsl --list --verbose
```

Salida esperada:

```
  NAME            STATE           VERSION
* Ubuntu-22.04    Running         2
```

> ✅ El número **2** en VERSION confirma que estás usando WSL2.
> 

---

## 📂 Paso 2 — Crear el directorio del curso en WSL

Todos los proyectos de este curso vivirán en `~/Curso-Scala` dentro de Ubuntu.

Abre la terminal de **Ubuntu** (búscala en el menú Inicio) y ejecuta:

```bash
mkdir -p ~/Curso-Scala
ls -la ~ | grep Curso-Scala
```

Salida esperada:

```
drwxr-xr-x 2 tuNombre tuNombre 4096 ... Curso-Scala
```

> 📌 **Ruta en WSL:** `/home/tuNombre/Curso-Scala`
> 
> 
> 📌 **Ruta desde Windows:** `\\wsl$\Ubuntu-22.04\home\tuNombre\Curso-Scala`
> 

---

## ☕ Paso 3 — Instalar Java JDK 17 en WSL (Ubuntu)

### 3.1 Instalar OpenJDK 17

En la terminal de Ubuntu:

```bash
sudo apt install -y openjdk-17-jdk
```

Tiempo estimado: **3-5 minutos**.

### 3.2 Verificar la instalación

```bash
java -version
```

Salida esperada:

```
openjdk version "17.x.x" 202x-xx-xx
OpenJDK Runtime Environment (build 17.x.x+x-Ubuntu-...)
OpenJDK 64-Bit Server VM (build 17.x.x+x-Ubuntu-..., mixed mode, sharing)
```

```bash
javac -version
# Salida esperada: javac 17.x.x
```

### 3.3 Configurar JAVA_HOME en WSL

```bash
echo 'export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64' >> ~/.bashrc
echo 'export PATH=$JAVA_HOME/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
echo $JAVA_HOME
# Salida: /usr/lib/jvm/java-17-openjdk-amd64
```

---

## 🔨 Paso 4 — Instalar sbt en WSL (Ubuntu)

### 4.1 Agregar el repositorio oficial de sbt

```bash
sudo apt install -y apt-transport-https curl gnupg
```

```bash
echo "deb https://repo.scala-sbt.org/scalasbt/debian all main" | \
  sudo tee /etc/apt/sources.list.d/sbt.list
```

```bash
curl -sL "https://keyserver.ubuntu.com/pks/lookup?op=get&search=0x2EE0EA64E40A89B84B2DF73499E82A75642AC823" | \
  sudo apt-key add
```

Salida esperada del último comando: `OK`

### 4.2 Instalar sbt

```bash
sudo apt update
sudo apt install -y sbt
```

### 4.3 Verificar la instalación

```bash
sbt --version
```

Salida esperada:

```
sbt version in this project: (none)
sbt script version: 1.x.x
```

> 💡 El mensaje `(none)` en "project" es normal cuando no hay proyecto abierto.
> 

---

## 🪟 Paso 5 — Instalar Java JDK 17 en Windows

IntelliJ IDEA necesita su propio JDK instalado **en el lado Windows**.

### 5.1 Descargar el instalador

1. Abre el navegador y ve a: [https://adoptium.net/temurin/releases/](https://adoptium.net/temurin/releases/)
2. Aplica estos filtros: **Version: 17** | **OS: Windows** | **Architecture: x64** | **Package Type: Installer (.msi)**
3. Descarga el archivo `.msi`

> 💡 **Eclipse Temurin** es la distribución OpenJDK más utilizada en entornos profesionales. Es gratuita y de código abierto.
> 

### 5.2 Instalar el JDK en Windows

Ejecuta el `.msi` descargado. En la pantalla de opciones, asegúrate de marcar:

```
✅ Set JAVA_HOME variable
✅ JavaSoft (Oracle) registry keys
```

Completa el asistente con los valores por defecto.

### 5.3 Verificar desde PowerShell

Abre una **nueva ventana** de PowerShell (sin administrador) y ejecuta:

```powershell
java -version
# Salida: openjdk version "17.x.x" ... Eclipse Adoptium

$env:JAVA_HOME
# Salida: C:\Program Files\Eclipse Adoptium\jdk-17.x.x.x-hotspot
```

---

## 🔨 Paso 6 — Instalar sbt en Windows

### 6.1 Descargar el instalador

1. Ve a [https://www.scala-sbt.org/download.html](https://www.scala-sbt.org/download.html)
2. Descarga el instalador **Windows (msi)**

### 6.2 Instalar sbt

Ejecuta el `.msi` y sigue el asistente con los valores por defecto.

### 6.3 Verificar desde PowerShell

Abre una **nueva ventana** de PowerShell:

```powershell
sbt --version
# Salida: sbt script version: 1.x.x
```

---

## 💡 Paso 7 — Instalar IntelliJ IDEA Community en Windows

### 7.1 Descargar el instalador

1. Ve a [https://www.jetbrains.com/idea/download](https://www.jetbrains.com/idea/download)
2. Selecciona la pestaña **Windows**
3. Descarga la versión **Community** — es el botón **negro/gris**

> ⚠️ **No descargues Ultimate** (botón azul). Ultimate es de pago. Community es gratuita y suficiente para este curso.
> 

### 7.2 Instalar IntelliJ

Ejecuta el `.exe` descargado. En la pantalla de opciones, marca:

```
✅ Add "Open Folder as Project"
✅ Add launchers to PATH
✅ Create Desktop shortcut (64-bit)
```

La instalación tarda aproximadamente **2-3 minutos**.

### 7.3 Primera ejecución

Abre IntelliJ IDEA:

- Acepta el acuerdo de licencia
- En "Data Sharing": elige según tu preferencia
- Elige un tema: **Darcula** (oscuro) o **IntelliJ Light** (claro)
- Haz clic en **Start using IntelliJ IDEA**

---

## 🔌 Paso 8 — Instalar el Plugin de Scala en IntelliJ

El plugin de Scala **no viene preinstalado**. Sin él, IntelliJ no reconocerá el código Scala.

### 8.1 Abrir el gestor de plugins

En la pantalla de bienvenida:

```
Plugins → pestaña Marketplace
```

### 8.2 Instalar el plugin

1. Escribe `Scala` en la barra de búsqueda
2. Selecciona **"Scala"** — autor: *JetBrains*
3. Haz clic en **Install**
4. Al terminar, haz clic en **Restart IDE**

> ✅ El plugin Scala activa: coloreado de sintaxis, autocompletado inteligente, integración con sbt, depuración y REPL de Scala.
> 

---

## 📁 Paso 9 — Crear el primer proyecto de práctica en WSL

### 9.1 Crear la estructura del proyecto

Abre la terminal de **Ubuntu** y ejecuta:

```bash
cd ~/Curso-Scala
mkdir hola-scala
cd hola-scala
```

Crea el archivo de configuración de sbt:

```bash
cat > build.sbt << 'EOF'
ThisBuild / version      := "0.1.0"
ThisBuild / scalaVersion := "3.3.1"

lazy val root = (project in file("."))
  .settings(
    name := "hola-scala"
  )
EOF
```

Crea la estructura de directorios estándar de sbt:

```bash
mkdir -p src/main/scala
mkdir -p src/test/scala
```

Crea el archivo principal de Scala:

```bash
cat > src/main/scala/Main.scala << 'EOF'
@main def main(): Unit =
  println("¡Hola, Scala desde IntelliJ + WSL!")

  val nombre = "Estudiante"
  val mensaje = s"Bienvenido al mundo de Scala, $nombre"
  println(mensaje)
EOF
```

Verifica la estructura del proyecto:

```bash
find . -type f
```

Salida esperada:

```
./build.sbt
./src/main/scala/Main.scala
```

### 9.2 Verificar compilación y ejecución desde WSL

```bash
sbt run
```

> ⏳ La **primera ejecución** descargará Scala y dependencias del compilador (~300 MB). Puede tardar **5-10 minutos**. Las siguientes ejecuciones serán inmediatas.
> 

Salida esperada:

```
[info] welcome to sbt 1.x.x
[info] compiling 1 Scala source ...
[info] running main
¡Hola, Scala desde IntelliJ + WSL!
Bienvenido al mundo de Scala, Estudiante
[success] Total time: xx s
```

> ✅ Si ves `[success]`, el entorno WSL funciona correctamente.
> 

---

## 🖥️ Paso 10 - Abrir el proyecto en IntelliJ IDEA

### 10.1 Localizar los archivos de WSL desde Windows

Los archivos de WSL son accesibles desde Windows en:

```
\\wsl$\Ubuntu-22.04\home\TU_USUARIO\Curso-Scala\hola-scala
```

También puedes navegar desde el **Explorador de archivos** → panel izquierdo → **Linux** → Ubuntu-22.04 → home → tuNombre → Curso-Scala → hola-scala.

### 10.2 Abrir el proyecto

1. En la pantalla de bienvenida de IntelliJ → **Open**
2. Navega a: `\\wsl$\Ubuntu-22.04\home\TU_USUARIO\Curso-Scala\hola-scala`
3. Selecciona la carpeta `hola-scala` → **OK**

### 10.3 Confiar en el proyecto

```
Trust and Open Project 'hola-scala'?  →  Trust Project
```

### 10.4 Configurar la importación sbt

IntelliJ detecta que es un proyecto sbt. En el diálogo de importación:

```
✅ Use sbt shell for build and import
JDK: selecciona "17" del desplegable
```

> ⚠️ **Si el JDK no aparece:** haz clic en **Add JDK...** y apunta a
> 
> 
> `C:\Program Files\Eclipse Adoptium\jdk-17.x.x.x-hotspot`
> 

Haz clic en **OK** y espera a que IntelliJ sincronice el proyecto (~1-2 minutos).

### 10.5 Verificar la importación

Deberías ver:

- `Main.scala` con coloreado de sintaxis (palabras clave en colores)
- La carpeta `src/main/scala` con ícono azul de fuente
- Panel **sbt** visible en la parte inferior

---

## ▶️ Paso 11 — Ejecutar el proyecto desde IntelliJ

### 11.1 Con el botón Run

1. Abre `src/main/scala/Main.scala` en el editor
2. Haz clic en el triángulo ▶ junto a `@main def main()`
3. Selecciona **Run 'main'**

### 11.2 Resultado esperado

```
¡Hola, Scala desde IntelliJ + WSL!
Bienvenido al mundo de Scala, Estudiante

Process finished with exit code 0
```

### 11.3 Desde la sbt Shell integrada

```
View → Tool Windows → sbt Shell
```

Comandos útiles:

```
sbt:hola-scala> compile    ← solo compila
sbt:hola-scala> run        ← compila y ejecuta
sbt:hola-scala> clean      ← borra archivos compilados
sbt:hola-scala> ~run       ← re-ejecuta al guardar (modo watch)
```

> 💡 El modo `~run` es muy útil mientras aprendes: recompila y ejecuta automáticamente cada vez que guardas un archivo.
> 

---

## 🔍 Paso 12 — Familiarizarse con la interfaz de IntelliJ

### Paneles principales

| Panel | Ubicación | Para qué sirve |
| --- | --- | --- |
| **Project** | Izquierda | Navegar la estructura de archivos |
| **Editor** | Centro | Escribir y editar código Scala |
| **sbt Shell** | Inferior | Ejecutar comandos sbt |
| **Run** | Inferior | Ver la salida del programa |
| **Terminal** | Inferior | Terminal integrada (abre Bash de WSL) |
| **Problems** | Inferior | Errores de compilación en tiempo real |

### Atajos de teclado esenciales

| Atajo | Acción |
| --- | --- |
| `Shift + F10` | Ejecutar el programa |
| `Ctrl + Espacio` | Autocompletado |
| `Ctrl + /` | Comentar / descomentar línea |
| `Ctrl + Alt + L` | Formatear código automáticamente |
| `Shift + Shift` | Búsqueda global |
| `Alt + Enter` | Sugerencias de corrección rápida |
| `Ctrl + B` | Ir a la definición |
| `Ctrl + Z` | Deshacer |

---

## 🧪 Ejercicio Guiado

Modifica `Main.scala` con el siguiente código y ejecútalo desde IntelliJ:

```scala
@main def main(): Unit =

  // --- Variables con tipos explícitos ---
  val nombre: String  = "Estudiante"
  val edad: Int       = 25
  val activo: Boolean = true

  println(s"Nombre:  $nombre")
  println(s"Edad:    $edad años")
  println(s"Activo:  $activo")

  // --- Lista y recorrido ---
  val herramientas = List("Scala", "sbt", "IntelliJ", "WSL2")

  println("\nHerramientas del curso:")
  herramientas.foreach(h => println(s"  ✔ $h"))

  // --- Expresión condicional ---
  val nivel = if edad < 30 then "Junior" else "Senior"
  println(s"\nNivel estimado: $nivel")
```

**Resultado esperado:**

```
Nombre:  Estudiante
Edad:    25 años
Activo:  true

Herramientas del curso:
  ✔ Scala
  ✔ sbt
  ✔ IntelliJ
  ✔ WSL2

Nivel estimado: Junior
```

---

## 💾 Paso 13 — Copiar las prácticas de WSL a Windows al terminar cada sesión

Al finalizar tus sesiones de práctica, es importante guardar una copia de tu trabajo en Windows. Esto te protege ante posibles problemas con WSL y te permite acceder a tus archivos aunque no tengas Ubuntu disponible.

> 📌 **Regla del curso:** al terminar cada sesión de prácticas, copia el contenido de `~/Curso-Scala` (WSL) a la carpeta `Curso-Scala-Copia` en el Escritorio de Windows.
> 

---

### 13.1 Crear la carpeta de destino en el Escritorio de Windows

Haz esto **una sola vez**. Abre **PowerShell** (sin necesidad de administrador) y ejecuta:

```powershell
New-Item -ItemType Directory -Path "$env:USERPROFILE\Desktop\Curso-Scala-Copia" -Force
```

Verifica que se creó:

```powershell
Test-Path "$env:USERPROFILE\Desktop\Curso-Scala-Copia"
# Salida: True
```

---

### 13.2 Copiar manualmente al terminar una sesión

Cada vez que termines de practicar, ejecuta este comando en **PowerShell**:

```powershell
# Reemplaza "tuNombre" por tu usuario real de Ubuntu
$origen  = "\\wsl$\Ubuntu-22.04\home\tuNombre\Curso-Scala"
$destino = "$env:USERPROFILE\Desktop\Curso-Scala-Copia"

Copy-Item -Path $origen -Destination $destino -Recurse -Force
```

> 💡 El parámetro `-Recurse` copia todas las subcarpetas y archivos. El parámetro `-Force` sobreescribe los archivos que ya existan en el destino, manteniendo siempre la versión más reciente.
> 

Verifica que la copia se realizó:

```powershell
Get-ChildItem "$env:USERPROFILE\Desktop\Curso-Scala-Copia"
```

Deberías ver la carpeta `Curso-Scala` con todos tus proyectos dentro.

---

### 13.3 Automatizar la copia con un script reutilizable (opcional)

Para no tener que recordar el comando cada vez, puedes crear un script PowerShell que ejecutes con un doble clic.

**Paso 1 — Crear el script**

En PowerShell, ejecuta:

```powershell
$scriptPath = "$env:USERPROFILE\Desktop\copiar-scala.ps1"

$contenido = @'
# =============================================
# Script: copiar-scala.ps1
# Descripción: Copia los proyectos de Curso-Scala
#              desde WSL al escritorio de Windows
# Uso: Ejecutar al terminar cada sesión de práctica
# =============================================

# --- Configuración ---
$usuarioWSL = "tuNombre"   # <-- cambia esto por tu usuario de Ubuntu
$origen     = "\\wsl$\Ubuntu-22.04\home\$usuarioWSL\Curso-Scala"
$destino    = "$env:USERPROFILE\Desktop\Curso-Scala-Copia"
$fecha      = Get-Date -Format "yyyy-MM-dd HH:mm:ss"

# --- Verificar que WSL está disponible ---
if (-not (Test-Path $origen)) {
    Write-Host "❌ No se encontró la carpeta de origen: $origen" -ForegroundColor Red
    Write-Host "   Asegúrate de que Ubuntu (WSL2) está en ejecución." -ForegroundColor Yellow
    pause
    exit 1
}

# --- Ejecutar la copia ---
Write-Host ""
Write-Host "📂 Copiando proyectos de Curso-Scala..." -ForegroundColor Cyan
Write-Host "   Origen:  $origen"
Write-Host "   Destino: $destino"
Write-Host ""

Copy-Item -Path $origen -Destination $destino -Recurse -Force

# --- Confirmación ---
Write-Host "✅ Copia completada el $fecha" -ForegroundColor Green
Write-Host ""

# Muestra los proyectos copiados
Write-Host "📁 Proyectos en Curso-Scala-Copia:" -ForegroundColor Cyan
Get-ChildItem "$destino\Curso-Scala" -Directory | ForEach-Object {
    Write-Host "   └── $($_.Name)"
}

Write-Host ""
pause
'@

Set-Content -Path $scriptPath -Value $contenido -Encoding UTF8
Write-Host "✅ Script creado en: $scriptPath"
```

**Paso 2 — Editar el nombre de usuario**

Abre el archivo `copiar-scala.ps1` que se creó en el escritorio con el Bloc de notas y cambia la línea:

```powershell
$usuarioWSL = "tuNombre"   # <-- cambia esto por tu usuario real de Ubuntu
```

Por ejemplo, si tu usuario de Ubuntu es `juan`:

```powershell
$usuarioWSL = "juan"
```

Guarda y cierra el archivo.

**Paso 3 — Permitir la ejecución del script**

Windows bloquea los scripts PowerShell por defecto. Ejecuta esto **una sola vez** en PowerShell como Administrador:

```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

Confirma con `S` cuando te lo pregunte.

**Paso 4 — Ejecutar el script**

Desde ahora, al terminar cada sesión de prácticas:

1. Abre PowerShell
2. Escribe:
O también puedes hacer **clic derecho** sobre `copiar-scala.ps1` en el escritorio → **Ejecutar con PowerShell**.
    
    ```powershell
    & "$env:USERPROFILE\Desktop\copiar-scala.ps1"
    ```
    

Salida esperada del script:

```
📂 Copiando proyectos de Curso-Scala...
   Origen:  \\wsl$\Ubuntu-22.04\home\juan\Curso-Scala
   Destino: C:\Users\juan\Desktop\Curso-Scala-Copia

✅ Copia completada el 2025-03-15 18:42:10

📁 Proyectos en Curso-Scala-Copia:
   └── hola-scala
```

---

### 13.4 Estructura resultante en Windows

Después de la primera copia, el escritorio tendrá esta estructura:

```
Escritorio de Windows
└── Curso-Scala-Copia\
    └── Curso-Scala\
        ├── hola-scala\
        │   ├── build.sbt
        │   └── src\
        │       └── main\
        │           └── scala\
        │               └── Main.scala
        └── (futuros proyectos del curso...)
```

> ⚠️ **Nota importante:** la carpeta `Curso-Scala-Copia` es solo una **copia de respaldo**. No edites archivos directamente en ella. El trabajo siempre se hace en WSL (`~/Curso-Scala`) y se copia a Windows al finalizar.
> 

---

## ✅ Checklist de Verificación Final

**En PowerShell (Windows):**

```powershell
java -version       # openjdk 17.x.x (Temurin / Eclipse Adoptium)
sbt --version       # sbt script version: 1.x.x
$env:JAVA_HOME      # C:\Program Files\Eclipse Adoptium\jdk-17...
```

**En la terminal de Ubuntu (WSL):**

```bash
java -version       # openjdk 17.x.x (Ubuntu)
sbt --version       # sbt script version: 1.x.x
echo $JAVA_HOME     # /usr/lib/jvm/java-17-openjdk-amd64
ls ~/Curso-Scala    # debe mostrar la carpeta hola-scala
```

**En IntelliJ:**

```
✅ El proyecto hola-scala abre sin errores
✅ Main.scala tiene coloreado de sintaxis
✅ El botón Run ▶ ejecuta el programa correctamente
✅ La sbt Shell responde comandos
```

**Copia de seguridad:**

```
✅ La carpeta Curso-Scala-Copia existe en el Escritorio de Windows
✅ El script copiar-scala.ps1 está en el Escritorio y se ejecuta sin errores
✅ Después de ejecutarlo, Curso-Scala-Copia contiene la carpeta hola-scala
```

---

## 🚨 Solución de Problemas Comunes

### ❌ PowerShell dice "acceso denegado"

**Causa:** No se abrió como Administrador.

**Solución:** Clic derecho en PowerShell → **Ejecutar como administrador**.

### ❌ Ubuntu no abre sola después del reinicio

**Solución:** Búscala en el menú Inicio como **Ubuntu 22.04** y ábrela manualmente. Completará la instalación la primera vez.

### ❌ `wsl --list --verbose` muestra VERSION 1

```powershell
wsl --set-version Ubuntu-22.04 2
wsl --set-default-version 2
```

### ❌ IntelliJ no encuentra el JDK al importar el proyecto

```
File → Project Structure → SDKs → + → Add JDK
Ruta: C:\Program Files\Eclipse Adoptium\jdk-17.x.x.x-hotspot
```

### ❌ El plugin de Scala no colorea la sintaxis

```
File → Invalidate Caches → Invalidate and Restart
```

### ❌ `sbt run` en WSL falla en la primera ejecución

**Causa:** Descarga interrumpida.

**Solución:** Vuelve a ejecutar `sbt run`. sbt retomará desde donde se quedó.

### ❌ Error "object Main is not a member of package"

Verifica que `build.sbt` tenga exactamente:

```scala
ThisBuild / scalaVersion := "3.3.1"
```

Y que `Main.scala` use la sintaxis de Scala 3:

```scala
@main def main(): Unit = ...
```

---

## 📚 Resumen de lo Instalado

| Herramienta | Versión | Entorno |
| --- | --- | --- |
| OpenJDK 17 (Eclipse Temurin) | 17 LTS | Windows 11 |
| sbt | 1.x.x | Windows 11 |
| IntelliJ IDEA Community | Última disponible | Windows 11 |
| Plugin Scala para IntelliJ | Última disponible | IntelliJ (Windows) |
| OpenJDK 17 (Ubuntu) | 17 LTS | WSL2 / Ubuntu 22.04 |
| sbt | 1.x.x | WSL2 / Ubuntu 22.04 |
| Scala 3 | 3.3.1 | Gestionado por sbt (ambos lados) |
| Directorio de trabajo | — | `~/Curso-Scala` en WSL2 |
| Copia de seguridad | — | `Escritorio\Curso-Scala-Copia` en Windows |
| Script de copia | — | `Escritorio\copiar-scala.ps1` en Windows |

---

## Vía 2: Jupyter Notebooks desde VSCode (Windows 11 + WSL2)

> 
> 
> 
> **Terminal principal:** PowerShell (Windows 11) + terminal Ubuntu (WSL2)
> 
> **Prerequisito:** Haber completado la **Vía 1** (WSL2 + Ubuntu 22.04 + Java JDK 17 instalados)
> 
> **Objetivo:** Configurar VSCode con Jupyter y el kernel Almond para programar Scala en notebooks interactivos
> 

---

## 📋 Punto de partida

Esta vía asume que ya se tiene:

- WSL2 con Ubuntu 22.04 funcionando (instalado en la Vía 1)
- Java JDK 17 instalado en WSL (instalado en la Vía 1)
- El directorio `~/Curso-Scala` creado en WSL (creado en la Vía 1)
- El script `copiar-scala.ps1` en el Escritorio de Windows (creado en la Vía 1)
- **VSCode ya instalado en Windows**

Lo que configuraremos en esta clase:

- Extensiones de VSCode: WSL, Python y Jupyter
- Python 3 y Jupyter en WSL
- El kernel **Almond** (Scala para Jupyter)
- Subcarpeta `notebooks` dentro de `~/Curso-Scala` para los notebooks

---

## 🗺️ Visión General del Stack

Al terminar esta clase tendrás este entorno adicional funcionando:

```
Windows 11
│
├── 🪟 LADO WINDOWS
│   └── VSCode
│       ├── Extensión: WSL (conecta VSCode con Ubuntu)
│       ├── Extensión: Python
│       └── Extensión: Jupyter
│
├── 🐧 LADO WSL2 (Ubuntu 22.04)
│   ├── Python 3 + pip
│   ├── Jupyter (jupyter-lab / jupyter notebook)
│   ├── Kernel Almond (Scala 3 para Jupyter)  ← el componente clave
│   └── ~/Curso-Scala/
│       ├── hola-scala/           (proyectos sbt — Vía 1)
│       └── notebooks/            ← nueva subcarpeta para notebooks
│           └── hola-notebook.ipynb
│
└── 🗂️ COPIA DE SEGURIDAD (Windows)
    └── Escritorio\Curso-Scala-Copia\   (script ya existente, actualizado)
```

### ¿Qué es Almond y por qué lo usamos?

**Almond** es el kernel de Scala más completo y mantenido activamente para Jupyter. Permite ejecutar código Scala celda a celda, exactamente como Python en un notebook. Es compatible con Scala 3 y soporta librerías del ecosistema Big Data. Su instalación es oficial y está respaldada por el equipo de Scala.

---

## 🔌 Paso 1 — Verificar VSCode e instalar las extensiones necesarias

Antes de instalar las extensiones, verifica que VSCode está correctamente en el PATH. Abre PowerShell y ejecuta:

```powershell
code --version
```

Salida esperada:

```
1.xx.x
xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
x64
```

> ⚠️ Si el comando no se reconoce, abre VSCode manualmente desde el menú Inicio y ejecuta: `Ctrl + Shift + P` → `Shell Command: Install 'code' command in PATH`. Luego abre una nueva ventana de PowerShell y vuelve a intentarlo.
> 

VSCode necesita tres extensiones para trabajar con Scala en Jupyter desde WSL.

### 1.1 Abrir VSCode

Ábrelo desde el menú Inicio o ejecutando `code` en PowerShell.

### 1.2 Abrir el panel de extensiones

```
Ctrl + Shift + X
```

O haz clic en el ícono de bloques apilados en la barra lateral izquierda.

### 1.3 Instalar la extensión WSL

1. Busca: `WSL`
2. Selecciona **"WSL"** — autor: *Microsoft*
3. Haz clic en **Install**

> Esta extensión permite que VSCode se conecte directamente al sistema de archivos y al entorno de Ubuntu, como si estuviera ejecutándose dentro de WSL.
> 

### 1.4 Instalar la extensión Python

1. Busca: `Python`
2. Selecciona **"Python"** — autor: *Microsoft*
3. Haz clic en **Install**

### 1.5 Instalar la extensión Jupyter

1. Busca: `Jupyter`
2. Selecciona **"Jupyter"** — autor: *Microsoft*
3. Haz clic en **Install**

> ✅ Esta extensión instala automáticamente sus dependencias: **Jupyter Keymap**, **Jupyter Notebook Renderers** y **Jupyter Cell Tags**.
> 

### 1.6 Verificar las extensiones instaladas

En el panel de extensiones, filtra por instaladas:

```
@installed
```

Deberías ver:

```
✅ WSL           (Microsoft)
✅ Python        (Microsoft)
✅ Jupyter       (Microsoft)
```

---

## 🐍 Paso 2 — Instalar Python y Jupyter en WSL

Almond necesita que Jupyter esté instalado en el entorno Linux para registrar su kernel.

### 2.1 Abrir la terminal de Ubuntu

Búscala en el menú Inicio como **Ubuntu 22.04** o ábrela desde el Terminal de Windows.

### 2.2 Instalar Python 3 y pip

Ubuntu 22.04 ya incluye Python 3 por defecto, pero hay que asegurarse de tener `pip` y las herramientas de entorno virtual:

```bash
sudo apt update
sudo apt install -y python3 python3-pip python3-venv
```

Verifica:

```bash
python3 --version
# Salida: Python 3.10.x  (o superior)

pip3 --version
# Salida: pip 22.x.x ...
```

### 2.3 Instalar Jupyter

```bash
pip3 install --user jupyter jupyterlab
```

> 💡 El flag `--user` instala Jupyter solo para tu usuario, sin necesitar `sudo`. Es la práctica recomendada en Linux.
> 

Añade la ruta de instalación de usuario al PATH:

```bash
echo 'export PATH=$HOME/.local/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
```

Verifica:

```bash
jupyter --version
```

Salida esperada (los números pueden variar):

```
Selected Jupyter core packages...
IPython          : 8.x.x
ipykernel        : 6.x.x
jupyter_client   : 8.x.x
jupyter_core     : 5.x.x
jupyterlab       : 4.x.x
notebook         : 7.x.x
```

---

## ⚙️ Paso 3 — Instalar el kernel Almond (Scala para Jupyter)

Almond es el puente entre Jupyter y Scala. Su instalación requiere una herramienta llamada `coursier`, que es el gestor de dependencias estándar del ecosistema Scala.

### 3.1 Instalar Coursier en WSL

En la terminal de Ubuntu:

```bash
curl -fL "https://github.com/coursier/launchers/raw/master/cs-x86_64-pc-linux.gz" | gzip -d > cs
chmod +x cs
./cs setup
```

Durante la instalación, `cs setup` puede preguntarte si deseas actualizar el PATH. Responde `y` (sí).

Después, recarga el entorno:

```bash
source ~/.bashrc
```

Verifica:

```bash
cs --version
# Salida: Coursier 2.x.x
```

### 3.2 Instalar el kernel Almond con Scala 3

```bash
cs launch --use-bootstrap almond:0.14.0-RC14 --scala 3.3.1 -- --install
```

> ⏳ Este proceso descarga el compilador de Scala 3, el runtime de Almond y sus dependencias. Puede tardar **5-10 minutos** la primera vez según tu conexión. Es completamente normal.
> 

Salida esperada al finalizar:

```
Installed scala kernelspec in /home/tuNombre/.local/share/jupyter/kernels/scala
```

### 3.3 Verificar que el kernel quedó registrado

```bash
jupyter kernelspec list
```

Salida esperada:

```
Available kernels:
  python3    /home/tuNombre/.local/share/jupyter/kernels/python3
  scala      /home/tuNombre/.local/share/jupyter/kernels/scala
```

> ✅ La presencia de `scala` en la lista confirma que Almond está correctamente instalado.
> 

---

## 📂 Paso 4 — Crear la subcarpeta de notebooks en el directorio del curso

Los notebooks de este curso se organizarán dentro del mismo directorio `~/Curso-Scala`, en una subcarpeta dedicada.

En la terminal de Ubuntu:

```bash
mkdir -p ~/Curso-Scala/notebooks
ls ~/Curso-Scala
```

Salida esperada:

```
hola-scala   notebooks
```

---

## 🔗 Paso 5 — Conectar VSCode a WSL

Esta es la forma correcta de trabajar: VSCode corre en Windows pero opera **dentro del entorno de Ubuntu**. Así tiene acceso directo a Python, Jupyter y el kernel Almond instalados en WSL.

### 5.1 Conectar VSCode a WSL desde PowerShell

En PowerShell ejecuta:

```powershell
code --remote wsl+Ubuntu-22.04 /home/tuNombre/Curso-Scala/notebooks
```

> ⚠️ Reemplaza `tuNombre` por tu usuario real de Ubuntu.
> 

Esto abrirá VSCode conectado a WSL y posicionado directamente en la carpeta de notebooks.

### 5.2 Verificar la conexión WSL en VSCode

En la esquina inferior izquierda de VSCode verás un indicador verde:

```
><  WSL: Ubuntu-22.04
```

> ✅ Este indicador confirma que VSCode está operando dentro del entorno de Ubuntu. Cualquier terminal que abras desde VSCode será una terminal Bash de Linux, no PowerShell.
> 

### 5.3 Forma alternativa: abrir desde VSCode

Si prefieres hacerlo desde la interfaz gráfica:

1. Abre VSCode normalmente
2. Presiona `F1` o `Ctrl + Shift + P` para abrir la paleta de comandos
3. Escribe: `WSL: Connect to WSL`
4. Selecciona la opción y espera a que VSCode se reconecte
5. Luego: `File → Open Folder` → navega a `/home/tuNombre/Curso-Scala/notebooks`

---

## 📓 Paso 6 — Crear el primer notebook de Scala

### 6.1 Crear un nuevo notebook

Con VSCode conectado a WSL y la carpeta `notebooks` abierta:

1. Presiona `Ctrl + Shift + P` para abrir la paleta de comandos
2. Escribe: `Jupyter: Create New Jupyter Notebook`
3. Selecciona la opción

Se abrirá un notebook vacío sin nombre. Guárdalo con `Ctrl + S`:

```
Nombre: hola-notebook.ipynb
Ubicación: /home/tuNombre/Curso-Scala/notebooks/
```

### 6.2 Seleccionar el kernel de Scala

En la esquina superior derecha del notebook verás el selector de kernel. Haz clic en él y selecciona:

```
Select Another Kernel → Jupyter Kernel → Scala (almond ... scala 3.3.1)
```

> ⚠️ Si no aparece el kernel de Scala, ejecuta en la terminal de Ubuntu:
> 
> 
> ```bash
> jupyter kernelspec list
> ```
> 
> Si `scala` aparece en la lista, cierra y vuelve a abrir VSCode conectado a WSL.
> 

### 6.3 Verificar que el kernel Scala está activo

En la esquina superior derecha del notebook deberías ver:

```
Scala (almond ... scala 3.3.1)  ●
```

El punto `●` indica que el kernel está activo.

---

## ▶️ Paso 7 — Ejecutar código Scala en el notebook

### 7.1 Primera celda — verificación del entorno

Haz clic en la primera celda del notebook, escribe el siguiente código y presiona `Shift + Enter` para ejecutarlo:

```scala
println("¡Hola desde Scala en Jupyter!")
println(s"Scala version: ${scala.util.Properties.versionString}")
```

> ⏳ La **primera ejecución** tarda entre **30-60 segundos** porque Almond inicializa el compilador de Scala. Las siguientes celdas se ejecutan en menos de un segundo.
> 

Salida esperada debajo de la celda:

```
¡Hola desde Scala en Jupyter!
Scala version: version 3.3.1
```

### 7.2 Segunda celda — variables y tipos

Añade una nueva celda (`Ctrl + Shift + Enter` o el botón `+` en la barra del notebook) y escribe:

```scala
// Variables inmutables
val nombre: String  = "Estudiante"
val edad: Int       = 25
val activo: Boolean = true

println(s"Nombre:  $nombre")
println(s"Edad:    $edad años")
println(s"Activo:  $activo")
```

Presiona `Shift + Enter`.

Salida esperada:

```
Nombre:  Estudiante
Edad:    25 años
Activo:  true
```

### 7.3 Tercera celda — colecciones

```scala
val herramientas = List("Scala", "Jupyter", "VSCode", "Almond")

println("Herramientas del entorno:")
herramientas.foreach(h => println(s"  ✔ $h"))
```

Salida esperada:

```
Herramientas del entorno:
  ✔ Scala
  ✔ Jupyter
  ✔ VSCode
  ✔ Almond
```

### 7.4 Cuarta celda — expresión con resultado visible

Una de las ventajas de los notebooks es que la **última expresión de una celda** muestra su valor automáticamente, sin necesidad de `println`:

```scala
val cuadrados = (1 to 5).map(n => n * n).toList
cuadrados
```

Salida esperada (valor de retorno automático):

```
cuadrados: List[Int] = List(1, 4, 9, 16, 25)
```

> 💡 Esta característica de los notebooks hace que explorar datos y probar expresiones sea mucho más ágil que en un proyecto sbt clásico.
> 

---

## 🔍 Paso 8 — Familiarizarse con la interfaz de notebooks en VSCode

### Modos del notebook

| Modo | Cómo activarlo | Para qué sirve |
| --- | --- | --- |
| **Edición** | Clic dentro de la celda | Escribir o editar código |
| **Comando** | `Esc` | Navegar entre celdas, añadir/eliminar |

### Atajos de teclado esenciales en el notebook

| Atajo | Acción |
| --- | --- |
| `Shift + Enter` | Ejecutar celda y avanzar a la siguiente |
| `Ctrl + Enter` | Ejecutar celda sin avanzar |
| `Alt + Enter` | Ejecutar celda e insertar una nueva debajo |
| `Esc` + `A` | Insertar celda **arriba** de la actual |
| `Esc` + `B` | Insertar celda **abajo** de la actual |
| `Esc` + `D D` | Eliminar celda actual |
| `Esc` + `M` | Convertir celda a Markdown (texto) |
| `Esc` + `Y` | Convertir celda a código |
| `Ctrl + Z` | Deshacer dentro de la celda |

### Tipos de celda

Los notebooks admiten dos tipos de celdas:

**Celda de código:** ejecuta Scala y muestra el resultado.

**Celda de Markdown:** permite escribir texto formateado, títulos, listas y notas. Muy útil para documentar el notebook. Para insertar una, usa `Esc + M` o el menú de tipo de celda.

Ejemplo de celda Markdown:

```markdown
## Mi primer notebook de Scala

Este notebook explora los tipos básicos de Scala:
- `val`: variable inmutable
- `var`: variable mutable
- Tipos: `Int`, `String`, `Boolean`, `Double`
```

---

## 🧪 Ejercicio Guiado

Crea un nuevo notebook llamado `tipos-basicos.ipynb` en `~/Curso-Scala/notebooks/` y completa las siguientes celdas:

**Celda 1 — Markdown (título):**

```markdown
# Tipos Básicos en Scala 3
Exploración interactiva de variables y tipos fundamentales.
```

**Celda 2 — Código:**

```scala
// val vs var
val inmutable = 100        // no se puede reasignar
var mutable   = 200        // sí se puede reasignar

mutable = 250              // esto funciona
println(s"inmutable: $inmutable")
println(s"mutable:   $mutable")
```

**Celda 3 — Código:**

```scala
// Inferencia de tipos
val entero   = 42
val decimal  = 3.14
val texto    = "Scala"
val bandera  = true

println(s"${entero.getClass.getSimpleName}:  $entero")
println(s"${decimal.getClass.getSimpleName}: $decimal")
println(s"${texto.getClass.getSimpleName}:  $texto")
println(s"${bandera.getClass.getSimpleName}: $bandera")
```

**Resultado esperado de la celda 3:**

```
Int:     42
Double:  3.14
String:  Scala
Boolean: true
```

**Celda 4 — Código (expresión directa):**

```scala
// Rango y operaciones
val rango = (1 to 10).toList
val pares = rango.filter(_ % 2 == 0)
val suma  = pares.sum

(pares, suma)
```

**Resultado esperado:**

```
(List(2, 4, 6, 8, 10), 30)
```

---

## 💾 Paso 9 — Actualizar el script de copia para incluir los notebooks

El script `copiar-scala.ps1` que creaste en la Vía 1 ya copia todo el directorio `~/Curso-Scala`, lo que incluye automáticamente la subcarpeta `notebooks`. No necesitas modificar nada.

Sin embargo, puedes verificarlo ejecutando el script después de crear los notebooks:

```powershell
& "$env:USERPROFILE\Desktop\copiar-scala.ps1"
```

Salida esperada actualizada:

```
📂 Copiando proyectos de Curso-Scala...
   Origen:  \\wsl$\Ubuntu-22.04\home\juan\Curso-Scala
   Destino: C:\Users\juan\Desktop\Curso-Scala-Copia

✅ Copia completada el 2025-xx-xx xx:xx:xx

📁 Proyectos en Curso-Scala-Copia:
   └── hola-scala
   └── notebooks
```

La estructura en Windows quedará así:

```
Escritorio\Curso-Scala-Copia\
└── Curso-Scala\
    ├── hola-scala\               (proyectos sbt — Vía 1)
    │   ├── build.sbt
    │   └── src\...
    └── notebooks\                (notebooks Jupyter — Vía 2)
        ├── hola-notebook.ipynb
        └── tipos-basicos.ipynb
```

> 📌 **Recuerda:** ejecuta el script `copiar-scala.ps1` al terminar cada sesión, tanto si trabajaste con proyectos sbt (Vía 1) como con notebooks (Vía 2).
> 

---

## ✅ Checklist de Verificación Final

**Extensiones de VSCode (Windows):**

```
✅ Extensión WSL instalada y activa
✅ Extensión Python instalada y activa
✅ Extensión Jupyter instalada y activa
```

**En la terminal de Ubuntu (WSL):**

```bash
python3 --version        # Python 3.10.x o superior
jupyter --version        # muestra versiones de los componentes
jupyter kernelspec list  # debe mostrar "scala" en la lista
cs --version             # Coursier 2.x.x
ls ~/Curso-Scala         # debe mostrar hola-scala y notebooks
```

**En VSCode conectado a WSL:**

```
✅ El indicador inferior izquierdo muestra ">< WSL: Ubuntu-22.04"
✅ El notebook hola-notebook.ipynb abre correctamente
✅ El kernel seleccionado es "Scala (almond ... scala 3.3.1)"
✅ Shift + Enter ejecuta celdas sin errores
```

**Copia de seguridad:**

```
✅ copiar-scala.ps1 copia también la carpeta notebooks
✅ Curso-Scala-Copia en el Escritorio contiene hola-scala y notebooks
```

---

## 🚨 Solución de Problemas Comunes

### ❌ VSCode no muestra el kernel de Scala en el selector

**Causa 1:** VSCode no está conectado a WSL.

**Solución:** Verifica que la esquina inferior izquierda muestre `>< WSL: Ubuntu-22.04`. Si no, conecta usando `Ctrl + Shift + P` → `WSL: Connect to WSL`.

**Causa 2:** Almond no está registrado en Jupyter.

**Solución:** En la terminal de Ubuntu ejecuta:

```bash
jupyter kernelspec list
```

Si `scala` no aparece, reinstala el kernel:

```bash
cs launch --use-bootstrap almond:0.14.0-RC14 --scala 3.3.1 -- --install
```

### ❌ `code --version` no funciona en PowerShell

**Causa:** VSCode no se añadió al PATH durante la instalación.

**Solución:** Abre VSCode manualmente desde el menú Inicio, luego:

```
Ctrl + Shift + P → Shell Command: Install 'code' command in PATH
```

Abre una nueva ventana de PowerShell y vuelve a intentarlo.

### ❌ `cs setup` falla o no descarga correctamente

**Causa:** Posible problema de red o permisos.

**Solución:**

```bash
rm -f cs
curl -fL "https://github.com/coursier/launchers/raw/master/cs-x86_64-pc-linux.gz" | gzip -d > cs
chmod +x cs
./cs setup --yes
source ~/.bashrc
```

### ❌ La primera ejecución de una celda tarda más de 2 minutos

**Causa:** Almond está descargando dependencias adicionales en el primer uso.

**Solución:** Esperar. Si pasan más de 5 minutos sin respuesta, reinicia el kernel desde VSCode:

```
Ctrl + Shift + P → Jupyter: Restart Kernel
```

### ❌ `jupyter --version` no se reconoce como comando

**Causa:** La ruta `~/.local/bin` no está en el PATH.

**Solución:**

```bash
echo 'export PATH=$HOME/.local/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
jupyter --version
```

### ❌ El notebook muestra error "Kernel died" al ejecutar

**Causa:** Falta de memoria RAM disponible o conflicto con otra instancia de Jupyter.

**Solución:**

```bash
# Cierra procesos jupyter huérfanos
pkill -f jupyter
```

Luego recarga el kernel en VSCode: `Ctrl + Shift + P` → `Jupyter: Restart Kernel`.

---

## 📚 Resumen de lo Instalado en la Vía 2

| Herramienta | Versión | Entorno |
| --- | --- | --- |
| VSCode | Última disponible | Windows 11 |
| Extensión WSL | Última disponible | VSCode (Windows) |
| Extensión Python | Última disponible | VSCode (Windows) |
| Extensión Jupyter | Última disponible | VSCode (Windows) |
| Python 3 | 3.10.x (Ubuntu) | WSL2 / Ubuntu 22.04 |
| Jupyter / JupyterLab | Última disponible | WSL2 / Ubuntu 22.04 |
| Coursier (cs) | 2.x.x | WSL2 / Ubuntu 22.04 |
| Almond (kernel Scala) | 0.14.0-RC14 | WSL2 / Ubuntu 22.04 |
| Subcarpeta notebooks | — | `~/Curso-Scala/notebooks` en WSL2 |

### Comparativa entre las dos vías

| Aspecto | Vía 1: IntelliJ + sbt | Vía 2: VSCode + Jupyter |
| --- | --- | --- |
| **Tipo de flujo** | Proyectos completos compilados | Exploración interactiva celda a celda |
| **Feedback** | Compilación → ejecución | Resultado inmediato por celda |
| **Ideal para** | Programas estructurados, OOP, producción | Aprendizaje, pruebas rápidas, análisis |
| **Visualización** | Consola de texto | Tablas, gráficos, texto enriquecido |
| **Archivos** | `.scala` + `build.sbt` | `.ipynb` |
| **Similitud con** | Proyecto Java/Python estándar | Google Colab / Jupyter con Python |

---

# Sesión 2 |  Fundamentos de Scala

---

> ⚠️ **Prerequisito:** El entorno de trabajo debe estar instalado y funcionando antes de esta sesión. Si aún no lo tienes, sigue la guía **" Preparación del Entorno (Vía 1)"** que cubre la instalación de WSL2, Ubuntu 22.04, Java JDK 17, sbt e IntelliJ IDEA.
> 

---

## 🧠 BLOQUE TEÓRICO

---

### 1. ¿Qué es Scala?

Scala es un lenguaje de programación **moderno, potente y conciso** que nació en 2003 en la École Polytechnique Fédérale de Lausanne (EPFL), Suiza, de la mano del profesor **Martin Odersky**.

El nombre viene de **"Scalable Language"** → un lenguaje diseñado para crecer contigo: sirve tanto para un script pequeño como para sistemas distribuidos a gran escala.

> 💡 **Analogía para principiantes:** Imagina que Java es un coche familiar robusto y seguro, pero algo rígido. Scala es una versión mejorada del mismo coche: misma potencia, misma carretera (la JVM), pero con cambios automáticos, asientos más cómodos y opciones extra que Java no tiene.
> 

---

### 2. ¿Por qué Scala existe? El problema que resuelve

Antes de Scala, los desarrolladores usaban **Java** para casi todo. Java es muy sólido, pero tiene problemas:

- Mucho código repetitivo (boilerplate) para hacer cosas simples
- No tiene soporte nativo para programación funcional
- Es verboso: necesitas muchas líneas para expresar ideas sencillas

Martin Odersky quería un lenguaje que:

1. Corriera sobre la **JVM** (para aprovechar el ecosistema Java)
2. Fuera mucho más **expresivo y conciso**
3. Soportara **programación orientada a objetos** Y **programación funcional** de forma unificada

Así nació Scala.

---

### 3. Scala: un lenguaje híbrido

Esta es la característica más importante de Scala y la que lo hace especial:

```
Scala = Orientado a Objetos + Funcional
```

**¿Qué significa esto?**

| Característica | Descripción | Ejemplo |
| --- | --- | --- |
| Orientado a Objetos | Todo es un objeto, incluso los números | `1.toString` es válido en Scala |
| Funcional | Las funciones son valores, se pueden pasar como parámetros | `val doble = (x: Int) => x * 2` |
| Tipado estático | El compilador detecta errores antes de ejecutar | Si sumas un número y un texto, falla en compilación |
| Inferencia de tipos | No siempre hace falta declarar el tipo | `val nombre = "Ana"` → Scala sabe que es String |

---

### 4. Scala y la JVM: ¿cómo funciona por debajo?

Scala **no tiene su propia máquina virtual**. Compila a **bytecode de Java**, el mismo que entiende la JVM.

```
Tu código .scala
       ↓
  Compilador scalac
       ↓
  Bytecode .class  (igual que Java)
       ↓
      JVM
       ↓
   Ejecución
```

**¿Qué ventajas tiene esto?**

- Puedes usar **cualquier librería Java** desde Scala
- Scala funciona en cualquier lugar donde funcione Java
- El rendimiento es comparable al de Java
- Apache Spark está escrito en Scala → integración perfecta

---

### 5. ¿Por qué Scala para Big Data?

Esta es la pregunta clave del módulo: ¿por qué no usamos Python o Java directamente?

**Apache Spark está escrito en Scala.** Cuando usas Spark con Scala:

- Accedes a la **API nativa** de Spark, sin capas intermedias
- Obtienes **verificación de tipos en tiempo de compilación** → menos errores en producción
- El rendimiento es **máximo**, ya que no hay traducción
- El código es más **conciso** que Java y más **eficiente** que Python para Big Data

> 💡 **Comparativa rápida:**
> 

| Lenguaje | Ventajas con Spark | Desventajas |
| --- | --- | --- |
| **Scala** | API nativa, máximo rendimiento, tipado fuerte | Curva de aprendizaje |
| Java | Familiar para muchos, tipado fuerte | Muy verboso, sin soporte funcional nativo |
| Python (PySpark) | Fácil de aprender, popular en Data Science | Overhead de traducción, menos eficiente para pipelines complejos |
| R (SparkR) | Bueno para estadística | Limitado para producción |

> Para un entorno profesional de Big Data, Scala + Spark es la combinación más potente.
> 

---

### 6. El ecosistema Scala

Scala no está solo. A su alrededor ha crecido un ecosistema robusto:

| Herramienta | ¿Qué es? |
| --- | --- |
| **sbt** | Scala Build Tool — el gestor de proyectos y dependencias |
| **IntelliJ IDEA** | El IDE más popular para Scala |
| **Apache Spark** | El framework de Big Data por excelencia |
| **Akka** | Framework para sistemas distribuidos y concurrentes |
| **Play Framework** | Framework web para aplicaciones Scala |
| **Cats / ZIO** | Librerías avanzadas de programación funcional |

> En este curso nos centraremos en **sbt, IntelliJ IDEA y Apache Spark**.
> 

---

### 7. Nuestro entorno de trabajo: IntelliJ + WSL2

En la Vía 1 del curso trabajamos con esta arquitectura:

```
Windows 11
│
├── 🪟 LADO WINDOWS
│   ├── Java JDK 17                    (requerido por IntelliJ)
│   ├── sbt                            (para que IntelliJ gestione proyectos)
│   ├── IntelliJ IDEA Community
│   │   └── Plugin: Scala
│   └── Escritorio\Curso-Scala-Copia\  ← copia de seguridad de tus prácticas
│
└── 🐧 LADO WSL2 (Ubuntu 22.04)
    ├── Java JDK 17           (para compilar y ejecutar desde terminal)
    ├── sbt                   (para compilar y ejecutar desde terminal)
    └── ~/Curso-Scala/        ← directorio de trabajo principal
        └── hola-scala/       (proyecto que usaremos hoy)
```

**¿Por qué esta arquitectura?**

- El **código fuente** vive en WSL (`~/Curso-Scala`) — entorno Linux profesional
- **IntelliJ** se conecta a esos archivos desde Windows para editar con comodidad
- La **sbt Shell** de IntelliJ compila y ejecuta dentro de WSL
- Al terminar cada sesión, el script `copiar-scala.ps1` hace una copia de seguridad en Windows

---

### 8. La sbt Shell: tu herramienta de trabajo diaria

Dentro de IntelliJ, la **sbt Shell** es la consola desde la que compilarás y ejecutarás todo tu código Scala. Es equivalente al terminal, pero integrada en el IDE.

Para abrirla:

```
View → Tool Windows → sbt Shell
```

Comandos que usarás constantemente:

```
sbt:nombre-proyecto> compile    ← compila el código sin ejecutar
sbt:nombre-proyecto> run        ← compila y ejecuta
sbt:nombre-proyecto> clean      ← borra archivos compilados
sbt:nombre-proyecto> ~run       ← modo watch: recompila al guardar
```

> 💡 El modo `~run` es especialmente útil mientras aprendes: cada vez que guardas un archivo, Scala recompila y ejecuta automáticamente. Así ves los resultados al instante.
> 

---

## 💻 BLOQUE PRÁCTICO

---

### 🔧 Ejercicio A — Verificación del entorno (15 min)

Antes de empezar a programar, confirma que todo está en orden.

**Paso 1 — Verificar desde la terminal de Ubuntu (WSL):**

Abre Ubuntu desde el menú Inicio y ejecuta:

```bash
java -version
# Esperado: openjdk version "17.x.x"

sbt --version
# Esperado: sbt script version: 1.x.x

echo $JAVA_HOME
# Esperado: /usr/lib/jvm/java-17-openjdk-amd64

ls ~/Curso-Scala
# Esperado: hola-scala
```

**Paso 2 — Verificar desde PowerShell (Windows):**

```powershell
java -version
# Esperado: openjdk version "17.x.x" (Eclipse Adoptium)

sbt --version
# Esperado: sbt script version: 1.x.x
```

**Paso 3 — Verificar IntelliJ:**

- Abre IntelliJ IDEA
- Comprueba que el plugin de Scala está instalado: `File → Settings → Plugins → Installed` → busca "Scala"

**✅ Criterio de éxito:** Los tres entornos responden sin error.

> ⚠️ Si algo falla → levantar la mano. No avanzar hasta tener el entorno funcionando.
> 

---

### 🔧 Ejercicio B — Abrir el proyecto en IntelliJ y explorar la interfaz (15 min)

El proyecto `hola-scala` ya fue creado durante la instalación del entorno. Vamos a abrirlo en IntelliJ.

**Paso 1 — Abrir el proyecto:**

1. En la pantalla de bienvenida de IntelliJ → **Open**
2. Navega a: `\\wsl$\Ubuntu-22.04\home\TU_USUARIO\Curso-Scala\hola-scala`
3. Selecciona la carpeta `hola-scala` → **OK**
4. Si aparece el diálogo de confianza → **Trust Project**

**Paso 2 — Configurar sbt si lo pide:**

Si IntelliJ muestra un diálogo de importación sbt:

```
✅ Use sbt shell for build and import
JDK: selecciona "17"
```

Haz clic en **OK** y espera ~1-2 minutos a que sincronice.

**Paso 3 — Explorar la estructura del proyecto:**

En el panel **Project** (izquierda) deberías ver:

```
hola-scala
├── build.sbt              ← configuración del proyecto
├── project/
│   └── build.properties   ← versión de sbt
└── src/
    ├── main/
    │   └── scala/
    │       └── Main.scala  ← tu código fuente
    └── test/
        └── scala/          ← tests (lo usaremos más adelante)
```

**Paso 4 — Familiarizarte con los paneles:**

| Panel | Ubicación | Para qué sirve |
| --- | --- | --- |
| **Project** | Izquierda | Navegar archivos del proyecto |
| **Editor** | Centro | Escribir y editar código |
| **sbt Shell** | Inferior | Compilar y ejecutar |
| **Run** | Inferior | Ver la salida del programa |
| **Problems** | Inferior | Errores de compilación en tiempo real |

**Paso 5 — Atajos esenciales que usarás hoy:**

| Atajo | Acción |
| --- | --- |
| `Shift + F10` | Ejecutar el programa |
| `Ctrl + Espacio` | Autocompletado |
| `Ctrl + /` | Comentar / descomentar línea |
| `Ctrl + Alt + L` | Formatear código automáticamente |
| `Ctrl + Z` | Deshacer |
| `Alt + Enter` | Sugerencias de corrección rápida |

---

### 🔧 Ejercicio C — Primer programa: modificar y ejecutar Main.scala

Abre `src/main/scala/Main.scala` desde el panel Project. Verás el código que se creó durante la instalación:

```scala
@main def main(): Unit =
  println("¡Hola, Scala desde IntelliJ + WSL!")

  val nombre = "Estudiante"
  val mensaje = s"Bienvenido al mundo de Scala, $nombre"
  println(mensaje)
```

**Paso 1 — Ejecutar el programa tal como está:**

Haz clic en el triángulo ▶ junto a `@main def main()` en el margen izquierdo → **Run 'main'**

Resultado esperado en el panel Run:

```
¡Hola, Scala desde IntelliJ + WSL!
Bienvenido al mundo de Scala, Estudiante

Process finished with exit code 0
```

**Paso 2 — Entender el código línea por línea:**

```scala
@main def main(): Unit =
```

- `@main` → anotación de Scala 3 que marca el punto de entrada del programa
- `def main()` → define una función llamada `main`
- `Unit` → tipo de retorno que significa "esta función no devuelve ningún valor" (equivalente a `void` en Java)

```scala
  println("¡Hola, Scala desde IntelliJ + WSL!")
```

- `println(...)` → imprime una línea de texto en la consola

```scala
  val nombre = "Estudiante"
```

- `val` → declara una variable **inmutable** (no se puede cambiar una vez asignada)
- Scala **infiere** el tipo `String` automáticamente

```scala
  val mensaje = s"Bienvenido al mundo de Scala, $nombre"
```

- `s"..."` → **interpolación de strings**: el prefijo `s` activa la inserción de variables
- `$nombre` → inserta el valor de la variable dentro del texto

**Paso 3 — Modificar el programa:**

Reemplaza todo el contenido de `Main.scala` con este código:

```scala
@main def main(): Unit =

  // Mi primer programa modificado en Scala
  println("=== Primer programa en Scala ===")

  val lenguaje  = "Scala"
  val version   = "3.3.1"
  val ide       = "IntelliJ IDEA"
  val terminal  = "WSL2 + Ubuntu 22.04"

  println(s"Lenguaje:  $lenguaje $version")
  println(s"IDE:       $ide")
  println(s"Terminal:  $terminal")
  println("")
  println("¡Entorno listo para Big Data!")
```

**Paso 4 — Compilar y ejecutar desde la sbt Shell:**

Abre la sbt Shell (`View → Tool Windows → sbt Shell`) y escribe:

```
sbt:hola-scala> run
```

Resultado esperado:

```
=== Primer programa en Scala ===
Lenguaje:  Scala 3.3.1
IDE:       IntelliJ IDEA
Terminal:  WSL2 + Ubuntu 22.04

¡Entorno listo para Big Data!
```

> 💡 Los comentarios en Scala empiezan con `//`. IntelliJ los muestra en gris para distinguirlos del código ejecutable.
> 

---

### 🔧 Ejercicio D — Presentación personal con tipos de datos

Ahora vas a escribir un programa más completo usando los tipos de datos básicos de Scala.

**Paso 1 — Reemplaza el contenido de `Main.scala` con este código:**

```scala
@main def main(): Unit =

  // ── Tipos de datos básicos en Scala ──────────────────
  val nombre:  String  = "Ana García"   // texto
  val edad:    Int     = 28             // número entero
  val altura:  Double  = 1.68           // número decimal
  val activo:  Boolean = true           // verdadero / falso
  val curso:   String  = "IFCD0115 — Procesamiento Big Data con Scala"

  // ── Imprimir con interpolación de strings ─────────────
  println("╔══════════════════════════════════════╗")
  println("║       PRESENTACIÓN DEL ALUMNO        ║")
  println("╚══════════════════════════════════════╝")
  println("")
  println(s"  Nombre:   $nombre")
  println(s"  Edad:     $edad años")
  println(s"  Altura:   $altura m")
  println(s"  Activo:   $activo")
  println(s"  Curso:    $curso")
  println("")

  // ── Operaciones con los datos ─────────────────────────
  val edadEnMeses   = edad * 12
  val alturaEnCm    = (altura * 100).toInt
  val añoNacimiento = 2025 - edad

  println(s"  Edad en meses:    $edadEnMeses meses")
  println(s"  Altura en cm:     $alturaEnCm cm")
  println(s"  Año nacimiento:   $añoNacimiento")
  println("")

  // ── Expresión condicional ─────────────────────────────
  val experiencia = if edad >= 30 then "Senior" else "Junior"
  println(s"  Nivel estimado:   $experiencia")
  println("")
  println("  ¡Bienvenido/a al curso de Big Data con Scala!")
```

**Paso 2 — Ejecutar desde la sbt Shell:**

```
sbt:hola-scala> run
```

Resultado esperado:

```
╔══════════════════════════════════════╗
║       PRESENTACIÓN DEL ALUMNO        ║
╚══════════════════════════════════════╝

  Nombre:   Ana García
  Edad:     28 años
  Altura:   1.68 m
  Activo:   true
  Curso:    IFCD0115 — Procesamiento Big Data con Scala

  Edad en meses:    336 meses
  Altura en cm:     168 cm
  Año nacimiento:   1997

  Nivel estimado:   Junior

  ¡Bienvenido/a al curso de Big Data con Scala!
```

**Paso 3 — Personaliza el programa:**

Cambia los valores de `nombre`, `edad` y `altura` por los tuyos propios. Vuelve a ejecutar con `run`.

> 📝 **Observa qué pasa con los tipos:**
> 
> - `edad * 12` funciona porque `edad` es `Int` y `12` es `Int` → resultado `Int`
> - `altura * 100` devuelve `Double` → usamos `.toInt` para convertirlo a entero
> - `if edad >= 30 then "Senior" else "Junior"` → en Scala, el `if` **devuelve un valor**. Esto es diferente a Java y lo exploraremos más en próximas sesiones.

---

### 🔧 Ejercicio E — Reto de refuerzo

Añade al final de `Main.scala` la siguiente sección que imprime una lista de herramientas:

```scala
  // ── Lista de herramientas ─────────────────────────────
  val herramientas = List("Scala 3.3.1", "Apache Spark", "sbt", "IntelliJ IDEA", "WSL2")

  println("  Herramientas que aprenderemos:")
  herramientas.foreach(h => println(s"    ✔ $h"))
```

Resultado esperado al final de la salida:

```
  Herramientas que aprenderemos:
    ✔ Scala 3.3.1
    ✔ Apache Spark
    ✔ sbt
    ✔ IntelliJ IDEA
    ✔ WSL2
```

> 🏆 E**xtra:** Prueba el modo watch. Escribe `~run` en la sbt Shell y luego modifica y guarda el archivo. Observa cómo IntelliJ recompila y ejecuta automáticamente sin que tengas que hacer nada más.
> 

---

### 💾 Al terminar la sesión — Copia de seguridad

Antes de cerrar IntelliJ, ejecuta el script de copia en PowerShell:

```powershell
& "$env:USERPROFILE\Desktop\copiar-scala.ps1"
```

Resultado esperado:

```
📂 Copiando proyectos de Curso-Scala...
✅ Copia completada el 2025-xx-xx xx:xx:xx

📁 Proyectos en Curso-Scala-Copia:
   └── hola-scala
```

> 📌 **Recuerda:** ejecutar este script al final de cada sesión de prácticas para no perder tu trabajo.
> 

---