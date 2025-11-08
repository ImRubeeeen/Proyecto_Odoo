# 1. Correos y puertos:

- tecnofix@odooserra.work.gd
PerenxisaTecnо2025

- patocomponentes@odooserra.work.gd

- apptorrent@odooserra.work.gd

- Entrante: imap.qboxmail.com
IMAP: 993
Seguridad SSL/TLS

- Saliente: smtp.qboxmail.com
SMTP: 465
Seguridad SSL/TLS

- Webmail: webmail.qboxmail.com

- Gerente: manuel@theinkgarage.com

# 2. Subir a Git:

- tar -cvzf nombre.tar volumesOdoo/ (en nuestro caso hacemos un tar para subirlo a github)
- tar cvf micomprimido.tar.xz -I 'xz -9' directorio que queremos comprimir (es para comprimirlo aún más)
- git init (en el directorio donde se encuentre lo que quieres subir, esto hace que puedas empezar a usar el git)
- git branch -m main
- git branch --show-current (comprobación para ver en que rama te encuentras, en este caso, lo uso para poder ver que se ha modificado correctamente el nombre)
- git config --global --add safe.directory directorio (hay veces que te lo va a pedir por seguridad, es simplemente para añadir el directorio que le digas como fichero seguro)
- git add ruta (prepara el estado de subida) / git add . (sube todo lo que tengas en el directorio en el que estés)
- git status (comprobar lo que se ha subido)
- git commit -m "Comentario" (si no has configurado tu usuario y correo de github, te pedirá que lo hagas, pones los 2 comandos que te salen y ya)
- git remote add origin nuestro repositorio que queremos usar
- git remote -v (comprueba que se vea el repositorio remoto)
- git push -u origin main (indicas que a partir de ahora la rama donde vas a subir las cosas es el main y también lo subes / para poder subirlo, pedirá por seguridad el usuario y en el apartado de la contraseña hay que poner el token que hayas generado desde github)

Si tenemos más de 1 rama, tenemos que hacer lo siguiente:

- git add ruta
- git commit -m "Comentario"
  - git remote add origin nuestro repositorio que queremos usar
- git remote -v
- git fetch --all
- git checkout la rama donde lo tengas
- git push -u origin main

En el caso de que no quereamos reemplazar nada y subir más ficheros/directorios, hay que hacer lo siguiente:

- git pull --rebase origin main (sin añadir cambios antes, es decir, NO HACER ningun "git add")
- git add directorio/fichero (ahora si que añades lo que quieras subir)
- git status
- git commit -m "Añadido el docker compose"
- git push origin main

Si queremos descargar lo que tenemos subido:

- git clone https://github.com/ImRubeeeen/Proyecto_Odoo.git (al ser un perfil y repositorio público no pide ningún tipo de credencial)
- tar xvf micomprimido.tar (para descomprimirlo)

# 3. Credenciales base de datos Odoo

- Database Name: odoo_db
- email: rubeninform123@gmail.com
- Password: Hola123_
