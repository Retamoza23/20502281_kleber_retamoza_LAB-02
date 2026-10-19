## Evaluacion del Escenario

### Pregunta 1: ¿Cuáles son los principales riesgos?
* **Compromiso masivo de cuentas:** Al estar las contraseñas almacenadas en texto claro, cualquier atacante que acceda a la tabla puede obtenerlas e intentar suplantar la identidad de los usuarios.
* **Exfiltración de datos sensibles (PII y financieros):** El RFC y la información financiera expuestos en texto claro vulneran la privacidad de los usuarios y violan regulaciones de protección de datos.
* **Ataques de red e intermediario (*Man-in-the-Middle*):** Las conexiones sin la validación adecuada permiten que el tráfico entre clientes y el servidor sea interceptado.
* **Compromiso de la seguridad criptográfica:** Exponer las claves criptográficas en archivos SQL destruye la utilidad del cifrado, permitiendo el descifrado masivo de información.
* **Vulnerabilidad en respaldos:** Los respaldos sin cifrar pueden ser robados, extraídos o consultados fuera del entorno controlado del motor de base de datos.

---

### Pregunta 2: ¿Qué mecanismo utilizarías para cada elemento?
* **Contraseña:** Hashing unidireccional con funciones criptográficas seguras (`pgcrypto` usando `crypt` y `gen_salt('bf')`).
* **RFC e Información financiera:** Cifrado simétrico en reposo (`pgp_sym_encrypt()` / `pgp_sym_decrypt()`).
* **Claves criptográficas:** Gestores de secretos dedicados (como HashiCorp Vault o AWS Secrets Manager) o variables de entorno protegidas (nunca dentro de scripts SQL).
* **Conexiones:** Configuración estricta de transporte cifrado mediante TLS con `sslmode=verify-full`.
* **Respaldos:** Cifrado de respaldos (*Backup Encryption*) antes de almacenarlos en medios locales o servicios en la nube.

---

### Pregunta 3: ¿Cuál de los problemas consideras más crítico y por qué?
La **exposición de las claves criptográficas en archivos SQL** (junto con el almacenamiento de contraseñas en texto claro).
* *Justificación:* Si las claves criptográficas quedan expuestas, todo el sistema de protección colapsa. Un atacante puede utilizarlas para descifrar inmediatamente toda la información financiera y los RFCs protegidos, anulando por completo la efectividad del cifrado.

---

### Pregunta 4: ¿Tener correctamente configurados los usuarios y permisos elimina estos riesgos?
**No, no los elimina.**
* *Justificación:* Los roles y permisos solo regulan el acceso dentro del motor de base de datos, pero no protegen contra:
  1. **Ataques de red:** Un usuario legítimo que se conecte a través de una red insegura sin TLS puede sufrir intercepción de su tráfico.
  2. **Robo de respaldos:** Un archivo de respaldo extraído permite consultar los datos sin pasar por los permisos de la base de datos.
  3. **Filtros en el código:** Los permisos de la base de datos no evitan que una clave filtrada en un repositorio de Git sea explotada externamente.
  4. **Cuentas comprometidas:** Si una cuenta con permisos legítimos es vulnerada, el atacante podrá leer todo lo que esté guardado en texto claro.