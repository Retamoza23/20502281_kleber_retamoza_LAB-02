## Verificación Final

### Credenciales
**¿Una contraseña debe cifrarse o protegerse mediante un mecanismo de derivación de claves para contraseñas?**
Debe protegerse mediante un **mecanismo de derivación o hashing de contraseñas** (como `crypt` con `gen_salt('bf')`, Argon2 o PBKDF2). El cifrado es bidireccional (reversible), lo cual no es adecuado para credenciales; el hashing es unidireccional e irreversible.

---

### Datos sensibles
**¿Qué diferencia existe entre enmascaramiento, seudonimización y cifrado?**
* **Enmascaramiento:** Oculta visualmente partes del dato original (ejemplo: `****-****-****-1234`) para ciertos roles de usuario, sin alterar el dato en la base de datos.
* **Seudonimización:** Sustituye los datos identificadores por valores falsos o tokens (ejemplo: reemplazar un RFC por un UUID), permitiendo reidentificar al usuario únicamente con una tabla de mapeo separada.
* **Cifrado:** Transforma matemáticamente la información en reposo utilizando un algoritmo y una clave criptográfica, de modo que sea totalmente ilegible a menos que se posea la clave para descifrarla.

---

### Comunicaciones
**¿Qué problema resuelve TLS?**
Garantiza la protección de los datos **en tránsito** a través de la red. Resuelve los riesgos de intercepción de tráfico (*sniffing*), manipulación de datos en el camino y ataques de intermediario (*Man-in-the-Middle*), además de permitir verificar la identidad del servidor.

---

### Claves
**¿Dónde debería almacenarse una clave criptográfica en un entorno real?**
En un **gestor de secretos dedicado** (como HashiCorp Vault, AWS Secrets Manager o Azure Key Vault) o a través de **variables de entorno protegidas** a nivel de infraestructura, manteniéndolas completamente fuera del código fuente y de la base de datos.

---

### Repositorio
**¿Es correcto subir una contraseña o clave real a GitLab aunque el repositorio sea privado?**
**No.** Un repositorio privado reduce exposición, pero no convierte un secreto en información apropiada para almacenar dentro del código.