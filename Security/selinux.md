
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
sudo reboot
```

## ✅ 4️⃣ Verificar estado después del reinicio

```bash
sestatus
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


