# INC-04 – Acceso SSH denegado para root

**Informe del Analista e Investigador (Rol 3)**

- Materia: Sistemas Operativos · UNIFRANZ · Gestión 2026
- Docente: Ing. Diego Cárdenas Sejas
- Integrante: [Tu nombre]
- Grupo: [Número o nombre del grupo]
- Fecha: 28 de octubre de 2026

## 1. Resumen del incidente

El solicitante perdió el acceso remoto por SSH con la cuenta root a un servidor Linux que prepara para pruebas de un servicio FTP. Desde la consola de Proxmox el acceso con root funciona, y desde PuTTY la conexión llega a la IP 192.168.50.107, pero la autenticación es rechazada. La causa no estaba confirmada y debía determinarse con evidencias.

Como analista, mi función fue formular hipótesis alternativas, consultar documentación técnica, contrastar los resultados de las pruebas del operador técnico y concluir qué se confirma, qué se descarta y qué queda abierto.

## 2. Estado inicial y datos del entorno

Los datos salen de las capturas del operador técnico (Captura 1 y Captura 2).

| Dato | Valor | Fuente |
| --- | --- | --- |
| Nombre del equipo | servicioftp | Captura 1 |
| Sistema operativo | Alpine Linux v3.24 | Captura 1 (`/etc/os-release`) |
| Interfaz y dirección IP | eth0, 192.168.50.107/24, estado UP | Captura 1 (`ip a`) |
| Gestor de servicios | OpenRC (`systemctl` no existe) | Captura 1 |
| Estado de sshd | started | Captura 2 (`rc-service sshd status`) |
| sshd al arrancar | Sí, en runlevel default | Captura 2 (`rc-update show`) |

Conclusión del estado inicial: la red y el servicio funcionan. El problema está en la autenticación o en la política de sshd, no en la conectividad. Como el cliente llega a pedir usuario y contraseña, también se descarta un bloqueo por firewall.

## 3. Hipótesis planteadas y contraste con la evidencia

| # | Hipótesis | Prueba o evidencia | Resultado | Estado |
| --- | --- | --- | --- | --- |
| H1 | `PermitRootLogin` impedía el acceso de root | Captura 3: la línea `PermitRootLogin yes` aparece descomentada y editada. Captura 4: se reinició sshd. Captura 5: root entra por PuTTY después del cambio | El acceso se recuperó tras el cambio, pero no se capturó el valor original de la directiva | **Probable, pendiente de confirmar** |
| H2 | `PasswordAuthentication no` | Captura 5: la contraseña de root funcionó por SSH | La autenticación por contraseña está habilitada | **Descartada** |
| H3 | Restricciones `AllowUsers` o `DenyUsers` | Captura 5: root entró sin modificar esas directivas | No hay bloqueo por lista de usuarios | **Descartada** |
| H4 | Un archivo adicional de configuración sobrescribe `sshd_config` | Captura 5: el acceso funciona con el cambio aplicado en `sshd_config` | La configuración editada es la efectiva | **Descartada** |
| H5 | PAM o la propia cuenta bloquean el acceso | Captura 5: la sesión de root se abrió normalmente | No hay bloqueo a nivel de cuenta | **Descartada** |
| H6 | Contraseña mal escrita o diferencia de teclado entre consola y PuTTY | Captura 5: primer intento `Access denied`, segundo intento correcto en la misma sesión | Es un factor posible, porque con la configuración ya aplicada el primer intento falló | **Abierta** |
| H7 | Se editó la configuración sin reiniciar el servicio | Captura 4: se ejecutó `rc-service sshd restart` | No sabemos si hubo una edición previa sin reinicio | **Abierta** |
| H8 | La configuración del FTP causó el problema | Captura 6: solo está instalado `openssh-sftp-server`, no hay archivos de FTP en `/etc` y el único puerto en escucha es el 22 | Al momento del diagnóstico no había FTP instalado (Captura 6). El FTP se instaló después, con el acceso de root ya recuperado (Capturas 8 a 11), y no hay evidencia de relación causal | **Descartada como causa, falta prueba de regresión** |

## 4. Análisis

### 4.1 Causa más probable

La evidencia apunta a que la causa fue la directiva `PermitRootLogin` de `/etc/ssh/sshd_config`. Se editó la línea a `yes` (Captura 3), se reinició sshd (Captura 4) y root pudo entrar por PuTTY (Captura 5). Según la documentación de OpenSSH, esta directiva controla si root puede iniciar sesión por SSH y sus valores son `yes`, `prohibit-password`, `forced-commands-only` y `no`. El valor por defecto de OpenSSH es `prohibit-password`, que impide el acceso de root con contraseña. Esto explica por qué root funciona en la consola local (que no pasa por sshd) y falla por SSH.

### 4.2 Limitación de la evidencia

No existe captura del valor que tenía la directiva antes de editarla, ni del mensaje exacto del log cuando root era rechazado. Por eso H1 queda como causa probable y no como causa demostrada. Para confirmarla de forma rigurosa habría que devolver la directiva a `prohibit-password`, reiniciar sshd, intentar entrar con root y revisar el log (por ejemplo `/var/log/messages`), siempre con autorización del docente.

### 4.3 El primer intento fallido en PuTTY

En la Captura 5 aparece `Access denied` y luego un segundo intento exitoso. Con `PermitRootLogin yes` ya aplicado, un rechazo en el primer intento sugiere que la contraseña se escribió mal o que hay diferencia de teclado entre la consola de Proxmox y PuTTY (H6). Queda registrado como observación abierta.

### 4.4 Relación entre el servicio FTP y la incidencia

FTP (File Transfer Protocol) sirve para transferir archivos entre equipos y usa normalmente el puerto 21. Envía usuario y contraseña en texto plano, por lo que hoy se prefieren alternativas cifradas como SFTP o FTPS. FTP y SSH son servicios independientes: no existe una relación directa entre ellos, salvo que se modifiquen archivos compartidos del sistema.

Al momento del diagnóstico, la Captura 6 mostraba que solo estaba instalado `openssh-sftp-server`, que es el subsistema SFTP y viaja dentro de SSH por el puerto 22. No había un servidor FTP instalado ni escuchando en el puerto 21. El FTP se instaló más tarde (sección 4.5), cuando el acceso de root por SSH ya estaba recuperado.

### 4.5 Instalación y configuración del servicio FTP

Después de recuperar el acceso, el operador técnico instaló y configuró vsftpd en el mismo servidor. Estas son las evidencias y lo que significan.

| Paso | Evidencia | Lectura del analista |
| --- | --- | --- |
| Instalación fallida | Captura 8: `apk add vsftpd` responde "no such package" | El repositorio `community` de Alpine estaba comentado en `/etc/apk/repositories`, y vsftpd está en ese repositorio |
| Corrección | Captura 8: se descomentó `community`, se ejecutó `apk update` y la instalación terminó bien (vsftpd 3.0.5-r3, 64 paquetes, 74.0 MiB) | El error era de repositorios, no de red ni de permisos |
| Configuración | Captura 9: `/etc/vsftpd/vsftpd.conf` con `anonymous_enable=NO`, `local_enable=YES` y `write_enable=YES` | Se bloquea el acceso anónimo, se permite iniciar sesión a usuarios locales y se permite escribir archivos |
| Servicio activo | Captura 10: `netstat -lnt` muestra el puerto 21 en escucha, además del 22 | El servidor FTP está corriendo y SSH sigue activo |
| Acceso desde la red | Captura 11: `Test-NetConnection 192.168.50.107 -Port 21` desde 192.168.50.247 devuelve `TcpTestSucceeded: True` | El puerto 21 es alcanzable desde el cliente, pero esto no prueba que el inicio de sesión FTP funcione |

Observaciones del analista:

- **Evidencia incompleta de la configuración.** En la Captura 9, vi marca el archivo como `[Modified]`, o sea con cambios sin guardar. Como en la Captura 10 el puerto 21 ya escucha, se asume que se guardó y se inició el servicio, pero no hay captura de ninguno de los dos pasos.
- **Seguridad.** Con `local_enable=YES`, los usuarios del sistema (incluido `adminlab`) pueden entrar por FTP con su contraseña, y FTP envía las credenciales en texto plano. Se recomienda preferir SFTP (puerto 22) o FTPS, y comprobar en la documentación de vsftpd que root no pueda iniciar sesión por FTP.
- **Relación con el incidente.** Según el orden de las capturas, el FTP se instaló después de recuperar el acceso SSH de root, y ninguna captura muestra cambios en `sshd_config` durante esa instalación. No hay evidencia de que el FTP causara la incidencia.
- **Pendiente.** Una prueba de regresión (confirmar que root y `adminlab` siguen entrando por SSH después de instalar vsftpd) y una prueba de inicio de sesión FTP con `adminlab`.

## 5. Recuperación del acceso y administración remota segura

La Captura 3 muestra que el acceso se recuperó habilitando `PermitRootLogin yes`. Sin embargo, mantener root con contraseña por SSH no es la forma adecuada de administrar un servidor, porque root es un blanco frecuente de ataques de fuerza bruta y no deja trazabilidad de quién hizo cada cambio.

La práctica recomendada es una cuenta nominativa con privilegios elevados bajo demanda:

1. Crear un usuario nominativo. Ya está hecho: `adminlab`, uid 1000 (Captura 7).
2. Dar una contraseña robusta. Pendiente: el sistema advirtió `Bad password: similar to username` y la contraseña se aceptó igualmente porque la cambió root (Captura 7).
3. Dar privilegios de administración. Pendiente: `adminlab` solo pertenece a su propio grupo (`groups=1000(adminlab)`). Debe añadirse al grupo `wheel` y configurarse `doas` o `sudo`.
4. Comprobar el acceso por SSH con `adminlab` y la elevación de privilegios con `doas` o `sudo`.
5. Restringir de nuevo root por SSH (`PermitRootLogin prohibit-password` o `no`), idealmente combinado con autenticación por clave pública.

## 6. Estado de las acciones

| Acción | Estado |
| --- | --- |
| Diagnóstico del entorno (SO, red, servicio) | Hecho |
| Recuperación del acceso de root por SSH | Hecho |
| Verificación de que no había servidor FTP instalado al momento del diagnóstico | Hecho |
| Creación de usuario nominativo `adminlab` | Hecho |
| Contraseña robusta para `adminlab` | Pendiente |
| Privilegios de administración (`wheel` y `doas` o `sudo`) | Pendiente |
| Prueba de acceso SSH con `adminlab` | Pendiente |
| Restringir nuevamente el acceso de root por SSH | Pendiente |
| Captura del estado original de `PermitRootLogin` y del log del rechazo | Pendiente |
| Instalación de vsftpd (corrigiendo el repositorio community) | Hecho |
| Configuración inicial de vsftpd.conf | Hecho (falta evidencia de que se guardó) |
| Verificación del puerto 21 en el servidor y desde el cliente | Hecho |
| Prueba de inicio de sesión FTP con adminlab | Pendiente |
| Prueba de regresión de SSH (root y adminlab) después de instalar FTP | Pendiente |

## 7. Conclusiones

1. La conectividad no es el problema: PuTTY alcanza el servidor y sshd está activo y habilitado al arranque.
2. La causa más probable es la directiva `PermitRootLogin` en `sshd_config`, ya que el acceso se recuperó al cambiarla a `yes` y reiniciar el servicio. Falta la evidencia del valor original para darla por demostrada.
3. El servicio FTP no aparece relacionado con la incidencia: se instaló después de recuperar el acceso de root y no hay evidencia de que alterara la configuración de SSH.
4. La solución a largo plazo no debe ser mantener root habilitado por SSH, sino administrar con una cuenta nominativa con privilegios elevados y, si es posible, claves públicas.
5. Quedan abiertas H6 (contraseña o teclado) y H7 (cambio sin reinicio), que pueden cerrarse repitiendo la prueba de forma controlada.

## 8. Fuentes consultadas

Por verificar y citar con la fecha de consulta antes de entregar:

- Manual de `sshd_config` (OpenSSH): directivas `PermitRootLogin`, `PasswordAuthentication`, `AllowUsers`, `DenyUsers`. Disponible con `man sshd_config` y en man.openbsd.org.
- Alpine Linux Wiki: página "Setting up a new user" (wiki.alpinelinux.org).
- Alpine Linux Wiki: página "Doas" (wiki.alpinelinux.org).
- Alpine Linux Wiki: página "OpenRC", para el manejo de servicios con `rc-service` y `rc-update`.

## 9. Uso de IA

Se usó un asistente de IA como apoyo para generar hipótesis alternativas, interpretar las capturas y proponer comandos de verificación. Sus resultados se contrastaron con la evidencia real del servidor (Capturas 1 a 11) y deben verificarse con la documentación oficial citada arriba. Las conclusiones del informe se apoyan en las capturas y no solo en lo sugerido por la IA.

## Anexo: índice de capturas

| N.º | Contenido |
| --- | --- |
| 1 | `ip a`, `cat /etc/os-release` y comprobación de que `systemctl` no existe |
| 2 | `rc-service sshd status` y `rc-update show` |
| 3 | Edición de `PermitRootLogin yes` en `/etc/ssh/sshd_config` |
| 4 | `vi /etc/ssh/sshd_config` y `rc-service sshd restart` |
| 5 | PuTTY: `Access denied`, segundo intento correcto, `whoami` devuelve root |
| 6 | `apk info`, `netstat -lnt` y `ls /etc` filtrando FTP |
| 7 | Creación del usuario `adminlab` con `adduser` e `id adminlab` |
| 8 | Error de `apk add vsftpd`, corrección del repositorio community, `apk update` e instalación correcta |
| 9 | Edición de `/etc/vsftpd/vsftpd.conf` (`anonymous_enable`, `local_enable`, `write_enable`) |
| 10 | `netstat -lnt` con los puertos 21 y 22 en escucha |
| 11 | `Test-NetConnection` al puerto 21 desde el cliente Windows (`TcpTestSucceeded` True) |
