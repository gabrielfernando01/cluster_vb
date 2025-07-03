![](https://raw.githubusercontent.com/gabrielfernando01/cluster_vb/main/images/cover.png)

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

**Configuración paso a paso**
**Paso 1: Configurar la red Host y VM**

