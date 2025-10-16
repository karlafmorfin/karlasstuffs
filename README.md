# karlasstuffs
Mi primera aplicación con Azure DevOps y GitHub


YALM
1. trigger: - main
Descripción: Esta sección define cuándo debe ejecutarse automáticamente tu pipeline.
trigger:: Indica que la pipeline se activará automáticamente por cambios en el repositorio.
- main: Especifica que los cambios en la rama llamada main (que es tu rama principal en GitHub) son los que activarán la pipeline.
En resumen: Cada vez que hagas un git push a la rama main en tu repositorio de GitHub, Azure DevOps detectará este cambio y comenzará una nueva ejecución de esta pipeline.
`
2. pool: vmImage: 'ubuntu-latest'
Descripción: Esta sección define el "agente" o la máquina virtual donde se ejecutarán todas las tareas de tu pipeline.
pool:: Agrupa la configuración del agente.
vmImage: 'ubuntu-latest': Especifica que quieres usar la imagen de máquina virtual más reciente de Ubuntu. Azure DevOps tiene una serie de imágenes preconfiguradas (Windows, Ubuntu, macOS) con las herramientas de desarrollo más comunes ya instaladas (como .NET SDK, Node.js, Python, Git, etc.). Ubuntu es una opción popular y eficiente para proyectos .NET Core.
En resumen: Tu pipeline se ejecutará en un servidor de Ubuntu limpio y actualizado, provisto por Azure DevOps.
`
3. variables:
Descripción: Aquí puedes definir variables que puedes reutilizar a lo largo de tu pipeline para evitar repetir valores y hacer el código más fácil de mantener.
buildConfiguration: 'Release': Define una variable llamada buildConfiguration con el valor 'Release'. Esto significa que tu código se compilará en modo "Release", que generalmente optimiza el código para el rendimiento y el tamaño final, a diferencia de "Debug" que incluye información de depuración.
projectPath: 'MiWebAppKarlasStuffs/': Define una variable projectPath que contiene la ruta a la carpeta de tu proyecto .NET (MiWebAppKarlasStuffs) dentro de la raíz del repositorio. Usar esta variable es muy útil porque si tuvieras múltiples tareas que necesitan saber dónde está el proyecto, solo tendrías que cambiar esta línea.
En resumen: Hemos creado dos "accesos directos" con nombres para configuraciones importantes que usaremos más adelante en los pasos.
`
4. steps:
Descripción: Esta es la sección principal donde defines la secuencia de tareas que la pipeline ejecutará. Cada - task: representa un paso distinto.
steps:: Inicia la lista de pasos.
`
- task: DotNetCoreCLI@2: Indica que vamos a usar la tarea de Azure DevOps para la línea de comandos de .NET Core, en su versión 2. Esta tarea permite ejecutar comandos dotnet (como restore, build, test, publish).
displayName: 'Dotnet Restore': Es un nombre amigable que verás en la interfaz de Azure DevOps mientras la pipeline se ejecuta.
inputs:: Define los parámetros o argumentos para esta tarea específica.
command: 'restore': Le dice a la tarea DotNetCoreCLI que ejecute el comando dotnet restore. Este comando descarga todas las dependencias (paquetes NuGet) que tu proyecto necesita, basándose en lo que está declarado en tu archivo .csproj.
projects: '$(projectPath)MiWebAppKarlasStuffs.csproj': Le indica a dotnet restore sobre qué proyecto debe actuar. Aquí, usa la variable $(projectPath) para construir la ruta completa al archivo .csproj de tu aplicación: MiWebAppKarlasStuffs/MiWebAppKarlasStuffs.csproj.
En resumen: Este paso asegura que todas las librerías externas que tu aplicación utiliza sean descargadas y estén disponibles antes de intentar compilar el código.
`
- task: DotNetCoreCLI@2: Nuevamente, usamos la tarea de línea de comandos de .NET Core.
displayName: 'Dotnet Build': Nombre amigable para este paso.
inputs::
command: 'build': Le dice a la tarea que ejecute el comando dotnet build. Este comando compila tu código fuente (.cs y otros archivos) en un formato ejecutable (archivos .dll).
projects: '$(projectPath)MiWebAppKarlasStuffs.csproj': Especifica el proyecto que se debe compilar, usando la misma ruta y nombre de archivo .csproj.
arguments: '--configuration $(buildConfiguration)': Pasa argumentos adicionales al comando dotnet build. Aquí, --configuration $(buildConfiguration) le dice que use la variable buildConfiguration (que definimos como 'Release') para compilar el proyecto en modo Release.
En resumen: Este paso transforma tu código fuente en una aplicación funcional. Si hay errores de sintaxis o de compilación, este paso fallará.
`
Esta sección está completamente comentada (#) lo que significa que no se ejecuta actualmente.
Si tuvieras un proyecto de pruebas (por ejemplo, MiWebAppKarlasStuffs.Tests.csproj), aquí es donde lo configurarías.
command: 'test': Ejecutaría el comando dotnet test para buscar y ejecutar tus pruebas unitarias, de integración, etc.
projects: '$(projectPath)MiWebAppKarlasStuffs.Tests.csproj': Tendría que apuntar al .csproj de tu proyecto de pruebas.
En resumen: Este es el lugar para asegurar la calidad de tu código mediante la ejecución automática de pruebas. Si alguna prueba falla, la pipeline se detendrá y te notificará.
`
- task: DotNetCoreCLI@2: Otra vez, la tarea de línea de comandos de .NET Core.
displayName: 'Dotnet Publish': Nombre amigable para este paso.
inputs::
command: 'publish': Ejecuta el comando dotnet publish. Este comando toma la aplicación compilada y la empaqueta junto con todas sus dependencias en una carpeta lista para ser desplegada en un servidor.
projects: '$(projectPath)MiWebAppKarlasStuffs.csproj': Especifica qué proyecto debe ser "publicado".
publishWebProjects: false: Esta opción se usa para que la tarea no intente adivinar si un proyecto es web y lo publique por su cuenta. Como ya especificamos explícitamente el projects arriba, la ponemos en false para tener más control.
arguments: '--configuration $(buildConfiguration) --output $(Build.ArtifactStagingDirectory)': Pasa argumentos adicionales:
--configuration $(buildConfiguration): Publica en modo Release.
--output $(Build.ArtifactStagingDirectory): Le dice dónde guardar los archivos publicados. $(Build.ArtifactStagingDirectory) es una variable predefinida de Azure DevOps que apunta a una carpeta temporal en el agente donde se deben colocar los artefactos antes de ser subidos.
zipAfterPublish: true: Después de que dotnet publish genere todos los archivos en la carpeta de salida, esta opción le dice a la tarea que comprima todos esos archivos en un único archivo .zip. Esto es muy útil porque facilita el almacenamiento y el despliegue posterior del artefacto.
En resumen: Este paso prepara tu aplicación para ser desplegada, creando un paquete compacto y listo para usar.
`
- task: PublishBuildArtifacts@1: Esta es una tarea estándar de Azure DevOps para tomar archivos de una ubicación en el agente y publicarlos como un "artefacto" asociado a la ejecución de la pipeline.
displayName: 'Upload Build Artifact': Nombre amigable.
inputs::
PathtoPublish: '$(Build.ArtifactStagingDirectory)': Le dice a la tarea qué carpeta contiene los archivos que quieres subir como artefacto. Aquí, es la misma carpeta donde el paso anterior guardó el archivo .zip.
ArtifactName: 'drop': Le da un nombre al artefacto. "drop" es un nombre convencional y común. Cuando veas la ejecución de la pipeline en Azure DevOps, verás un artefacto llamado "drop" que puedes descargar.
publishLocation: 'Container': Indica que el artefacto debe publicarse en el almacenamiento interno de Azure DevOps (un "contenedor" dentro del contexto de la build).
En resumen: Este paso toma tu aplicación zip preparada y la guarda en Azure DevOps, de modo que puedas acceder a ella más tarde (por ejemplo, para descargarla o para que una pipeline de despliegue la use).
`






Explicacion de los pasos con ejemplo

4. steps: (Los Pasos a Seguir)
Esta es la lista de todas las acciones que tu "robot" (el agente de Azure DevOps) va a realizar en el orden que las escribas. Piensa en cada - task: como una instrucción específica para hacer algo.
Ejemplo cotidiano: Cuando preparas una taza de café, tus "steps" podrían ser:
Poner agua en la cafetera.
Poner café en el filtro.
Encender la cafetera.
Servir en una taza.
En tu pipeline, cada - task: es un "mini-programa" predefinido de Azure DevOps que sabe cómo hacer algo específico, como trabajar con código .NET.
Paso 1: Dotnet Restore (¡Consigue los ingredientes!)
`
task: DotNetCoreCLI@2
displayName: 'Dotnet Restore'
inputs:
command: 'restore'
projects: '$(projectPath)MiWebAppKarlasStuffs.csproj'
`
Explicación sencilla: Piensa que tu proyecto .NET necesita muchas "librerías" o "paquetes" externos para funcionar. Son como los ingredientes que no fabricas tú, sino que compras en el supermercado (por ejemplo, el azúcar, la leche).
command: 'restore': Le estás diciendo al "robot": "¡Ve y busca todos los ingredientes que mi proyecto necesita!" El robot mirará la lista de tu proyecto (.csproj) y descargará todo lo necesario de internet (de NuGet, que es como el "supermercado de paquetes" de .NET).
projects: '$(projectPath)MiWebAppKarlasStuffs.csproj': Le dices dónde buscar la lista de ingredientes, en qué archivo de proyecto.
Ejemplo: Imagina que quieres hacer una tarta y necesitas harina, huevos, azúcar. Este paso es como ir a la tienda y asegurarte de tener todos esos elementos antes de empezar a mezclar. Si te falta algo, la tarta no saldrá.
Paso 2: Dotnet Build (¡Mezcla y hornea!)
`
task: DotNetCoreCLI@2
displayName: 'Dotnet Build'
inputs:
command: 'build'
projects: '$(projectPath)MiWebAppKarlasStuffs.csproj'
arguments: '--configuration $(buildConfiguration)'
`
Explicación sencilla: Una vez que tienes todos los ingredientes, el siguiente paso es usarlos para crear tu aplicación. "Build" (construir) significa tomar todo tu código (lo que tú escribiste) y transformarlo en un programa que una computadora pueda entender y ejecutar. Es como cuando mezclas los ingredientes y horneas la tarta.
command: 'build': Le dices al "robot": "¡Compila mi código y haz un programa con él!"
arguments: '--configuration $(buildConfiguration)': Esto es como decir: "Haz esta tarta en modo 'fiesta' (Release), no en modo 'prueba' (Debug)". El modo "Release" suele hacer que el programa sea más pequeño y rápido.
Ejemplo: Con la harina, huevos y azúcar, este paso sería mezclarlos, amasarlos y luego meter la masa en el horno. Si hay un error en tu código (por ejemplo, olvidaste un punto y coma), es como si la tarta no subiera o se quemara. ¡El "build" fallaría!
Paso 3: (Tareas de Prueba) (¡Prueba un pedacito!)
`
- task: DotNetCoreCLI@2
displayName: 'Dotnet Test'
inputs:
command: 'test'
projects: '$(projectPath)MiWebAppKarlasStuffs.Tests.csproj'
`
Explicación sencilla: Esta sección está comentada (#), lo que significa que ahora mismo tu "robot" se la salta. Pero si estuviera activa, sería el momento de comprobar que tu aplicación funciona correctamente.
command: 'test': Aquí el "robot" ejecutaría tus "pruebas unitarias". Son como pequeños experimentos que tú escribes para asegurarte de que cada parte de tu código hace lo que se espera.
Ejemplo: Después de hornear la tarta, antes de servirla, ¡pruebas un trocito! Así te aseguras de que sabe bien, que no está cruda, etc. Si el trocito está malo, sabes que algo anda mal con la tarta completa.
Paso 4: Dotnet Publish (¡Empaqueta la tarta para llevar!)
`
task: DotNetCoreCLI@2
displayName: 'Dotnet Publish'
inputs:
command: 'publish'
projects: '$(projectPath)MiWebAppKarlasStuffs.csproj'
publishWebProjects: false
arguments: '--configuration $(buildConfiguration) --output $(Build.ArtifactStagingDirectory)'
zipAfterPublish: true
`
Explicación sencilla: Una vez que tu aplicación está construida y lista, necesitas prepararla para llevarla a donde va a funcionar de verdad (un servidor web, por ejemplo). "Publish" (publicar) significa juntar tu programa con todo lo que necesita para ejecutarse (librerías, archivos de configuración) en una carpeta organizada y compacta. Es como poner la tarta ya horneada en una caja bonita, lista para que alguien se la lleve.
command: 'publish': Le dices al "robot": "¡Prepara mi aplicación para desplegarla!"
--output $(Build.ArtifactStagingDirectory): Le dices dónde poner la caja de la tarta, en una carpeta temporal especial que Azure DevOps prepara.
zipAfterPublish: true: Le dices: "Cuando termines de poner todo en la caja, ¡comprime la caja en un archivo .zip para que sea más fácil de mover!"
Ejemplo: Tienes la tarta horneada. Este paso es ponerla con cuidado en una caja de cartón especial, asegurándote de que no le falte nada para que quien la reciba pueda disfrutarla directamente. Y luego, por comodidad, esa caja la metes en una bolsa zip para que ocupe menos y sea más fácil de transportar.
Paso 5: PublishBuildArtifacts (¡Pon la caja de la tarta en el camión de reparto!)
`
task: PublishBuildArtifacts@1
displayName: 'Upload Build Artifact'
inputs:
PathtoPublish: '$(Build.ArtifactStagingDirectory)'
ArtifactName: 'drop'
publishLocation: 'Container'
`
Explicación sencilla: Ya tienes tu aplicación empacada en un archivo .zip dentro de esa carpeta temporal. Ahora, este paso es crucial: guardarlo en un lugar seguro y accesible dentro de Azure DevOps, para que puedas usarlo después. Es como cuando la empresa de tartas pone la caja de la tarta en un camión de reparto para que llegue a su destino.
PathtoPublish: '$(Build.ArtifactStagingDirectory)': Le dices al "robot": "¡El archivo que quiero guardar está en esta carpeta temporal!" (donde el paso anterior lo dejó).
ArtifactName: 'drop': Le pones un nombre al archivo guardado, en este caso "drop". Es como etiquetar la caja de la tarta para que sea fácil identificarla.
publishLocation: 'Container': Le dices: "Guarda esto dentro del 'almacén' de Azure DevOps para esta ejecución de la pipeline".
Ejemplo: El camión de reparto (Azure DevOps) llega a tu casa, recoge la caja de la tarta (el .zip de tu aplicación) y la lleva a un almacén central. Desde ese almacén, tú o alguien más podrá recogerla más tarde (descargar el .zip) para llevarla al servidor final donde se ejecutará tu aplicación.











Me ayudaste a crear una pagina web y tengo duda de desde donde se ejecuta cuando alguien la abre, ayer yo cree un agente autoalojado en mi computadora desde mi cuenta de azure devops, explicame si tiene relacion
Es muy común confundir cómo se relaciona el agente autoalojado con la ejecución de tu página web. Vamos a aclararlo.
Tienes razón al sentir que hay una relación, pero es importante diferenciar entre dos cosas:
La ejecución de tu pipeline (que construye y empaqueta tu página web).
La ejecución de tu página web (cuando un usuario la visita en su navegador).
¿Desde dónde se ejecuta tu página web cuando alguien la abre?
Cuando alguien abre tu página web en su navegador, esta se ejecuta desde un servidor web que está accesible en internet las 24 horas del día, los 7 días de la semana.
Imagina que tu página web es un restaurante.
El servidor web es el edificio del restaurante, siempre abierto y listo para atender a los clientes.
Tu página web son los platos que se preparan y se sirven en ese restaurante.
Cuando un usuario teclea la dirección de tu página (URL) en su navegador, lo que ocurre es:
El navegador envía una solicitud a la dirección de ese servidor web.
El servidor web "sirve" los archivos de tu página (HTML, CSS, JavaScript, imágenes) al navegador del usuario.
El navegador del usuario interpreta esos archivos y los muestra en pantalla.
Este servidor web es, generalmente, un servicio en la nube (como Azure App Service, máquinas virtuales en Azure, o servicios de otros proveedores como AWS o Google Cloud) o un servidor físico en algún centro de datos.

¿Qué relación tiene tu agente autoalojado con esto?
Tu agente autoalojado que creaste en tu computadora es una pieza fundamental, ¡pero no para servir la página web a los usuarios! Su función es:
Ayudar en la construcción y empaquetado de tu página web, no en su ejecución final para los usuarios.
Vamos a ver los dos escenarios:
Agentes alojados en la nube (Hosted Agents): Cuando usas pool: vmImage: 'ubuntu-latest', tu pipeline se ejecuta en una máquina virtual que Azure DevOps te presta temporalmente. Esta máquina existe solo para construir y empaquetar tu código, y luego desaparece.
Relación con tu web: Es la "cocina" donde se prepara la tarta (tu web). Una vez hecha la tarta, la "cocina" se desmantela.
Agente autoalojado (Self-Hosted Agent) en tu PC: Si configuraste tu pipeline para usar tu propio agente (en lugar de ubuntu-latest), entonces tu computadora se convierte en esa "cocina". Cuando haces un git push a main, Azure DevOps envía las instrucciones de tu pipeline a tu PC, y es tu PC la que:
Descarga el código.
Restaura los paquetes (dotnet restore).
Compila el código (dotnet build).
Publica y empaqueta la aplicación (dotnet publish, creando el .zip).
Sube el paquete .zip a Azure DevOps como un artefacto.
Relación con tu web: Tu PC es la "cocina" donde se prepara la tarta. Sin embargo, nadie come la tarta directamente de tu cocina. La tarta terminada (el .zip que se sube como artefacto) aún necesita ser llevada al "restaurante" (el servidor web en la nube) para que los clientes la prueben.
En resumen:
Tu agente autoalojado (tu PC) se usa para el proceso de Continuous Integration (CI): tomar tu código, compilarlo, probarlo y empaquetarlo. Este proceso termina con un "artefacto" (tu .zip de la aplicación).
Para que tu página web esté disponible para los usuarios, necesitas un paso adicional llamado Continuous Deployment (CD): tomar ese artefacto (.zip) y desplegarlo en un servidor web (por ejemplo, Azure App Service). Es ese servidor web el que estará siempre encendido y será accesible a todo el mundo.
Si tu página web actualmente se puede abrir por otras personas, significa que ya está desplegada en algún servidor web en internet (quizás en un servicio como Azure App Service que configuraste en algún momento). Tu agente autoalojado, en sí mismo, no "sirve" la web a los usuarios.



