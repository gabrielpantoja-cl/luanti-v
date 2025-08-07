

# **Autoalojamiento de Supabase en DigitalOcean: Una Guía Exhaustiva de Despliegue, Seguridad y Análisis Estratégico**

## **Sección 1: Introducción y Análisis Estratégico**

### **1.1. Resumen Ejecutivo: La Decisión de Autoalojar**

Supabase se ha consolidado como una de las alternativas de código abierto más potentes a Firebase, ofreciendo un conjunto de herramientas integradas sobre una base de PostgreSQL. Si bien su plataforma gestionada (SaaS) ofrece una vía rápida y eficiente para el desarrollo de aplicaciones, la opción de autoalojamiento (self-hosting) se presenta como una alternativa atractiva para equipos que buscan un control absoluto sobre su infraestructura. La decisión de autoalojar Supabase en una plataforma como DigitalOcean no es meramente técnica; es una decisión estratégica fundamental que intercambia la conveniencia del servicio gestionado por una soberanía total sobre los datos, la configuración y la escalabilidad.

Este informe se ha elaborado para guiar a líderes técnicos, arquitectos de soluciones y desarrolladores senior a través del complejo panorama del autoalojamiento de Supabase en DigitalOcean. El objetivo es ir más allá de una simple lista de comandos para proporcionar un análisis profundo que abarque el ciclo de vida completo de la implementación: desde la planificación arquitectónica y el despliegue inicial hasta la configuración de producción, el fortalecimiento de la seguridad y las operaciones de mantenimiento a largo plazo.

El núcleo de esta decisión estratégica reside en un compromiso ineludible: al autoalojar, la organización asume la responsabilidad total de la disponibilidad, la seguridad, las actualizaciones y los respaldos.1 Si bien los componentes de código abierto proporcionados por Supabase son robustos y de nivel empresarial, la experiencia de autoalojamiento está en gran medida respaldada por la comunidad, lo que requiere un nivel significativo de experiencia en DevOps y administración de sistemas.3 Por lo tanto, la pregunta fundamental que este documento busca responder no es "¿Es posible autoalojar Supabase?", sino "¿Es la decisión correcta para nuestra organización, y estamos preparados para las responsabilidades que conlleva?".

### **1.2. Perfiles Ideales para el Autoalojamiento**

El autoalojamiento no es una solución universal. Su valor se maximiza en escenarios específicos donde las limitaciones de un modelo SaaS se convierten en un obstáculo para los objetivos del negocio. Los perfiles que más se benefician de una implementación autoalojada son aquellos cuyas necesidades no pueden ser satisfechas por la oferta estándar.

* **Empresas con Requisitos de Soberanía de Datos y Cumplimiento Normativo:** Organizaciones en sectores altamente regulados como la salud (requerimientos tipo HIPAA), las finanzas o el sector legal a menudo enfrentan mandatos estrictos sobre la localización física y la gestión de los datos de los usuarios. La plataforma SaaS de Supabase, alojada en un número selecto de regiones de AWS, puede no cumplir con estas normativas.4 El autoalojamiento en un centro de datos de DigitalOcean en una jurisdicción específica permite a estas empresas mantener un control total sobre la residencia de los datos, un requisito indispensable para el cumplimiento.2  
* **Proyectos con Necesidades de Personalización Extrema del Entorno:** La plataforma gestionada de Supabase, por su naturaleza, ofrece un entorno estandarizado. Los equipos que requieren acceso a nivel de sistema operativo para optimizaciones de bajo nivel, ajustes de hardware específicos (por ejemplo, patrones de disco para cargas de trabajo de escritura intensiva), o la instalación de extensiones de PostgreSQL que no están disponibles en los planes gestionados, encontrarán en el autoalojamiento la única vía para lograr la flexibilidad que necesitan.4  
* **Aplicaciones a Gran Escala Sensibles a la Latencia:** Para aplicaciones globales cuyo rendimiento depende críticamente de la baja latencia, la ubicación del backend es primordial. Si la base de usuarios principal o la infraestructura de la aplicación reside en una región geográfica donde Supabase SaaS no tiene presencia, la latencia de red puede degradar significativamente la experiencia del usuario.4 Desplegar una instancia autoalojada en un Droplet de DigitalOcean en un centro de datos cercano a los usuarios o a la aplicación puede reducir drásticamente los tiempos de respuesta de la base de datos y la API.  
* **Organizaciones con Equipos de DevOps/SRE Dedicados y Maduros:** El autoalojamiento es, en esencia, un proyecto de infraestructura. Las empresas que ya cuentan con equipos de ingeniería de fiabilidad del sitio (SRE) o DevOps poseen la experiencia y los recursos humanos necesarios para gestionar sistemas complejos. Para estas organizaciones, el costo de las horas de ingeniería dedicadas al mantenimiento no se ve como un gasto, sino como una inversión estratégica para obtener control, personalización y, potencialmente, una mayor eficiencia de costos a gran escala.1

### **1.3. Ventajas y Desventajas Fundamentales**

Antes de sumergirse en los detalles técnicos, es crucial tener una visión clara y equilibrada de los pros y los contras inherentes al autoalojamiento.

**Ventajas:**

* **Control Total y Soberanía de Datos:** La ventaja más significativa es el control absoluto sobre cada componente de la pila y, lo que es más importante, sobre los datos. La organización decide dónde se almacena la información, quién tiene acceso y cómo se gestiona, lo cual es fundamental para el cumplimiento normativo y la privacidad.2  
* **Personalización y Flexibilidad:** Se puede ajustar cada aspecto del entorno, desde la versión de PostgreSQL hasta la configuración del sistema operativo y la red, para optimizar el rendimiento y la funcionalidad según las necesidades específicas de la aplicación.4  
* **Potencial de Ahorro de Costos a Escala:** Aunque existen costos iniciales y operativos, para aplicaciones con un uso muy elevado, el costo fijo de la infraestructura de DigitalOcean puede ser significativamente menor que los costos variables y los precios por uso del servicio gestionado, que escalan con el número de usuarios, el almacenamiento y el ancho de banda.6 Un usuario reporta un costo de aproximadamente 35 USD/mes para una configuración funcional.7  
* **Reducción de la Dependencia del Proveedor:** Al gestionar la propia infraestructura, se mitiga el riesgo asociado a cambios en los precios, los términos de servicio o la estrategia de producto del proveedor de BaaS.2

**Desventajas:**

* **Carga Operativa y de Mantenimiento:** Esta es la contrapartida más importante. La responsabilidad de las actualizaciones de software, la aplicación de parches de seguridad, la monitorización del rendimiento y la resolución de problemas recae enteramente en el equipo interno. Es una tarea continua que consume tiempo y requiere experiencia.1  
* **Complejidad de la Configuración y Gestión:** La pila de Supabase consta de múltiples servicios interconectados. La configuración inicial, especialmente para un entorno de producción seguro, es compleja y propensa a errores. La gestión continua de este sistema distribuido requiere un conocimiento profundo de cada componente.7  
* **Responsabilidad Total sobre Seguridad y Respaldo:** A diferencia del modelo SaaS, no hay respaldos automáticos ni un equipo de seguridad dedicado por parte de Supabase. La implementación de una estrategia robusta de respaldos, recuperación ante desastres y fortalecimiento de la seguridad es una responsabilidad crítica y no trivial del equipo que autoaloja.1  
* **Disparidad de Características y Soporte:** La versión autoalojada a menudo carece de las últimas características disponibles en la plataforma en la nube y de las interfaces de usuario para la configuración. El soporte se limita a la comunidad (foros, GitHub), mientras que los planes de pago de Supabase ofrecen soporte profesional con SLAs.9 Esta brecha existe porque el enfoque comercial de Supabase es su producto SaaS, no una versión "on-premise" oficialmente soportada.4

En última instancia, la elección de autoalojar no se reduce a una simple comparación de costos entre un Droplet de DigitalOcean y un plan de Supabase. Es un cálculo más profundo que debe valorar el costo de las horas de ingeniería, el riesgo de tiempo de inactividad por una mala configuración y el impacto potencial de una brecha de seguridad. Es una evaluación de la madurez organizacional y la tolerancia al riesgo.

## **Sección 2: Arquitectura de una Instancia Autoalojada de Supabase**

Para operar con éxito una instancia autoalojada de Supabase, es imperativo comprender que no se está gestionando una aplicación monolítica, sino un ecosistema de microservicios de código abierto, orquestados para funcionar en conjunto. Cada componente tiene un rol específico, y un fallo en uno de ellos puede tener efectos en cascada en toda la plataforma.

### **2.1. El Ecosistema de Servicios de Supabase**

La pila de Supabase, tal como se define en su configuración de Docker, se compone de los siguientes servicios principales 11:

* **PostgreSQL:** Es el núcleo de toda la arquitectura. No solo actúa como la base de datos relacional para la aplicación, sino que también es la fuente única de verdad para los metadatos de los servicios de autenticación y almacenamiento. Su robustez y su sistema de extensiones son la base sobre la que se construye todo lo demás.11  
* **Kong:** Actúa como la puerta de enlace de la API (API Gateway). Es el único punto de entrada para todo el tráfico externo. Kong se encarga de recibir las solicitudes HTTP, aplicar políticas (como la autenticación) y enrutarlas al servicio interno apropiado.11  
* **GoTrue:** Es el servidor de autenticación responsable de la gestión de usuarios. Maneja los registros, los inicios de sesión (correo/contraseña, OAuth, enlaces mágicos) y la emisión de JSON Web Tokens (JWT) que se utilizan para autorizar las solicitudes en el resto de la pila.11  
* **PostgREST:** Un componente que transforma la base de datos PostgreSQL en una API RESTful de forma automática. Expone las tablas, vistas y funciones de la base de datos como puntos finales de la API, respetando los roles y permisos de PostgreSQL.11  
* **Realtime:** Un servidor construido en Elixir que proporciona funcionalidades en tiempo real. Utiliza la capacidad de replicación lógica de PostgreSQL para escuchar cambios en la base de datos (inserciones, actualizaciones, eliminaciones) y los transmite a los clientes suscritos a través de WebSockets.11  
* **Storage:** Proporciona una API RESTful para la gestión de archivos (objetos), como imágenes de perfil o documentos. Utiliza PostgreSQL para gestionar los metadatos y los permisos de acceso a los archivos, mientras que los archivos en sí se almacenan en un backend, que por defecto es el sistema de archivos local, pero en producción debería ser un servicio compatible con S3 como DigitalOcean Spaces.11  
* **postgres-meta:** Una API de utilidad que permite gestionar la propia base de datos, como obtener una lista de tablas y columnas, añadir roles o ejecutar consultas. Es utilizada principalmente por el Supabase Studio.11  
* **Supavisor:** Es un pooler de conexiones de alto rendimiento para PostgreSQL. Su función es gestionar de manera eficiente un gran número de conexiones de clientes a la base de datos, reduciendo la sobrecarga en el servidor de Postgres y mejorando la escalabilidad.15  
* **Analytics Server (Logflare):** Un componente opcional para la ingesta, el almacenamiento y la consulta de logs de la plataforma. En un entorno de producción, se recomienda encarecidamente que este servicio utilice su propia base de datos PostgreSQL, separada de la base de datos principal de la aplicación, para evitar la contención de recursos.16

La modularidad de esta arquitectura es, simultáneamente, su mayor fortaleza y su principal debilidad. Permite un control granular y la capacidad de reemplazar componentes individuales (por ejemplo, usar una base de datos externa gestionada en lugar del contenedor de Postgres 17), pero también introduce múltiples puntos de fallo y una curva de aprendizaje pronunciada. Gestionar "Supabase" en modo autoalojado es, en realidad, operar un sistema distribuido de aproximadamente nueve servicios interdependientes.

### **2.2. Orquestación con Docker y Docker Compose**

La magia que une todos estos servicios dispares en un host único es Docker y, más específicamente, Docker Compose. El fichero docker-compose.yml proporcionado en el repositorio de Supabase es el plano de la arquitectura.11 Este fichero define:

* **Servicios:** Cada uno de los componentes mencionados anteriormente se define como un servicio de Docker.  
* **Imágenes:** Especifica qué imagen de Docker se debe usar para cada servicio (por ejemplo, supabase/postgres, supabase/gotrue, etc.).  
* **Variables de Entorno:** Inyecta la configuración necesaria en cada contenedor, a menudo desde un fichero .env externo.  
* **Redes:** Crea una red interna de Docker a través de la cual los contenedores se comunican entre sí de forma segura, sin exponer sus puertos internos al mundo exterior.  
* **Volúmenes:** Define cómo persistir los datos, como los datos de la base de datos de PostgreSQL o los archivos de configuración, para que no se pierdan cuando se reinician los contenedores.  
* **Dependencias:** Utiliza la directiva depends\_on para asegurar que los servicios se inicien en el orden correcto (por ejemplo, los servicios de API no se inician hasta que la base de datos esté lista).

Aunque han surgido debates en la comunidad sobre la posibilidad de empaquetar todo en un único contenedor de Docker para simplificar el despliegue, el enfoque oficial y estándar de la industria es la arquitectura multi-contenedor.18 Esta modularidad es esencial para la mantenibilidad, la depuración y la actualización independiente de los componentes.

### **2.3. Arquitectura de Red Típica en DigitalOcean**

Al desplegar esta pila en DigitalOcean, la arquitectura de red conceptual sigue un flujo lógico diseñado para maximizar la seguridad y la eficiencia:

1. **Punto de Entrada:** El tráfico de Internet llega a una **IP Reservada** de DigitalOcean asignada al Droplet. El uso de una IP reservada asegura que la dirección IP pública del servicio no cambie si el Droplet se redimensiona o se recrea.  
2. **Primera Capa de Defensa:** Un **Firewall en la Nube** de DigitalOcean actúa como el primer filtro. Este firewall debe configurarse para permitir el tráfico entrante únicamente en los puertos 80 (HTTP) y 443 (HTTPS) para el tráfico web, y en el puerto 22 (SSH) solo desde direcciones IP de confianza para la administración. Todos los demás puertos deben estar bloqueados desde el exterior.  
3. **Terminación SSL y Enrutamiento:** Dentro del Droplet, un **servidor proxy inverso** (como Caddy o Nginx) escucha en los puertos 80 y 443\. Su función es triple: redirigir todo el tráfico HTTP a HTTPS, terminar la conexión SSL (descifrar el tráfico) y actuar como un único punto de enrutamiento.  
4. **Puerta de Enlace de la API:** El proxy inverso reenvía las solicitudes HTTP descifradas a la puerta de enlace de la API de Supabase, **Kong**, que se ejecuta internamente en un puerto no expuesto públicamente (por ejemplo, el puerto 8000).11  
5. **Comunicación Interna de Servicios:** Kong, basándose en la ruta de la solicitud (por ejemplo, /auth/v1, /rest/v1), enruta la solicitud al servicio interno apropiado (GoTrue, PostgREST, etc.) a través de la red privada de Docker.  
6. **Acceso a la Base de Datos:** Los servicios internos, como PostgREST o GoTrue, se comunican con la base de datos **PostgreSQL** a través de la misma red interna de Docker. La base de datos en sí no debe ser accesible directamente desde Internet.

Un aspecto arquitectónico crucial con profundas implicaciones operativas es la dependencia del servicio **Realtime** de la replicación lógica de PostgreSQL.13 Esto significa que cualquier base de datos PostgreSQL utilizada (ya sea la que se ejecuta en el contenedor o una externa) debe tener esta característica avanzada habilitada y configurada correctamente. Además, el rendimiento y la gestión del Write-Ahead Log (WAL) de la base de datos impactan directamente en la fiabilidad de las características en tiempo real. Si el servidor Realtime se desconecta o se retrasa, la base de datos podría eliminar segmentos del WAL que el servidor aún necesita, lo que provocaría una pérdida de datos en el flujo de tiempo real.13 Esto crea un acoplamiento muy estrecho entre la salud operativa de la base de datos y una funcionalidad central de Supabase, imponiendo una carga de mantenimiento que va más allá de la administración estándar de bases de datos.

## **Sección 3: Guía de Despliegue en DigitalOcean: Métodos y Procedimientos**

Existen varios caminos para desplegar Supabase en DigitalOcean, cada uno con sus propias ventajas, desventajas y niveles de complejidad. La elección del método adecuado depende del objetivo final: el aprendizaje, la creación rápida de prototipos o la implementación de un sistema de producción robusto y repetible.

### **3.1. Prerrequisitos Esenciales**

Independientemente del método de despliegue elegido, se deben cumplir los siguientes requisitos previos:

* **Cuenta de DigitalOcean:** Una cuenta activa con un método de facturación válido.  
* **Nombre de Dominio Registrado:** Se necesita un nombre de dominio propio (por ejemplo, mi-proyecto-supabase.com) que se configurará para apuntar al Droplet de Supabase.11  
* **Configuración de DNS:** Es necesario tener la capacidad de gestionar los registros DNS del dominio. La forma más sencilla es delegar la gestión del DNS a DigitalOcean cambiando los servidores de nombres (nameservers) en el registrador de dominios a ns1.digitalocean.com, ns2.digitalocean.com y ns3.digitalocean.com.20 Alternativamente, se pueden crear registros A o CNAME directamente en el proveedor de DNS para que apunten a la IP del Droplet.22  
* **Herramientas Locales:** En la máquina local del administrador, se deben tener instaladas las herramientas de línea de comandos git y docker (que incluye docker-compose).15

### **3.2. Método 1: Despliegue Manual en un Droplet Ubuntu (El Enfoque Didáctico)**

Este método es el más fundamental y se recomienda para aquellos que deseen comprender en profundidad cada componente y el proceso de configuración. Aunque no es el más rápido, proporciona el mayor nivel de aprendizaje.

* Paso 1: Creación del Droplet  
  En el panel de control de DigitalOcean, crear un nuevo Droplet. Se recomienda seleccionar una imagen de "Marketplace" que ya incluya Docker preinstalado para simplificar la configuración inicial. Como mínimo, se debe elegir un plan con 2 GB de RAM y 1 vCPU, aunque un plan con 4 GB de RAM y 2 vCPUs ofrecerá un mejor rendimiento inicial.19 Seleccionar la región del centro de datos más cercana a los usuarios finales y configurar la autenticación mediante una clave SSH, que es más segura que una contraseña.  
* Paso 2: Configuración Inicial del Servidor  
  Conectarse al Droplet recién creado a través de SSH: ssh root@\<ip-del-droplet\>. Es una buena práctica de seguridad no operar como root. Crear un nuevo usuario no privilegiado y añadirlo al grupo sudo para permitirle ejecutar comandos con privilegios elevados 19:  
  Bash  
  adduser miusuario  
  usermod \-aG sudo miusuario

  Para que el nuevo usuario pueda gestionar Docker sin usar sudo constantemente, hay que añadirlo al grupo docker:  
  Bash  
  usermod \-aG docker miusuario

  Finalmente, cambiar al nuevo usuario: su \- miusuario.  
* Paso 3: Clonación del Repositorio de Supabase  
  Desde el directorio personal del nuevo usuario, clonar el repositorio oficial de Supabase. El uso de \--depth 1 descarga solo la última versión, ahorrando espacio y tiempo.19  
  Bash  
  git clone \--depth 1 https://github.com/supabase/supabase.git

* Paso 4: Configuración Inicial  
  Navegar al directorio que contiene la configuración de Docker y copiar el fichero de ejemplo de variables de entorno para crear el fichero de configuración local.19  
  Bash  
  cd supabase/docker  
  cp.env.example.env

  Este fichero .env contendrá todos los secretos y la configuración específica de la instancia.  
* Paso 5: Primer Arranque (Inseguro)  
  Con la configuración por defecto (que es insegura y solo para pruebas), iniciar todos los servicios de Supabase en modo "detached" (-d) para que se ejecuten en segundo plano.  
  Bash  
  docker-compose up \-d

  La primera vez, este comando tardará varios minutos en descargar todas las imágenes de Docker necesarias.15 Se puede verificar el estado de los contenedores con  
  docker-compose ps.  
* Paso 6: Acceso Inicial  
  Una vez que todos los servicios estén en funcionamiento, se puede acceder al Supabase Studio a través del navegador web apuntando a la dirección IP pública del Droplet en el puerto 8000: http://\<ip-del-droplet\>:8000.15 Este acceso es inseguro (HTTP) y no está protegido por contraseña, lo que subraya la necesidad de la sección de seguridad de este informe.

### **3.3. Método 2: Despliegue Automatizado con Terraform (El Enfoque de Producción)**

Para cualquier despliegue serio o de producción, la automatización es clave. El uso de herramientas de Infraestructura como Código (IaC) como Terraform garantiza que el entorno sea repetible, versionable y menos propenso a errores humanos. DigitalOcean mantiene un repositorio oficial (supabase-on-do) que utiliza Packer y Terraform para un despliegue completo y robusto.11 Este es el método recomendado para producción.

* Paso 1: Prerrequisitos Adicionales  
  Instalar en la máquina local las CLIs de packer y terraform. En el panel de control de DigitalOcean, generar un token de acceso a la API con permisos de lectura y escritura. Además, crear un par de claves de acceso (key y secret) para DigitalOcean Spaces, que se utilizará para el almacenamiento de objetos.11  
* **Paso 2: Clonar el Repositorio supabase-on-do**  
  Bash  
  git clone https://github.com/digitalocean/supabase-on-do.git  
  cd supabase-on-do

* Paso 3: Construir la Imagen con Packer  
  Packer es una herramienta que automatiza la creación de imágenes de máquina. En este caso, creará un "snapshot" de un Droplet con todo el software necesario preinstalado.  
  Bash  
  cd packer  
  cp supabase.auto.pkrvars.hcl.example supabase.auto.pkrvars.hcl

  Editar el fichero supabase.auto.pkrvars.hcl para configurar las variables necesarias. Luego, inicializar Packer y construir la imagen 11:  
  Bash  
  packer init.  
  packer build.

  Este proceso puede tardar varios minutos y dará como resultado un snapshot personalizado en la cuenta de DigitalOcean.  
* Paso 4: Aprovisionar la Infraestructura con Terraform  
  Terraform leerá los ficheros de configuración para aprovisionar toda la infraestructura necesaria.  
  Bash  
  cd../terraform  
  cp terraform.tfvars.example terraform.tfvars

  Editar terraform.tfvars y rellenar todas las variables requeridas, como el token de la API de DO, el nombre del dominio, la región, las claves de Spaces y el nombre del snapshot de Packer creado en el paso anterior. Luego, ejecutar los comandos de Terraform 11:  
  Bash  
  terraform init  
  terraform plan  \# Opcional, para previsualizar los cambios  
  terraform apply \# Para crear los recursos

  Terraform creará el Droplet a partir del snapshot, un Volumen para el almacenamiento persistente de la base de datos, un bucket en Spaces, un Firewall y los registros DNS necesarios para el dominio.  
* Paso 5: Obtener Credenciales  
  Una vez que terraform apply haya finalizado, las contraseñas y secretos generados aleatoriamente se pueden recuperar utilizando el comando terraform output.11  
  Bash  
  terraform output htpasswd        \# Contraseña para el dashboard  
  terraform output psql\_pass      \# Contraseña de PostgreSQL  
  terraform output jwt\_secret      \# Secreto JWT

La existencia de este repositorio mantenido por DigitalOcean es una fuerte indicación de que la Infraestructura como Código es la mejor práctica respaldada por la plataforma para un despliegue de producción. Aunque la complejidad inicial es mayor, los beneficios en cuanto a consistencia y mantenibilidad son invaluables.

### **3.4. Método 3: Uso de la Aplicación 1-Click del Marketplace (El Enfoque de Inicio Rápido)**

Para la creación rápida de prototipos o entornos de prueba, el Marketplace de DigitalOcean ofrece una aplicación "1-Click" que simplifica enormemente el proceso de despliegue.25

* Paso 1: Creación desde el Marketplace  
  En el Marketplace de DigitalOcean, buscar "Supabase" y hacer clic en el botón "Create Supabase Droplet". Seleccionar el plan de Droplet deseado (se aplican las mismas recomendaciones de tamaño que en el método manual) y crear el Droplet.  
* Paso 2: Script de Configuración Inicial  
  Tras la creación del Droplet, se debe esperar unos 10 minutos para que se inicialicen todos los contenedores. Al conectarse por primera vez al Droplet vía SSH, se ejecutará automáticamente un script de configuración interactivo. Este script solicitará el nombre de dominio y una dirección de correo electrónico. Utilizará esta información para configurar automáticamente un proxy inverso y obtener un certificado SSL/TLS de Let's Encrypt, asegurando el acceso a través de HTTPS.25  
* Paso 3: Localizar Credenciales  
  Las credenciales de acceso generadas (usuario y contraseña para el dashboard) se almacenan en el fichero /srv/supabase/supabase/docker/.env dentro del Droplet. Se debe acceder a este fichero para recuperarlas.25

Este método es el más rápido y sencillo, pero ofrece la menor flexibilidad. La versión de Supabase y sus componentes puede no ser la más reciente, y la configuración subyacente está más oculta que en los otros métodos. Es ideal para probar la plataforma, pero para producción se recomienda uno de los dos métodos anteriores. La automatización del SSL en este método revela un punto crítico: la configuración de SSL es un paso esencial y no trivial que los otros métodos deben abordar manualmente.

### **Tabla 1: Comparativa de Métodos de Despliegue**

| Característica | Despliegue Manual en Droplet | Despliegue Automatizado con Terraform | Aplicación 1-Click del Marketplace |
| :---- | :---- | :---- | :---- |
| **Complejidad** | Media | Alta | Baja |
| **Velocidad de Despliegue** | Lenta | Rápida (tras configuración inicial) | Muy Rápida |
| **Flexibilidad/Control** | Muy Alto | Muy Alto | Bajo |
| **Mantenibilidad** | Media (manual) | Alta (basada en código) | Baja (caja negra) |
| **Caso de Uso Ideal** | Aprendizaje, depuración, personalización profunda. | Entornos de producción, staging, despliegues repetibles. | Prototipos, pruebas rápidas, evaluación. |

## **Sección 4: Configuración y Personalización para Producción**

Un despliegue por defecto de Supabase no está listo para producción. Es imperativo realizar una serie de configuraciones para asegurar la estabilidad, seguridad y funcionalidad completa de la plataforma. El centro neurálgico de esta configuración es el fichero .env.

### **4.1. El Fichero .env: El Centro de Control**

El fichero .env, ubicado en el directorio supabase/docker, es donde se definen todas las variables de entorno que controlan el comportamiento de los servicios de Supabase. Es de vital importancia que este fichero nunca se incluya en un repositorio de control de versiones público, ya que contiene todos los secretos de la aplicación.15

* **Generación de Secretos Seguros:** Los valores por defecto en .env.example son inseguros y deben ser reemplazados.  
  * POSTGRES\_PASSWORD: Debe ser una contraseña larga, compleja y única para el superusuario de la base de datos. Se puede generar con un gestor de contraseñas o un comando como openssl rand \-base64 32\.  
  * JWT\_SECRET: Es la clave utilizada para firmar todos los JSON Web Tokens. Debe ser una cadena aleatoria de al menos 32 caracteres. Un método para generarla es openssl rand \-base64 32\.15  
* **Configuración de URLs Públicas:**  
  * SITE\_URL: La URL base de la aplicación frontend. GoTrue la utiliza para construir los enlaces de redirección y los que se envían por correo electrónico (confirmación, reseteo de contraseña).15  
  * API\_EXTERNAL\_URL: La URL pública a través de la cual se accederá a la API de Supabase (por ejemplo, https://api.mi-proyecto-supabase.com). Esta URL es utilizada por los servicios internos para comunicarse entre sí y por el Supabase Studio para conectarse a las APIs.19

### **Tabla 2: Variables de Entorno Esenciales para Producción**

| Variable | Descripción | Ejemplo / Generación | Requerido/Opcional |
| :---- | :---- | :---- | :---- |
| POSTGRES\_PASSWORD | Contraseña para el rol postgres de la base de datos. | Generar una contraseña fuerte y aleatoria. | **Requerido** |
| JWT\_SECRET | Secreto para firmar los tokens JWT. Mínimo 32 caracteres. | Generar una cadena aleatoria y segura. | **Requerido** |
| ANON\_KEY | Clave de API pública (anónima). Se genera a partir del JWT\_SECRET. | Generar usando la herramienta de Supabase. | **Requerido** |
| SERVICE\_ROLE\_KEY | Clave de API de servicio (privilegiada). Se genera a partir del JWT\_SECRET. | Generar usando la herramienta de Supabase. | **Requerido** |
| SITE\_URL | URL base del sitio web o aplicación cliente. | https://mi-proyecto-supabase.com | **Requerido** |
| API\_EXTERNAL\_URL | URL pública completa de la instancia de Supabase. | https://api.mi-proyecto-supabase.com | **Requerido** |
| SMTP\_HOST | Host del servidor SMTP para el envío de correos. | smtp.sendgrid.net | Requerido para Auth |
| SMTP\_PORT | Puerto del servidor SMTP. | 587 | Requerido para Auth |
| SMTP\_USER | Usuario para la autenticación SMTP. | apikey (para SendGrid) | Requerido para Auth |
| SMTP\_PASS | Contraseña para la autenticación SMTP. | Clave de API de SendGrid. | Requerido para Auth |
| STORAGE\_BACKEND | Define el backend de almacenamiento. | s3 (para DigitalOcean Spaces) | Opcional (Recomendado) |
| GLOBAL\_S3\_BUCKET | Nombre del bucket en DO Spaces. | mi-bucket-de-storage | Requerido para s3 |
| GLOBAL\_S3\_REGION | Región del bucket de DO Spaces. | nyc3 | Requerido para s3 |

### **4.2. Generación y Aplicación de Claves de API**

La configuración de las claves de API es un proceso de dos pasos que a menudo causa confusión. Es crucial realizar ambos pasos correctamente para que la autenticación funcione.

1. **Generar el JWT\_SECRET:** Como se mencionó anteriormente, crear un secreto fuerte y único y colocarlo en la variable JWT\_SECRET del fichero .env.  
2. **Generar y Aplicar ANON\_KEY y SERVICE\_ROLE\_KEY:** Estas claves son en realidad JWTs pre-firmados con el JWT\_SECRET. Supabase proporciona una herramienta en su documentación para generarlos.15 Una vez generados, deben ser actualizados en  
   **dos lugares diferentes**:  
   * En el fichero .env, en las variables ANON\_KEY y SERVICE\_ROLE\_KEY.  
   * En el fichero de configuración de Kong, volumes/api/kong.yml, bajo la sección consumers.17

Este es un punto de fallo común. Si las claves no coinciden entre .env y kong.yml, Kong (la puerta de enlace) rechazará las solicitudes con un error de autenticación, aunque el token sea técnicamente válido, porque su configuración interna no coincide con la que esperan los demás servicios. Este diseño de "doble fuente de verdad" para las claves de API requiere una atención meticulosa durante la configuración y las actualizaciones.

### **4.3. Integración de Servicios Externos**

Para un entorno de producción robusto, se recomienda encarecidamente desacoplar ciertos componentes de la pila principal y utilizar servicios gestionados externos. Esto mejora la fiabilidad, la escalabilidad y la mantenibilidad.

* **Servicio SMTP para Autenticación:** El servicio GoTrue necesita enviar correos electrónicos para funciones críticas como la confirmación de cuenta y el restablecimiento de contraseña. DigitalOcean bloquea el tráfico saliente en el puerto 25 por defecto para prevenir el spam, lo que hace inviable ejecutar un servidor de correo propio en el Droplet.11 La solución es utilizar un servicio de correo transaccional externo como SendGrid, que ofrece un generoso plan gratuito. La configuración se realiza estableciendo las variables  
  SMTP\_\* en el fichero .env con las credenciales proporcionadas por SendGrid.  
* **Almacenamiento de Objetos con DigitalOcean Spaces:** Por defecto, el servicio de Storage de Supabase utiliza el sistema de archivos local del Droplet. Esto no es ideal para producción, ya que no es escalable ni altamente disponible. La mejor práctica es utilizar un backend de almacenamiento de objetos compatible con S3, como DigitalOcean Spaces.11 Para configurarlo, se deben establecer las siguientes variables de entorno en  
  .env 15:  
  * STORAGE\_BACKEND=s3  
  * GLOBAL\_S3\_BUCKET: El nombre del bucket creado en DO Spaces.  
  * GLOBAL\_S3\_REGION: La región del bucket (por ejemplo, nyc3).  
  * Establecer las variables de credenciales de Spaces (\*\_ACCESS\_KEY\_ID y \*\_SECRET\_ACCESS\_KEY).  
* **Base de Datos Externa Gestionada:** La recomendación más importante para producción es separar la base de datos de la pila de aplicaciones.17 Ejecutar la base de datos en el mismo Droplet que los servicios de la API crea un único punto de fallo y contención de recursos. La solución es utilizar una Base de Datos Gestionada de PostgreSQL de DigitalOcean. Esto ofrece beneficios como alta disponibilidad, respaldos automáticos y escalado independiente. Para realizar esta integración:  
  1. Crear un clúster de bases de datos gestionado en DigitalOcean.  
  2. En el fichero .env de Supabase, actualizar las variables POSTGRES\_\* (POSTGRES\_HOST, POSTGRES\_PORT, POSTGRES\_USER, POSTGRES\_PASSWORD, POSTGRES\_DB) para que apunten a la cadena de conexión de la base de datos gestionada.  
  3. En el fichero docker-compose.yml, comentar o eliminar por completo la definición del servicio db y cualquier directiva depends\_on: \[db\].  
  4. Es crucial asegurarse de que la base de datos externa tenga la replicación lógica habilitada para que el servicio Realtime funcione correctamente.

La transición a una arquitectura de producción implica inherentemente una evolución desde un modelo simple y autocontenido en un único Droplet hacia un sistema distribuido más complejo. Este cambio introduce nuevas consideraciones de red, seguridad y costos, pero es un paso necesario para construir una aplicación fiable y escalable.

## **Sección 5: Fortalecimiento de la Seguridad: Un Pilar Fundamental**

La seguridad en un entorno autoalojado no es una opción, es una obligación. La configuración por defecto de Supabase con Docker es inherentemente insegura para la exposición pública y debe ser fortalecida mediante un enfoque de "defensa en profundidad".15 El administrador de la plataforma es responsable de construir cada una de estas capas de seguridad.

### **5.1. Implementación de un Reverse Proxy con SSL/TLS Automático**

Un proxy inverso es un componente esencial que se sitúa entre Internet y la pila de Supabase. Cumple varias funciones críticas: actúa como un único punto de entrada, simplifica el enrutamiento del tráfico y, lo más importante, gestiona el cifrado SSL/TLS.

* **Guía para Caddy:** Caddy es una opción moderna y muy recomendada por su simplicidad y su gestión automática de HTTPS.19 Su fichero de configuración, el  
  Caddyfile, es notablemente conciso. Caddy detectará automáticamente los dominios en su configuración, se comunicará con una autoridad de certificación como Let's Encrypt para obtener certificados SSL/TLS, y los renovará automáticamente antes de que expiren.28  
  Un Caddyfile de ejemplo para Supabase, que se colocaría en /etc/caddy/Caddyfile en el Droplet, podría tener la siguiente estructura 19:  
  \# Dominio para el Studio y las APIs  
  mi-proyecto-supabase.com {  
      \# Proteger el Studio con autenticación básica  
      basicauth / {  
          mi\_usuario\_admin $2a$12$....hashed.password....  
      }

      \# Enrutar las APIs a Kong  
      reverse\_proxy /auth/v1/\* localhost:8000  
      reverse\_proxy /rest/v1/\* localhost:8000  
      reverse\_proxy /storage/v1/\* localhost:8000  
      reverse\_proxy /realtime/v1/\* localhost:8000

      \# Enrutar todo lo demás al Studio  
      reverse\_proxy \* localhost:3000  
  }

  La contraseña hash se genera con el comando caddy hash-password.  
* **Guía para Nginx:** Nginx es una alternativa más tradicional, potente y configurable. El repositorio supabase-on-do utiliza una imagen de Docker llamada swag (Secure Web Application Gateway), que empaqueta Nginx, certbot para la gestión de certificados SSL y fail2ban para la prevención de intrusiones.11 Esta es una solución más compleja pero también más rica en características. La configuración implicaría definir bloques  
  server y location en los ficheros de configuración de Nginx para realizar el proxy inverso a los servicios de Supabase y configurar certbot para que gestione los certificados.

La elección entre Caddy y Nginx/SWAG es una decisión entre simplicidad y potencia. Para la mayoría de los casos de uso, la facilidad de configuración y mantenimiento de Caddy es una ventaja significativa. Para aquellos que ya tienen experiencia con Nginx o necesitan sus características avanzadas, SWAG es una excelente opción.

### **5.2. Aseguramiento del Supabase Studio**

El Supabase Studio (el dashboard de administración) en la versión autoalojada no incluye su propio sistema de autenticación de usuarios.10 Exponerlo directamente a Internet sin protección es un riesgo de seguridad catastrófico, ya que daría a cualquiera acceso completo a la gestión de la base de datos.

* **Método 1: Autenticación Básica (Basic Auth):** Como se muestra en el ejemplo de Caddy anterior, la forma más sencilla de proteger el Studio es configurar la autenticación básica en el proxy inverso. Esto mostrará un cuadro de diálogo de inicio de sesión del navegador que solicita un nombre de usuario y una contraseña antes de permitir el acceso al dashboard.19  
* **Método 2: Restricción por IP o VPN:** Un enfoque aún más seguro es no exponer el Studio a la Internet pública en absoluto. Esto se puede lograr de dos maneras:  
  * **Restricción por IP:** Configurar el firewall de DigitalOcean o las reglas del proxy inverso para permitir el acceso al Studio solo desde una lista de direcciones IP de confianza (por ejemplo, la IP de la oficina).  
  * **VPN:** Configurar una red privada virtual (VPN) como WireGuard o OpenVPN en el Droplet o en la misma VPC. El Studio solo sería accesible para los usuarios conectados a la VPN.

### **5.3. Configuración de los Firewalls de DigitalOcean**

El Cloud Firewall de DigitalOcean es la primera línea de defensa a nivel de red. Debe configurarse para seguir el principio de mínimo privilegio, permitiendo solo el tráfico estrictamente necesario.

* **Reglas de Entrada (Inbound):**  
  * Permitir tráfico TCP en el puerto 443 (HTTPS) desde Cualquier origen.  
  * Permitir tráfico TCP en el puerto 80 (HTTP) desde Cualquier origen (necesario para la redirección a HTTPS y para el desafío de validación de Let's Encrypt).  
  * Permitir tráfico TCP en el puerto 22 (SSH) **únicamente** desde las direcciones IP de los administradores del sistema.  
* **Reglas de Salida (Outbound):** Generalmente, se puede permitir todo el tráfico saliente, aunque entornos de alta seguridad podrían restringirlo a los destinos conocidos.

Todos los demás puertos de los servicios de Supabase (8000, 3000, 5432, etc.) deben estar bloqueados para el acceso externo por este firewall. La comunicación con ellos solo debe ocurrir a través del proxy inverso.

### **5.4. Gestión de Secretos en Producción**

Reiterar los secretos por defecto es un riesgo inaceptable.15 Si bien el uso de un fichero

.env es común para el desarrollo, en producción es propenso a fugas accidentales (por ejemplo, si se comete en un repositorio de git). La práctica recomendada para entornos de producción es utilizar un gestor de secretos dedicado. Herramientas como([https://www.doppler.com/](https://www.doppler.com/)) o [Infisical](https://infisical.com/) permiten centralizar la gestión de secretos, controlar el acceso y inyectarlos de forma segura en el entorno de ejecución en el momento del despliegue, sin almacenarlos en ficheros de texto plano en el servidor.15

## **Sección 6: Operaciones y Mantenimiento a Largo Plazo**

El despliegue inicial es solo el comienzo del viaje del autoalojamiento. La responsabilidad más significativa es la operación y el mantenimiento continuo de la plataforma. Esto incluye la gestión de actualizaciones, la implementación de una estrategia de respaldo robusta y la monitorización del estado del sistema.

### **6.1. Estrategias de Actualización**

A diferencia del servicio gestionado, donde Supabase se encarga de las actualizaciones, en un entorno autoalojado esta tarea recae sobre el administrador. La pila de Supabase se actualiza con frecuencia para añadir nuevas características y corregir errores.

* **Proceso de Actualización de un Servicio:**  
  1. **Identificar la Nueva Versión:** El primer paso es consultar el Docker Hub o el repositorio de GitHub del servicio que se desea actualizar (por ejemplo, supabase/studio) para encontrar la etiqueta de la última versión estable.15  
  2. **Actualizar docker-compose.yml:** Editar el fichero docker-compose.yml y cambiar la etiqueta de la imagen del servicio correspondiente a la nueva versión.  
  3. **Descargar la Nueva Imagen:** Ejecutar docker-compose pull \<nombre-del-servicio\> para descargar la nueva imagen de Docker al Droplet.  
  4. **Reiniciar el Servicio:** Finalmente, reiniciar el contenedor para que utilice la nueva imagen. El uso de \--force-recreate asegura que se cree un nuevo contenedor con la nueva configuración.15  
     Bash  
     docker-compose up \-d \--force-recreate \<nombre-del-servicio\>

* **Manejo de Cambios Disruptivos (Breaking Changes):** Este proceso manual conlleva riesgos. Antes de actualizar cualquier componente, especialmente los críticos como postgres o postgrest, es fundamental leer detenidamente las notas de la versión (release notes) en busca de cambios que puedan romper la compatibilidad.31 La mejor práctica es tener un entorno de "staging" (una copia de la producción) donde las actualizaciones se puedan probar de forma segura antes de aplicarlas al entorno de producción real.32

Un riesgo inherente al proceso de actualización manual es la "deriva de configuración" (configuration drift). Con el tiempo, se pueden actualizar unos servicios y otros no, resultando en una combinación de versiones que nunca ha sido probada conjuntamente por el equipo de Supabase. Para mitigar esto y facilitar los retrocesos (rollbacks), es una práctica altamente recomendada gestionar toda la configuración del despliegue (incluyendo docker-compose.yml y los ficheros de configuración) en un repositorio de git. Esto permite rastrear cada cambio, entender el historial del despliegue y, si una actualización causa problemas, volver a un estado anterior y funcional simplemente revirtiendo a un commit anterior. Este enfoque eleva la estrategia de mantenimiento de simples comandos a un flujo de trabajo similar a GitOps.

### **6.2. Gestión de Respaldos (Backups): Una Responsabilidad Crítica**

En un entorno autoalojado, no hay respaldos automáticos. Si no se implementa una estrategia de respaldo, una fallo del disco o un error humano podría resultar en la pérdida total e irreversible de los datos.33

* Estrategia de Respaldo de la Base de Datos:  
  El método para respaldar la base de datos debe ser elegido con cuidado. La herramienta de línea de comandos de Supabase (supabase db dump) parece una opción obvia, pero tiene una limitación crítica para el autoalojamiento: excluye deliberadamente los esquemas gestionados por Supabase, como auth y storage.35 Esto significa que un respaldo creado con esta herramienta no contendrá los datos de los usuarios, la configuración de autenticación ni los metadatos de los archivos. Una restauración desde este tipo de respaldo sería catastrófica para la mayoría de las aplicaciones.  
  Por lo tanto, es necesario un enfoque más directo usando la herramienta nativa de PostgreSQL, pg\_dump. La estrategia recomendada, validada por la comunidad, es la siguiente 35:  
  1. **Utilizar pg\_dumpall o pg\_dump con el usuario supabase\_admin:** Este usuario parece tener los permisos necesarios para realizar un volcado completo de la base de datos, incluyendo los esquemas auth y storage.  
  2. **Ejecutar el comando desde el host:** Se puede ejecutar el comando pg\_dump dentro del contenedor de PostgreSQL a través de docker exec.  
  3. **Automatizar con cron:** Crear un script que ejecute el comando de respaldo y lo almacene en una ubicación segura (preferiblemente fuera del Droplet, como en un bucket de DigitalOcean Spaces). Luego, configurar una tarea de cron en el Droplet para que ejecute este script diariamente.

Un ejemplo de script de respaldo (backup.sh):Bash  
\#\!/bin/bash  
TIMESTAMP=$(date \+"%Y%m%d%H%M")  
BACKUP\_FILE="/backups/db\_backup\_${TIMESTAMP}.sql.gz"  
docker exec \-t \<nombre\_contenedor\_postgres\> pg\_dumpall \-U supabase\_admin | gzip \> $BACKUP\_FILE  
\# Opcional: Subir a DO Spaces  
\# s3cmd put $BACKUP\_FILE s3://mi-bucket-de-respaldos/

* **Estrategia de Respaldo de Storage:** Los respaldos de la base de datos no incluyen los archivos físicos almacenados por el servicio de Storage.34 Si se utiliza el sistema de archivos local (un Volumen de Docker), es necesario respaldar el contenido de ese volumen. Si se utiliza DigitalOcean Spaces, los archivos ya están en un sistema de almacenamiento redundante, pero aún así es una buena práctica crear respaldos en otra región o proveedor. Una herramienta como  
  rclone puede utilizarse para sincronizar el contenido del volumen o del bucket de Spaces a otra ubicación de respaldo.  
* **Point-in-Time Recovery (PITR):** Para una recuperación mucho más granular (a un punto específico en el tiempo, hasta de segundos), se requiere una configuración de PITR. Supabase utiliza la herramienta wal-g para esto en su plataforma gestionada.34 Configurar  
  wal-g en un entorno autoalojado es un proceso avanzado que implica archivar continuamente los logs de transacciones de PostgreSQL (ficheros WAL), pero ofrece el nivel más alto de protección de datos.

### **6.3. Monitorización y Escalado**

* **Monitorización:** Es vital vigilar la salud del Droplet. Las herramientas de monitorización integradas de DigitalOcean proporcionan gráficos sobre el uso de la CPU, la memoria RAM, la E/S del disco y el tráfico de red. Se deben configurar alertas para notificar al administrador si alguna de estas métricas supera un umbral crítico, lo que podría indicar un problema de rendimiento o un ataque.36  
* **Escalado Vertical:** Si la aplicación crece y el Droplet actual se queda sin recursos, la forma más sencilla de escalar es el escalado vertical. Esto implica redimensionar el Droplet a un plan superior con más CPU, RAM y almacenamiento a través del panel de control de DigitalOcean.  
* **Escalado Horizontal:** Para aplicaciones más grandes, el escalado horizontal ofrece mayor resiliencia. Esto implica distribuir la carga entre múltiples Droplets. El primer paso, y el más impactante, es el que ya se ha discutido: mover la base de datos a un clúster de bases de datos gestionado por DigitalOcean. Pasos más avanzados podrían incluir la ejecución de múltiples instancias de los servicios de la API de Supabase detrás de un balanceador de carga de DigitalOcean.

## **Sección 7: Análisis Comparativo: Autoalojado vs. Servicio Gestionado de Supabase**

La elección final entre autoalojar y utilizar el servicio gestionado depende de una evaluación cuidadosa de los costos, las características, la experiencia de usuario y otros factores cualitativos. Esta sección proporciona una comparación directa para facilitar esa decisión.

### **7.1. Desglose Detallado de Costos**

Una comparación de costos superficial puede ser engañosa. El costo real depende en gran medida del uso.

* **Escenario Autoalojado en DigitalOcean:** Los costos se componen de varios elementos:  
  * **Droplet:** El costo mensual del servidor virtual. Un plan "Basic" con 2 vCPUs y 4 GB de RAM cuesta aproximadamente 24 USD/mes.37  
  * **Almacenamiento:** Si se utiliza un Volumen de Bloque para la base de datos y/o Spaces para el almacenamiento de objetos, estos tienen costos adicionales. Por ejemplo, 100 GB de almacenamiento en Spaces cuestan alrededor de 5 USD/mes.38  
  * **Ancho de Banda:** Los Droplets incluyen una generosa cuota de transferencia saliente. El tráfico adicional se factura a 0.01 USD/GB.36  
  * **Costo de Ingeniería:** Este es el costo "oculto" pero más significativo. El tiempo que un ingeniero dedica a la configuración, mantenimiento, actualizaciones y resolución de problemas es un costo real para la empresa.1  
* **Escenario de Servicio Gestionado de Supabase:** La estructura de precios es un modelo de suscripción con componentes de pago por uso.  
  * **Plan Pro:** Comienza en 25 USD/mes. Esto incluye una cuota base de recursos: 8 GB de base de datos, 100 GB de almacenamiento de archivos y 250 GB de ancho de banda.39  
  * **Costos de Uso Adicional (Overages):** Cualquier uso por encima de las cuotas incluidas se factura por separado (por ejemplo, 0.125 USD/GB para almacenamiento de base de datos adicional).39  
  * **Add-ons:** Características como Point-in-Time Recovery (PITR), tamaños de cómputo más grandes o dominios personalizados tienen costos adicionales.39

El verdadero costo del autoalojamiento no es la factura mensual de DigitalOcean, sino el "Costo Total de Propiedad" (TCO), que debe incluir el salario del tiempo de los ingenieros responsables de las operaciones. El precio del servicio gestionado es, en efecto, una prima de seguro contra estos costos operativos. Para la mayoría de las pequeñas y medianas empresas, el servicio gestionado es casi con toda seguridad más económico cuando se considera el TCO.

### **Tabla 3: Análisis de Costos Estimados: Autoalojado vs. Gestionado**

| Recurso / Escenario | Autoalojado (Estimación Mensual) | Plan Supabase Pro (Estimación Mensual) |
| :---- | :---- | :---- |
| **Escenario Pequeño** |  |  |
| (10GB DB, 50GB Storage, 100GB BW) | Droplet: $24, Storage: $5 \= **$29** | Base: $25, DB Overage: $2.50 \= **$27.50** |
| **Escenario Mediano** |  |  |
| (50GB DB, 200GB Storage, 500GB BW) | Droplet: $48, Storage: $10, BW: $2.50 \= **$60.50** | Base: $25, DB Overage: $52.50, Storage Overage: $2.10, BW Overage: $22.50 \= **$102.10** |
| **Escenario Grande** |  |  |
| (100GB DB, 500GB Storage, 1TB BW) | Droplet: $96, Storage: $25, BW: $7.50 \= **$128.50** | Base: $25, DB Overage: $115, Storage Overage: $8.40, BW Overage: $67.50 \= **$215.90** |
| **Costo de Ingeniería (TCO)** | **Alto (no cuantificado)** | **Bajo (incluido en el precio)** |

*Nota: Las estimaciones son aproximadas y no incluyen todos los posibles costos adicionales como compute add-ons en Supabase o backups en DigitalOcean. El punto de cruce donde el autoalojamiento se vuelve más barato monetariamente depende en gran medida del uso de la base de datos y el ancho de banda.*

### **7.2. Paridad de Características y Experiencia de Usuario**

Un usuario podría asumir que "autoalojado" significa obtener exactamente el mismo producto, solo que en servidores propios. Esta suposición es incorrecta. Existen diferencias significativas en las características y la experiencia del desarrollador.

* **Interfaces de Configuración:** En la plataforma gestionada, la configuración de proveedores de OAuth, la personalización de plantillas de correo electrónico y otras opciones se realizan a través de una interfaz de usuario amigable en el dashboard. En un entorno autoalojado, toda esta configuración debe realizarse manualmente editando ficheros de configuración (.env, docker-compose.yml).10  
* **Gestión Multi-proyecto:** El dashboard autoalojado está diseñado para un solo proyecto. No es posible gestionar múltiples proyectos de Supabase desde una única instancia del dashboard, a diferencia de la plataforma en la nube.10  
* **Disponibilidad de Características de Plataforma:** Las características más avanzadas y centradas en la plataforma, como "Branching" (ramificación de bases de datos para desarrollo), el escalado de cómputo con un solo clic o los add-ons de la UI, se lanzan primero (y a veces exclusivamente) en el servicio gestionado.9  
* **Edge Functions:** Aunque las funciones Edge ahora se pueden ejecutar en un entorno autoalojado, su gestión, monitorización y depuración a través del dashboard son muy limitadas o inexistentes en comparación con la experiencia integrada que ofrece la nube.10

### **Tabla 4: Matriz de Paridad de Características**

| Característica | Servicio Gestionado Supabase | Autoalojado en DigitalOcean |
| :---- | :---- | :---- |
| **UI de Configuración de Auth** | Sí (completa) | No (vía fichero de configuración) |
| **UI de Plantillas de Email** | Sí | No (vía fichero de configuración) |
| **Respaldos Automáticos** | Sí (diarios, con retención) | No (responsabilidad del usuario) |
| **Point-in-Time Recovery (PITR)** | Sí (como add-on de pago) | No (requiere configuración manual avanzada con wal-g) |
| **Escalado de Cómputo con un Clic** | Sí | No (requiere redimensionamiento manual del Droplet) |
| **Branching de Base de Datos** | Sí (en planes de pago) | No |
| **Gestión Multi-proyecto** | Sí (desde una cuenta) | No (una instancia por proyecto) |
| **Soporte Técnico Profesional** | Sí (con SLAs en planes de pago) | No (soporte comunitario vía GitHub/Discord) |
| **Cumplimiento (SOC2, HIPAA)** | Sí (en plan Team/Enterprise) | No (responsabilidad total del usuario) |

### **7.3. Evaluación Cualitativa: Más Allá del Dinero**

* **Tiempo de Ingeniería y Enfoque:** El factor más crítico. El tiempo dedicado a la gestión de la infraestructura es tiempo que no se dedica a construir el producto principal. El servicio gestionado compra este enfoque.1  
* **Fiabilidad y Tiempo de Actividad (Uptime):** Con el servicio gestionado, Supabase es contractualmente responsable de la disponibilidad de la plataforma. En el autoalojamiento, un error de configuración, un fallo de hardware o un mantenimiento mal planificado pueden provocar tiempos de inactividad cuya responsabilidad y resolución recaen 100% en el equipo interno.1  
* **Seguridad:** Si bien el autoalojamiento ofrece control, también presenta una superficie de ataque que debe ser gestionada activamente. El servicio gestionado se beneficia de un equipo de seguridad dedicado y de auditorías regulares como SOC2, algo que es costoso y complejo de replicar internamente.4

La elección entre autoalojado y gestionado es una clásica decisión de "construir vs. comprar" (build vs. buy) aplicada a las operaciones de la plataforma. "Comprar" (el servicio gestionado) optimiza la velocidad de salida al mercado y el enfoque en el producto. "Construir" (el autoalojamiento) optimiza el control y la reducción de costos a largo plazo y a gran escala, pero a expensas de la velocidad inicial y del enfoque del equipo de desarrollo.

## **Sección 8: Conclusiones y Recomendaciones Finales**

### **8.1. Resumen de Hallazgos Clave**

El análisis exhaustivo del autoalojamiento de Supabase en DigitalOcean revela una serie de hallazgos críticos. Primero, es técnicamente factible y existen múltiples vías para el despliegue, desde métodos manuales didácticos hasta enfoques de producción automatizados con Infraestructura como Código, siendo este último el recomendado. Segundo, la responsabilidad operativa que se asume es inmensa, abarcando la seguridad, las actualizaciones, los respaldos y la monitorización, tareas que en el servicio gestionado son invisibles para el usuario. Tercero, la seguridad no es opcional; la configuración por defecto es insegura y requiere un fortalecimiento deliberado y multicapa. Finalmente, la comparación con el servicio gestionado muestra un claro compromiso: el autoalojamiento ofrece un control sin precedentes y un potencial de ahorro a escala, pero a costa de una mayor complejidad, una carga operativa significativa y una disparidad de características.

### **8.2. Matriz de Decisión: ¿Qué Camino Tomar?**

Basándose en los hallazgos de este informe, se proponen las siguientes recomendaciones estratégicas adaptadas a diferentes perfiles de proyecto:

* **Para Proyectos Personales, Hobby y Prototipos:**  
  * **Recomendación:** **Utilizar el plan gratuito del servicio gestionado de Supabase.**  
  * **Justificación:** El autoalojamiento representa una sobrecarga de trabajo y complejidad completamente innecesaria para esta escala. El plan gratuito de Supabase ofrece recursos más que suficientes para experimentar, aprender y validar ideas sin ningún costo monetario ni carga operativa.7  
* **Para Startups y Pequeñas y Medianas Empresas (PYMES) en Crecimiento:**  
  * **Recomendación:** **Comenzar con el plan Pro del servicio gestionado de Supabase.**  
  * **Justificación:** En esta etapa, la velocidad de desarrollo y la capacidad de iterar sobre el producto son mucho más valiosas que el ahorro marginal en costos de infraestructura. El costo del plan Pro (a partir de 25 USD/mes) es una inversión justificable para externalizar por completo las operaciones de la plataforma, permitiendo que el equipo de desarrollo se centre en crear valor para el cliente.1 La migración al autoalojamiento solo debe considerarse si se alcanzan límites de escala que hacen que los costos de uso adicional sean prohibitivos, o si surgen requisitos de cumplimiento normativo que el servicio gestionado no puede satisfacer.  
* **Para Grandes Empresas y Casos de Uso Específicos (Regulados, Sensibles a la Latencia):**  
  * **Recomendación:** **El autoalojamiento es una opción viable y potente, pero debe ser abordado como un proyecto de infraestructura de primer nivel.**  
  * **Justificación:** Para organizaciones con las necesidades y los recursos adecuados, el autoalojamiento desbloquea el máximo potencial de Supabase. El despliegue debe realizarse utilizando el enfoque automatizado con Terraform para garantizar la consistencia y la repetibilidad. Se debe asignar un equipo de DevOps o SRE dedicado para el mantenimiento, la seguridad y las operaciones continuas. Esta es la única vía para satisfacer requisitos estrictos de soberanía de datos, personalización extrema o latencia ultra baja.4

### **8.3. Reflexiones Finales**

El autoalojamiento de Supabase en DigitalOcean representa la máxima expresión de la filosofía del código abierto: proporciona las herramientas para construir una alternativa potente a las soluciones propietarias, otorgando al usuario final una libertad y un control absolutos. Sin embargo, esta libertad no es gratuita. Viene acompañada de una responsabilidad proporcional. El viaje del autoalojamiento es un camino que ofrece un control total sobre el destino de la propia infraestructura, pero exige a cambio una vigilancia constante, una experiencia técnica profunda y un compromiso ineludible con la excelencia operativa. La elección, por tanto, no debe tomarse a la ligera, sino como una decisión estratégica informada que alinee las capacidades de la organización con las demandas de la tecnología.

#### **Obras citadas**

1. Downside of self-hosting Supabase? \- Reddit, fecha de acceso: agosto 6, 2025, [https://www.reddit.com/r/Supabase/comments/1it991b/downside\_of\_selfhosting\_supabase/](https://www.reddit.com/r/Supabase/comments/1it991b/downside_of_selfhosting_supabase/)  
2. What is Self-Hosted Software | An Overview with Pros and Cons \- Blog, fecha de acceso: agosto 6, 2025, [https://blog.dreamfactory.com/the-pros-and-cons-of-self-hosted-software-solutions](https://blog.dreamfactory.com/the-pros-and-cons-of-self-hosted-software-solutions)  
3. Self-Hosting | Supabase Docs, fecha de acceso: agosto 6, 2025, [https://supabase.com/docs/guides/self-hosting](https://supabase.com/docs/guides/self-hosting)  
4. The Case For Better Self-hosting Supabase Support | by Sachin Agarwal | Medium, fecha de acceso: agosto 6, 2025, [https://medium.com/@sachin\_49501/the-case-for-better-self-hosting-supabase-support-dbdab63df3fa](https://medium.com/@sachin_49501/the-case-for-better-self-hosting-supabase-support-dbdab63df3fa)  
5. Self-hosting with Supabase \- DEV Community, fecha de acceso: agosto 6, 2025, [https://dev.to/chronsyn/self-hosting-with-supabase-1aii](https://dev.to/chronsyn/self-hosting-with-supabase-1aii)  
6. Appwrite vs Supabase: When Does Self-Hosting Outperform Cloud? \- Blog \- Codigee, fecha de acceso: agosto 6, 2025, [https://codigee.com/blog/appwrite-vs-supabase-when-does-self-hosting-outperform-cloud](https://codigee.com/blog/appwrite-vs-supabase-when-does-self-hosting-outperform-cloud)  
7. Is Self-Hosting Supabase Worth It? \- Reddit, fecha de acceso: agosto 6, 2025, [https://www.reddit.com/r/Supabase/comments/1i3mduv/is\_selfhosting\_supabase\_worth\_it/](https://www.reddit.com/r/Supabase/comments/1i3mduv/is_selfhosting_supabase_worth_it/)  
8. Supabase self hosted vs hosted? \- Reddit, fecha de acceso: agosto 6, 2025, [https://www.reddit.com/r/Supabase/comments/1iknrqx/supabase\_self\_hosted\_vs\_hosted/](https://www.reddit.com/r/Supabase/comments/1iknrqx/supabase_self_hosted_vs_hosted/)  
9. Troubleshooting | Are all features available in self-hosted Supabase?, fecha de acceso: agosto 6, 2025, [https://supabase.com/docs/guides/troubleshooting/are-all-features-available-in-self-hosted-supabase-THPcqw](https://supabase.com/docs/guides/troubleshooting/are-all-features-available-in-self-hosted-supabase-THPcqw)  
10. Is supabase self hosted good? \- Reddit, fecha de acceso: agosto 6, 2025, [https://www.reddit.com/r/Supabase/comments/15an011/is\_supabase\_self\_hosted\_good/](https://www.reddit.com/r/Supabase/comments/15an011/is_supabase_self_hosted_good/)  
11. digitalocean/supabase-on-do \- GitHub, fecha de acceso: agosto 6, 2025, [https://github.com/digitalocean/supabase-on-do](https://github.com/digitalocean/supabase-on-do)  
12. Minimal Docker Compose setup for self-hosting Supabase \- GitHub, fecha de acceso: agosto 6, 2025, [https://github.com/secretarybird97/supabase-docker](https://github.com/secretarybird97/supabase-docker)  
13. Self-Hosting Realtime \- Supabase, fecha de acceso: agosto 6, 2025, [https://supabase.com/docs/reference/self-hosting-realtime/introduction](https://supabase.com/docs/reference/self-hosting-realtime/introduction)  
14. Self-Hosting Storage \- Supabase Docs, fecha de acceso: agosto 6, 2025, [https://supabase.com/docs/reference/self-hosting-storage/introduction](https://supabase.com/docs/reference/self-hosting-storage/introduction)  
15. Self-Hosting with Docker | Supabase Docs, fecha de acceso: agosto 6, 2025, [https://supabase.com/docs/guides/self-hosting/docker](https://supabase.com/docs/guides/self-hosting/docker)  
16. Self-Hosting Analytics \- Supabase, fecha de acceso: agosto 6, 2025, [https://supabase.com/docs/reference/self-hosting-analytics/introduction](https://supabase.com/docs/reference/self-hosting-analytics/introduction)  
17. Self-Hosting with Docker | Supabase Docs \- Vercel, fecha de acceso: agosto 6, 2025, [https://docs-n3gxhwtbf-supabase.vercel.app/docs/guides/self-hosting/docker](https://docs-n3gxhwtbf-supabase.vercel.app/docs/guides/self-hosting/docker)  
18. Self-Host Supabase in a \*Single\* Docker Container \- Reddit, fecha de acceso: agosto 6, 2025, [https://www.reddit.com/r/Supabase/comments/1ipulw1/selfhost\_supabase\_in\_a\_single\_docker\_container/](https://www.reddit.com/r/Supabase/comments/1ipulw1/selfhost_supabase_in_a_single_docker_container/)  
19. Self-hosting Supabase on Ubuntu and Digital Ocean | by Kelvin ..., fecha de acceso: agosto 6, 2025, [https://medium.com/@kelvinpompey.me/self-hosting-supabase-on-ubuntu-and-digital-ocean-9bff8a819250](https://medium.com/@kelvinpompey.me/self-hosting-supabase-on-ubuntu-and-digital-ocean-9bff8a819250)  
20. Point to DigitalOcean Name Servers From Common Domain Registrars, fecha de acceso: agosto 6, 2025, [https://docs.digitalocean.com/products/networking/dns/getting-started/dns-registrars/](https://docs.digitalocean.com/products/networking/dns/getting-started/dns-registrars/)  
21. How can I point my domain to a DigitalOcean Droplet using the DigitalOcean DNS?, fecha de acceso: agosto 6, 2025, [https://www.digitalocean.com/community/questions/how-can-i-point-my-domain-to-a-digitalocean-droplet-using-the-digitalocean-dns](https://www.digitalocean.com/community/questions/how-can-i-point-my-domain-to-a-digitalocean-droplet-using-the-digitalocean-dns)  
22. How to Manage Domains in App Platform | DigitalOcean Documentation, fecha de acceso: agosto 6, 2025, [https://docs.digitalocean.com/products/app-platform/how-to/manage-domains/](https://docs.digitalocean.com/products/app-platform/how-to/manage-domains/)  
23. droplet vs website to deploy a website and link my own domain : r/digital\_ocean \- Reddit, fecha de acceso: agosto 6, 2025, [https://www.reddit.com/r/digital\_ocean/comments/1iunp7a/droplet\_vs\_website\_to\_deploy\_a\_website\_and\_link/](https://www.reddit.com/r/digital_ocean/comments/1iunp7a/droplet_vs_website_to_deploy_a_website_and_link/)  
24. Self Host Supabase on DigitalOcean \- YouTube, fecha de acceso: agosto 6, 2025, [https://www.youtube.com/watch?v=dDhy6pk282U](https://www.youtube.com/watch?v=dDhy6pk282U)  
25. Supabase | DigitalOcean Marketplace 1-Click App, fecha de acceso: agosto 6, 2025, [https://marketplace.digitalocean.com/apps/supabase](https://marketplace.digitalocean.com/apps/supabase)  
26. DigitalOcean Marketplace Droplet 1-Click Apps, fecha de acceso: agosto 6, 2025, [https://docs.digitalocean.com/products/marketplace/droplet-1-click-apps/](https://docs.digitalocean.com/products/marketplace/droplet-1-click-apps/)  
27. DigitalOcean Marketplace, fecha de acceso: agosto 6, 2025, [https://marketplace.digitalocean.com/](https://marketplace.digitalocean.com/)  
28. Automatic HTTPS — Caddy Documentation, fecha de acceso: agosto 6, 2025, [https://caddyserver.com/docs/automatic-https](https://caddyserver.com/docs/automatic-https)  
29. SSL reverse proxy with Caddy, Docker and Let's Encrypt \- pdemro.com, fecha de acceso: agosto 6, 2025, [https://www.pdemro.com/ssl-reverse-proxy-with-caddy-docker-and-lets-encrypt/](https://www.pdemro.com/ssl-reverse-proxy-with-caddy-docker-and-lets-encrypt/)  
30. How To Set Up A Reverse Proxy With Free SSL Using Caddy \- Programonaut, fecha de acceso: agosto 6, 2025, [https://www.programonaut.com/how-to-set-up-a-reverse-proxy-with-free-ssl-using-caddy/](https://www.programonaut.com/how-to-set-up-a-reverse-proxy-with-free-ssl-using-caddy/)  
31. Upgrading | Supabase Docs, fecha de acceso: agosto 6, 2025, [https://supabase.com/docs/guides/platform/upgrading](https://supabase.com/docs/guides/platform/upgrading)  
32. How do you update your Self-Hosted Supabase? \- Reddit, fecha de acceso: agosto 6, 2025, [https://www.reddit.com/r/Supabase/comments/1jy8ch7/how\_do\_you\_update\_your\_selfhosted\_supabase/](https://www.reddit.com/r/Supabase/comments/1jy8ch7/how_do_you_update_your_selfhosted_supabase/)  
33. Database backups | Supabase Features, fecha de acceso: agosto 6, 2025, [https://supabase.com/features/database-backups](https://supabase.com/features/database-backups)  
34. Database Backups | Supabase Docs, fecha de acceso: agosto 6, 2025, [https://supabase.com/docs/guides/platform/backups](https://supabase.com/docs/guides/platform/backups)  
35. How to properly backup/restore self-hosted instance : r/Supabase \- Reddit, fecha de acceso: agosto 6, 2025, [https://www.reddit.com/r/Supabase/comments/1f3aa1w/how\_to\_properly\_backuprestore\_selfhosted\_instance/](https://www.reddit.com/r/Supabase/comments/1f3aa1w/how_to_properly_backuprestore_selfhosted_instance/)  
36. DigitalOcean Droplets | Scalable Cloud Compute Starting at $4/mo, fecha de acceso: agosto 6, 2025, [https://www.digitalocean.com/products/droplets](https://www.digitalocean.com/products/droplets)  
37. Droplet Pricing \- DigitalOcean, fecha de acceso: agosto 6, 2025, [https://www.digitalocean.com/pricing/droplets](https://www.digitalocean.com/pricing/droplets)  
38. Budget-Friendly Cloud Server Pricing \- DigitalOcean, fecha de acceso: agosto 6, 2025, [https://www.digitalocean.com/pricing](https://www.digitalocean.com/pricing)  
39. Pricing & Fees \- Supabase, fecha de acceso: agosto 6, 2025, [https://supabase.com/pricing](https://supabase.com/pricing)  
40. Auth Self-hosting Config | Supabase Docs, fecha de acceso: agosto 6, 2025, [https://supabase.com/docs/guides/self-hosting/auth/config](https://supabase.com/docs/guides/self-hosting/auth/config)  
41. Self-Hosting Functions \- Supabase Docs, fecha de acceso: agosto 6, 2025, [https://supabase.com/docs/reference/self-hosting-functions/introduction](https://supabase.com/docs/reference/self-hosting-functions/introduction)