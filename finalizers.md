
# Procedimiento de eliminación de finalizers asociados a Rancher

Debido a conflictos generados con Openshift tras las instalacion de Rancher/Fleet, se desarolló un procedimiento para eliminar todos los finalizers en recursos asociados.


### Scripts

En la home folder de azureuser de rancher1 se encuentran las utilidades necesarias:

`/home/azureuser/rancher-cleanup`

los scripts son `cleanup.sh` y `verify.shp` se despliegan en el cluster objetivo con los .yaml que hay incluidos dentro de la carpeta `/deploy`.

El primero es más completo y pesado, revisa, tanto a nivel cluster como namespaces, todos los recursos asociados con Rancher además de la parte de monitorizacion, logging, gatekeeper, etc...

El segundo es un script de verificación, más ligero, orientado a tareas de mantenimiento, que sería mas optimo en caso de tener que desplegar un CronJob, se centra solamente en recursos que contienen `cattle.io` en el nombre.

### Despliegue y validación

Para ejecutar los Jobs asociados es necesario aplicar los .yamls dentro de la carpeta `/deploy`:

`oc apply -f /home/azureuser/rancher-cleanup/deploy/rancher-cleanup.yaml`

`oc apply -f /home/azureuser/rancher-cleanup/deploy/verify.yaml`

Esto genera un job dentro del namespace `kube-system`, una vez completado se puede verificar el proceso en los logs asociados a la pod correspondiente.

### Consideraciones

En caso de que se modificasen los scripts a posteriori, los cambios no tendrán efecto hasta que se actualize la imagen de Docker (docker build/docker push) que despliegan los archivos .yaml, línea 38.

La Dockerfile y una copia de los scripts se encuentran dentro de la carpeta `/package`. 









