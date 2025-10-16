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
