### Escuela Colombiana de Ingeniería
### Arquitecturas de Software - ARSW

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

    [](images/lab/maquina_creada.jpg)

![Imágen 1](images/part1/part1-vm-basic-config.png)

2. Para conectarse a la VM use el siguiente comando, donde las `x` las debe remplazar por la IP de su propia VM (Revise la sección "Connect" de la virtual machine creada para tener una guía más detallada).

    `ssh scalability_lab@xxx.xxx.xxx.xxx`

    [Imagen 2](images/lab/conectando_maquina.jpg)

3. Instale node, para ello siga la sección *Installing Node.js and npm using NVM* que encontrará en este [enlace](https://linuxize.com/post/how-to-install-node-js-on-ubuntu-18.04/).
4. Para instalar la aplicación adjunta al Laboratorio, suba la carpeta `FibonacciApp` a un repositorio al cual tenga acceso y ejecute estos comandos dentro de la VM:

    `git clone <your_repo>`

    `cd <your_repo>/FibonacciApp`

    `npm install`

5. Para ejecutar la aplicación puede usar el comando `npm FibinacciApp.js`, sin embargo una vez pierda la conexión ssh la aplicación dejará de funcionar. Para evitar ese compartamiento usaremos *forever*. Ejecute los siguientes comando dentro de la VM.

    ` node FibonacciApp.js`

    Despues de ejecutar este comando podremos hacer las peticiones que se veran de la siguiente manera

    [](images/lab/usando_la_app.jpg)

6. Antes de verificar si el endpoint funciona, en Azure vaya a la sección de *Networking* y cree una *Inbound port rule* tal como se muestra en la imágen. Para verificar que la aplicación funciona, use un browser y user el endpoint `http://xxx.xxx.xxx.xxx:3000/fibonacci/6`. La respuesta debe ser `The answer is 8`.

![](images/part1/part1-vm-3000InboudRule.png)

Ahora usaremos http para verificar el endpoint 


7. La función que calcula en enésimo número de la secuencia de Fibonacci está muy mal construido y consume bastante CPU para obtener la respuesta. Usando la consola del Browser documente los tiempos de respuesta para dicho endpoint usando los siguintes valores:
    * 1000000
    * 1010000
    * 1020000
    * 1030000
    * 1040000
    * 1050000
    * 1060000
    * 1070000
    * 1080000
    * 1090000

    podemos ver que con estas consultas la mmaquina se demora mucho en realizar una respuesta.    

8. Dírijase ahora a Azure y verifique el consumo de CPU para la VM. (Los resultados pueden tardar 5 minutos en aparecer).

![Imágen 2](images/part1/part1-vm-cpu.png)


9. Ahora usaremos Postman para simular una carga concurrente a nuestro sistema. Siga estos pasos.
    * Instale newman con el comando `npm install newman -g`. Para conocer más de Newman consulte el siguiente [enlace](https://learning.getpostman.com/docs/postman/collection-runs/command-line-integration-with-newman/).
    * Diríjase hasta la ruta `FibonacciApp/postman` en una maquina diferente a la VM.
    * Para el archivo `[ARSW_LOAD-BALANCING_AZURE].postman_environment.json` cambie el valor del parámetro `VM1` para que coincida con la IP de su VM.
    * Ejecute el siguiente comando.

    ```
    newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
    newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10
    ```

    [](images/lab/registros_app.jpg)

10. La cantidad de CPU consumida es bastante grande y un conjunto considerable de peticiones concurrentes pueden hacer fallar nuestro servicio. Para solucionarlo usaremos una estrategia de Escalamiento Vertical. En Azure diríjase a la sección *size* y a continuación seleccione el tamaño `B2ms`.

![Imágen 3](images/part1/part1-vm-resize.png)


[](images/lab/uso%20de%20cpu.jpg)


[](images/lab/escalando_vertical.jpg)

11. Una vez el cambio se vea reflejado, repita el paso 7, 8 y 9.
[](images/lab/cambio_de_tamaño.jpg)

12. Evalue el escenario de calidad asociado al requerimiento no funcional de escalabilidad y concluya si usando este modelo de escalabilidad logramos cumplirlo.
    podemos ver como la mejoria es bastante notable ya que en un principio aparecian los registros despues de 14 segundos y despues de hacer un escalamiento ahora solo se demora 10
    [](images/lab/registros_get.jpg)


13. Vuelva a dejar la VM en el tamaño inicial para evitar cobros adicionales.

[](images/lab/escalando_vertical.jpg)

**Preguntas**

1. ¿Cuántos y cuáles recursos crea Azure junto con la VM?
    se crean 7 recursos:
      - Maquina virtial
      - Red Virtual
      - Interfaz de Red
      - Disco de arranque
      - Direccion ip publica
      - Reglas de puerto

2. ¿Brevemente describa para qué sirve cada recurso?
   - VM: Ejecuta el sistema operativo y la aplicación.
   - VNet y NIC: Permiten la comunicación entre la VM y otros servicios.
   - OS Disk: Contiene el sistema operativo y archivos críticos.
   - Dirección IP pública: Facilita el acceso remoto a la VM.
   - Grupo de recursos: Organiza y gestiona todos los recursos relacionados.
   - Reglas de puerto: Controlan qué tráfico se permite hacia la VM.


3. ¿Al cerrar la conexión ssh con la VM, por qué se cae la aplicación que ejecutamos con el comando `npm FibonacciApp.js`? ¿Por qué debemos crear un *Inbound port rule* antes de acceder al servicio?

   La aplicación se cae cuando se cierra la conexión SSH porque, en su forma predeterminada, los procesos en ejecución asociados a esa sesión de SSH se terminan. Si el comando    npm FibonacciApp.js no se ha configurado para ejecutarse en segundo plano, se detendrá al cerrarse la sesión SSH.
   la creacion de *Inbound port rule*  permite explícitamente el tráfico en los puertos que el servicio utiliza. Las reglas de entrada permiten establecer qué tráfico puede       acceder a la VM.
   
4. Adjunte tabla de tiempos e interprete por qué la función tarda tando tiempo.
5. Adjunte imágen del consumo de CPU de la VM e interprete por qué la función consume esa cantidad de CPU.
6. Adjunte la imagen del resumen de la ejecución de Postman. Interprete:
    * Tiempos de ejecución de cada petición.
    * Si hubo fallos documentelos y explique.
7. ¿Cuál es la diferencia entre los tamaños `B2ms` y `B1ls` (no solo busque especificaciones de infraestructura)?
8. ¿Aumentar el tamaño de la VM es una buena solución en este escenario?, ¿Qué pasa con la FibonacciApp cuando cambiamos el tamaño de la VM?
9. ¿Qué pasa con la infraestructura cuando cambia el tamaño de la VM? ¿Qué efectos negativos implica?
10. ¿Hubo mejora en el consumo de CPU o en los tiempos de respuesta? Si/No ¿Por qué?
11. Aumente la cantidad de ejecuciones paralelas del comando de postman a `4`. ¿El comportamiento del sistema es porcentualmente mejor?

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
1. ¿Cuáles son los tipos de balanceadores de carga en Azure y en qué se diferencian?, ¿Qué es SKU, qué tipos hay y en qué se diferencian?, ¿Por qué el balanceador de carga necesita una IP pública?

      ### Tipos de Balanceadores de Carga
      
         - Azure Load Balancer: Es un balanceador de carga de capa 4 (transporte), que distribuye el tráfico de red entrante hacia máquinas virtuales dentro de un conjunto de             disponibilidad o a través de instancias de backend.
      
         - Classic Load Balancer: Es la versión anterior, con menos características de escalabilidad y seguridad.
        
         - Standard Load Balancer: Es la versión más avanzada, con características de alta disponibilidad, mejor escalabilidad, soporte de reglas de NAT y un mayor control.
        
         - Application Gateway: Este balanceador de carga opera en la capa 7 (aplicación), y es adecuado para aplicaciones web. Ofrece características adicionales como la terminación SSL, el redireccionamiento de URL y el enrutamiento basado en URL.
         
         - Azure Traffic Manager: Es un balanceador de carga global que distribuye el tráfico entre diferentes regiones de Azure. Está basado en DNS y permite el enrutamiento de 
         tráfico según la proximidad geográfica, el rendimiento, la disponibilidad o las políticas personalizadas.
      
      ### Tipos de SKU
      SKU (Stock Keeping Unit) es un identificador que se usa para especificar la configuración y características de un servicio en Azure. Existen dos tipos principales de SKU en servicios como el Load Balancer:
      
      - Basic SKU: Tiene funcionalidades más limitadas y está diseñado para cargas más ligeras.
      - Standard SKU: Proporciona una mayor escalabilidad, características de seguridad mejoradas, y es recomendado para aplicaciones de mayor escala y más exigentes.
        
        ###  IP pública en balanceador de carga
        El balanceador de carga necesita una IP pública para permitir el acceso desde el exterior (Internet) hacia los recursos de backend de Azure. La IP pública se asocia con la entrada de tráfico, que luego se distribuye de manera equitativa entre las instancias de backend a través de las reglas de balanceo de carga.
        
     
2. ¿Cuál es el propósito del *Backend Pool*?
   
      El Backend Pool es un grupo de recursos, como máquinas virtuales, instancias de contenedor, o instancias de servicio, que recibirán el tráfico balanceado por el Load Balancer. Cada una de estas instancias recibirá una parte del tráfico que el balanceador distribuye.

   
3. ¿Cuál es el propósito del *Health Probe*?

      El Health Probe verifica el estado de salud de las instancias de backend del pool. Si una instancia no pasa la prueba de salud, el balanceador de carga deja de enviar tráfico a esa instancia hasta que vuelva a estar saludable. Esto garantiza que solo las instancias operativas reciban tráfico.


4. ¿Cuál es el propósito de la *Load Balancing Rule*? ¿Qué tipos de sesión persistente existen, por qué esto es importante y cómo puede afectar la escalabilidad del sistema?.

      ###proposito
      
      La Load Balancing Rule define cómo se distribuirá el tráfico entre las instancias del Backend Pool. Esta regla establece cómo los puertos de entrada se deben redirigir a las instancias de backend y qué tipo de tráfico debe ser balanceado.
      
      ### tipos de sesión persistente
      
      - Sticky Sessions (Session Affinity): Asegura que una vez que un cliente se conecta a una instancia de backend, todo su tráfico posterior se dirige a la misma instancia.
      - Non-sticky Sessions: El tráfico se distribuye de manera aleatoria entre todas las instancias.
      
        Importancia y escalabilidad:
      
      Las sesiones persistentes son necesarias para aplicaciones que requieren que un cliente interactúe con la misma instancia a lo largo de una sesión. Esto puede afectar la escalabilidad, ya que si un solo servidor recibe un tráfico excesivo debido a la afinidad de sesión, el sistema podría volverse menos eficiente.

5. ¿Qué es una *Virtual Network*? ¿Qué es una *Subnet*? ¿Para qué sirven los *address space* y *address range*?

      ### Virtual Network
      
      Una Virtual Network es un espacio aislado dentro de Azure que proporciona comunicación privada entre los recursos de Azure. Se puede configurar subredes, control de tráfico y reglas de seguridad dentro de una VNet.
      
      ### Subnet
      
      Una Subnet es una división dentro de una VNet que organiza los recursos en segmentos más pequeños y facilita el control del tráfico entre diferentes partes de la red.
      
      ### address
      - Address Space: Es el rango de direcciones IP que se puede usar dentro de una VNet.
      - Address Range: Especifica el rango de direcciones IP que se asignan a cada Subnet dentro de la VNet.

6. ¿Qué son las *Availability Zone* y por qué seleccionamos 3 diferentes zonas?. ¿Qué significa que una IP sea *zone-redundant*?
   
      ### Availability Zone
      
      Las Availability Zones son ubicaciones físicas dentro de una región de Azure que están separadas y tienen su propia infraestructura de energía, redes y refrigeración. Se seleccionan tres zonas para garantizar la alta disponibilidad y resiliencia de la infraestructura. Si una zona falla, las otras dos continúan operando, evitando la interrupción de los servicios.
      
      ### Ip zone-redundant
      
      es una IP que se distribuye a través de múltiples zonas de disponibilidad, lo que garantiza que la dirección IP siga siendo accesible incluso si una zona de disponibilidad se ve afectada.


7. ¿Cuál es el propósito del *Network Security Group*?

      es un conjunto de reglas de seguridad que controla el acceso a los recursos dentro de una red virtual. Puedes definir reglas para permitir o denegar tráfico entrante y saliente según direcciones IP, puertos y protocolos.

* Informe de newman 1 (Punto 2)
* Presente el Diagrama de Despliegue de la solución.




