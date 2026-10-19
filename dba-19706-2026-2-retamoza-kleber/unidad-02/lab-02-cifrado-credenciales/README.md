# Laboratorio 02: Cifrado y Protección de Credenciales

## 1. Introducción
En este laboratorio se abordaron las vulnerabilidades asociadas al almacenamiento inseguro de datos sensibles y credenciales dentro de un Sistema Gestor de Bases de Datos (SGBD). Se analizaron los riesgos de exponer información en texto plano, la interceptación de datos en tránsito a través de la red y el manejo inadecuado de claves criptográficas dentro de scripts de código o repositorios. El objetivo principal fue implementar mecanismos robustos de protección en reposo y en tránsito mediante funciones de hashing, cifrado simétrico y configuraciones seguras de TLS en PostgreSQL.

---

## 2. Protección de Credenciales
Existe una diferencia fundamental entre los mecanismos de manejo de información:
* **Texto claro:** Los datos se almacenan sin ningún tipo de transformación, lo que permite que cualquier persona con acceso de lectura a las tablas o respaldos observe directamente la información.
* **Hash:** Es una transformación matemática unidireccional e irreversible (`pgcrypto` con `crypt` y `gen_salt('bf')`). Se utiliza para autenticar credenciales comparando hashes sin necesidad de conocer o recuperar la contraseña original.
* **Cifrado:** Es una transformación bidireccional mediante algoritmos matemáticos y claves criptográficas (`pgp_sym_encrypt` / `pgp_sym_decrypt`). Permite ocultar los datos pero conservando la capacidad de recuperarlos mediante la clave correcta.

---

## 3. Protección de Datos Sensibles
Se protegieron datos personales identificables (PII) y financieros, como el **RFC de los clientes**. Se aplicó cifrado simétrico en reposo debido a que, a diferencia de las contraseñas, los datos operativos requieren ser recuperados y leídos por usuarios o servicios autorizados de la aplicación sin quedar expuestos en texto claro dentro de los archivos del disco duro.

---

## 4. TLS (Transport Layer Security)
TLS resuelve el problema de la seguridad **en tránsito**, previniendo la intercepción de tráfico (*sniffing*) y los ataques de intermediario (*Man-in-the-Middle*). En PostgreSQL se verificó que la conexión utilizara TLS activo (`pg_stat_ssl`), analizando la versión del protocolo (`TLSv1.3` / `TLSv1.2`) y el algoritmo de cifrado negociado, además de revisar el impacto de configurar parámetros estrictos como `sslmode=verify-full`.

---

## 5. Gestión de Claves
Las claves criptográficas **nunca deben almacenarse junto con los scripts SQL ni subirse a repositorios de código**. Incluirlas en archivos de código provoca la exposición accidental ante desarrolladores, registros de ejecución (*logs*), respaldos o filtraciones. Si la clave se compromete, todo el esquema de cifrado colapsa independientemente de lo robusto que sea el algoritmo.

---

## 6. Resultados
* Implementación exitosa de tablas seguras para credenciales utilizando hashing con *salt* aleatorio (`bf`).
* Cifrado y descifrado correcto de información sensible en tablas de clientes mediante algoritmos PGP simétricos.
* Identificación de malas prácticas en la gestión de secretos e integración de políticas para el ciclo de vida de claves criptográficas.
* Verificación y diagnóstico de conexiones cifradas TLS en la instancia de PostgreSQL.

---

## 7. Problemas Encontrados
* **Manejo de claves en consultas:** Se observó la dificultad técnica y el riesgo que representa ejecutar funciones como `pgp_sym_decrypt()` pasando la clave directamente en la consulta SQL, lo cual evidencia la necesidad de integrar la base de datos con un gestor externo de secretos.
* **Configuración de SSL:** Al verificar la tabla `pg_stat_ssl`, se identificó que las conexiones locales predeterminadas no siempre fuerzan TLS a menos que la directiva de configuración del servidor y cliente lo requieran explícitamente.

---

## 8. Conclusiones
Se comprendió que la seguridad en bases de datos es un esquema por capas (*Defense in Depth*). Los roles y permisos no son suficientes por sí solos. En un entorno productivo aplicaría:
1. Hashing obligatorio para contraseñas usando algoritmos adaptativos (Argon2 / Bcrypt).
2. Uso de gestores de secretos dedicados (AWS Secrets Manager / HashiCorp Vault) para llaves criptográficas.
3. Conexiones obligatorias en tránsito usando TLS con `sslmode=verify-full`.
4. Cifrado de respaldos en reposo antes de su transferencia a almacenamiento externo.

---

## 9. Preguntas de Reflexión

1. **¿Por qué una contraseña no debe almacenarse como texto claro?**  
   Porque si la base de datos sufre una filtración o acceso no autorizado, el atacante obtendría acceso inmediato a las cuentas de los usuarios y podría aplicar ataques de reuso de credenciales (*credential stuffing*) en otros servicios.

2. **¿Por qué un hash no es equivalente a un cifrado?**  
   Un hash es una función unidireccional e irreversible (no se puede deshacer para obtener el texto original), mientras que el cifrado es una transformación bidireccional que requiere una clave para descifrar el dato.

3. **¿Por qué el cifrado de una columna no sustituye al control de acceso?**  
   El cifrado protege los datos en reposo frente a accesos directos al disco o respaldos robados. El control de acceso (roles/permisos) regula qué usuarios dentro del motor tienen permiso de ejecutar operaciones (SELECT, INSERT, DELETE). Ambos se complementan.

4. **¿Por qué TLS no sustituye a los mecanismos de autorización de PostgreSQL?**  
   TLS asegura que la comunicación entre cliente y servidor no sea interceptada en la red, pero no valida qué tablas o registros tiene permitido consultar un usuario una vez autenticado dentro de la base de datos.

5. **¿Qué ocurriría si una organización pierde una clave utilizada para cifrar información?**  
   La información cifrada con esa clave se volvería permanentemente inaccesible de forma irrecuperable (pérdida total de datos), ya que no existe un método matemático para descifrar el contenido sin la clave correcta.

6. **¿Qué ocurriría si una clave criptográfica se publica accidentalmente en GitLab?**  
   La clave debe considerarse inmediatamente comprometida. Un atacante podría rastrearla en el historial y usarla para descifrar toda la información protegida. Se debe revocar, rotar la clave y purgar el historial del repositorio.

7. **¿Por qué un repositorio privado no debe considerarse un almacén de secretos?**  
   Porque un repositorio privado otorga acceso a múltiples colaboradores, servicios de CI/CD y respaldos locales, aumentando la superficie de exposición. No proporciona auditoría ni control de ciclo de vida seguro para secretos.

8. **¿Qué diferencia existe entre proteger información almacenada y proteger información durante su transmisión?**  
   Proteger información almacenada (en reposo) previene el robo de discos, tablas o respaldos. Proteger información en transmisión (en tránsito) evita que intercepten y lean las consultas mientras viajan por la red.

9. **¿Por qué la gestión de claves es tan importante como el algoritmo criptográfico?**  
   Porque la seguridad del sistema depende completamente de la confidencialidad de la clave. Aunque el algoritmo sea invulnerable, si la clave se filtra en código o registros, todo el cifrado queda neutralizado.

10. **¿Qué mecanismo adicional implementarías en una organización para proteger los respaldos de las bases de datos?**  
    Cifrado de respaldos antes de su almacenamiento (*Backup Encryption*), almacenamiento en ubicaciones físicamente separadas con políticas de control de acceso estricto y almacenamiento Inmutable/WORM (*Write Once, Read Many*) para prevenir ataques de *ransomware*.