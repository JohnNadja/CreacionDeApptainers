# Conexión de InfluxDB y Telegraf

En esta configuración, conectaremos InfluxDB en su instancia de servidor y cli para almacenar métricas de rendimiento en una base de datos de series temporales con diferentes contenedores de Apptainer. Los archivos de definición de contenedores de InfluxDB se pueden encontrar más adelante en este documento. 




En esta ocasión será realizado con la distribución de Ubuntu 22.04 LTS e integraciones de red para así poder realizar la conexión de InfluxDB y Telegraf.

Telegraf es un agente de servidor que recopila y envía métricas y eventos de bases de datos, sistemas, aplicaciones y servicios. Es un plugin de InfluxDB integrado que se puede usar para enviar métricas a InfluxDB.

Estos son los prrequisitos:
``apt-get -y install curl wget python3 iproute2 net-tools vim python3-pip``


Se creará un contenedor de InfluxDB:
    
```bash
    # influxDB
   curl -LO https://download.influxdata.com/influxdb/releases/influxdb2-2.7.6_linux_amd64.tar.gz
   tar xvzf ./influxdb2-2.7.6_linux_amd64.tar.gz
   cp ./influxdb2-2.7.6/usr/bin/influxd /usr/local/bin/
   mkdir /var/run/influxdb
   chown $USER:$USER /var/run/influxdb
```
El archivo de definición utilizado se puede encontrar en el siguiente enlace: [Archivo de definición de InfluxDB](InfluxDB.def)
    
Para el contenedor de Telegraf, tendrá que tener además el cliente CLI de InfluxDB para poder enviar las métricas a la base de datos. Para ello, se instalará el cliente CLI de InfluxDB en el contenedor de Telegraf:

```bash
# InfluxDB CLI
wget https://download.influxdata.com/influxdb/releases/influxdb2-client-2.7.5-linux-amd64.tar.gz
tar xvzf ./influxdb2-client-2.7.5-linux-amd64.tar.gz
cp ./influx /usr/local/bin/
```
```bash
#instalación telegraf
wget -q https://repos.influxdata.com/influxdata-archive_compat.key
echo '393e8779c89ac8d958f81f942f9ad7fb82a25e133faddaf92e15b16e6ac9ce4c influxdata-archive_compat.key' | sha256sum -c && cat influxdata-archive_compat.key | gpg --dearmor | tee /etc/apt/trusted.gpg.d/influxdata-archive_compat.gpg > /dev/null
echo 'deb [signed-by=/etc/apt/trusted.gpg.d/influxdata-archive_compat.gpg] https://repos.influxdata.com/debian stable main' | tee /etc/apt/sources.list.d/influxdata.list

apt-get update && apt-get install telegraf
```

El archivo de definición utilizado se puede encontrar en el siguiente enlace: [Archivo de definición de Telegraf](influx-telegraf.def)


# Creación de contenedores 
Usando los comandos por individual para la creación de los contenedores de InfluxDB y Telegraf:

```bash
apptainer build -s Influx-server/ def_files/influxDB.def
```
```bash	
apptainer build -s Telegraf/ def_files/influx-telegraf.def
```

Una vez creados ambos contenedores, se deberá iniciar el contenedor de InfluxDB y Telegraf con los siguientes comandos:

```bash
apptainer shell -w Influx-server/
```
```bash
apptainer shell -w Telegraf/
```

## Configuración de InfluxDB y Telegraf

### InfluxDB server

Para configurar InfluxDB, se deberá ejecutar el siguiente comando para iniciar el contenedor de InfluxDB:

```bash
influxd --http-bind-address=:<port>
```

![influxdb-bind](/img/influxdb-bind.png)

Donde <port> será el puerto en el que InfluxDB escuchará las solicitudes HTTP. Por defecto, InfluxDB escucha en el puerto 8086. Si se desea cambiar el puerto, se deberá especificar el puerto deseado en lugar de <port>.

Una vez que InfluxDB esté en funcionamiento, se podrá acceder a la interfaz web de InfluxDB en un navegador web visitando la dirección IP del servidor y el puerto en el que InfluxDB está escuchando. Por ejemplo, si InfluxDB está escuchando en el puerto 8086 en el servidor con la dirección IP

```plaintext
http://<server_ip>:<port>
```

Primeramente se pedirá un usarname y password para admin, se deberá ingresar un nombre de usuario y una contraseña para el usuario administrador de InfluxDB. Una vez que se haya ingresado la información, se deberá hacer clic en el botón "Continue" para continuar con la configuración de InfluxDB.


Una vez dentro, se podrá visualizar lo siguiente para la configuración de Telegraf para la conexión con InfluxDB:
![panelInflux](/img/panelInflux.png)

Se busca generar una conexión como fuente de Carga de datos (Load Data), se esioge la opción de Telegraf y se elige la opción de:
- System
 
![panelInflux-2](/img/panelInflux-2.png)


Aparecerá un panel de configuración de agente de Telegraf, se deberá configurar la conexión con los siguientes parámetros:
![agenteTelegraf](/img/agente-Telegraf.png)

Guardamos la configuración y se deberá ver la siguiente pantalla:
![agenteTelegraf-2](/img/agente-Telegraf-2.png)

Aquí existen 2 elementos cruciales para la configuración de Telegraf:
- El token de acceso
- La configuración de Telegraf mediante el cliente CLI de InfluxDB

El token de acceso es único para esta configuración. Aunque se puede restablecer, se recomienda guardar el token de acceso en un lugar seguro para futuras referencias.




### Telegraf

Para configurar Telegraf, se deberá ejecutar el siguiente comando para iniciar el contenedor de Telegraf:

```bash
export INFLUX_TOKEN=<token>
```

Donde <token> es el token de acceso generado en la configuración de InfluxDB. Este token de acceso se utilizará para autenticar Telegraf con InfluxDB.

Una vez que se haya configurado el token de acceso, se deberá ejecutar el siguiente comando para configurar Telegraf:

```bash
telegraf --config http://<server_ip>:<port>/api/v2/telegrafs/<telegraf_id>
```

Que básicamente es la configuración de Telegraf para la conexión con InfluxDB del comando anterior a esta sección.

Se deberá ver algo similar a la siguiente imagen:
![telegraf-test](/img/telegraf-config.png)

## Prueba de ingesta de datos

InfluxDB provee la posibilidad de visualizar los datos de manera directa en su servicio de interfaz web. Para ello, se deberá ingresar a la interfaz web de InfluxDB y seleccionar la opción de "Data Explorer" en el panel lateral izquierdo.

La forma de consultar datos se divide en 2 partes:
- Query Builder
- Script Editor

En Query Builder, se puede seleccionar la base de datos, la medida y los campos a visualizar a través de botones; mientras que Script Editor, se puede escribir consultas personalizadas en el lenguaje de consulta de InfluxDB, Flux (lenguaje similar a SQL).

Aqui se puede ver un ejemplo de la visualización de datos en InfluxDB con el Script Editor:
![influx-data](/img/influx-data.png)

Otra forma con el Query Builder:
![influx-data-2](/img/influx-data-2.png)

Con esto, se ha configurado con éxito la conexión de InfluxDB y Telegraf para la ingesta de datos en una base de datos de series temporales. Se puede continuar con la configuración de Grafana para visualizar los datos en un panel de control.

#### [Volver al inicio](../README.md)
#### [Configuración Grafana](/BD_influx/Grafana_&_Influx.md)