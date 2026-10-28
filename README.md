# Reporte de Incidente - Cuadrilla 1

# SIMULACIÓN DE INCIDENTE — MESA DE SERVICIO DE TI

## 1. Integrantes y Responsabilidades

- **Coordinador del incidente:** Luis Fernando Callejas Jaime
- **Operador técnico:** Luis Angel Caceres
- **Analista e investigador:** Brandon Gonzales
- **Responsable de evidencias y documentación:** Rudi Choque
- **Responsable de continuidad y relevo:** Adam Fabricio Sanchez

## 2. Ticket del Incidente

- **Código:** INC-04
- **Descripción:** Acceso SSH denegado para root.
- **Categoría:** Incidente de acceso remoto / administración Linux.
- **Prioridad:** Alta.
- **Estado inicial:** Abierto.
- **Solicitante:** Desarrollador de sistemas.
- **Equipo asignado:** Soporte técnico de TI.
- **Servidor:** Linux en Proxmox.
- **Dirección IP:** 192.168.50.107.
- **Servicio relacionado:** Servidor Linux para implementación de FTP.

### Descripción de la incidencia

El solicitante informa que puede iniciar sesión como root desde la consola de la máquina virtual en Proxmox. Sin embargo, cuando intenta conectarse desde su laptop mediante PuTTY a la dirección 192.168.50.107, la conexión llega al servidor, pero la autenticación es rechazada.

Se requiere identificar la causa, recuperar el acceso administrativo remoto de forma segura y documentar las pruebas realizadas.

## 3. Estado Inicial y Análisis

### Estado inicial

- **Acceso local por consola:** funciona, según lo informado por el solicitante.
- **Acceso remoto mediante PuTTY:** la autenticación de root es rechazada.
- **Dirección IP indicada:** 192.168.50.107.
- **Estado del servicio SSH:** pendiente de verificación.
- **Configuración de autenticación SSH:** pendiente de revisión.
- **Registros del servidor:** pendientes de análisis.
- **Causa raíz:** no confirmada.

### Análisis inicial

El funcionamiento del acceso local y el rechazo del acceso SSH pueden deberse a diferencias entre las políticas de autenticación local y remota.

Se investigarán las siguientes posibilidades:

1. La configuración SSH podría impedir el inicio de sesión remoto de root.
2. La autenticación mediante contraseña podría estar deshabilitada.
3. Podrían existir restricciones adicionales de acceso o problemas de autenticación.
4. Podría haber diferencias entre la contraseña utilizada y el método de autenticación configurado.

Estas posibilidades son hipótesis iniciales y no deben considerarse causas confirmadas hasta completar las pruebas.

## 4. Hipótesis y Pruebas Realizadas

### Hipótesis de trabajo

| ID | Hipótesis | Comprobación necesaria |
|---|---|---|
| H-01 | SSH podría prohibir el acceso remoto de root. | Revisar el valor efectivo de `PermitRootLogin`. |
| H-02 | La autenticación mediante contraseña podría estar deshabilitada. | Revisar `PasswordAuthentication`. |
| H-03 | El servicio SSH podría presentar algún problema. | Comprobar el estado del servicio y sus registros. |
| H-04 | Podría existir otra restricción de autenticación. | Revisar los mensajes de error y las políticas aplicables. |

### Pruebas que realizará el operador técnico

**PR-01. Verificación de la dirección IP**

Comprobar que el servidor tenga asignada la dirección 192.168.50.107.

Comando:

`ip addr`

**PR-02. Verificación del servicio SSH**

En Alpine Linux:

`rc-service sshd status`

Registrar el resultado obtenido y comprobar si el servicio está activo.

**PR-03. Revisión de la configuración efectiva**

Comando:

`sshd -T | grep -E 'permitrootlogin|passwordauthentication'`

Registrar los valores que devuelve el servidor.

**PR-04. Revisión de los registros**

Consultar los registros disponibles para identificar el motivo del rechazo de autenticación.

En Alpine Linux puede utilizarse:

`logread | grep -i sshd`

**PR-05. Validación después de la corrección**

Realizar una nueva conexión mediante PuTTY y documentar si el acceso funciona con la cuenta autorizada.

### Resultados de las pruebas

Las pruebas anteriores quedan pendientes de ejecución y documentación. Se completará esta sección con los comandos realmente ejecutados, los resultados observados y las conclusiones del analista.

No se confirmará una causa raíz hasta contar con resultados suficientes.

## 5. Evidencias

**Responsable de evidencias y documentación:** Rudi Choque.

Se recopilarán las capturas de pantalla y los registros necesarios para demostrar el estado inicial, las pruebas, los cambios aplicados y la validación final.

| ID | Evidencia | Estado |
|---|---|---|
| EVID-01 | Captura del rechazo de autenticación en PuTTY. | Pendiente |
| EVID-02 | Captura del acceso local a través de la consola de Proxmox. | Pendiente |
| EVID-03 | Captura de la dirección IP del servidor mediante `ip addr`. | Pendiente |
| EVID-04 | Captura del estado del servicio SSH. | Pendiente |
| EVID-05 | Captura de la configuración efectiva de autenticación. | Pendiente |
| EVID-06 | Captura de la revisión de los registros del servidor. | Pendiente |
| EVID-07 | Captura de la conexión después de aplicar la solución. | Pendiente |
| EVID-08 | Evidencia de la verificación de la cuenta administrativa autorizada. | Pendiente |

### Procedimiento para registrar las evidencias

1. Recibir las capturas del operador técnico.
2. Comprobar que cada captura corresponda a una prueba realizada.
3. Asignar el identificador EVID correspondiente.
4. Subir los archivos al repositorio de GitHub, dentro de la carpeta `INC-04/evidencias/`.
5. Actualizar esta tabla y registrar el resultado de cada prueba.

No se deben publicar contraseñas, claves privadas ni otros datos confidenciales.

## 6. Bitácora de Trabajo y Conclusiones

### Bitácora de trabajo

| Fecha y hora | Responsable | Actividad | Resultado |
|---|---|---|---|
| [Completar fecha y hora] | Rudi Choque | Preparación del registro documental del incidente INC-04. | Registrar una vez guardados los cambios en GitHub. |
| [Completar fecha y hora] | Luis Angel Caceres | Ejecución de las comprobaciones técnicas de SSH. | Pendiente de documentar. |
| [Completar fecha y hora] | Brandon Gonzales | Investigación y análisis de las posibles causas. | Pendiente de documentar. |
| [Completar fecha y hora] | Luis Fernando Callejas Jaime | Coordinación del incidente y seguimiento de las tareas. | Pendiente de documentar. |
| [Completar fecha y hora] | Adam Fabricio Sanchez | Preparación de la continuidad y transferencia del incidente. | Pendiente de documentar. |

### Acciones correctivas previstas

1. Identificar la causa del rechazo mediante la configuración y los registros.
2. Aplicar una corrección acorde con la causa identificada.
3. Evaluar la creación o utilización de una cuenta nominativa para la administración remota.
4. Evitar mantener habilitado el acceso directo de root mediante contraseña como configuración permanente.
5. Verificar el acceso SSH después de aplicar la solución.
6. Registrar los cambios efectuados y las evidencias correspondientes.

### Conclusión provisional

El incidente INC-04 se encuentra en fase de investigación. El acceso local por consola funciona según el solicitante, pero el acceso remoto de root es rechazado.

Todavía no existe información suficiente para establecer una causa raíz definitiva. La resolución dependerá de las pruebas técnicas, de los cambios que correspondan y de la comprobación final de la conexión SSH.

## 7. Estado Final y Continuidad (Relevo)

**Responsable:** Adam Fabricio Sanchez.

### Estado actual del incidente

- **Estado:** Abierto; pendiente de diagnóstico y validación.
- **Causa raíz:** pendiente de confirmar.
- **Solución aplicada:** pendiente de documentar.
- **Prueba final de acceso SSH:** pendiente.
- **Evidencias completas:** pendiente.

### Criterios para cerrar el incidente

El incidente podrá considerarse resuelto cuando:

1. Se haya identificado y documentado la causa del rechazo SSH.
2. Se haya aplicado una solución segura y documentada.
3. Se compruebe que la cuenta administrativa autorizada puede acceder remotamente.
4. Se verifique que la configuración final de SSH cumple con las medidas de seguridad previstas.
5. Se hayan incorporado las evidencias y actualizado la bitácora de trabajo.

### Transferencia y continuidad

Si el incidente continúa abierto, se deberá informar al siguiente responsable sobre las pruebas realizadas, los resultados obtenidos, las acciones pendientes y las evidencias disponibles.

El repositorio de GitHub deberá permanecer actualizado para que los integrantes puedan continuar la investigación sin repetir innecesariamente las pruebas.

**Conclusión final:** pendiente de completar después de confirmar la causa raíz, validar la solución y registrar los resultados finales.
