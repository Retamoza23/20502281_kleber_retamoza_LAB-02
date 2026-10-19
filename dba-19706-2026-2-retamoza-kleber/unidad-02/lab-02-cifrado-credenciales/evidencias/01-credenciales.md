*¿Qué ocurriría si una persona obtiene acceso de lectura a esta tabla?*

Cualquier persona con acceso a esa tabla vería todas las contraseñas reales en texto plano (por ejemplo, 'Password123').
Esto generaría graves consecuencias:

- Compromiso total de cuentas: Cualquier atacante o usuario malintencionado con acceso a la consulta podría suplantar la identidad de los usuarios legítimos en la aplicación.

- Efecto dominó (credential stuffing): Como las personas suelen repetir contraseñas en diferentes sitios y servicios, el atacante podría intentar usar estas mismas credenciales para acceder a los correos u otras cuentas personales de las víctimas.

- Violación de normativas de seguridad: Incumple cualquier estándar básico de desarrollo seguro y leyes de protección de datos al no almacenar las credenciales mediante funciones de hashing seguras (como bcrypt, argon2 o PBKDF2).

---------------------------------------------------------------------------------------------------------------------------

1. ¿Son iguales?
No, los resultados son diferentes cada vez que ejecutas la consulta.

2. Si no son iguales, ¿por qué?
Porque gen_salt() crea un valor aleatorio único (salt) en cada ejecución. Aunque la contraseña sea exactamente la misma ('Password123'), el resultado cambia debido a ese factor aleatorio.

3. ¿Qué función cumple gen_salt()?
Genera una cadena de datos aleatoria (conocida como sal o salt) utilizando un algoritmo específico (en este caso, Blowfish o 'bf'). Esto sirve para evitar que contraseñas iguales tengan el mismo resultado y protege contra ataques informáticos masivos.

4. ¿Qué función cumple crypt()?
Toma la contraseña en texto plano y la mezcla con el salt para generar un hash criptográfico seguro y único.

5. ¿Esto es cifrado reversible?
No. No es un cifrado reversible. Es un proceso de un solo sentido (hashing unidireccional); esto significa que no se puede "des-cifrar" el resultado para recuperar la contraseña original.

---------------------------------------------------------------------------------------------------------------------------

1. ¿Qué resultado obtuviste con la contraseña correcta?
Se obtuvo un valor true (verdadero), lo que indica que la contraseña ingresada coincide correctamente con el hash almacenado en la base de datos.

2. ¿Qué resultado obtuviste con la contraseña incorrecta?
Se obtuvo un valor false (falso), indicando que la contraseña no coincide y el acceso debe ser denegado.

3. ¿Por qué no es necesario recuperar la contraseña original?
Porque el sistema no necesita saber cuál es la contraseña en texto plano para autenticar al usuario. Cuando el usuario intenta iniciar sesión, la aplicación solo vuelve a aplicar la función crypt() a la contraseña escrita y compara el resultado con el hash guardado; si coinciden, se le permite el acceso.

4. ¿Por qué un hash no debe confundirse con cifrado?
El cifrado es bidireccional: Se puede cifrar un dato y, utilizando una clave o contraseña, se puede descifrar para recuperar el texto original.

El hash es unidireccional: Es una transformación matemática irreversible. No se puede "deshacer" ni descifrar el hash para volver a obtener la contraseña original.

