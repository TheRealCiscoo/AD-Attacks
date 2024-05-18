# AS-REQ Roast

# ¿Cómo surge un AS-REQ ROAST?

---

Este ataque ocurre con la captura un paquete de Kerberos del tipo `KRB_AS_REQ` en el primer intercambio entre el cliente y el Domain Controller. En una petición `KRB_AS_REQ` se encuentra el **nombre de usuario** que realiza la petición, el **nombre de dominio** y un **timestamp cifrado** con la contraseña del usuario. Con estos datos podemos estructurar un hash que luego podemos intentar crackear para adquirir la contraseña de ese usuario.

El hash se sigue la siguiente estructura:

```bash
$krb5pa$18$NOMBRE_USUARIO$DOMINIO$TIMESTAMP_CIFRADO
```

Antes de capturar los paquetes con fines educativos primero vamos a eliminar los TGT que se han pedido antes desde el cliente para forzar que se realice la petición `KRB_AS_REQ`, para ellos vamos a abrir una powershell en el equipo cliente y con el comando ***`klist`*** listamos para ver los TGTs que tiene ese cliente.

![Untitled](images/Untitled.png)

Y con ***`klist purge`*** limpiamos dichos TGTs.

![Untitled](images/Untitled%201.png)

Para capturar el paquete utilizaremos wireshark y aplicaremos el filtro ***`kerberos.as_req_element`***.

![Untitled](images/e27d661e-6db8-4b74-b61b-52029404f509.png)

Luego desde el cliente accederemos a cualquier recurso SMB existente, tal como lo haría un usuario normal, con el fin de que se produzca la autenticación de Kerberos.

![Untitled](images/Untitled%202.png)

Si miramos el wireshark veremos algunos paquetes del tipo `AS-REQ`.

![Untitled](images/Untitled%203.png)

Buscaremos el paquete que contenga los valores cipher, CNameString y realm.

![Untitled](images/Untitled%204.png)

Con estos datos estructura estructuramos el hash como antes habíamos explicado.

```bash
$krb5pas$18$EAnderson$CISCO$3cd6b6e9b10de664f942f8e06bbf4ac171ea1d054642cf3bb3d814b2314c818c6d0a023524605a6320faf8ce8a2948e67f4de1519cb490d6
```

# Crackeo de los hashes encontrados

---

Para crackear este hash con Hashcat solo introducimos el hash en un archivo de texto y hacemos uso del siguiente comando indicando que el módulo que usaremos será el 19900, el tipo de ataque será de fuerza bruta y, por último, el archivo donde se encuentra el hash y el diccionario que usaremos para crackear la contraseña.

```bash
hashcat -a 0 -m 19900 ARCHIVO_HASH DICCIONARIO
```