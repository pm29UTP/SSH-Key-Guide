# Guía de llaves de SHH para Windows
Esta es una guía paso a paso para crear llaves de SSH en Windows y copiarlas a Linux, específicamente a Fedora Server.

## En Fedora Server
Si no existe, crea el directorio de SSH en el `$HOME` de tu usuario (por ejemplo, `/home/nombre.apellido`) y modifica los permisos para que solo tu usuario lo pueda acceder:
```sh
mkdir .ssh && chmod 700 .ssh
```
Luego crea el archivo `authorized_keys` dentro del directorio `.ssh` creado en el paso anterior, y dale permisos de lectura y escritura exclusivamente a tu usuario:
```sh
touch .ssh/authorized_keys && chmod 600 .ssh/authorized_keys
```
## En Windows
Abre el símbolo del sistema (CMD) presionando las teclas `Windows + R` para abrir la ventana de diálogo "Ejecutar" y escribe "cmd". Opcionalmente, puedes buscar "cmd" en el buscador de programas de Windows.

Una vez en la ventana de CMD, escribe el comando para generar las llaves de SSH:
```
ssh-keygen -t ed25519 -C "Nombre Apellido"
```
Este comando hace lo siguiente:
- `ssh-keygen`: Invoca la utilidad de llaves de autenticación de OpenSSH[^1].
- `-t ed25519`: Esta opción indica el tipo de llave, en este caso, usando el algoritmo de cifrado Ed25519[^2].
- `-C "Nombre Apellido"`: Con esta opción agregamos un comentario dentro de las comillas para identificar la llave.

Cuando ejecutan el comando tendran una salida parecida a la siguiente:
```
Generating public/private ed25519 key pair.
Enter file in which to save the key (C:\Users\<Usuario>\.ssh\id_ed25519):
Enter passphrase for "C:\Users\<Usuario>\.ssh\id_ed25519" (empty for no passphrase):
Enter same passphrase again: 
Your identification has been saved in C:\Users\<Usuario>\.ssh\id_ed25519.
Your public key has been saved in C:\Users\<Usuario>\.ssh\id_ed25519.pub.
The key fingerprint is:
SHA256:SqtyNulQjRuCHLMoG/YWPZDhoYy9bQg+Zr1+q48XaeI Nombre Apellido
The key's randomart image is:
+--[ED25519 256]--+
|   o             |
|o.o +            |
|o=.+             |
|+o=+oo           |
|=O+o*o+ S        |
|=oo=oB.o         |
|. o++.+          |
|  +E*+           |
|   OB+.          |
+----[SHA256]-----+
```
Ahora si inspeccionan los contenidos de la carpeta oculta `.ssh` van a ver las dos llaves creadas: una pública y una privada.

Como siguiente paso, vamos a copiar la llave pública a nuestro servidor de Fedora:
```
scp .ssh\id_ed25519.pub nombre.apellido@<ip-del-servidor>:~/llave_ssh
```
Con este comando invocamos la utilidad de copia segura de archivos de OpenSSH, llamada `scp`[^3], y le pasamos como argumento la ubicación de la llave pública que queremos copiar hacia el servidor de Fedora. Seguido escribimos el usuario y dirección IP del servidor, y al final añadimos el nombre y la ubicación del archivo copiado dentro del servidor, que en este caso lo llamaremos `llave_ssh` dentro del `$HOME` del usuario.

## En Fedora Server
Regresamos al servidor de Fedora y verificamos que la llave se copió exitosamente, usando el comando `ls`.

Ahora vamos a copiar la llave dentro del archivo `authorized_keys` que creamos al inicio:
```sh
cat llave_ssh >> ~/.ssh/authorized_keys
```
Para estar seguros, mostremos los contenidos de `authorized_keys` usando el comando `cat`, y deberíamos tener algo así:
```
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIKR7nEGZA3DTpfgmA73VcSDyJnaZzLncaqgqSQB1Fu+w Nombre Apellido
```
Esto significa que la llave fue agregada exitosamente a la lista de llaves autorizadas por nuestro servidor.

## En Windows
Abrimos otra ventana de CMD **sin cerrar la anterior** y vamos a iniciar sesión con SSH, como lo haríamos normalmente:
```
ssh nombre.apellido@<ip-del-servidor>
```
Pero esta vez, debería iniciar sesión sin solicitar la contraseña.

Si fue así: ¡Felicidades!, configuraste el inicio de sesión por SSH usando llaves.

## Consideraciones finales
Como ya no es necesario ingresar la contraseña, puedes incrementar la seguridad de tu servidor al deshabilitar el inicio de sesión por contraseñas en SSH. Esto se puede hacer modificando el archivo `/etc/ssh/sshd_config`[^4].

> [!CAUTION]
> Asegúrate que el inicio de sesión sin contraseñas funciona correctamente y mantén siempre una sesión iniciada dentro del servidor en caso de que algo salga mal. De lo contrario te quedarás sin acceso al servidor.

# Referencias
[^1]: https://man.openbsd.org/ssh-keygen.1
[^2]: https://ed25519.cr.yp.to/
[^3]: https://man.openbsd.org/scp.1
[^4]: https://man.openbsd.org/sshd_config#PasswordAuthentication
