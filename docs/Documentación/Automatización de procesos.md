### **1. Crea un script para los commits automáticos**

El script debe hacer lo siguiente:

1. Cambiar al directorio del repositorio.
2. Agregar todos los cambios al área de preparación.
3. Realizar un commit con un mensaje predefinido.
4. Opcionalmente, hacer un push al repositorio remoto.

#### Ejemplo de script en Bash (Linux/macOS)

```bash
#!/bin/bash

# Cambiar al directorio del repositorio
cd /ruta/a/tu/repositorio || exit

# Agregar cambios al staging
git add .

# Realizar commit
fecha=$(date "+%Y-%m-%d %H:%M:%S")
git commit -m "Commit automático: $fecha"

# Subir cambios al repositorio remoto (opcional)
git push origin main
```

Guarda este archivo como `auto_commit.sh` y dale permisos de ejecución:

```bash
chmod +x auto_commit.sh
```

---

#### Ejemplo de script en Batch (Windows)

```shell
@echo off
:: Cambiar al directorio del repositorio
cd /d C:\ruta\a\tu\repositorio

:: Agregar cambios
git add .

:: Realizar commit con fecha
for /f "tokens=2 delims==" %%i in ('wmic os get localdatetime /value')
do
	set datetime=%%i
	set formattedDate=%datetime:~0,4%-%datetime:~4,2%-%datetime:~6,2% %datetime:~8,2%:%datetime:~10,2%:%datetime:~12,2%
	git commit -m "Commit automático: %formattedDate%" 

:: Subir cambios al repositorio remoto (opcional)
git push origin main

```
Guarda este archivo como `auto_commit.bat`.

---

### **2. Programa el script para ejecutarse automáticamente**

#### **En Linux/macOS (usando `cron`):**

1. asd
```bash
crontab -e
```
2. Agrega una línea para programar la ejecución diaria del script:
```bash
0 18 * * * /ruta/a/auto_commit.sh
```
	Esto ejecutará el script todos los días a las 18:00.

#### **En Windows (usando Tareas Programadas):**

1. Abre el **Programador de Tareas**.
2. Crea una **tarea básica**:
    - Dale un nombre y selecciona "Diariamente".
    - Configura la hora a la que quieres que se ejecute.
    - En "Acción", elige "Iniciar un programa" y selecciona el archivo `auto_commit.bat`.
3. Guarda la tarea.

#cron #automation 


