# Instalación y configuración de Rancher 2.7
## 1. Kubernetes.
En primer lugar se instaló un cluster RKE2, en las máquinas rancher-1, rancher-2 y rancher-3.
Este procedimiento se encuentra descrito en detalle en el siguiente enlace.
El archivo de configuración de los nodos se encuentra en el path:
```
/etc/rancher/rke2/config.yaml
```

El archivo kubeconfig está almacenado en:
```
/root/.kube/config/rke2.yaml
```
Se instaló también Kubectl y Helm como parte de las herramientas CLI necesarias.

## 2. Rancher server.
El servidor se desplegó usando un helm chart con TLS certificates auto-generados, para ello se
instaló el cert-manager, en un namespace con el mismo nombre en rancher-1.
La url de acceso es: 

https://10.217.192.sslip.io/dashboard

Las instrucciones detalladas de instalación están en este enlace: 

https://ranchermanager.docs.rancher.com/pages-for-subheaders/install-upgrade-on-a-kubernetes-cluster

## 3. Cluster UAT.
Se importó el cluster UAT en la consola de rancher utilizando el comando que genera el servidor
para desplegar un proyecto y un namespace bajo el nombre cattle-system:

``` 
azureuser@rancher-1:~> curl --insecure -sfL
https://10.217.192.189.sslip.io/v3/import/kp89gh7zcs76qz7l5d42rf6bcxwvxhz9f6k5vrmbn74mcx2
587fldg_c-m-l7bqqs7h.yaml | oc apply -f -
clusterrole.rbac.authorization.k8s.io/proxy-clusterrole-kubeapiserver created
clusterrolebinding.rbac.authorization.k8s.io/proxy-role-binding-kubernetes-master created
namespace/cattle-system created
serviceaccount/cattle created
clusterrolebinding.rbac.authorization.k8s.io/cattle-admin-binding created
secret/cattle-credentials-d173f3b created
clusterrole.rbac.authorization.k8s.io/cattle-admin created
deployment.apps/cattle-cluster-agent created
service/cattle-cluster-agent created
azureuser@rancher-1:~>
```

Hubo varios problemas durante el proceso, con los work-arounds descritos a continuación.

### 3.1 Error ‘failed to ensure secret’

El cluster quedaba en estado pending dentro del GUI de rancher con el siguiente error:

```
time="2023-03-21T12:31:53Z" level=fatal msg="looking up cattle-system/cattle ca/token:
failed to ensure secret for service account cattle-system/cattle: error ensuring secret
for service account [cattle-system:cattle]: timed out waiting for the condition"
```

Tanto el secret como el token estaban correctamente creados, pero por alguna razón, la service
account fallaba al localizarlos, para remediarlo se creó un dummy token con el nombre
rancher-token, el archivo se encuentra en el path `/root/rancher-token.yaml` de rancher-1, se
aplicó y se editó el service account cattle, sustituyendo los secrets que traia por defecto con el
dummy token. Con esto quedó solventado y una vez registró correctamente, se eliminó el token
que creamos y se devolvió el service account a su estado original.

### 3.2 Error ‘400 Bad Request: Request Header too large’
Tras solventar el problema del secret el cluster seguía sin levantar en el GUI de Rancher con un
error en el nginx ingress por un tamaño excesivo de la request header, hubo que editar la
configuración del servidor nginx para aumentar el tamaño máximo de las cabeceras, se escribió
un HelmChartConfig, el archivo se encuentra en el path:

``` 
/var/lib/rancher/rke2/server/manifests/rke2-ingress-nginx-config.yaml
```

Tras aplicar los cambios el cluster se registró correctamente y se encuentra disponible para
gestionar en la consola de Rancher.