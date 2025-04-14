# Instalar Certbot

- **Instalar el paquete "core"**  
    Este comando instala el paquete "core" desde Snap, que es el entorno base necesario para que funcionen las aplicaciones instaladas con Snap.
    
    ```terminal
    sudo snap install core
    ```
    
- **Actualizar "core"**  
    Aquí se actualiza el paquete "core" a su versión más reciente. Esto garantiza que el entorno fundamental de Snap esté actualizado.
    
    ```terminal
    sudo snap refresh core
    ```
    
- **Instalar Certbot**  
    Con este comando instalas Certbot utilizando Snap. El flag `--classic` indica que la aplicación se instala con confinamiento clásico, lo que le permite acceder a más recursos del sistema (necesario para que Certbot pueda interactuar con la configuración de NGINX y el sistema en general)
    
    ```terminal
    sudo snap install --classic certbot
    ```
    
- **Vincular el ejecutable**  
    Este comando crea un enlace simbólico (similar a un atajo) que vincula el ejecutable de Certbot instalado vía Snap (`/snap/bin/certbot`) a una ubicación tradicionalmente usada para comandos del sistema (`/usr/bin/certbot`). De esta manera, al ejecutar el comando `certbot` en la terminal, se usará la versión recién instalada por Snap.
    
    ```terminal
    sudo ln -s /snap/bin/certbot /usr/bin/certbot
    ```



Gracias por compartir todo el contexto, esto aclara perfectamente el problema.

Lo que te está pasando es lo siguiente:

---

### ❗ Estás ejecutando una versión **vieja de Certbot** (la del sistema) en vez de la nueva instalada por Snap

Aunque hiciste bien en instalar Certbot por Snap (✅), todavía se está ejecutando la versión antigua del sistema (la que tiene el bug con `urllib3` y `requests_toolbelt`).

Esto lo podemos confirmar porque en el traceback aparece esto:

```
File "/usr/bin/certbot", line 33, in <module>
...
certbot==1.21.0
```

Pero vos **instalaste la versión `certbot 4.0.0`** con Snap. Esa debería ser la que se ejecute.

---

### 📍 ¿Por qué pasa esto?

Porque la ruta `/usr/bin/certbot` **ya existía** y no la pudiste sobrescribir:

```
ln: failed to create symbolic link '/usr/bin/certbot': File exists
```

Entonces, al ejecutar `certbot`, el sistema está llamando a la vieja, no a la del Snap (`/snap/bin/certbot`).

---

### ✅ Solución paso a paso

1. **Verificá cuál certbot estás usando:**
    

```bash
which certbot
```

Probablemente diga `/usr/bin/certbot`.

---

2. **Eliminá el viejo symlink (o binario del sistema):**
    

```bash
sudo rm /usr/bin/certbot
```

---

3. **Creá el symlink nuevo correctamente:**
    

```bash
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

---

4. **Verificá que apunta al correcto:**
    

```bash
which certbot
certbot --version
```

Deberías ver algo como:

```
/usr/bin/certbot
certbot 4.0.0
```

---

5. **Volvé a ejecutar el comando con confianza:**
    

```bash
sudo certbot --nginx -d urgara-api.cradel.tech
```

---

### Si todo va bien 🎉

Certbot debería ahora:

- Leer correctamente tu config de NGINX.
    
- Validar tu dominio.
    
- Emitir el certificado.
    
- Agregar automáticamente la redirección HTTPS y recargar NGINX.
    

---

¿Querés que revisemos tu configuración de NGINX también para asegurarnos de que todo está listo antes de ejecutar el comando?