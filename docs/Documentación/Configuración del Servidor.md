
# Exponer puertos con Nginx

 Para exponer varios puertos en tu servidor Ubuntu con Nginx, debes configurar **bloques de servidor (server blocks)** para cada puerto y aplicación. Aquí te explico los pasos:

---

### **1. Instalar Nginx (si no está instalado)**
```bash
sudo apt update
sudo apt install nginx
```

---

### **2. Verificar el firewall (UFW)**
Asegúrate de que los puertos que quieres exponer estén habilitados:
```bash
sudo ufw allow <puerto>/tcp  # Ej: 80, 443, 8080, 3000, etc.
sudo ufw reload
```

---

### **3. Configurar múltiples puertos en Nginx**
Cada puerto requiere una configuración específica. Puedes hacerlo de dos formas:

#### **Opción A: Configurar en archivos separados (recomendado)**
1. Crea un archivo de configuración para cada puerto en `/etc/nginx/sites-available/`:
   ```bash
   sudo nano /etc/nginx/sites-available/puerto-8080
   ```

2. **Ejemplo para el puerto 8080** (servir contenido estático):
   ```nginx
   server {
       listen 8080;
       listen [::]:8080;
       server_name _;

       root /var/www/mi-app-1;
       index index.html;

       location / {
           try_files $uri $uri/ =404;
       }
   }
   ```

3. **Ejemplo para el puerto 3000** (proxy inverso a una app):
   ```nginx
   server {
       listen 3000;
       listen [::]:3000;
       server_name _;

       location / {
           proxy_pass http://localhost:5000;  # Redirige al puerto interno 5000
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
       }
   }
   ```

4. Crea enlaces simbólicos a `sites-enabled`:
   ```bash
   sudo ln -s /etc/nginx/sites-available/puerto-8080 /etc/nginx/sites-enabled/
   sudo ln -s /etc/nginx/sites-available/puerto-3000 /etc/nginx/sites-enabled/
   ```

#### **Opción B: Usar un solo archivo con múltiples bloques**
Edita el archivo principal `/etc/nginx/nginx.conf` y añade bloques `server` para cada puerto.

---

### **4. Probar y recargar Nginx**
```bash
sudo nginx -t           # Verifica errores en la configuración
sudo systemctl reload nginx  # Aplica los cambios sin reiniciar
```

---

### **5. Verificar puertos en uso**
```bash
sudo ss -tulpn | grep nginx  # Deberías ver los puertos configurados (ej: 8080, 3000)
```

---

### **Notas importantes**
- **SSL/HTTPS**: Si necesitas exponer puertos con HTTPS (ej: 443), añade los certificados con `ssl_certificate` y `ssl_certificate_key` en el bloque `server`.
- **Reglas de firewall**: Asegúrate de abrir los puertos en el firewall de tu proveedor de cloud (AWS, DigitalOcean, etc.).
- **Logs**: Revisa los logs en `/var/log/nginx/error.log` si hay problemas.

¿Necesitas ayuda con un caso específico? ¡Dime más detalles! 🚀

# Firewall

(Due to technical issues, the search service is temporarily unavailable.)

¡Entendido! Vamos a solucionar el problema paso a paso. Cuando una conexión se queda cargando sin respuesta, suele deberse a problemas de **firewall**, **configuración incorrecta de Nginx**, o **la aplicación no está escuchando en el puerto esperado**. Aquí las soluciones:

---

### **1. Verifica el firewall (UFW y proveedor de cloud)**
- **Local (UFW):**
  ```bash
  sudo ufw status
  ```
  Asegúrate de que el puerto `8080/tcp` esté en estado **ALLOW**.  
  Si no, habilítalo y recarga:
  ```bash
  sudo ufw allow 8080/tcp
  sudo ufw reload
  ```

- **Firewall del proveedor (AWS, DigitalOcean, etc.):**  
  Si usas un servidor en la nube, verifica que el puerto **8080** esté abierto en las reglas de seguridad (Security Groups o Firewall).

---

### **2. Revisa la configuración de Nginx**
#### **a. ¿Nginx está escuchando en el puerto 8080?**
  ```bash
  sudo ss -tulpn | grep nginx
  ```
  Debes ver una línea como `:8080` en la salida.  
  Si no aparece, hay un error en la configuración del bloque `server` para el puerto 8080.

#### **b. Prueba la sintaxis de Nginx:**
  ```bash
  sudo nginx -t
  ```
  Si hay errores, corrígelos en los archivos de configuración (`/etc/nginx/sites-available/puerto-8080`).

#### **c. Verifica el archivo de configuración del puerto 8080:**
  - Asegúrate de que el `root` apunte a una ruta válida (ej: `/var/www/mi-app-1`).
  - Si usas proxy inverso, verifica que la aplicación esté corriendo en el puerto local configurado (ej: `proxy_pass http://localhost:5000`).

---

### **3. ¿La aplicación está funcionando?**
- **Si es un proxy inverso:**  
  Asegúrate de que la aplicación detrás del proxy (ej: en `localhost:5000`) esté **activa** y escuchando conexiones.
  ```bash
  curl http://localhost:5000  # Reemplaza con el puerto de tu app
  ```

- **Si sirves archivos estáticos:**  
  Verifica que el directorio `root` exista y tenga permisos adecuados:
  ```bash
  ls -l /var/www/mi-app-1  # Reemplaza con tu ruta
  sudo chmod -R 755 /var/www/mi-app-1  # Si hay problemas de permisos
  ```

---

### **4. Revisa las conexiones de red**
#### **a. Prueba desde el servidor local:**
  ```bash
  curl http://localhost:8080
  ```
  - Si funciona: El problema es externo (firewall, red, DNS).  
  - Si no funciona: Revisa Nginx o la aplicación nuevamente.

#### **b. Prueba desde tu máquina local:**
  ```bash
  telnet <ip-del-servidor> 8080
  ```
  - Si la conexión se establece pero no hay respuesta: Problema de Nginx/aplicación.  
  - Si la conexión es rechazada: **Firewall bloqueando el puerto**.

---

### **5. Analiza los logs de Nginx**
  ```bash
  sudo tail -f /var/log/nginx/error.log  # Errores en tiempo real
  sudo tail -f /var/log/nginx/access.log # Solicitudes entrantes
  ```
  Busca mensajes como `permission denied`, `connection refused`, o rutas no encontradas.

---

### **6. Casos comunes**
- **Problema de SELinux (en sistemas como CentOS/RHEL):**  
  Ubuntu no usa SELinux por defecto, pero si has modificado políticas de seguridad, podría bloquear el puerto.

- **Conflicto de puertos:**  
  Verifica que otro servicio no esté usando el puerto 8080:
  ```bash
  sudo lsof -i :8080
  ```

- **Nginx no recargó la configuración:**  
  Asegúrate de haber ejecutado:
  ```bash
  sudo systemctl reload nginx
  ```

---

### **Resumen de comandos clave**
```bash
sudo ufw status                   # Verificar firewall
sudo ss -tulpn | grep ":8080"     # Puerto en uso
sudo nginx -t                     # Validar sintaxis
curl http://localhost:8080        # Test local
telnet <IP> 8080                  # Test remoto
```

**¡Cuéntame qué encuentras en cada paso y seguimos depurando!** 🔍

# Dominio y Subdominio

Para asignar **subdominios a puertos específicos** en tu servidor, debes combinar la **configuración DNS** con **bloques de servidor en Nginx** que actúen como proxy inverso. Aquí te explico cómo hacerlo paso a paso:

---

### **1. Configuración DNS (Registrar subdominios)**
En tu proveedor de DNS (Cloudflare, GoDaddy, etc.), crea registros **A** o **CNAME** para cada subdominio apuntando a la IP de tu servidor:

| Tipo   | Nombre (Subdominio) | Valor (IP o dominio)       |
|--------|---------------------|----------------------------|
| A      | app1.tudominio.com  | 192.168.1.1 (IP del server)|
| A      | app2.tudominio.com  | 192.168.1.1                |

**Nota:** Los DNS no manejan puertos, por lo que Nginx se encargará de redirigir el tráfico según el subdominio.

---

### **2. Configuración de Nginx como Proxy Inverso**
Crea un **bloque de servidor (server block)** para cada subdominio, escuchando en los puertos estándar (80/443) y redirigiendo al puerto interno correspondiente.

#### **Ejemplo para `app1.tudominio.com` (puerto 8080):**
1. Crea el archivo de configuración:
   ```bash
   sudo nano /etc/nginx/sites-available/app1.tudominio.com
   ```

2. Configuración básica (HTTP):
   ```nginx
   server {
       listen 80;
       listen [::]:80;
       server_name app1.tudominio.com;  # Subdominio

       location / {
           proxy_pass http://localhost:8080;  # Redirige al puerto 8080
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
       }
   }
   ```

#### **Ejemplo para `app2.tudominio.com` (puerto 3030):**
1. Crea otro archivo:
   ```bash
   sudo nano /etc/nginx/sites-available/app2.tudominio.com
   ```

2. Configuración básica (HTTP):
   ```nginx
   server {
       listen 80;
       listen [::]:80;
       server_name app2.tudominio.com;  # Segundo subdominio

       location / {
           proxy_pass http://localhost:3030;  # Redirige al puerto 3030
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
       }
   }
   ```

---

### **3. Habilitar las configuraciones**
Crea enlaces simbólicos a la carpeta `sites-enabled`:
```bash
sudo ln -s /etc/nginx/sites-available/app1.tudominio.com /etc/nginx/sites-enabled/
sudo ln -s /etc/nginx/sites-available/app2.tudominio.com /etc/nginx/sites-enabled/
```

---

### **4. Configurar SSL (HTTPS Opcional)**
Usa **Certbot** para obtener certificados gratis de Let’s Encrypt:
```bash
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx -d app1.tudominio.com -d app2.tudominio.com
```
Certbot modificará automáticamente los archivos de configuración para agregar HTTPS.

---

### **5. Recargar Nginx y verificar**
```bash
sudo nginx -t           # Verifica errores
sudo systemctl reload nginx
```

---

### **6. Verificar conexiones**
- Accede a los subdominios desde tu navegador:
  - `http://app1.tudominio.com` → Debe redirigir al servicio en el puerto 8080.
  - `http://app2.tudominio.com` → Debe redirigir al puerto 3030.

---

### **Notas clave**
- **Firewall:** Asegúrate de que los puertos **80 y 443** estén abiertos en tu servidor y proveedor de cloud.
- **Aplicaciones locales:** Las aplicaciones en `localhost:8080` y `localhost:3030` deben estar en ejecución.
- **Pruebas locales:** Si no tienes los DNS configurados aún, puedes modificar tu archivo `hosts` local para pruebas:
  ```plaintext
  # En tu computadora (ej: /etc/hosts en Linux o C:\Windows\System32\drivers\etc\hosts en Windows)
  192.168.1.1 app1.tudominio.com
  192.168.1.1 app2.tudominio.com
  ```

---

### **Ejemplo de flujo completo**
1. Un usuario visita `app1.tudominio.com`.
2. El DNS redirige la solicitud a tu IP del servidor.
3. Nginx detecta el subdominio `app1.tudominio.com` y redirige el tráfico a `localhost:8080`.
4. La aplicación en el puerto 8080 responde.

¡Con esto, tus subdominios estarán vinculados a sus respectivos puertos! ¿Necesitas ayuda con algo más? 😊




# PM2
(Due to technical issues, the search service is temporarily unavailable.)

¡Genial! Para subir tu backend de Express al servidor y hacerlo accesible a través de Nginx, sigue estos pasos:

---

### **1. Preparar el entorno en el servidor**
#### **Instalar Node.js y npm**
```bash
sudo apt update
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -  # Reemplaza "18.x" por tu versión de Node.js
sudo apt install -y nodejs
node -v  # Verifica la instalación
```

#### **Instalar PM2 (gestor de procesos para Node.js)**
```bash
sudo npm install -g pm2
```

---

### **2. Subir el código de Express al servidor**
#### **Opción A: Clonar repositorio Git**
```bash
cd ~
git clone https://github.com/tu-usuario/tu-repo-backend.git
cd tu-repo-backend
```

#### **Opción B: Transferir archivos manualmente (vía SCP)**
Desde tu máquina local:
```bash
scp -r ./tu-directorio-backend usuario@tuserver.com:/ruta/destino/
```

---

### **3. Instalar dependencias del proyecto**
```bash
cd /ruta/al/proyecto  # Ej: ~/tu-repo-backend
npm install
```

---

### **4. Configurar variables de entorno**
- Crea un archivo `.env` en la raíz del proyecto con las variables necesarias (puerto, bases de datos, etc.):
  ```bash
  nano .env
  ```
  Ejemplo:
  ```env
  PORT=3000
  NODE_ENV=production
  DB_URI=mongodb://localhost:27017/midb
  ```

---

### **5. Ejecutar la aplicación con PM2**
```bash
pm2 start src/index.js --name "mi-backend"  # Reemplaza "src/index.js" por tu archivo principal
pm2 save  # Guarda los procesos para que persistan después de reinicios
pm2 startup  # Configura PM2 para iniciar automáticamente al reiniciar el servidor
```

#### **Verificar estado:**
```bash
pm2 list  # Deberías ver tu aplicación en "online"
```

---

### **6. Configurar Nginx como Proxy Inverso**
Crea un nuevo archivo de configuración para tu subdominio (ej: `api.tudominio.com`):
```bash
sudo nano /etc/nginx/sites-available/api.tudominio.com
```

#### **Configuración básica (HTTP):**
```nginx
server {
    listen 80;
    listen [::]:80;
    server_name api.tudominio.com;  # Subdominio para el backend

    location / {
        proxy_pass http://localhost:3000;  # Puerto donde escucha Express
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_cache_bypass $http_upgrade;
    }
}
```

#### **Habilitar el sitio:**
```bash
sudo ln -s /etc/nginx/sites-available/api.tudominio.com /etc/nginx/sites-enabled/
```

---

### **7. Configurar HTTPS (Opcional pero recomendado)**
Usa Certbot para obtener un certificado SSL:
```bash
sudo certbot --nginx -d api.tudominio.com
```
Certbot modificará automáticamente el archivo de configuración de Nginx para redirigir tráfico a HTTPS.

---

### **8. Recargar Nginx y probar**
```bash
sudo nginx -t           # Verificar errores
sudo systemctl reload nginx
```

---

### **9. Verificar el funcionamiento**
#### **Desde el servidor:**
```bash
curl http://localhost:3000  # Debe responder tu API
```

#### **Desde tu máquina local:**
```bash
curl https://api.tudominio.com  # Debe devolver la respuesta del backend
```

---

### **Solución de problemas comunes**
1. **La aplicación no responde:**
   - Verifica que Express esté escuchando en `0.0.0.0` y no en `localhost`:
     ```javascript
     app.listen(3000, '0.0.0.0', () => {
         console.log('Servidor en puerto 3000');
     });
     ```
   - Revisa los logs de PM2:
     ```bash
     pm2 logs mi-backend
     ```

2. **Error 502 Bad Gateway:**
   - Asegúrate de que la aplicación esté corriendo en el puerto correcto.
   - Verifica que Nginx tenga permisos para acceder al puerto de Express.

3. **Firewall bloqueando conexiones:**
   ```bash
   sudo ufw allow 3000/tcp  # Solo necesario si accedes directamente al puerto (no si usas Nginx como proxy)
   ```

---

### **Notas adicionales**
- **Puertos:** Si quieres exponer otro backend en el puerto `3030`, repite los pasos 6-8 con un nuevo subdominio (ej: `app2.tudominio.com`).
- **Escalabilidad:** Para manejar más tráfico, considera usar clusters de PM2:
  ```bash
  pm2 start src/index.js -i max --name "mi-backend"
  ```

¡Con esto, tu backend de Express estará accesible desde `api.tudominio.com`! ¿Necesitas ayuda con algo más? 😊