# Lab 6 Microservices - Project Report

## 1. Configuration Setup

**Configuration Repository**: https://github.com/daniel-844388/lab6-microservices

Describe the changes you made to the configuration:

- What did you modify in `accounts-service.yml`?
Se ha cambiado el puerto en el que se expone el servicio del valor 3333 al valor 2222.
- Why is externalized configuration useful in microservices?
Permite realizar cambios en la configuración del servicio y que todas las instancias que arranquen a continuación apliquen la nueva configuración, evitando cambiar manualmente el fichero de configuración de cada instancia.

---

## 2. Service Registration (Task 1)

### Accounts Service Registration

![Accounts Registration Log](docs/screenshots/accounts-registration.png)

1. Se prepara e inicializa cliente Eureka:
```text
Setting initial instance status as: STARTING
Initializing Eureka in region us-east-1
Resolving eureka endpoints via configuration
```
2. Se obtiene la lista actual de servicios registrados:
```text
Getting all instance registry info from the eureka server
The response status is 200
```
3. Se inicia el heartbeat y el replicador:
```text
Starting heartbeat executor: renew interval is: 30
InstanceInfoReplicator onDemand update allowed rate per min is 4
```
4. Se registra el servicio con estado UP:
```text
Registering application ACCOUNTS-SERVICE with eureka with status UP
Saw local status change event StatusChangeEvent [timestamp=1764951972071, current=UP, previous=STARTING]
DiscoveryClient_ACCOUNTS-SERVICE/10.255.255.254:accounts-service:3333: registering service...
```
5. Se completa el inicia del servidor:
```text
Tomcat started on port 3333 (http) with context path '/'
```
6. Se confirma el registro:
```text
DiscoveryClient_ACCOUNTS-SERVICE/10.255.255.254:accounts-service:3333 - registration status: 204
```

### Web Service Registration

![Web Registration Log](docs/screenshots/web-registration.png)

Explain how the web service discovers the accounts service.
1. El servcio web pregunta a Eureka por instancias de "ACCOUNTS-SERVICE" mediante RestTemplate.
2. Eureka selecciona una instancia disponible con balanceo de carga.
3. Se sustituye "ACCOUNTS-SERVICE" por la URL y puerto en el que se encuentra la instancia devuelta por Eureka.

---

## 3. Eureka Dashboard (Task 2)

![Eureka Dashboard](docs/screenshots/eureka-dashboard.png)

Describe what the Eureka dashboard shows:

- Which services are registered?
Aparecen registrados el servidor de configuración, el servicio 'accounts' y el servicio 'web'.
- What information does Eureka track for each instance?
1. Application: El nombre de la aplicación o servicio.
2. AMIs: Información sobre Amazon Machine Image (no aplica) y número de instancias.
3. Availability Zones: zonas en las que hay instancias desplegadas (no aplica, entorno local) y número de instancias.
4. Status: Estado e instancias en dicho estado y dirección de acceso al servicio.  

---

## 4. Multiple Instances (Task 4)

![Multiple Instances](docs/screenshots/multiple-instances.png)

Answer the following questions:

- What happens when you start a second instance of the accounts service?
Aparece una nueva instancia del servicio, con su correspondiente dirección de acceso (en este caso con otro puerto)
- How does Eureka handle multiple instances?
Eureka permite varias instancias de un mismo servicio, cada una con su dirección y puerto de forma que se diferencien.
- How does client-side load balancing work with multiple instances?
El cliente solicita un listado de todas las instancias del servicio, y es el propio cliente, con su política de balanceo, el que determina qué instancia utiliza.

---

## 5. Service Failure Analysis (Task 5)

### Initial Failure

![Error Screenshot](docs/screenshots/failure-error.png)

Al realizar una nueva petición la página queda en blanco y se produce un error 500 ante petición GET.
En el log del servicio 'web' se ve un error en la conexión con el servicio 'accounts', porque la instancia que tenía registrada ya no está activa.

### Eureka Instance Removal

![Instance Removal](docs/screenshots/instance-removal.png)

Explain how Eureka detects and removes the failed instance:

- How long did it take for Eureka to remove the dead instance?
Unos pocos segundos.
- What mechanism does Eureka use to detect failures?
Utiliza 'heartbeats' que los distintos servicios envían a Eureka cada 5 segundos (lease-renewal-interval-in-seconds: 5).
Así, si Eureka no ha recibido 'heartbeat' de un servicio en 10 segundos (lease-expiration-duration-in-seconds: 10) establece la instancia como caída.

---

## 6. Service Recovery Analysis (Task 6)

![Recovery State](docs/screenshots/recovery.png)

Answer the following questions:

- Why does the web service eventually recover?
Porque hay otra instancia activa del servicio al que realizar la petición.
- How long did recovery take?
Casi inmediatamente.
- What role does client-side caching play in the recovery process?
El cliente guarda en caché las instancias del servicio, que actualiza periodicamente (eureka.client.registry-fetch-interval-seconds), y si la petición a una instancia falló tan solo tiene que probar con otras.

---

## 7. Conclusions

Summarize what you learned about:

- Microservices architecture
La arquitectura de microservicios permite que una aplicación pueda dividirse según funcionalidades, permitiendo que estos servicios sean independientes unos de otros; desde el lenguaje en el que se implementan, la independencia para desarrollarlos y modificarlos así como un escalado asimétrico, donde se pueden crear múltiples instancias de cada servicio según necesidades.
- Service discovery with Eureka
Eureka permite que varios servicios puedan descubrirse entre sí para comunicarse. Son los diferentes servicios los que se registran en Eureka y preguntan por otros servicios. Así, ni el resto de servicios ni el propio servidor Eureka deben conocer al resto de servicios de antemano. Además, el sistema permite registrar múltiples instancias de cada servicio y gestionar si estas están activas o caídas. 
- System resilience and self-healing
El sistema cuenta con un mecanismo basado en 'heatbeats' que garantiza a Eureka conocer si las instancias registradas están disponibles o no. Además, la caché en el lado del cliente garantiza mantener una copia actualizada de las instancias de los servicios en caso de que el servidor Eureka pueda fallar temporalmente.
- Challenges you encountered and how you solved them
Quería comprobar el tiempo real que tarda Eureka en detectar una caída. El dashboard no actualiza automáticamente, y al refrescar manualmente la página las caídas aparecen al instante (no esperan ausencia de heartbeats), y no he conseguido saber el motivo.

---

## 8. AI Disclosure

**Did you use AI tools?** (ChatGPT, Copilot, Claude, etc.)

Si, he usado ChatGPT para comprender algunos logs de la consola y para consultar algunos aspectos concretos del fucionamiento de Eureka.
Las cuestiones obtenidas mediante ChatGTP han sido verificadas en otras fuentes, como por ejemplo en documentación oficial de Spring:
https://spring.io/blog/2015/07/14/microservices-with-spring
https://docs.spring.io/spring-cloud-netflix/reference/spring-cloud-netflix.html
https://docs.spring.io/spring-cloud-netflix/reference/configprops.html

**Important**: Explain your own understanding of microservices patterns and Eureka behavior, even if AI helped you write parts of this report.

