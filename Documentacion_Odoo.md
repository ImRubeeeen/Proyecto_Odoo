# INSTALACIÓN DEL SERVIDOR DE UBUNTU Y DEL SISTEMA ERP ODOO 18

## 1. Creación e instalación del servidor

1. Descargar la ISO que necesitamos, en mi caso, he descargado el Ubuntu Server 24.04.03.
2. Creamos la máquina virtual en VirtualBox.

    - Mi configuración: (https://imgur.com/a/6dLx97v)

3. Iniciamos la máquina y elegimos el idioma del servidor y del teclado.
4. Elegimos la base del Ubuntu server entre:
    
    -   Ubuntu server (la opción que he elegido)
    -   El minimizado

    (https://imgur.com/a/olP3H1Z)

5. Podemos hacer particiones, elegir todo el disco y encriptarlo. En mi caso, uso todo el disco y no lo encripto porque no lo necesito ahora mismo.

    (https://imgur.com/a/JbAFT2M) y (https://imgur.com/a/kCfFfeg)

6. Por último, elijimos el nombre del servidor, de usuario, etc y esperamos a que se instale para reiniciar.

    (https://imgur.com/a/NFhzCl0) y (https://imgur.com/a/1UHPkBF)

## 2. Instalación del sistema ERP ODOO 18

### 2.1 - Instalación en paquetes

Hay otras 3 opciones para instalar Odoo:

- En línea
- Origen
- Docker (en otro momento lo instalaremos desde aquí)

Empezamos con la instalación en paquetes:

#### 2.1.1 - Instalación del servidor PostgreSQL

```shell
sudo apt install postgresql -y
```

#### 2.1.2 - Instalación Odoo desde repositorio

```shell
wget -q -O - https://nightly.odoo.com/odoo.key | sudo gpg --dearmor -o /usr/share/keyrings/odoo-archive-keyring.gpg
```

Este comando descarga la clave GPG de Odoo, la convierte a formato binario y la instala en "/usr/share/keyrings/" para que el sistema pueda validar los paquetes de Odoo al instalar/actualizar desde su repositorio.

Parametros:

- "wget": herramienta para descargar archivos desde la web.
- "-q": ejecuta en modo silencioso, sin mostrar progreso ni mensajes innecesarios.
- "-O": redirige la salida a la terminal, en lugar de guardar el archivo en disco.
- "gpg": GNU Privacy Guard, usado para manejar claves de cifrado y firmas.
- "--deamor": convierte la clave descargada del formato ASCII-armored (texto) a formato binario que entiende APT (el gestor de paquetes de Ubuntu/Debian).
- "-o": guarda el resultado en el archivo.

```shell
echo 'deb [signed-by=/usr/share/keyrings/odoo-archive-keyring.gpg] https://nightly.odoo.com/18.0/nightly/deb/ ./' | sudo tee /etc/apt/sources.list.d/odoo.list
```

Este comando agrega el repositorio de paquetes de Odoo 18.0 (nightly builds) al sistema, vinculándolo con la clave GPG que se guardó antes, para que luego puedas instalar Odoo con apt install odoo.

Parametros:

- "echo": imprime en la salida estándar el texto entre comillas.
- "deb": indica que es un repositorio binario de paquetes Debian/Ubuntu (los .deb que puedes instalar).
- "[signed-by=/usr/share/keyrings/odoo-archive-keyring.gpg]":
    - Le dice a APT que solo confíe en los paquetes de este repo si están firmados con la clave guardada en ese keyring.
    - Esto mejora la seguridad, porque limita la confianza a esa clave en particular (en lugar de usar todas las claves del sistema).
- "https://nightly.odoo.com/18.0/nightly/deb/": Es la URL del repositorio de Odoo versión 18.0 nightly (se actualiza cada noche con las últimas builds).

```shell
sudo apt-get update && sudo apt-get install odoo
```

### 2.2 - Configuración de Virtualbox para redigir puertos

En mi caso, como uso un virtualizador de máquinas (Virtualbox) en la configuración de red de la máquina, hay un ajuste que es de los "reenvio de puertos".

- Puerto anfitrion: puerto que voy a usar para conectarme desde la máquina real (PC).
- Puerto invitado: es el puerto real que usa el servicio, en este caso Odoo.

Aquí mi configuración: (https://imgur.com/a/xGcoHm0)

### 2.3 - Configuración/Creación de la base de datos

Una vez que hayas hecho todos los pasos anterior, desde la máquina anfitriona podrás acceder al odoo de forma gráfica. Tenemos que crear una base de datos con la infomación que pide al entrar.

(https://imgur.com/a/khI3g6F)

## 3. Instalación del sistema ERP ODOO 18 (En docker)

### 3.1 - Instalación de docker

#### 3.1.1 - Configurar repositorio apt

Antes de instalarlo tenemos que parar el servicio de Odoo:

```shell
# Parar el servicio
sudo systemctl stop odoo

# Desactivar el servicio para que no se active cuando inicias la máquina
sudo systemctl disable odoo
```

Ahora hay que configurar el repositorio "apt" de docker para después poder instalarlo.

```shell
# Añadir la llave GPG (llave pública-privada que se usa para verificar que el archivo con la llave pública fue firmado por l llave privada) oficial de Docker:

sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Añadir los repositorio a los recursos apt:

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

#### 3.1.2 - Instalar los paquetes de docker

Una vez configurado el repositorio apt, tenemos que  instalar los paquetes

```shell
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
#### 3.1.3 - Comprobación de que funciona

Cuando esté instalado, tenemos que comprobar que todo funcione correctamente.

```shell
# Ver el estado del servicio de docker 
sudo systemctl status docker

# Si está apagado, usa el siguiente comando:
sudo systemctl start docker

# Si quieres que el servicio se ejecute cuando encindes la máquina, haz esto:
sudo systemctl enable docker
```

Creamos un contenedor de prueba para ver que va correctamente la función principal de docker:
```shell
sudo docker run hello-world
```

Para no tener que usar todo el rato el "sudo" y el comando de docker, podemos darle los siguientes permisos:

```shell
sudo usermod -aG docker $USER
sudo chown root:docker /var/run/docker.sock
```

## 3.2 - Instalación de Odoo en docker

### 3.2.1 - Contenedor Odoo para desarrollo

Para lanzar Odoo en un contenedor preparado para desarrollo, hay que crear dos contenedores:

Creamos el contenedor de PostgreSQL con:

```shell
docker run -d -v /home/usuario/OdooDesarrollo/dataPG:/var/lib/postgresql/data -e
POSTGRES_USER=odoo -e POSTGRES_PASSWORD=odoo -e POSTGRES_DB=postgres --name db
postgres:15
```

Y ahora creamos otro contenedor, este será el de Odoo:

```shell
docker run -d -v /home/usuario/OdooDesarrollo/volumesOdoo/addons:/mnt/extra-addons -v
/home/usuario/OdooDesarrollo/volumesOdoo/firestore:/var/lib/odoo/filestore -v
/home/usuario/OdooDesarrollo/volumesOdoo/sessions:/var/lib/odoo/sessions -p 8069:8069
--name odoodev --user="root" --link db:db -t odoo:18 --dev=all
```

Parametros que se usan:

- “-v /home/usuario/OdooDesarrollo/addons:/mnt/extra-addons”: la imagen de Odoo 18
    por defecto carga los módulos del directorio del contenedor “/mnt/extra-addons”, por lo cual mapeamos ese directorio a nuestro directorio de la máquina anfitrión.
- “/home/usuario/OdooDesarrollo/addons”, donde desarrollaremos usando un IDE externo.
- “-v /home/usuario/OdooDesarrollo/volumesOdoo/firestore:/var/lib/odoo/filestore
- “-v /home/usuario/OdooDesarrollo/volumesOdoo/sessions:/var/lib/odoo/sessions”:
    como en desarrollo es posible que paremos y montemos muchas veces los contenedores
    Docker, montamos estos volúmenes para tener persistencia de los directorios de Odoo
    “firestore” y la “sessions”. Para ello, mapeamos esos directorios del contenedor a nuestra
    máquina anfitrión dentro del directorio “/home/usuario/OdooDesarrollo/volumesOdoo/”.
- “--dev=all”: le pasa ese parámetro a Odoo para facilitar tareas de desarrollo.

Es recomendado para no tener problemas, darle al directorio de addons todos los permisos con:

```shell
sudo chmod -R 777 /home/usuario/OdooDesarrollo/volumesOdoo/addons
```
## 3.3 - Instalar Odoo con docker-compose

Creamos en el directorio que queremos que se instale el Odoo el fichero "docker-compose.yml". Con esto lo que hacemos es lo mismo que en el apartado anterior, pero en vez de hacerlo comando a comando, lo hacemos con un fichero que se encargará de hacer todo como si pusiesemos los comandos usados anteriormente.

```yml
version: '3.3'

services:
#Definimos el servicio Web, en este caso Odoo
  web:
    #Indicamos que imagen de Docker Hub utilizaremos
    image: odoo:18
    container_name: odoo-web
    #Indicamos que depende de "db", por lo cual debe ser procesada primero "db"
    depends_on:
        - db

    # Port Mapping: indicamos que el puerto 8069 del contenedor se mapeara con el mismo puerto en el anfritrion
    # Permitiendo acceder a Odoo mediante http://localhost:8069
    ports:
      - 8069:8069

    # Mapeamos el directorio de los contenedores (como por ejemplo" /mnt/extra-addons" )
    # en un directorio local (como por ejemplo en un directorio "./volumesOdoo/addons")
    # situado en el lugar donde ejecutemos "Docker compose"
    volumes:
      - ./volumesOdoo/addons:/mnt/extra-addons
      - ./volumesOdoo/odoo-web-data:/var/lib/odoo
    #Indicamos que el contenedor funcionara con usuario root y no con usuario odoo
    user: root
    # Definimos variables de entorno de Odoo
    environment:
      - DB_HOST=db
      - DB_PORT=5432
      - DB_USER=odoo
      - DB_PASSWORD=Hola123_
      - DB_NAME=odoo_db
#Definimos el servicio de la base de datos
  db:
    image: postgres:15
    container_name: odoo-db
    # Definimos variables de entorno de PostgreSQL
    environment:
      - POSTGRES_PASSWORD=odoo
      - POSTGRES_USER=odoo
      - POSTGRES_DB=postgres
    # Mapeamos el directorio del contenedor "var/lib/postgresql/data" en un directorio "./volumesOdoo/dataPostgreSQL"
    # situado en el lugar donde ejecutemos "Docker compose"
    volumes:
      - ./volumesOdoo/dataPostgreSQL:/var/lib/postgresql/data
```

- Para instalar/iniciar el docker-compose, usamos el siguiente comando: docker compose up -d (el -d se utiliza para que se quede en segundo plano)
- Para detener el docker-compose se usa: docker compose down (se recomienda antes de apagar la máquina el docker compose)

# 4. Personalización de plantillas de correo y su automatizaciones en Odoo

## 4.1 - Activar modo desarrollador

Antes de nada, hay que activar el modo desarrollador. Con esto puedes obtener todas las herramientas que no se pueden ver si no está activado. Como por ejemplo:

- Parámetros del sistema
- Alias de correo
- Modelos de datos
- Secuencias de documentos

Ve a `Ajustes → Acerca de Odoo → Activar modo desarrollador`.

## 4.2 - Personalización de las plantillas

Ahora podemos crear la plantilla de correo eléctronico en `Ajustes → Técnico → Correo electrónico → Plantillas de correo electrónico`.

Tenemos varios campos que personalizar aquí:

- Nombre: Aquí pondremos el nombre de nuestra plantilla.
- Aplica a: Destino al que hace referencia la plantilla.
- Asunto: Proporciona un adelanto sobre el contenido del mensaje sin necesidad de abrirlo (típico de un email).
- Contenido: Todo lo que quieres que contenga el correo.

En mi caso he puesto lo siguiente:

- Nombre: [CUSTOM] Correo electrónico automático
- Aplica a: Pedido de venta
- Asunto: Gracias por tu pedido {{object.name}}
- Contenido: 

```
Hola, '¡object.partner.id'!

¡Tu pedido ha sido confirmado!

Gracias por tu compra. Adjuntamos la confirmación de tu pedido 'object.name' para tu revisión.

Recuerda que tienes que pagar 'object.amount.total'€.

Procederemos con el procesamiento y envío de tu pedido. 

Te mantendremos informado sobre el estado de la entrega. 

Saludos cordiales, 'object.company_id.name'.
```

## 4.3 - Automatización del correo automático del pedido de ventas

Ahora que ya hemos creado la plantilla, podemos empezar a hacer la automátización que cuando confirme una venta envie el correo eléctronico.

Necesitamos antes de nada instalar el modulo llamado "Reglas de automatización", cuando lo tengamos, seguimos los siguientes pasos:

`Ajustes → Técnico → Automatización → Reglas de automatización`.

Cuando estamos dentro de las reglas de automatización, para crear uno tendremos que darle a "Nuevo".

[CUSTOM] Envio mail venta automática
Modelo: Pedido de venta
Activador: El estado está establecido como Pedido de venta
Acciones pendientes > Añadir una acción > Tipo (Enviar correo electrónico) > Detalles de la acción (Plantilla de correo electrónico > Enviar correo electrónico como Mensaje)

Creamos una venta, la confirmamos y debería de salir el mensaje automáticamente al confirmarlo.
