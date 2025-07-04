![](https://raw.githubusercontent.com/gabrielfernando01/cluster_vb/main/image/cover.png)

# Cluster Kubuntu (host) - Debian (virtual machine).

Recursos:

- Host (Kubuntu):
  - Hardware: HP Laptop 15-ef1501la, 16 GB RAM (12 GB para el clúster), 130 GB SSD (100 GB para la VM).
  - CPU: AMD Athlon Silver 3050U (2 núcleos) @ 2.3 GHz.
  - SO: Kubuntu 24.04.2, Kernel 6.8.0-52-generic.
  - Software:
    - Java: OpenJDK 11.0.26 ($JAVA_HOME: /usr/lib/jvm/java-11-openjdk-amd64).
    - Scala: 2.13.8 ($SCALA_HOME: /usr/local/share/scala).
    - Maven: 3.8.7 (/usr/share/maven).
    - sbt: 1.10.7.
    - Spark: 3.5.1 (/opt/spark).
    - IDE: IntelliJ IDEA 24.1.
- VM (Debian):
  - Hardware: 4 GB RAM, 30 GB almacenamiento.
  - SO: Debian GNU/Linux 12, Kernel 6.1.0-37-amd64.
  - Software: Sin Java, Scala, Spark, Maven ni sbt instalados.

**Nota**: Como la VM solo será un nodo slave (worker), solo necesita Java y Spark. No es necesario instalar Scala, Maven ni sbt en la VM, ya que estos se usan principalmente para desarrollo, no para ejecutar workers.

***

**⚙️ Configuración paso a paso**
**Paso 1: Configurar la red entre «Host» y «VM»**.

Para que el master (host) y el slave (VM) se comuniquen, usa una red **Bridge Adapter en VirtualBox**:

1. En VirtualBox, selecciona la VM (Debian) > Configuración > Red > Adaptador 1.
2. Configura:
- Habilitar adaptador de red,
- Tipo: **Bridge Adapter**.
- Selecciona la interfaz de red del host (por ejemplo, <code>wlan0</code> para Wi-Fi o <code>eth0</code> para Ethernet).
3. Inicial la VM y verifica la IP local.

bash
```
hostname -I
```

Anota la IP de la VM.

4. Desde el host, verifica la conectividad:

bash
```
ping <ip_local_vm>
```

5. Desde la VM, haz ping al host.

bash
```
ping <ip_local_host>
```

6. Asegúrate de que los puertos <code>7077</code> (master), <code>8080</code> (web UI del master), y <code>8081</code> (web UI del worker) estén abiertos. En ambas máquinas, si usas un firewall:

bash
```
sudo ufw allow 7077
sudo ufw allow 8080
sudo ufw allow 8081
```

***
**Paso 2: Instalar software en la VM (Debian 12)**.

La VM solo necesita Java y Spark para actuar como nodo slave.

1. **Instalar Java (OpenJDK 11)**:

bash
```
sudo apt update
sudo apt install openjdk-11-jdk
```

Verifica

bash
```
java -verison
```

Configura <code>$JAVA_HOME</code> en <code>~/.bahsrc</code>:

bash
```
echo 'export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64' >> ~/.bashrc
source ~/.bashrc
``` 

2. **Instalar Spark 3.5.1**: Descarga la misma versión que en el host para evitar incompatibilidades:

bash
```
wget https://archive.apache.org/dist/spark/spark-3.5.1/spark-3.5.1-bin-hadoop3.tgz
tar -xzf spark-3.5.1-bin-hadoop3.tgz
sudo mv spark-3.5.1-bin-hadoop3 /opt/spark
```

Configura las variables de entorno en <code>~/.bashrc</code>:

bash
```
echo 'export SPARK_HOME=/opt/spark' >> ~/.bashrc
echo 'export PATH=$PATH:$SPARK_HOME/bin:$SPARK_HOME/sbin' >> ~/.bashrc
source ~/.bashrc
```

***
**Paso 3: Configurar el nodo Master (Host - Kubuntu)**.

1. Edita el archivo de configuración de Spark (<code>$SPARK_HOME/conf/spark-env.sh</code>):

bash
```
cp $SPARK_HOME/conf/spark-env.sh.template $SPARK_HOME/conf/spark-env.sh
nano $SPARK_HOME/conf/spark-env.sh
```

Agrega:

```
export SPARK_MASTER_HOST=<ip_local_host>
export SPARK_MASTER_PORT=7077
export SPARK_MASTER_WEBUI_PORT=8080
export SPARK_WORKER_MEMORY=8g
export SPARK_WORKER_CORES=1
```

2. Inicia el master:

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