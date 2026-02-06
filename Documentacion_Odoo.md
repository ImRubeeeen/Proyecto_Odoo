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

# 4. Personalización de las plantillas de correos eléctronicos y sus automatizaciones en Odoo

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

- Nombre: Aquí pondremos el nombre que queramos para la plantilla.
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

Para poner los ``object`` en el contenido tienes que usar la ``/`` y dentro hay que usar ``Marcador de posición dinámico``.

## 4.3 - Automatización del correo automático del pedido de ventas

Ahora que ya hemos creado la plantilla, podemos empezar a hacer la automátización que cuando confirme una venta envie el correo eléctronico.

Necesitamos antes de nada instalar el modulo llamado "Reglas de automatización", cuando lo tengamos, seguimos los siguientes pasos:

`Ajustes → Técnico → Automatización → Reglas de automatización`.

Cuando estamos dentro de las reglas de automatización, para crear uno tendremos que darle a "Nuevo".

- Nombre: [CUSTOM] Envio mail venta automática
- Modelo: Pedido de venta
- Activador: El estado está establecido como Pedido de venta
- Acciones pendientes > Añadir una acción > Tipo (Enviar correo electrónico) > Detalles de la acción (Plantilla de correo electrónico > Enviar correo electrónico como Mensaje)

Creamos una venta, la confirmamos y debería de salir el mensaje (abajo) automáticamente al confirmarlo.

## 4.4 - Prácticas de las automatizaciones

### 4.4.1 - Mail automático - Automátización de detección de venta que supere los 20.000€

1. Tener instalado el modulo ``CRM``. Para hacerlo, hacemos lo siguiente: En el menú le damos a ``Aplicaciones`` > ``CRM`` > ``Activar``.
2. Creación de la plantilla del correo
    - Nombre: [CUSTOM] Oportunidad de venta > 20.000€
    - Aplica a: Lead/Oportunidad
    - Asunto: Oportunidad de venta > a 20.000€ en ``{{object.name}}``
    - Cuerpo del correo:

      ```
      Se ha detectado una oportunidad de venta mayor a 20.000€ en la empresa "object.partner_id.name (Cliente → Nombre)":
      - Nombre de la Oportunidad de venta: object.name (Oportunidad)
      - Ingresos esperados: object.expected_revenue€ (Ingresoso esperados)
      ```
3. Creación de la regla:
    - Nombre: [CUSTOM] Nueva oportunidad de venta > 20.000
    - Modelo: Lead/Oportunidad
    - Activador: ``Al guardar``
    - Aplicar a: ``Ingresos esperados > 20.000 ``
    - Acciones pendientes:
        - Tipo: Enviar correo eléctronico
        - Plantilla de correo eléctronico: [CUSTOM] Oportunidad de venta > 20.000€ (plantilla creada anteriormente)
        - Enviar correo eléctronico como: ``Mensaje`` 

### 4.4.2 - Regla Automátizada con scripts (opcional)

Vamos a hacer una regla de borrador de servicio para un pedido de un server y su instalación. Para eso tenemos que hacer lo siguiente:

1. Crear los siguientes 2 productos: `` Servidor HP Enterprise 2000`` y ``Servicio de Instalación y Configuración de Servidores``.
2. Hacer la automatización.
    - Nombre: [CUSTOM] Borrador de servicio para pedido de un server
    - Modelo: Pedido de venta
    - Activador: El estado está establecido como - Pedido de venta
    - Aplicar a:
        - Estado = Pedido de venta
        - Líneas del pedido - no contiene - ``Servicio de Instalación y Configuración de Servidores``
        - Líneas del pedido - no contiene - ``Servidor HP Enterprise 2000``
    - Acciones pendientes: Ejecutar código > Detalles de la acción
    
      ```py
      # 1. Obtener el producto de servicio (Producto B)
      # Se recomienda usar el ID, pero buscaremos por nombre para simplificar
        product_service = env['product.product'].search([('name', '=', 'Servicio de Instalación y Configuración de Servidores')], limit=1)
        
        if product_service:
            # 2. Crear el nuevo Pedido de Venta (Borrador)
            # 'record' es la variable que representa el Pedido de Venta (sale.order) actual confirmado.
            new_so = env['sale.order'].create({
                'partner_id': record.partner_id.id,
                'state': 'draft', # Lo creamos como borrador
            })
        
            # 3. Crear la línea de pedido de servicio en el nuevo PV
            env['sale.order.line'].create({
                'order_id': new_so.id,
                'product_id': product_service.id,
                'name': product_service.display_name,
                'product_uom_qty': 1.0,
                'price_unit': product_service.list_price,
            })
      ```

# 5. Configurar Sitio Web

Antes de empezar con la configuración, debemos instalar la aplicación de ``Sitio Web``. Cuando la instalamos ya podemos proceder con la configuración.

Nos pedirá ajustes básicos, como el logo, elegir plantilla, etc. Al hacer eso, ya se puede modificarla como queramos.

En el caso que queramos crear nuevas páginas, si queremos que se nos abra primero una web antes que otro podemos ponerla como la principal en ``Sitio Web → Configuración → Sitios web``.

## 5.1 - Personalización web

Para acceder a la página y empezar a modificarla, tenemos que ir al menú de arriba a la izquierda de las aplicaciones y darle clic a ``Sitio web``.

Cuando accedemos a ella se puede modificar todo lo que quieras, incluso puedes cambiar el ``HTML`` y ``CSS``.

# 6. Modulos personalizados de Odoo

## 6.1 - Crear los directorios y ficheros necesarios

**RECOMENDACIÓN:** Usar un IDE para la personalización.

Los modulos se hacen en ``/home/$USER/TecnoFix/volumesOdoo/addons`` (ten en cuenta que esta es la ruta donde lo tengo yo, pero depende donde tengas el ``volumesOdoo``).

Una vez que sabemos esto, creamos el primer modelo. Tenemos que crear un directorio dentro de ``addons`` (directorio por cada modulo que quieras hacer).

```bash
/home/$USER/TecnoFix/volumesOdoo/addons/prueba
```

Creamos los ficheros ``__init__.py`` y ``__manifest__.py`` y añadimos las siguientes lineas a ``__manifest__.py``:

```bash
{
    'name': "Prueba",
    'author':"TecnoFix"
}
```

## 6.2 - Conexión al contenedor y generación del modulo automático

Acceso al contenedor:

```bash
docker exec -it odoo-web bash
```

Generación del modulo automático:

```bash
odoo scaffold prueba /mnt/extra-addons
```

## 6.3 - Configuración y personalización del modulo

Personalizar el .py de models dentro de su mismo directorio:

```py
# -*- coding: utf-8 -*-

from odoo import models, fields, api

class prueba(models.Model):
    _name = 'prueba.fruta'
    _description = 'Modelo para la fruta'
    _rec_name = 'nombre'            #Para que busque la nueva variable que sustituirá name

    nombre = fields.Char()         
    tipo = fields.Selection([
            ('tipo_hueso','Hueso'),
            ('tipo_citrico','Citrico')
    ], string = 'Tipo de fruta')    #Campo de selecciones con sus campos
    peso = fields.Integer(string='Peso')    #Peso como numero entero
    peso_total = fields.Integer(string='Peso Total',compute='_compute_peso_total') #Peso_total como un decimal ademas del uso de 'compute' para definir que sera modificada por una api

    @api.depends('peso')
    def _compute_peso_total(self):
        for registro in self:
            registro.peso_total = registro.peso * 10
```

Personalización del .xml de vistas:

```xml
<odoo>
  <data>
    <!-- explicit list view definition -->

    <record model="ir.ui.view" id="fruta_list">
      <field name="name">Lista de frutas</field>
      <field name="model">prueba.fruta</field>
      <field name="arch" type="xml">
        <list>
          <field name="nombre"/>
          <field name="tipo"/>
          <field name="peso"/>
          <field name="peso_total"/>
        </list>
      </field>
    </record>

    <!-- actions opening views on models -->

    <record model="ir.actions.act_window" id="fruta_action_window">
      <field name="name">Lista de frutas</field>
      <field name="res_model">prueba.fruta</field>
      <field name="view_mode">list,form</field>
    </record>

    <!-- Top menu item -->

    <menuitem name="prueba" id="prueba_menu"/>

    <!-- menu categories -->

    <menuitem name="Frutas" id="prueba_fruta" parent="prueba_menu"/>


    <!-- actions -->

    <menuitem name="Ver Frutas" id="prueba_menu_ver_fruta" parent="prueba_fruta"
              action="fruta_action_window"/>

  </data>
</odoo>
```

## 6.4 - Instalación del modulo personalizado en Odoo

Para instalarlo debemos acceder a Odoo desde la página y hacer lo siguiente:

1. Ir a aplicaciones
2. Pulsar ``Actualizar lista de aplicaciones``
3. Luego buscar el modulo que hemos personalizado (borra los filtros que vienen puestos) y busca por el nombre que le establecimos al módulo, en mi caso ``Prueba``.
4. Pulsamos en ``Activar`` y cuando abajo a la derecha no salga ``Cargando`` significa que ya se ha instalado. Se puede ver en la lista de modulos de Odoo que esta arriba a la izquierda.

## 6.5 - Prácticas

### 6.5.1 - TIENDA DE VIDEOJUEGOS - REQUISITOS

1. Modelo
    - Nombre (Char): Título del juego.
    - Consola (Selection): Opciones entre 'PS5', 'Xbox', 'Switch', 'PC'.
    - Precio Base (Float): El precio sin impuestos (ej. 50.0).
    - Unidades (Integer): Cuántos juegos hay en stock.
    - Estado (Selection): 'Nuevo' o 'Segunda Mano'.
    - Descuento (Boolean): Un check para decidir si se aplica rebaja.
2. Campos Calculados
    - Campo 1:
        - Campo: valor_stock (Float).
        - Lógica: Multiplicar precio_base * unidades.
    - Campo 2:
        - Campo: precio_final (Float).
        - Lógica:
            - Si el check de descuento está marcado, el precio final es el
            precio_base menos 10 euros.
            - Si no está marcado, el precio final es igual al precio_base.
            - Ademas he agregado un 21% al precio final

### 6.5.2 - Directorios y archivos para Modulos personalizados

Crearemos el directorio del modelo ``tienda_videojuegos`` en su respectivo directorio que hemos indicado anteriormente.

```bash
/home/$USER/TecnoFix/volumesOdoo/addons/tienda_videojuegos
```

Luego creamos los ficheros `__init__.py` y `__manifest__.py` de los cuales agregaresmos lo siguiente a `__manifest__.py`
```
{
    'name': "Tienda videojuegos",
    'author':"TecnoFix"
}
```

### 6.5.3 - Acceso al contenedor y creación de la plantilla del modulo

Accedemos al contenedor con lo siguiente:

```bash
docker exec -it odoo-web bash
```
Generamos la plantilla del modulo personalizado

```bash
odoo scaffold tienda_videojuegos /mnt/extra-addons
```

#### 6.5.4 Configuración y personalización del modulo

Modificación del código de ``models/models.py``:

```py
# -*- coding: utf-8 -*-

from odoo import models, fields, api


class tienda_videojuegos(models.Model):
    _name = 'tienda_videojuegos.tienda_videojuegos'
    _description = 'tienda_videojuegos.tienda_videojuegos'
    _rec_name = 'nombre'

    nombre = fields.Char(string="Nombre del Juego", required=True)
    consola = fields.Selection([('ps5','PS5'),
                                ('xbox','XBOX'),
                                ('switch','SWITCH'),
                                ('pc','PC')])
    precio_base = fields.Float()
    unidades = fields.Integer()
    estado = fields.Selection([('nuevo','Nuevo'),('usado','Segunda Mano')])
    descuento = fields.Boolean ()
    valor_stock = fields.Float(compute="_stock_total", store=True)
    precio_final = fields.Float(compute="_precio_total", store=True)

    @api.depends('precio_base','unidades')
    def _stock_total(self):
        for record in self:
            record.valor_stock = float(record.precio_base) * int(record.unidades)

    @api.depends('precio_base','descuento')
    def _precio_total(self):
        for record in self:
            if record.descuento:
                record.precio_final = float(record.precio_base) * 1.21 - 10
            else:
                record.precio_final = float(record.precio_base) * 1.21
```

Modificación del código de ``views/views.xml``:

```py
<odoo>
  <data>
    <!-- explicit list view definition -->

    <record model="ir.ui.view" id="tienda_videojuegos_list">
      <field name="name">Tienda_videojuegos list</field>
      <field name="model">tienda_videojuegos.tienda_videojuegos</field>
      <field name="arch" type="xml">
        <list>
          <field name = "nombre"/>
          <field name = "consola"/>
          <field name = "precio_base"/>
          <field name = "unidades"/>
          <field name = "estado"/>
          <field name = "descuento"/>
          <field name = "valor_stock"/>
          <field name = "precio_final"/>
        </list>
      </field>
    </record>

    <!-- actions opening views on models -->

    <record model="ir.actions.act_window" id="tienda_videojuegos_action_window">
      <field name="name">Tienda de videojuegos</field>
      <field name="res_model">tienda_videojuegos.tienda_videojuegos</field>
      <field name="view_mode">list,form</field>
    </record>

        <!-- Top menu item -->
    <menuitem name="Tienda de videojuegos" id="tienda_videojuegos_menu"/>

    <!-- menu categories -->
    <menuitem name="Ver juegos" id="tienda_videojuegos_tienda_videojuegos" parent="tienda_videojuegos_menu"/>

    <!-- actions -->
    <menuitem name="Lista de juegos" id="tienda_videojuegos_listar_videojuegos" parent="tienda_videojuegos_tienda_videojuegos"
              action="tienda_videojuegos_action_window"/>

  </data>
</odoo>
```

Modificación del código de ``__manifest__.py``.

```py
# -*- coding: utf-8 -*-
{
    'name': "Tienda videojuegos",

    'summary': "Short (1 phrase/line) summary of the module's purpose",

    'description': """
Long description of module's purpose
    """,

    'author': "Tecnofix",
    'website': "https://www.yourcompany.com",

    # Categories can be used to filter modules in modules listing
    # Check https://github.com/odoo/odoo/blob/15.0/odoo/addons/base/data/ir_module_category_data.xml
    # for the full list
    'category': 'Uncategorized',
    'version': '0.1',

    # any module necessary for this one to work correctly
    'depends': ['base'],

    # always loaded
    'data': [
        'security/ir.model.access.csv',
        'views/views.xml',
        'views/templates.xml',
    ],
    # only loaded in demonstration mode
    'demo': [
        'demo/demo.xml',
    ],
}
```
> **SOLO** he descomentado en data la línea ``'security/ir.model.access.csv',``.

### 6.5.5 - Activar módulo

Cuando ya hemos modificado el código como queremos para nuestro módulo, vamos a Odoo, actualizamos los paquetes y lo activamos.
