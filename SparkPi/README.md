![](https://raw.githubusercontent.com/gabrielfernando01/cluster_vb/main/SparkPi/image/cover.png)

# ☀️ Ejecutar un proyecto <code>.jar</code> en el spark cluster.

### Pasos para ejecutar el ejemplo <code>SparkPi</code> y reflejarlo en la UI del master

1. Verificar el entorno y configuración

+ **Misma version spark, java**

bash
```
spark-shell --version
```

+ **Configurar variables de entorno**

en <code>~/.bashrc</code>

texto:
```
export SPARK_HOME=/ruta/a/tu/spark
export PATH=$SPARK_HOME/bin:$PATH
export JAVA_HOME=/ruta/a/tu/jdk17
export PATH=$JAVA_HOME/bin:$PATH
```

Aplicar cambios.

bash
```
source ~/.bashrc
```

+ **Verificar la conexión**

bash
```
sudo ufw allow 7077
sudo ufw allow 8080
```

En todos los nodos de tener el firewall activo.

2. **Iniciar el clúster Spark**:

+ **En el nodo master**

bash
```
start-master-sh
```

Vericar que esté corriendo:

bash
```
netstat -tuln | grep 7077
```

Debes ver al puerto <code>7077</code> como «LISTEN». Obtén la URL del master desde los logs en <code>$SPARK_HOME/logs</code> o en la UI (<code>http://«tu-ip»:8080</code>). Será algo como <code>spark://«tu-ip»:7077</code>.

+ **Iniciar el worker local en master en maquina virtual y equipo independiente**:

bash
```
start-worker.sh spark://<tu-ip>:7077
```

![](https://raw.githubusercontent.com/gabrielfernando01/cluster_vb/main/SparkPi/image/cluster.png)

## ✨ Ejecutar el ejemplo <code>SparkPi</code>

Para comprobar que se ejecute el ejemplo <code>.jar</code> lo ejecutamos en modo local[*] 

bash
```
$SPARK_HOME/bin/spark-submit \
  --class org.apache.spark.examples.SparkPi \
  --master local[*] \
  $SPARK_HOME/examples/jars/spark-examples_2.13-4.0.0.jar 1000
```

Deberia ver algo como: 

text:
```
Pi is roughly 3.14...
```

**Confirmamos el estado del cluster**

1. Inicializamos master, worker1 (master), worker2 (virtual machine), worker3.

2. Ejecutar <code>SparkPi</code> con conexión al master:

+ Usaremos <code>spark-submit</code> para ejecutar <code>SparkPi</code> conectándo al master:

```
$SPARK_HOME/bin/spark-submit \
  --class org.apache.spark.examples.SparkPi \
  --master spark://192.168.0.103:7077 \
  --conf spark.driver.host=192.168.0.103 \
  --conf spark.driver.bindAddress=0.0.0.0 \
  $SPARK_HOME/examples/jars/spark-examples_2.13-4.0.0.jar 1000
```

Deberia ver algo como: 

text:
```
Pi is roughly 3.14...
```

![](https://raw.githubusercontent.com/gabrielfernando01/cluster_vb/main/SparkPi/image/ui_master.png)