## Setup minikube

ref: [minikube setup](https://minikube.sigs.k8s.io/docs/start/macos/)

Ejecutar en consola los comandos:

```
brew install minikube
```

```
minikube start --vm-driver=hyperkit
```

Para chequear que este todo levantado:

```
kubectl get po -A
```

Deberian ver algo como:

```
NAMESPACE              NAME                                         READY   STATUS      RESTARTS   AGE
kube-system            coredns-6955765f44-2cbwr                     1/1     Running     0          11h
kube-system            coredns-6955765f44-kwxqt                     1/1     Running     0          11h
kube-system            etcd-minikube                                1/1     Running     0          11h
kube-system            kube-addon-manager-minikube                  1/1     Running     0          11h
kube-system            kube-apiserver-minikube                      1/1     Running     0          11h
kube-system            kube-controller-manager-minikube             1/1     Running     0          11h
kube-system            kube-proxy-p56dx                             1/1     Running     0          11h
kube-system            kube-scheduler-minikube                      1/1     Running     0          11h
kube-system            nginx-ingress-controller-6fc5bcc8c9-lb682    1/1     Running     0          9h
kube-system            storage-provisioner                          1/1     Running     0          11h
```

## Setup namespace

ref: [k8ns/create-new-namespaces](https://kubernetes.io/docs/tasks/administer-cluster/namespaces-walkthrough/#create-new-namespaces)

Para generar un namespace y asociarle un context, crear un archivo `namespace.json` (debería ser yaml, pero en el ejemplo esta asi) con el siguiente contenido:

```
{
  "apiVersion": "v1",
  "kind": "Namespace",
  "metadata": {
    "name": "development",
    "labels": {
      "name": "development"
    }
  }
}
```

Luego generar el namespace en minikube asi:

```
kubectl create -f namespace.json
```

Para verificar que se haya creado correctamente:

```
kubectl get namespaces --show-labels
```

Deberia devolver algo asi:

```
NAME          STATUS    AGE       LABELS
default       Active    32m       <none>
development   Active    29s       name=development
```

## Setup context

ref: [k8ns/create-context](https://kubernetes.io/docs/tasks/administer-cluster/namespaces-walkthrough/#create-pods-in-each-namespace)

Ejecutar:

```
kubectl config current-context
```

Deberia devolver algo como:

```
minikube
```

Luego para generar el contexto nuevo para el namespace que se creo previamente:

```
kubectl config set-context dev --namespace=development \
  --cluster=minikube \
  --user=minikube
```

Si ejecutan:

```
kubectl config view
```

Deberian ver dentro de los context el nuevo que crearon, tambien esto se puede ver desde el iconito de docker, en el menu de Kubernetes, deberia aparecer tambien.

Para empezar a trabajar en el context que crearon, ejecutar:

```
kubectl config use-context dev
```

A partir de ahora ya se puede hacer lo mismo que en el workshop, excepto que hay algunas cosas que modificar dado que no tenemos el LoadBalancer autogenerado por AWS.

## Setup LoadBalancer

ref: [minikube/LoadBalancer](https://minikube.sigs.k8s.io/docs/tasks/loadbalancer/)

Cuando actualicen el whoami service, si lo listan con `kubectl get services` van a ver que dice en el status `pending`, lo que hay que hacer es utilizar un comando de minikube que emula y crea un LoadBalancer.

Ejecutar:

```
minikube tunnel
```

Al rato, veran que deja de estar `pending` y empiezan a funcionar los requests.

## Setup Ingress Controller

ref: [k8ns/ingress-controller-setup](https://kubernetes.io/docs/tasks/access-application-cluster/ingress-minikube/#enable-the-ingress-controller)

Para crear el ingress controller, ejecutar:

```
minikube addons enable ingress
```

Para ver que se crea correctamente, ejecutar:

```
kubectl get pods -n kube-system
```

Luego crear el ingress igual que en el workshop, pero adaptandolo para el nginx creado:

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: whoami
  namespace: <tu namespace>
  labels:
    app: whoami
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$1
spec:
  rules:
  - host: whoami2.<tu namespace>.k8ns.com
    http:
      paths:
      - path: /
        backend:
          serviceName: whoami
          servicePort: http
```

Para poder accederlo via el hostname, modificar el archivo de hosts en /etc/hosts y agregarlo con la info que da ejecutar esto:

```
kubectl get ingress
```

Listo!
