# Conexión de InfluxDB a Glances

En esta configuración, conectaremos InfluxDB a Glances para almacenar métricas de rendimiento en una base de datos de series temporales con diferentes contenedores de Apptainer. Los archivos de definición de contenedores de Glances e InfluxDB se pueden encontrar más adelante en este documento.

En esta ocasión será realizado con la distribución de Ubuntu 22.04 LTS e integraciones de red para así poder realizar la conexión de InfluxDB a Glances

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
El archivo de definición utilizado se puede encontrar en el siguiente enlace: [InfluxDB]()
    
Para el contenedor de Glances, tendrá que tener además el cliente CLI de InfluxDB para poder enviar las métricas a la base de datos. Para ello, se instalará el cliente CLI de InfluxDB en el contenedor de Glances:

```bash
# InfluxDB CLI
wget https://download.influxdata.com/influxdb/releases/influxdb2-client-2.7.5-linux-amd64.tar.gz
tar xvzf ./influxdb2-client-2.7.5-linux-amd64.tar.gz
cp ./influx /usr/local/bin/
```
```bash
# Glances
pip install 'glances[all]'
python3 -m venv ~/virtualenv/glances
. ~/virtualenv/glances/bin/activate
```

El archivo de definición utilizado se puede encontrar en el siguiente enlace: [Glances](glances.def)


# Configuración de Glances

## Método 1: Configuración de Glances a través de un archivo de configuración

En el archivo de /usr/local/share/doc/glances/glances.conf donde se configuró el contenedor de Glances, se buscará por la línea 580: 
    
```plaintext
[influxdb2]
# Configuration for the --export influxdb2 option
# https://influxdb.com/
host=localhost
port=8086
protocol=http
org=nicolargo
bucket=glances
token=EjFUTWe8U-MIseEAkaVIgVnej_TrnbdvEcRkaB1imstW7gapSqy6_6-8XD-yd51V0zUUpDy-kAdVD1purDLuxA= 

```
Donde se configurará el host, puerto, protocolo, organización, cubo y token de acceso de InfluxDB:
- `host`: La dirección IP o el nombre de host de la instancia de InfluxDB. Puede ser localhost si InfluxDB se está ejecutando en el mismo servidor que Glances o 0.0.0.0
- `port`: El puerto en el que InfluxDB está escuchando. Por defecto, InfluxDB escucha en el puerto 8086. Puede ser otro o alguno que esté disponible bajo las condiciones de la red.
- `protocol`: El protocolo de comunicación con InfluxDB. Puede ser http o https.
- `org`: La organización a la que pertenece el token de acceso (que se configuró en Influx via web). Puede ser el nombre de la organización o el ID de la organización.
- `bucket`: El cubo en el que se almacenarán las métricas de Glances. Puede ser el nombre del cubo que ya se haya creado en InfluxDB.
- `token`: Aquí se usará el token de acceso de InfluxDB para la autenticación creado en pasos anteriores.

Una vez que se haya configurado el archivo de configuración de Glances, se reiniciará el contenedor de Glances para que los cambios surtan efecto.

## Método 2: Configuración de Glances a través de InfluxDB CLI y Telegraf

Para configurar Glances para enviar métricas a InfluxDB a través de la CLI de InfluxDB, se ejecutará se deberá hacer uso del plugin incorporado de InfluxDB llamado `Telegraf`. Para ello, se ejecutará el siguiente comando:

```bash
#!/bin/bash
# influxdata-archive_compat.key GPG fingerprint:
#     9D53 9D90 D332 8DC7 D6C8 D3B9 D8FF 8E1F 7DF8 B07E
wget -q https://repos.influxdata.com/influxdata-archive_compat.key
echo '393e8779c89ac8d958f81f942f9ad7fb82a25e133faddaf92e15b16e6ac9ce4c influxdata-archive_compat.key' | sha256sum -c && cat influxdata-archive_compat.key | gpg --dearmor | tee /etc/apt/trusted.gpg.d/influxdata-archive_compat.gpg > /dev/null
│echo 'deb [signed-by=/etc/apt/trusted.gpg.d/influxdata-archive_compat.gpg] https://repos.influxdata.com/debian stable main' | tee /etc/apt/sources.list.d/influxdata.list
apt-get update && apt-get install telegraf

```





