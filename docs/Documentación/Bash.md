### **Manipulación de archivos y directorios**

1. **Buscar archivos o carpetas**:
```bash
	find /ruta -type f -name "archivo*"
```
1. **Eliminar archivos grandes** (más de 100MB):
```bash
    find /ruta -size +100M -exec rm -i {} \;
``` 
1. **Ver el espacio usado por directorios**:
```bash
    du -h --max-depth=1
```
1. **Listar archivos ordenados por tamaño**:
```bash
    ls -lhS
```
1. **Unir varios archivos en uno**:
```bash
	cat archivo1 archivo2 > combinado.txt
```

---

### **Procesamiento de texto**

1. **Buscar texto en un archivo**:
```bash
    grep "palabra" archivo.txt
```
1. **Contar palabras, líneas o caracteres**:
```bash
    wc archivo.txt  # -w para palabras, -l para líneas, -c para caracteres.
```    
3. **Sustituir texto en archivos**:
```bash
	sed -i 's/viejo/nuevo/g' archivo.txt
```    
4. **Mostrar las primeras o últimas líneas de un archivo**:
```bash
    head -n 10 archivo.txt  # Primeras 10 líneas.
    tail -n 10 archivo.txt  # Últimas 10 líneas.
```

---

### **Redes**

1. **Ver IP pública**:
```bash
	curl ifconfig.me
```
2. **Verificar conectividad con un host**:
```bash
    ping ejemplo.com
```
3. **Ver puertos abiertos**:
```bash
    netstat -tuln
```
4. **Descargar un archivo desde internet**:
```bash
    wget http://ejemplo.com/archivo.zip
```    

---

### **Monitorización y análisis del sistema**

1. **Ver procesos en ejecución**:
```bash
   htop  # Si está instalado, más visual; usa `ps aux` como alternativa.
```    
2. **Ver el uso de memoria**:
```bash
    free -h
```
3. **Ver uso de disco**:
```bash
    df -h
```
4. **Ver tiempo de actividad del sistema**:
```bash
    uptime
```

---

### **Productividad y automatización**

1. **Crear un archivo vacío rápidamente**:
```bash
    touch nuevo_archivo.txt
```
2. **Correr un comando periódicamente**:
```bash
    watch -n 5 'comando'  # Ejecuta el comando cada 5 segundos.
```
3. **Generar una secuencia de números**:
```bash
    seq 1 10
```
4. **Compactar archivos en un tar.gz**:
```bash
    tar -czvf archivo.tar.gz carpeta/
```
5. **Descomprimir un tar.gz**:
```bash
    tar -xzvf archivo.tar.gz
```

---

### **Trucos y personalización**

1. **Autocompletar nombres de archivo**:  
    >Usa `TAB` mientras escribes el nombre de un archivo o comando.    
2. **Historial de comandos**:
```bash
    history | grep "comando"
```
3. **Ejecutar el último comando que contenía "texto"**:
```bash
    !texto
```
4. **Editar un comando del historial antes de ejecutarlo**:
```bash
    fc
```
5. **Definir alias para comandos frecuentes**:
```bash
    alias gs="git status"
```
6. **Guardar alias permanentemente** (edita `.bashrc`):
```bash
    echo 'alias gs="git status"' >> ~/.bashrc
```

