### **¿Qué es el protocolo SSH?**

SSH (Secure Shell) es un protocolo de red que permite a los usuarios acceder y administrar de manera segura dispositivos remotos. Fue diseñado para reemplazar métodos inseguros como Telnet y FTP, ya que cifra toda la comunicación entre el cliente y el servidor.

### Conexión:
```bash
ssh -p<port> <user>@<server>
```

### Transferencia de archivos:

```bash
scp [opciones] origin dest
```

**Subida de archivos**
```bash
scp -P<port>   ./file.sh    <user>@<server>:/path/to/dest/
```
**Descarga de archivos**
```bash
scp -P<port>    <user>@<server>:/path/to/folder.tgz   ./path/to/download/
```


#ssh #scp #server