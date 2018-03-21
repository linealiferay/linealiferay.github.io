---
layout: default
---

# [](#header-1)1.  Objetivo concreto. 

Tras la propuesta de uno de los clientes con los que trabajamos desde la línea Liferay de tener un time to market lo más bajo posible nos pusimos el reto desde Sevilla no solo de darle al cliente lo que necesitaba en ese momento concreto sino que pensamos en los bloqueos actuales que teníamos en la mayoría de nuestros proyectos para poder realizar una integración continua y de depliegue de calidad que pudiese alcanzar de principio a fin desde nuestros entornos locales a todos los entornos de los clientes con los que trabajamos.

 Actualmente debido al uso limitado que podemos hacer del jenkins de steps por problemas de conectividad con los clientes finales pensamos en una alternativa que pudiesemos usar de forma global no solo en los proyectos de Liferay sino en todos los de Everis que tienen el mismo problema. Para ello tras hablar con el equipo de steps lanzamos y ejecutamos una propuesta del uso de esclavos que son invocados desde las propias máquinas de steps en los que cada uno de los proyectos pueda configurar sus propias herramientas de compilación y que tuviesen comunicación con los entornos finales de clientes, liberando así a la máquina en la que se encuentra instalado jenkins y delegando en éstos esclavos la ejecución, compilación y despliegue hacia entornos finales. 

## [](#header-11)¿Que hemos hecho?
*   Identificación de jobs comunes a todos los procesos de CI/CD.
*   Programación individual de los jobs.
*   Ejecución mediante workflow.
*   Ajuste y verificación de uso de pipelines con procesos paralelos.
*   Estandarización de proceso de despliegue.

## [](#header-12)¿Que hemos conseguido?
*   Reducción del time-to-market en la integración continua.
*   Reutilización de activos para nuevos proyectos

![](https://raw.githubusercontent.com/dmcisneros/dmcisneros.github.io/master/assets/images/ci/jenkins-docker-01.png)


# [](#header-1)2. Descripción del software reutilizable o herramienta de soporte al desarrollo: 

Desde el comienzo del estudio de realizar un aplicativo escalable, replicable y configurable para realizar una integración continua iniciamos nuestro estudio sobre las posibilidades del mercado para conseguir dicho objetivo y finalmente lo abordamos utilizando docker por su expansión en el mercado para gestionar las imágenes que necesitabamos de jenkins y los distintos esclavos. 

## [](#header-2)a.  Tecnologías y lenguajes de programación utilizados. 

### [](#header-3)Imágenes docker

#### [](#header-4)Jenkins
Partimos de una imágen de jenkins con la última versión de dockerhub a la que realizamos configuraciones y configuración de jobs para disminuir el tiempo de paquetización de nuestros artefactos lo mínimo posible. Para poder conseguir mejor tiempo de respuesta utilizamos jobs paralelos con pipelines ya que muchas de las tareas de compilación podían ejecutarse de forma paralela y no dependian entre ellas.

##### Las tareas/jobs utilizadas en todos los proyectos liferay son lo siguientes:

![](https://raw.githubusercontent.com/dmcisneros/dmcisneros.github.io/master/assets/images/ci/jenkins-jobs.png)

*   **Task_Update_Repository:** Actualizará el código del repositorio.
*   **Task_QA:** Ejecución de análisis estático de código.
*   **Task_Sonar:** Envío de código a analizar por sonar.
*   **Task_Junit:** Ejecución de pruebas unitarias
*   **Task_Build_Layouts:** Compilación con gulp de layouts
*   **Task_Build_Modules:** Compilación con gradle de módulos
*   **Task_Build_Themes:** Compilación gulp
*   **Task_Upload_Nexus:** Subida a nexus
*   **Task_Deploy:** Invocación de .sh para el despliegue al servidor liferay



#### [](#header-4)Esclavos-centos
Partimos de una imágen de centos muy liviana en la que instalamos las herramientas comunes de compilación de los proyectos de Liferay entre las que se encuentran:

*   **git**
*   **gradle**
*   **jdk8**
*   **node**

### [](#header-3)Pipeline: Parallels Jobs 

![](https://raw.githubusercontent.com/dmcisneros/dmcisneros.github.io/master/assets/images/ci/jenkins-jobs-parallels.png)

La configuración de workflows se puede realizar de una forma decalarativa en lugar de con las tareas de tipo flow, para ello podemos hacer uso de tareas de tipo pipeline. 
Ventajas que nos aportan los pipelines respecto a los flows:
*   Nos darán mayor flexibilidad de configurar el workflow de ejecución mediante código groovy 
*   El script lo podemos centralizar en un repositorio de control de versiones en lugar de escribir el código directamente en Jenkins y que antes de ejecutarse se actualice automáticamente. 
*   Nos permite lanzar la ejecución de tareas en paralelo, en nuestro caso es muy útil para disminuir el tiempo de ejecución del workflow.
*   Mejor visualización del proceso en su ejecución.

### [](#header-3)Despliegues
El proceso de despliegue ha quedado totalmente parametrizable y automatizado, mediante la creación de tags en git se iniciará el proceso de despliegue de los artefactos nexus para posteriormente poder ejecutar la tarea de despliegue, dicha tarea en nuestros proyectos la hemos delegado en un perfil de release-manager para que sea el que controle que versiones desplegar en los entornos.

![](https://raw.githubusercontent.com/dmcisneros/dmcisneros.github.io/master/assets/images/ci/nexus.png)

##### [](#header-5)Proceso de despliegue

1.  Descarga Artefactos de Nexus.
2.  Copia desde el esclavo al host de destino.
3.  Despliega a la carpeta deploy los artefactos.

![](https://raw.githubusercontent.com/dmcisneros/dmcisneros.github.io/master/assets/images/ci/deployment.png)

## [](#header-2)b.  Software libre utilizado. 
*   **git**
*   **gradle**
*   **docker**
*   **jdk8**
*   **nexus**

## [](#header-2)c.  Resumen de pasos para su instalación y prueba. 
Los pasos necesarios para adaptar la maqueta de integración continua serían los siguientes: 
*   Solicitar Herramientas de producción de proyecto: Solicitar mediante jira el alta de git, nexus, sonar, y Jenkins. En el caso de tener que conectar con entornos finales de cliente que requieren de una vpn será necesario utilizar las imágenes de docker preparadas para ello consultándolo con los arquitectos.
*   Configuración de pipeline: Configurar un jenkinsFile con los parámetros concretos de cada proyecto
*   Configuración de script de despliegue: Será necesario conocer las credenciales de acceso a los entornos finales para su configuración, en algunos casos será necesaria una autenticación directa entre el esclavo y la máquina destino
*   Configuración de los parámetros por defecto de la invocación del build de jenkins: Será necesario indicar la url de nexus, url de git, branch, environments,versión a desplegar, usuario, password y la ruta de la carpeta de despliegue.


## [](#header-2)d.  Información de entrada y de salida esperada en su utilización. 
La información de entrada será los parámetros de despliegue de los artefactos y la salida será el despliegue en los entornos finales.
![](https://raw.githubusercontent.com/dmcisneros/dmcisneros.github.io/master/assets/images/ci/deployment.png)

# [](#header-1)3. Inversión realizada. Detalle del esfuerzo en horas desglosado por persona. 



| Perfil        | Horas de dedicación          | 
|:-------------|:------------------|
| Arquitecto           | 360 horas | 




# [](#header-1)4. Indicadores utilizados para medir la mejora en el tiempo de desarrollo o en la calidad. 
El indicador principal es el tiempo necesario para poder desplegar un artefacto para su puesta en producción, con la implantación continua en uno de los proyectos no solo hemos reducido tiempos de despliegue sino que hemos obligado a que todos los artefactos pasen por unas fases de calidad para dar un producto final estable y de calidad. La paquetización de éste producto supone la redución del 80% de trabajo de un arquitecto para poder integrarlo en los nuevos proyectos gracias a su parametrización y flexibilidad. Por lo que si la dedicación para llevarlo a cabo fué de 360 horas en proyectos posteriores será de 72 horas.

# [](#header-1)5. Resultados obtenidos. Indicadores antes y después de la experiencia piloto realizada. 
Los resultados en los proyectos en los que se ha utilzado ésta solución paquetizada son excelentes, actualmente se ha integrado en orange, sacyr y ministerio de justicia pero el objetivo desde el centro de excelencia es el uso en todos los nuevos proyectos que se inicien porque la calidad y el valor añadido que se le aporta al cliente mejora enormemente ya que anteriormente no se estaban realizando tareas de automatización salvo algún proceso pequeño de paquetización.

# [](#header-1)6. Posibilidad de aplicación a otros servicios de Centros y requisitos para su utilización
Actualmente estamos trabajando de manera conjunta con Madrid, Barcelona y Londres para poder implantar el proyecto como mecanismo estandarizado para la integración continua de proyectos Liferay, se ha llevado a cabo en dos proyectos de forma exitosa y se va a implantar en los siguientes con la idea de que sea un estandar defacto cuando steps migre a su nueva versión. 

El uso de ésta paquetización a otros proyectos del centro es totalmente viable ya que solo sería necesario que cada proyecto instalase en su esclavo sus herramientas de compilación. Todos los demás pasos que se utilizan son comunes a la gran mayoría de proyectos del mercado.


![](https://raw.githubusercontent.com/dmcisneros/dmcisneros.github.io/master/assets/images/ci/team.jpg)




**L** _i_ **F** _e_ **R** _a_ **Y** ~~||~~.
