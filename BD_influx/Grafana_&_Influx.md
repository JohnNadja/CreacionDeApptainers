En este presente se mostrará como instalar Grafana con Apptainer y mostrar los datos de monitoreo obtenidos de InfluxDB.

La versión de Grafana que será usada es la versión Enterprise.

## Paso 1: Prerrequisitos
Tener un archivo de definición de contenedor con el siguiente contenido:

```plaintext
Bootstrap: docker
From: ubuntu:22.04

%post
    apt-get update
    apt-get install -y python3
```

## Paso 2: Instalar Grafana con Apptainer
1. Dentro del mismo archivo de definición de contenedor objetivo, se puede agregar la instalación de Grafana. Por ejemplo, agregue las siguientes líneas al archivo `mycontainerGrafana.def`:

```plaintext
BootStrap: docker
From: ubuntu:22.04

%post
   touch /etc/localtime
   apt-get -y update
   apt-get -y install curl wget python3 iproute2 net-tools vim python3-pip python3-venv apt-transport-https software-properties-common

#Grafana install
   mkdir -p /etc/apt/keyrings/
   wget -q -O - https://apt.grafana.com/gpg.key | gpg --dearmor | tee /etc/apt/keyrings/grafana.gpg > /dev/null
   echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://apt.grafana.com stable main" | tee -a /etc/apt/sources.list.d/grafana.list

   apt-get update
   apt-get install grafana-enterprise -y

%environment
   export LC_ALL=C
   #para idioma en español Mexico, descomentar la siguiente linea
   #export LANGUAGE=es_MX:es

%test

   echo "Finalizando instalaciones. Ejecutando pruebas:"
   echo "Grafana CLI...: "
   (grafana-cli -v) && echo "Exito en la instalación"
   echo "Grafana Server...: "
   (grafana-server -v) && echo "Éxito en la instalación"
```

Una vez creado el archivo de definición, se puede construir el contenedor con el siguiente comando:

```bash
apptainer build -s GrafanaApp/ mycontainerGrafana.def
```
## Paso 3: Configurar Grafana
Antes de iniciar el servicio, se harán unas configuraciones previas en Grafana Enterprise para que pueda conectarse a InfluxDB.
1. Se debe ingresar al contenedor creado con el siguiente comando:

```bash
apptainer shell -w GrafanaApp/
```

2. Dentro del contenedor, se debe configurar su archivo .ini (con algún editor de preferencia) de la siguiente ruta:

```bash
/etc/grafana/grafana.ini
```

Se debe modificar las secciones `[paths]` y `[server]`  para que quede de la siguiente manera (se puede copiar y pegar). Los puertos son de ejemplo, se pueden cambiar según sea necesario además del parámetro `http_addr`:

```bash
[paths]
# Path to where grafana can store temp files, sessions, and the sqlite3 db (if that is used)
data = /var/lib/grafana

homepath = /grafana/data/
```

```bash
[server]
# Protocol (http, https, h2, socket)
protocol = http

# This is the minimum TLS version allowed. By default, this value is empty. Accepted values are: TLS1.2, TLS1.3. If nothing is set TLS1.2 would be taken
;min_tls_version = ""

# The ip address to bind to, empty will bind to all interfaces
http_addr = 0.0.0.0

# The http port  to use
http_port = <puerto>
```
![grafana-ini](/img/grafana-ini.png)


3. Se debe iniciar el servicio de Grafana con el siguiente comando:

```bash
service grafana-server start
```
Se puede verificar que el servicio esté corriendo con el siguiente comando:

```bash
service grafana-server status
```
Y la salida debería ser similar a la siguiente:

```plaintext
* grafana is running
```

## Paso 4: Configurar Grafana para InfluxDB
1. Ingresar a la interfaz web de Grafana con la siguiente URL. La IP es la que se configuró en el archivo `.ini` al igual que el puerto:

```plaintext
http://<IP>:18001
```

2. Ingresar con las credenciales por defecto:

```plaintext
Usuario: admin
Contraseña: admin
```

![grafana-login](/img/grafana-login.png)

Puede haber que se tenga que cambiar la contraseña en la primera vez que se ingrese.

3. Una vez dentro, se debe configurar la conexión con InfluxDB. Para ello, se puede ingresar al panel lateral izquierdo y seleccionar `Configuration` y luego `Data Sources`. 

![grafana-data-source](/img/grafana-data-source.png)
![grafana-data-source-2](/img/grafana-data-source-2.png)
Se debe seleccionar `Add data source` y seleccionar `InfluxDB`.

![grafana-data-source-3](/img/grafana-data-source-3.png)

4. Se debe configurar la conexión con los siguientes parámetros. Los valores de IP y puerto tienen que ser referentes a la dirección de InfluxDB y su puerto donde se localizan los datos:

```plaintext
Name: InfluxDB
Query Language: Flux
HTTP URL: http://<IP>:<Port>
```

![grafana-influxdb](/img/grafana-influxdb.png)

Detalles de autenticación:

```plaintext
User: <usuarioInfluxDB>
Password: <contraseñaInfluxDB>
```

InfluxDB Details:

```plaintext
Organization: <nombreOrganización>
Token: <tokenGeneradoEnInfluxDB>*
Default Bucket: <nombreBucketObjetivo>
Min time interval: 1s
Max series: 10000
```


El token se puede obtener en InfluxDB en la sección de `Data` y luego en `API Tokens`. Dicho token debe ser uno de tipo "custom" al ser generado con los permisos necesarios para que Grafana pueda acceder a los datos (escritura y lectura). Además, este token es único y no se puede recuperar una vez generado, por lo que se recomienda guardarlo en un lugar seguro.

Este token puede "regenerarse", pero las buenas prácticas de seguridad recomiendan que se genere uno nuevo en caso de que se haya comprometido el token actual y solo se recomiende hacerlo cada 6 meses.

5. Una vez configurado, se puede guardar y probar la conexión. Si todo está correcto, se debería ver un mensaje de éxito. Por ejemplo:

```plaintext
Data source is working. 1 bucket found
Next, you can start to visualize data by building a dashboard, or by quering data in the Explore view.
```

![grafana-influxdb-2](/img/grafana-influxdb-2.png)


## Paso 5: Crear un Dashboard
1. Para crear un dashboard, se puede seleccionar el panel lateral izquierdo y seleccionar `Create` y luego `Dashboard`.

![grafana-dashboard](/img/grafana-dashboard.png)

![grafana-dashboard-2](/img/grafana-dashboard-2.png)

2. Se puede seleccionar `Add new panel` y seleccionar el tipo de gráfico que se desea visualizar. Por ejemplo, se puede seleccionar `Graph`.

![gra](/img/gra.png)

Aquí hay un ejemplo de uso de lenguaje Flux para obtener los datos de InfluxDB. Se puede usar los elementos de la interfaz web de consulta hecha en InfluxDB para agilizar el proceso de consulta.



