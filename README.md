![](https://raw.githubusercontent.com/gabrielfernando01/cluster_vb/main/image/cover.png)

# Cluster Master (host) - worker (virtual machine).

Recursos:

- Host (192.168.0.102):
  - Hardware: 12 GB RAM, 100 GB SSD.
  - CPU: AMD Athlon Silver 3050U (1 núcleos) @ 2.3 GHz.
  - SO: Kubuntu 24.04.2, Kernel 6.8.0-52-generic.
  - Software:
  	- Java: OpenJDK 17.0.15 ($JAVA_HOME: /usr/lib/jvm/java-17-openjdk-amd64).
    - Spark: 4.0 (/opt/spark).
    - Maven: 3.8.7 (/usr/share/maven).
    - sbt: 1.10.7.
    - IDE: IntelliJ IDEA 24.1.

- Virtual Machine (192.168.0.103):
  - Hardware: 4 GB RAM, 30 GB almacenamiento.
  - CPU: AMD Athlon Silver 3050U (1 núcleos) @ 2.3 GHz.
  - SO: Debian GNU/Linux 12, Kernel 6.1.0-37-amd64.
  - Software:
	  + Java: OpenJDK 17.0.15 ($JAVA_HOME: /usr/lib/jvm/java-17-openjdk-amd64).
	  + Spark: 4.0 (/opt/spark).

**Nota**: Como la VM solo será un nodo slave (worker), solo necesita Java y Spark. No es necesario instalar Scala, Maven ni sbt en la VM, ya que estos se usan principalmente para desarrollo, no para ejecutar workers.

***

### 🔥 Revisar recursos del sistema:

**Configuración de la red local**

Muestra la IPs locales(privadas) con:

bash
```
hostname -I
```

ip_local_host: xxx.xxx.x.xxx

ip_local_worker: xxx.xxx.x.xxx

1. Conectividad básica

Desde la terminal en master(host) hacemos ping al worker(vm). 

bash
```
$ ping <ip_local_worker>
```

Desde la terminal en worker(vm) hacemos ping al master(host).

bash
```
$ ping <ip_machine_worker>
```

2. Comprobar el estado del servicio SSH en el host. Verifica si el servicio SSH está activo en el host:

bash
```
$ sudo systemctl status sshd
```

Si no está activo, inícialo:

bash
```
$ sudo systemctl start sshd
$ sudo systemctl enable sshd
```

3. Verificar si OpenSSH está instalado

bash
```
$ dpkg -l | grep openssh-server
```

Si no está instalado, instálalo ejecutando:

bash
```
$ sudo apt update
$ sudo apt install openssh-server
```

Volvemos a comprobar el estado del servicio ssh

bash
```
$ sudo systemctl status ssh
```

4. Configurar el firewall en el host.

Asegúrate de que el puerto 22 (SSH) y los puertos utilizados por Spark (como 7077 y otros dinámicos) estén abiertos en el firewall del host.

UFW (firewall predeterminado en Ubuntu/Kubuntu):

bash
```
sudo ufw allow 22/tcp
sudo ufw allow 7077/tcp
sudo ufw allow 4040-4050/tcp  # Puertos dinámicos típicos de Spark
sudo ufw reload
```

5. Configurar Spark para evitar problemas de rsync

Modificar el fichero del host(master) y del slave(wo) <code>/opt/spark/conf/spark-env.sh</code>

bash
```
cd /opt/spark/conf/
cp spark-env.sh.templeate cp spark-env.sh
```

En el fichero <code>spark.env.sh</code> al final escribimos:

texto
```
export SPARK_WORKER_INSTANCES=1
export SPARK_MASTER_IP=<ip_machine_host>
export SPARK_MASTER_PORT=7077			# Puerto por defecto
export SPARK_MASTER_WEBUI_PORT=8080
export SPARK_WORKER_CORES=<#_cores>  	# Ajusta según tus recursos
export SPARK_WORKER_MEMORY=<#_memory>	# Ajusta según tus recursos
```

Modificar el fichero del slave(vm): /opt/spark/conf/spark-env.sh

texto
```
export SPARK_MASTER_HOST=<ip_machine_host>
export SPARK_MASTER_PORT=7077
# Directorio donde se guardarán logs y trabajos
export SPARK_LOG_DIR=/opt/spark/logs
export SPARK_WORKER_DIR=/opt/spark/work/
export SPARK_WORKER_CORES=<#_cores>
export SPARK_WORKER_MEMORY=<memory_worker>
# Directorio temporal de Spark
export SPARK_LOCAL_DIRS=/opt/spark/tmp
```

**⚙️ Configuración de la Virtual Machine**

**Paso 1: Configurar la red entre «Host» y «VM»**.

Para que el master (host) y el slave (VM) se comuniquen, usa una red **Bridge Adapter en VirtualBox**:

1. En VirtualBox, selecciona la VM (Debian) > Configuración > Red > Adaptador 1.
2. Configura:
- Habilitar adaptador de red,
- Tipo: **Bridge Adapter**.
- Selecciona la interfaz de red del host (por ejemplo, <code>wlan0</code> para Wi-Fi o <code>eth0</code> para Ethernet).
3. Inicial la VM y verifica la IP local.

![](https://raw.githubusercontent.com/gabrielfernando01/cluster_vb/main/image/bridge_adapter.png)

***
**Paso 2: Instalar software en la VM (Debian 12)**.

La VM solo necesita Java y Spark para actuar como nodo slave.

1. **Instalar Java (OpenJDK 17)**:

bash
```
sudo apt update
sudo apt install openjdk-17-jdk
```

Verifica

bash
```
java -verison
```

Configura <code>$JAVA_HOME</code> en <code>~/.bahsrc</code>:

bash
```
echo 'export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64' >> ~/.bashrc
source ~/.bashrc
``` 

2. **Instalar Spark 4.0**: Descarga la misma versión que en el host para evitar incompatibilidades:

bash
```
wget https://dlcdn.apache.org/spark/spark-4.0.0/spark-4.0.0-bin-hadoop3.tgz
tar -xzf spark-4.0.0-bin-hadoop3.tgz
sudo mv spark-4.0.0-bin-hadoop3 /opt/spark
```

Configura las variables de entorno en <code>~/.bashrc</code>:

bash
```
echo 'export SPARK_HOME=/opt/spark' >> ~/.bashrc
echo 'export PATH=$PATH:$SPARK_HOME/bin:$SPARK_HOME/sbin' >> ~/.bashrc
source ~/.bashrc
```

3. Inicia el master:

bash
```
start-master.sh
```

3. Vefica que el master esté activo:

- Abre un navegador en el host y accede a <code>http://<IP_local_host>:8080</code>
- Deberías ver la interfaz web de Spark con el master activo (sin workers aún).

4. (Opcional) Configura el host como worker. Si quieres que el host también ejecute tareas, inicia un worker:

bash
```
$SPARK_HOME/sbin/start-worker.sh spark://<ip_local_host>:7077
```

***
**Paso 4: Configurar el nodo Slave (VM - Debian)**.

1. Edita <code>$SPARK_HOME/conf/spark-env.sh</code> en la VM:

bash
```
cp $SPARK_HOME/conf/spark-env.sh.template $SPARK_HOME/conf/spark-env.sh
nano $SPARK_HOME/conf/spark-env.sh
```

Agrega:

```
export SPARK_WORKER_MEMORY=3g
export SPARK_WORKER_CORES=1
```

2. Inicia el worker y conéctalo al master:

bash
```
$SPARK_HOME/sbin/start-worker.sh spark://<ip_local_host>:7077
```