1. ¿Qué diferencia existe entre crypt() y pgp_sym_encrypt()?
crypt(): Crea un hash de un solo sentido (es irreversible y sirve para validar contraseñas).

pgp_sym_encrypt(): Realiza un cifrado simétrico (es bidireccional, es decir, el dato se oculta pero está diseñado para poder recuperarse después).

2. ¿Cuál de los dos mecanismos permite recuperar el dato original?
pgp_sym_encrypt() (junto con su función complementaria pgp_sym_decrypt()), ya que al estar cifrado de forma simétrica, se puede revertir con la clave correcta. crypt() no permite recuperar el dato original bajo ninguna circunstancia.

3. ¿Qué ocurre si se utiliza una clave incorrecta?
La base de datos arroja un error de descifrado o no puede procesar la información, ya que la llave equivocada es incapaz de desencriptar los datos protegidos.

4. ¿Qué problema de seguridad representa colocar la clave directamente en una consulta?
Que la clave secreta queda expuesta en texto plano en los historiales de consultas (logs), en los archivos de scripts o a la vista de cualquier persona con acceso al sistema, lo que vulnera por completo la seguridad del cifrado.

