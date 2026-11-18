Documentación de la Investigación - UT2.A4

Autor: Michael Steven Ospina Ocampo

1. Problema detectado
El responsable de seguridad detectó que el cortafuegos del servidor web estaba desactivado (`firewall=no` en la sección `[seguridad]` del archivo `servidor.conf`), dejando el servidor expuesto.

2. Investigación en el historial de Git
Para localizar el origen del fallo se utilizaron los siguientes comandos:
* `git log --oneline`: Para listar el historial de commits de forma resumida y localizar el mensaje sospechoso.
* `git show fd9afdf`: Para inspeccionar los cambios exactos introducidos en ese commit y confirmar la línea modificada (`-firewall=si` por `+firewall=no`).
* `git diff 36fa240 fd9afdf`: Para comparar el estado del archivo antes y después de la modificación.

Identificador del commit que introdujo el error: `fd9afdf` (Mensaje: "Actualizacion de seguridad").
Cómo se localizó: Revisando el listado de `git log --oneline` se identificó el commit referente a seguridad y mediante `git show` se verificó que fue el punto exacto donde se desactivó el firewall.

3. Recuperación de una versión anterior (Respuestas Sección 6)
Se utilizó el comando `git checkout 36fa240 -- servidor.conf` para traer la versión previa al fallo.

1. ¿Qué versión del archivo has recuperado?**
   Se ha recuperado la versión correspondiente al commit `36fa240` (*"Cambio de puerto"*), donde el puerto ya era el `8080` y el cortafuegos seguía activado (`firewall=si`).
2. ¿Se han perdido temporalmente las modificaciones posteriores?**
   Sí, en el directorio de trabajo y en el área de preparación (*staging*) desaparecieron temporalmente los cambios posteriores (como la nueva página principal y el límite de conexiones).
3. ¿Han desaparecido los commits posteriores del historial?**
   No, los commits posteriores siguen intactos en el historial del repositorio (`git log`), ya que `git checkout` sobre un archivo solo sobrescribe el archivo actual sin borrar commits.
4. ¿Por qué no conviene utilizar directamente esta versión antigua como solución definitiva?**
   Porque si hiciéramos commit de esa versión antigua tal cual, eliminaríamos configuraciones válidas y correcciones posteriores que se realizaron en los siguientes commits.

4. Cambios que debían conservarse y Solución definitiva
Cambios válidos conservados:
Puerto `8080`.
La configuración correcta de la página principal y su posterior corrección.
Límite de conexiones (`max_conexiones=100`).
Solución definitiva aplicada:
  Se restauró la versión más reciente del archivo, se ajustó la línea del cortafuegos a `firewall=si` para reactivarlo manteniendo intactos todos los demás parámetros válidos, y se registró el commit correspondiente a la corrección de seguridad.

5. Utilidad de Git para administrar archivos de configuración
Git permite llevar una auditoría completa de qué cambios se hacen en un servidor, cuándo se realizaron y por qué. Ante una caída del servicio o brecha de seguridad, facilita comparar versiones con `diff`/`show` y revertir errores puntuales en segundos sin perder el resto de configuraciones válidas del sistema.