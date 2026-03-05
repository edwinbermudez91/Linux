
# Instalar SELINUX

## 🔎 1️⃣ Verificar si SELinux está instalado

```bash
sestatus
```

Si dice command not found, no está instalado.

## 📦 2️⃣ Instalar SELinux en Ubuntu

```bash
sudo apt update
sudo apt install selinux-utils selinux-basics auditd -y
```

## ⚙️ 3️⃣ Activar SELinux

```bash
sudo selinux-activate
```

Reiniciar el sistema:

```bash
sudo systemctl reboot
```

## ✅ 4️⃣ Verificar estado después del reinicio

```bash
sestatus
```

**validar logs**

```bash
sudo audit2why --all |less
```

Revisar eventos SELinux denegados
```bash
sudo ausearch -m AVC,USER_AVC -ts recent
```

📄 Revisar directamente el log de auditoría
```bash
sudo less /var/log/audit/audit.log
```


## 🔒 5️⃣ Cambiar a modo Enforcing

```bash
sudo vim /etc/selinux/config
```

cambiar:

```bash
SELINUX=permissive
```

por:

```bash
SELINUX=enforcing
```

**📌 Modos de SELinux**

| Modo       | Descripción                   |
| ---------- | ----------------------------- |
| Enforcing  | Bloquea accesos no permitidos |
| Permissive | Solo registra logs            |
| Disabled   | Deshabilitado                 |

Cambiar temporalmente sin reiniciar:

```bash
sudo setenforce 0   # permissive
sudo setenforce 1   # enforcing
```

**⚠️ Importante (Arquitectura Ubuntu)**

Ubuntu está diseñado para funcionar con AppArmor, no con SELinux.
Tener ambos puede generar conflictos.

Ver estado de AppArmor:

```bash
sudo aa-status
```

Si vas a usar SELinux en serio, es recomendable deshabilitar AppArmor:

```bash
sudo systemctl disable apparmor
sudo systemctl stop apparmor
```

---
## Diagnóstico y Corrección de Políticas SELinux para SSH

1. Verificar el contexto SELinux del proceso SSH

```bash
ps -eZ |grep sshd_t
```
Este comando muestra todos los procesos activos junto con su contexto de seguridad SELinux.

Parámetros:

- `ps -e` → muestra todos los procesos en ejecución.
- `-Z` → incluye el contexto SELinux del proceso.
- `grep sshd_t` → filtra procesos que pertenecen al dominio SELinux de SSH.

Explicación del contexto:

```bash
usuario:rol:tipo:nivel
```

| Campo   | Descripción         |
| ------- | ------------------- |
| usuario | Usuario SELinux     |
| rol     | Rol asociado        |
| tipo    | Dominio del proceso |
| nivel   | Nivel de seguridad  |

El dominio `sshd_t` indica que el proceso se está ejecutando bajo la política de seguridad definida para SSH.


2. Verificar el contexto SELinux del binario SSH

```bash
ls -lZ /usr/sbin/sshd
```

Este comando muestra el contexto SELinux del ejecutable de SSH.

Ejemplo de salida:

```bash
-rwxr-xr-x root root system_u:object_r:sshd_exec_t:s0 /usr/sbin/sshd
```

| Campo       | Descripción         |
| ----------- | ------------------- |
| system_u    | Usuario SELinux     |
| object_r    | Rol del objeto      |
| sshd_exec_t | Tipo del ejecutable |
| s0          | Nivel de seguridad  |

El tipo `sshd_exec_t` permite que, cuando el binario se ejecute, el proceso cambie al dominio `sshd_t`.

3. Analizar bloqueos de SELinux y generar un módulo de política

```bash
sudo audit2why --all -M mymodule
```

Este comando analiza los registros de auditoría del sistema para identificar bloqueos generados por SELinux y genera un módulo de política personalizado.

Funciones principales:

- Analiza eventos de denegación (AVC)
- Explica por qué ocurrió el bloqueo
- Genera archivos de política SELinux

Archivos generados:

| Archivo       | Descripción                  |
| ------------- | ---------------------------- |
| `mymodule.te` | Código fuente de la política |
| `mymodule.pp` | Módulo compilado de SELinux  |

Ejemplo de regla generada:
```bash
allow sshd_t var_log_t:file read;
```

Esto permite que el proceso SSH lea archivos en un directorio específico que antes estaba bloqueado.

4. Instalar el módulo de política generado

```bash
semodule -i mymodule.pp
```

Este comando instala el módulo generado dentro de la política activa de SELinux.

Después de instalarlo:

- SELinux permitirá las acciones previamente bloqueadas.
- El sistema aplicará la nueva regla sin modificar la política base.

Para verificar los módulos instalados:

```bash
semodule -l
```

5. Activar modo Enforcing en SELinux

```bash
setenforce 1
```

El comando setenforce 1 activa el modo Enforcing, donde las políticas se aplican estrictamente.

Para verificar el estado actual:

```bash
sestatus
```