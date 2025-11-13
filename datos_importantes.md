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

- tar -cvzf nombre.tar volumesOdoo/ (En nuestro caso hacemos un tar para subirlo a github)
- tar cvf micomprimido.tar.xz -I 'xz -9' directorio que queremos comprimir (Es para comprimirlo aún más)
- git config --global init.defaultBranch main (Se usa para que cuando hagas el git init se cree la rama automáticamente con el nombre de "main) 
- git init (En el directorio donde se encuentre lo que quieres subir, esto hace que puedas empezar a usar el git)
- git branch --show-current (Comprobación para ver en que rama te encuentras, en este caso, lo uso para poder ver que se ha modificado correctamente el nombre)
- git config --global --add safe.directory directorio (Hay veces que te lo va a pedir por seguridad, es simplemente para añadir el directorio que le digas como fichero seguro)
- git add ruta (prepara el estado de subida) / git add . (Sube todo lo que tengas en el directorio en el que estés)
- git status (Comprobar lo que se ha subido, es opcional)
- git commit -m "Comentario" (Si no has configurado tu usuario y correo de github, te pedirá que lo hagas, pones los 2 comandos que te salen y ya)
- git remote add origin nuestro repositorio que queremos usar
- git remote -v (Comprueba que se vea el repositorio remoto)
- git push -u origin main (Indicas que a partir de ahora la rama donde vas a subir las cosas es el main y también lo usas para poder subir los cambios que acabas de hacer. Te pedirá por seguridad el usuario y la contraseña. En el apartado de la contraseña hay que poner el token que hayas generado desde github)

Si tenemos más de 1 rama, tenemos que hacer lo siguiente:

- git add ruta
- git commit -m "Comentario"
- git remote add origin nuestro repositorio que queremos usar
- git remote -v
- git fetch --all
- git checkout la rama donde lo tengas
- git push -u origin main

En el caso de que no quereamos reemplazar nada y subir más ficheros/directorios, hay que hacer lo siguiente:

Tienes 2 opciones:

  - git pull --rebase origin main (En el caso que ya tengas commits locales que quieres conservar y el repositorio remoto también cambió desde tu último pull. Esto reordena los commits para mantener un historial limpio y evitar merges innecesarios. Un merge lo que hace es que combina los cambios de dos ramas diferentes (o de un remoto y tu local) en una sola línea de desarrollo)
  - git fetch origin y git reset --hard origin/main (Úsalo cuando NO quieres conservar tus cambios locales y quieres dejar tu carpeta idéntica a lo que hay en GitHub (Se recomienda cuando: Estás empezando desde cero con el git init, quieres descartar errores o limpiar tu entorno local, estás configurando una nueva máquina o entorno, o tu repositorio local se ha “roto” y quieres “bajarlo limpio”. Básicamente fuerza tu copia local a ser idéntica al contenido del repositorio remoto))

Ahora ya puedes hacer lo siguiente:

- git add directorio/s
- git commit -m "comentario"
- git status (opcional)
- git push origin main

Si queremos descargar lo que tenemos subido:

- git clone repositorio que quieres descargar (Si es un repositorio público, no pedirá crendenciales. Se descargará en el directorio donde te encuentres)
- tar xvf micomprimido.tar (Para descomprimirlo)

# 3. Credenciales base de datos Odoo

- Database Name: odoo_db
- email: rubeninform123@gmail.com
- Password: Hola123_
