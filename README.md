### Escuela Colombiana de Ingeniería
### Arquitecturas de Software - ARSW

## Desarrollo por:

+ José Manuel Gamboa Gómez
+ Zuly Valentina Vargas Ramírez 


## Escalamiento en Azure con Maquinas Virtuales, Sacale Sets y Service Plans

### Dependencias
* Cree una cuenta gratuita dentro de Azure. Para hacerlo puede guiarse de esta [documentación](https://azure.microsoft.com/es-es/free/students/). Al hacerlo usted contará con $100 USD para gastar durante 12 meses.

### Parte 0 - Entendiendo el escenario de calidad

Adjunto a este laboratorio usted podrá encontrar una aplicación totalmente desarrollada que tiene como objetivo calcular el enésimo valor de la secuencia de Fibonnaci.

**Escalabilidad**
Cuando un conjunto de usuarios consulta un enésimo número (superior a 1000000) de la secuencia de Fibonacci de forma concurrente y el sistema se encuentra bajo condiciones normales de operación, todas las peticiones deben ser respondidas y el consumo de CPU del sistema no puede superar el 70%.

### Parte 1 - Escalabilidad vertical

1. Diríjase a el [Portal de Azure](https://portal.azure.com/) y a continuación cree una maquina virtual con las características básicas descritas en la imágen 1 y que corresponden a las siguientes:
    * Resource Group = SCALABILITY_LAB
    * Virtual machine name = VERTICAL-SCALABILITY
    * Image = Ubuntu Server 
    * Size = Standard B1ls
    * Username = scalability_lab
    * SSH publi key = Su llave ssh publica

![Imágen 1](images/part1/part1-vm-basic-config.png)

+ Creación:

![Creacion](images/solution/maquina.png)
![Creacion](images/solution/maquina2.png)

2. Para conectarse a la VM use el siguiente comando, donde las `x` las debe remplazar por la IP de su propia VM (Revise la sección "Connect" de la virtual machine creada para tener una guía más detallada).

    `ssh scalability_lab@xxx.xxx.xxx.xxx`

Desde la ubicación de la llave privada descargada nos conectamos por ssh a la máquina recien creada:

![Conexion](images/solution/conexionSSH.png)
    


3. Instale node, para ello siga la sección *Installing Node.js and npm using NVM* que encontrará en este [enlace](https://linuxize.com/post/how-to-install-node-js-on-ubuntu-18.04/).

![Conexion](images/solution/instalacion1.png)

Se reinicia la conexión para guardar los cambios. 

![Conexion](images/solution/instalacion2.png)

Continuamos con la instalación:

![Conexion](images/solution/instalacion3.png)


4. Para instalar la aplicación adjunta al Laboratorio, suba la carpeta `FibonacciApp` a un repositorio al cual tenga acceso y ejecute estos comandos dentro de la VM:

    `git clone <your_repo>`

![Clonar](images/solution/clone.png)

    `cd <your_repo>/FibonacciApp`

    `npm install`

5. Para ejecutar la aplicación puede usar el comando `npm FibinacciApp.js`, sin embargo una vez pierda la conexión ssh la aplicación dejará de funcionar. Para evitar ese compartamiento usaremos *forever*. Ejecute los siguientes comando dentro de la VM.

    ` node FibonacciApp.js`

![App](images/solution/app.png)

6. Antes de verificar si el endpoint funciona, en Azure vaya a la sección de *Networking* y cree una *Inbound port rule* tal como se muestra en la imágen. Para verificar que la aplicación funciona, use un browser y user el endpoint `http://xxx.xxx.xxx.xxx:3000/fibonacci/6`. La respuesta debe ser `The answer is 8`.

![](images/part1/part1-vm-3000InboudRule.png)

![3000](images/solution/puerto.png)

![3000](images/solution/puerto2.png)

Verificamos ingresando a la dirección correspondiente:

![3000](images/solution/funcionamiento.png)

7. La función que calcula en enésimo número de la secuencia de Fibonacci está muy mal construido y consume bastante CPU para obtener la respuesta. Usando la consola del Browser documente los tiempos de respuesta para dicho endpoint usando los siguintes valores:
    * 1000000
![r1](images/solution/r1.png)

    * 1010000
![r2](images/solution/r2.png)

    * 1020000
![r3](images/solution/r3.png)

    * 1030000
![r4](images/solution/r4.png)

    * 1040000
![r5](images/solution/r5.png)

    * 1050000
![r6](images/solution/r6.png)

    * 1060000
![r7](images/solution/r7.png)

    * 1070000
![r8](images/solution/r8.png)

    * 1080000
![r9](images/solution/r9.png)

    * 1090000  
![r10](images/solution/r10.png)

8. Dírijase ahora a Azure y verifique el consumo de CPU para la VM. (Los resultados pueden tardar 5 minutos en aparecer).

![Imágen 2](images/part1/part1-vm-cpu.png)

Resultados obtenidos:

![metricas](images/solution/metricas.png)


9. Ahora usaremos Postman para simular una carga concurrente a nuestro sistema. Siga estos pasos.
    * Instale newman con el comando `npm install newman -g`. Para conocer más de Newman consulte el siguiente [enlace](https://learning.getpostman.com/docs/postman/collection-runs/command-line-integration-with-newman/).

    Instalamos lo necesario en una nueva máquina:

![postman](images/solution/postman.png)


    * Diríjase hasta la ruta `FibonacciApp/postman` en una maquina diferente a la VM.
    * Para el archivo `[ARSW_LOAD-BALANCING_AZURE].postman_environment.json` cambie el valor del parámetro `VM1` para que coincida con la IP de su VM.


![ip](images/solution/vm1.png)


    * Ejecute el siguiente comando.

    ```
    newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
    newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10
    ```
![run](images/solution/postman-10.png)


10. La cantidad de CPU consumida es bastante grande y un conjunto considerable de peticiones concurrentes pueden hacer fallar nuestro servicio. Para solucionarlo usaremos una estrategia de Escalamiento Vertical. En Azure diríjase a la sección *size* y a continuación seleccione el tamaño `B2ms`.

![Imágen 3](images/part1/part1-vm-resize.png)

Cambiamos el tamaño y reiniciamos la máquina para guardar los cambios:

![run](images/solution/size.png)


11. Una vez el cambio se vea reflejado, repita el paso 7, 8 y 9.

Se espera que con en cambio de tamaño realizado se mejoren los tiempos de respuesta y además se disminuya el consumo de CPU.

* 1000000

![r1-2](images/solution/r1-2.png)

* 1010000

![r2-2](images/solution/r2-2.png)

* 1020000

![r3-2](images/solution/r3-2.png)

* 1030000

![r4-2](images/solution/r4-2.png)

* 1040000

![r5-2](images/solution/r5-2.png)

* 1050000

![r6-2](images/solution/r6-2.png)

* 1060000

![r7-2](images/solution/r7-2.png)

* 1070000

![r8-2](images/solution/r8-2.png)

* 1080000

![r9-2](images/solution/r9-2.png)

* 1090000 

![r10-2](images/solution/r10-2.png)

Rendimienyo CPU
![metricas2](images/solution/metricas2.png)

12. Evalue el escenario de calidad asociado al requerimiento no funcional de escalabilidad y concluya si usando este modelo de escalabilidad logramos cumplirlo.

Se puede observar que el tiempo de respuesta con el tamaño inicial es en promedio de 3 minutos, en cambio, al aumentar su tamaño se obtiene la respuesta en un promedio 
de 20 segundos.

Postman:

![postman](images/solution/postman-10-2.png)

13. Vuelva a dejar la VM en el tamaño inicial para evitar cobros adicionales.


**Preguntas**

1. ¿Cuántos y cuáles recursos crea Azure junto con la VM?


![recursos](images/solution/recursos-i.png)

Se crean 7 recursos junto a la VM.

2. ¿Brevemente describa para qué sirve cada recurso?

    2.1 Hace referencia a la red virtual.

    2.2 Hace referencia a la interfaz de red.

    2.3 Hace referencia a la máquina virtual creada.

    2.4 Hace referencia a la dirección ip pública.

    2.5 Hace referencia a el grupo de seguridad de red.

    2.6 Hace referencia al disco de la máquina virtual.

    2.7 Hace referencia a la clave SSH. 

3. ¿Al cerrar la conexión ssh con la VM, por qué se cae la aplicación que ejecutamos con el comando `npm FibonacciApp.js`? ¿Por qué debemos crear un *Inbound port rule* antes de acceder al servicio?
    3.1 La aplicación se cae ya que para cerrrar la conexión SSH es necesario detener la ejecución de la aplicación.  
    Debemos crear un Inbound port rule debido a que antes de esto el puerto se encuentra cerrado y no permitiría realizar la consulta. 

4. Adjunte tabla de tiempos e interprete por qué la función tarda tando tiempo.

    ![recursos](images/solution/tabla.png)
    La función tarda demasiado debido a que no se esta implementando alguna forma de guardar valores anteriores por lo que para cada consulta debe calcular el valor desde 0 haciendo que se lleve el consumo de la CPU al limite. 

5. Adjunte imágen del consumo de CPU de la VM e interprete por qué la función consume esa cantidad de CPU.

    Antes del cambio:

    ![metricas](images/solution/metricas.png)

    La función consume esta cantidad de CPU ya que no se realiza una busqueda recursiva. 

6. Adjunte la imagen del resumen de la ejecución de Postman. Interprete:
    * Tiempos de ejecución de cada petición.
    * Si hubo fallos documentelos y explique.

![postman](images/solution/postman-10-2.png)

El tiempo de ejecución total de todas las consultas es de 4m 36 s, en promedio cada consulta tarda 29.5s.

Se obtuvieron dos fallos. Estos errores se obtienen debido a que la aplicación se cierra al recibir una sobrecarga de peticiones.

7. ¿Cuál es la diferencia entre los tamaños `B2ms` y `B1ls` (no solo busque especificaciones de infraestructura)?

![comparacion](images/solution/comparacion.png)

El tamaño B1ls solo puede ser montado a un sistema operativo Linux, en cambio B2ms puede ser cargado sobre otros sistemas operativos.

8. ¿Aumentar el tamaño de la VM es una buena solución en este escenario?, ¿Qué pasa con la FibonacciApp cuando cambiamos el tamaño de la VM?

No del todo ya qué permite reducir considerablemente el tiempo de ejecución pero sigue tardándose  bastante. 

9. ¿Qué pasa con la infraestructura cuando cambia el tamaño de la VM? ¿Qué efectos negativos implica?

El cambio de tamaño implica un mayor costo de pago y además tarda bastante en estar nuevamente activa la máquina por lo que el servicio estará no disponible en ese cambio.

10. ¿Hubo mejora en el consumo de CPU o en los tiempos de respuesta? Si/No ¿Por qué?

Si hubo una mejora ya que al principio se tuvo un consumo promedio de 50.1713%, luego del cambio se obtuvo un consumo de 12.9847%.

11. Aumente la cantidad de ejecuciones paralelas del comando de postman a `4`. ¿El comportamiento del sistema es porcentualmente mejor?

Resultados:

![comparacion](images/solution/postman4.png)

### Parte 2 - Escalabilidad horizontal

#### Crear el Balanceador de Carga

Antes de continuar puede eliminar el grupo de recursos anterior para evitar gastos adicionales y realizar la actividad en un grupo de recursos totalmente limpio.

1. El Balanceador de Carga es un recurso fundamental para habilitar la escalabilidad horizontal de nuestro sistema, por eso en este paso cree un balanceador de carga dentro de Azure tal cual como se muestra en la imágen adjunta.

![](images/part2/part2-lb-create.png)

2. A continuación cree un *Backend Pool*, guiese con la siguiente imágen.

![](images/part2/part2-lb-bp-create.png)

3. A continuación cree un *Health Probe*, guiese con la siguiente imágen.

![](images/part2/part2-lb-hp-create.png)

4. A continuación cree un *Load Balancing Rule*, guiese con la siguiente imágen.

![](images/part2/part2-lb-lbr-create.png)

5. Cree una *Virtual Network* dentro del grupo de recursos, guiese con la siguiente imágen.

![](images/part2/part2-vn-create.png)

#### Crear las maquinas virtuales (Nodos)

Ahora vamos a crear 3 VMs (VM1, VM2 y VM3) con direcciones IP públicas standar en 3 diferentes zonas de disponibilidad. Después las agregaremos al balanceador de carga.

1. En la configuración básica de la VM guíese por la siguiente imágen. Es importante que se fije en la "Avaiability Zone", donde la VM1 será 1, la VM2 será 2 y la VM3 será 3.

![](images/part2/part2-vm-create1.png)

2. En la configuración de networking, verifique que se ha seleccionado la *Virtual Network*  y la *Subnet* creadas anteriormente. Adicionalmente asigne una IP pública y no olvide habilitar la redundancia de zona.

![](images/part2/part2-vm-create2.png)

3. Para el Network Security Group seleccione "avanzado" y realice la siguiente configuración. No olvide crear un *Inbound Rule*, en el cual habilite el tráfico por el puerto 3000. Cuando cree la VM2 y la VM3, no necesita volver a crear el *Network Security Group*, sino que puede seleccionar el anteriormente creado.

![](images/part2/part2-vm-create3.png)

4. Ahora asignaremos esta VM a nuestro balanceador de carga, para ello siga la configuración de la siguiente imágen.

![](images/part2/part2-vm-create4.png)

5. Finalmente debemos instalar la aplicación de Fibonacci en la VM. para ello puede ejecutar el conjunto de los siguientes comandos, cambiando el nombre de la VM por el correcto

```
git clone https://github.com/daprieto1/ARSW_LOAD-BALANCING_AZURE.git

curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.34.0/install.sh | bash
source /home/vm1/.bashrc
nvm install node

cd ARSW_LOAD-BALANCING_AZURE/FibonacciApp
npm install

npm install forever -g
forever start FibonacciApp.js
```

Realice este proceso para las 3 VMs, por ahora lo haremos a mano una por una, sin embargo es importante que usted sepa que existen herramientas para aumatizar este proceso, entre ellas encontramos Azure Resource Manager, OsDisk Images, Terraform con Vagrant y Paker, Puppet, Ansible entre otras.

#### Probar el resultado final de nuestra infraestructura

1. Porsupuesto el endpoint de acceso a nuestro sistema será la IP pública del balanceador de carga, primero verifiquemos que los servicios básicos están funcionando, consuma los siguientes recursos:

```
http://52.155.223.248/
http://52.155.223.248/fibonacci/1
```

2. Realice las pruebas de carga con `newman` que se realizaron en la parte 1 y haga un informe comparativo donde contraste: tiempos de respuesta, cantidad de peticiones respondidas con éxito, costos de las 2 infraestrucruras, es decir, la que desarrollamos con balanceo de carga horizontal y la que se hizo con una maquina virtual escalada.

3. Agregue una 4 maquina virtual y realice las pruebas de newman, pero esta vez no lance 2 peticiones en paralelo, sino que incrementelo a 4. Haga un informe donde presente el comportamiento de la CPU de las 4 VM y explique porque la tasa de éxito de las peticiones aumento con este estilo de escalabilidad.

```
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10
```

**Preguntas**

* ¿Cuáles son los tipos de balanceadores de carga en Azure y en qué se diferencian?, ¿Qué es SKU, qué tipos hay y en qué se diferencian?, ¿Por qué el balanceador de carga necesita una IP pública?
* ¿Cuál es el propósito del *Backend Pool*?
* ¿Cuál es el propósito del *Health Probe*?
* ¿Cuál es el propósito de la *Load Balancing Rule*? ¿Qué tipos de sesión persistente existen, por qué esto es importante y cómo puede afectar la escalabilidad del sistema?.
* ¿Qué es una *Virtual Network*? ¿Qué es una *Subnet*? ¿Para qué sirven los *address space* y *address range*?
* ¿Qué son las *Availability Zone* y por qué seleccionamos 3 diferentes zonas?. ¿Qué significa que una IP sea *zone-redundant*?
* ¿Cuál es el propósito del *Network Security Group*?
* Informe de newman 1 (Punto 2)
* Presente el Diagrama de Despliegue de la solución.




