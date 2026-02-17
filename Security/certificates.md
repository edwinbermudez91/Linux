
# Transport Layer Security (TLS)

## 🔐 ¿Qué es TLS?

Transport Layer Security (TLS) es un protocolo criptográfico que garantiza:

- 🔒 Confidencialidad → Los datos viajan cifrados.
- 🛡️ Integridad → No pueden ser modificados sin detección.
- 👤 Autenticación → Verifica la identidad del servidor (y opcionalmente del cliente).


TLS es la evolución de SSL (**Secure Sockets Layer**), hoy obsoleto.

## 🔄 ¿Cómo funciona TLS? (Handshake)

**1️⃣ Client Hello**
- El cliente envía:
- Versión TLS soportada
- Cipher suites
- Número aleatorio

**2️⃣ Server Hello**

El servidor responde:
- Certificado digital
- Cipher seleccionada
- Clave pública

**3️⃣ Verificación**

El cliente:

- Valida el certificado contra una CA
- Genera una clave de sesión

**4️⃣ Comunicación cifrada**

Ambos usan cifrado simétrico para la sesión.

## 🔐 Tipos de cifrado involucrados

| Tipo                   | Uso                          |
| ---------------------- | ---------------------------- |
| Asimétrico (RSA/ECDSA) | Intercambio seguro de claves |
| Simétrico (AES)        | Cifrado rápido de datos      |
| Hash (SHA-256)         | Integridad                   |


## 📊 Versiones modernas recomendadas

| Versión | Estado         |
| ------- | -------------- |
| TLS 1.0 | ❌ Obsoleto     |
| TLS 1.1 | ❌ Obsoleto     |
| TLS 1.2 | ✅ Soportado    |
| TLS 1.3 | 🚀 Recomendado |

## 🏗️ Mejor práctica en producción

- Usar certificados firmados por CA confiable
- Usar TLS 1.2 o 1.3
- Habilitar Perfect Forward Secrecy
- Deshabilitar cipher suites débiles
- Rotación periódica de certificados


## Linux TLS Lab – Nivel SysAdmin

🎯 Objetivo

- Crear CA local.
- Firmar certificado de servidor.
- Levantar servidor HTTPS con Python.
- Validar conexión.
- Analizar desde Linux (puertos, procesos, permisos).



**📂 Estructura del repo**

linux-tls-lab/
│
├── ca/
│   ├── ca.key
│   ├── ca.crt
│
├── server/
│   ├── server.key
│   ├── server.csr
│   ├── server.crt
│
└── README.md


### 🧪 EJERCICIO 1 – Crear tu propia CA

🔐 1️⃣ Generar clave privada de la CA

```bash
mkdir -p ca server
openssl genrsa -out ca/ca.key 4096
```

📄 2️⃣ Crear certificado raíz

```bash
openssl req -x509 -new -nodes \
-key ca/ca.key \
-sha256 -days 365 \
-out ca/ca.crt \
-subj "/CN=Linux-CA"
```

### 🧪 EJERCICIO 2 – Crear certificado del servidor

🔐 1️⃣ Generar clave privada del servidor

```bash
openssl genrsa -out server/server.key 4096
```

📄 2️⃣ Crear CSR

```bash
openssl req -new \
    -key server/server.key \
    -out server/server.csr \
    -subj "/CN=localhost"
```

✍️ 3️⃣ Firmar con tu CA

```bash
openssl x509 -req \
    -in server/server.csr \
    -CA ca/ca.crt \
    -CAkey ca/ca.key \
    -CAcreateserial \
    -out server/server.crt \
    -days 365 -sha256
```

### 🧪 EJERCICIO 3 – Levantar servidor HTTPS con Python

🖥️ 1️⃣ Crear archivo app/server.py

```python
import http.server
import ssl

HOST = "0.0.0.0"
PORT = 8443

httpd = http.server.HTTPServer(
    (HOST, PORT),
    http.server.SimpleHTTPRequestHandler
)

context = ssl.SSLContext(ssl.PROTOCOL_TLS_SERVER)
context.load_cert_chain(
    certfile="../server/server.crt",
    keyfile="../server/server.key"
)

httpd.socket = context.wrap_socket(httpd.socket, server_side=True)

print(f"Servidor HTTPS corriendo en https://{HOST}:{PORT}")
httpd.serve_forever()

```

▶️ 2️⃣ Ejecutar servidor

Desde la carpeta app:

```bash
cd app
python3 server.py
```

Deberias ver

```bash
Servidor HTTPS corriendo en https://0.0.0.0:8443
```


### 🔎 EJERCICIO 4 – Validar conexión con OpenSSL

Desde otra terminal:

```bash
openssl s_client -connect localhost:8443 -CAfile ca/ca.crt
```


### 🧠 EJERCICIO 5 – Análisis Linux

📡 Ver puerto abierto

```bash
ss -tulnp | grep 8443
```

🔍 Ver proceso escuchando

```bash
lsof -i :8443
```

🧾 Ver PID y detalles
```bash
ps aux | grep server.py
```

🔐 Validar permisos correctos

```bash
ls -l server/server.key
```

Permiso recomendado:

```bash
-rw------- (600)
```

Si no:

```bash
chmod 600 server/server.key
```

### 🧪 EJERCICIO 6 – Prueba de error TLS

Intentar conexión sin CA:

```bash
openssl s_client -connect localhost:8443
```

Debería fallar verificación.

### 🧪 EJERCICIO 7 – Forzar versión TLS

```bash
openssl s_client -connect localhost:8443 -tls1_2
```

```bash
openssl s_client -connect localhost:8443 -tls1_3
```


### 🔎 EJERCICIO 8 – Inspeccionar certificado

```bash
openssl x509 -in server/server.crt -text -noout
```

Buscar:

- Issuer (tu CA)
- Subject
- Signature Algorithm
- Key Size
- Validity


**Generar clave + CSR (para que una CA lo firme)**
```
openssl req -newkey rsa:4096 -keyout priv.key -out cert.csr
```
**Verificar certificado**

```
openssl x509 -in server.crt -text -noout
```

**Generar certificado con SAN (Subject Alternative Name)**

```
openssl req -newkey rsa:4096 -nodes \
    -keyout priv.key \
    -out cert.csr \
    -subj "/CN=midominio.com" \
    -addext "subjectAltName=DNS:midominio.com,DNS:www.midominio.com"
```


**Generar clave + certificado autofirmado (para pruebas)**

```bash
openssl req -x509 -newkey rsa:2048 -keyout server.key -out server.crt -days 365 -nodes
```