# FLEET Manual de usuario
## ¿Qué es Fleet?
Fleet es la herramienta de CD integrada en Rancher,
que nos permite añadir y gestionar repositorios de
código a los clusters manejados por la herramienta.

Para acceder, tenemos que abrir el menú desplegable
en la esquina superior izquierda del interfaz de
Rancher y seleccionar ‘Continuous Delivery’,
una vez dentro, se nos presentan una serie de
opciones donde podemos visualizar los clusters,
repositorios y clustergroup que hay creados.

Añadir nuevos clusters es una gestión que se realiza
desde rancher, en este documento nos centraremos
en las otras dos opciones.

### Repositorios

Para añadir un nuevo repo solamente tenemos que pulsar el botón `Add Repository`
que aparece en la parte superior derecha dentro de la opción `Git Repos`:

![alt text][repos]

[repos]: https://github.com/cristian-vila/imgs/blob/main/2.png?raw=true "Git Repos"


Se nos presenta una pantalla de configuración que nos requiere, de manera
obligatoria, un nombre y un `branch name`, este último siendo crítico, ya que si hay
cualquier errata fleet fallará al sincronizar el contenido el repositorio y no completará
el despliegue. El resto de opciones son la URL del repositorio, las opciones de
seguridad/autentificación y `Paths`, esta es la opción que nos permite especificar,
para un repositorio concreto, los directorios en los que fleet debe buscar, por
defecto, si se deja en blanco, asume el directorio raíz.

![alt text][create]

[create]: https://github.com/cristian-vila/imgs/blob/main/3.png?raw=true "Create Repo"

La siguiente pantalla es el selector de targets, donde especificaremos los clusters
sobre los que se va a desplegar, pudiendo añadir de manera opcional labels,
annotations, service accounts y namespaces.

![alt text][create2]

[create2]: https://github.com/cristian-vila/imgs/blob/main/4.png?raw=true "Create Repo"

Para más información sobre el estado de clusters y repositorios:
https://fleet.rancher.io/cluster-bundles-state


### Fleet.yaml

Este archivo opcional puede ser incluido en el repositorio para cambiar y
personalizar el modo en que los recursos son desplegados, `fleet.yaml`
siempre debe estar presente en el directorio raíz, si un fleet encuentra un `fleet.yaml`
dentro de un subdirectorio, define automáticamente un nuevo bundle con los
contenidos de este.
A nivel interno, los bundles son unidades de orquestación de recursos totalmente
independientes, pueden estar compuestos de manifiestos, configuraciones, o
cualquier otro recurso, debemos pensar en los bundles como la unidad fundamental
de despliegue de fleet.
La referencia de opciones que podemos personalizar en este archivo se encuentra
en la documentación oficial:


https://fleet.rancher.io/ref-fleet-yaml
### Cluster groups

Por último, tenemos la opción de crear cluster groups, con labels, annotations y
rules, estas son las que nos permiten seleccionar qué clusters serán incluidos en el
grupo, en función de una serie de operadores lógicos

![alt text][cgroups]

[cgroups]: https://github.com/cristian-vila/imgs/blob/main/5.png?raw=true "Cluster Groups"

### Problemas conocidos y work-arounds.


- Error `nil pointer evaluating interface`:

Cuando en alguna de las templates se hace referencia a una variable no declarada en `values.yaml`, Helm devuelve un error al añadir el repositorio en Fleet.

Esto es debido a que no es posible usar el valor `default` en un árbol de valores que tiene más de un nivel de profundidad, ejemplo:

```
spec:
    type: {{ .Values.service.type | default "ClusterIP" }}
```

Devolvería error: 

`Error: template: helm-chart/templates/service.yaml:6:18: executing "helm-chart/templates/service.yaml" at <.Values.service.type>: nil pointer evaluating interface {}.type`

En casos como este sería necesario declara un valor por defecto en `values.yaml`, bastaría con añadir: 

```
service: {}
```

al archivo `values.yaml`










