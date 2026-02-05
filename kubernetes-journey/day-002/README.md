# Día 02 - Crear un Deployment en Kubernetes

## Problema / Desafío

Crear un Deployment llamado `httpd` usando la imagen `httpd:latest`.

## Solución

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: httpd
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: httpd
  template:
    metadata:
      labels:
        app: httpd
    spec:
      containers:
        - name: httpd
          image: httpd:latest
```

### Desglose del manifiesto

**`apiVersion: apps/v1`**: Los Deployments pertenecen al grupo de API `apps`, a diferencia de los Pods que usan `v1` directamente. Esto es porque los Deployments son recursos de mayor nivel que gestionan ReplicaSets y Pods de forma declarativa.

**`kind: Deployment`**: Define el tipo de recurso. Un Deployment a diferencia de un Pod, no crea un contenedor directamente. En su lugar, crea un ReplicaSet que a su vez crea y gestiona los Pods. La cadena es: **Deployment → ReplicaSet → Pod(s)**.

**`spec.replicas`**: Número de réplicas del Pod que el Deployment debe mantener corriendo. Si un Pod muere, el Deployment lo recrea automáticamente para mantener el estado deseado.

**`spec.selector.matchLabels`**: Define qué Pods son gestionados por este Deployment. El selector **debe coincidir** con los labels definidos en `template.metadata.labels`. Si no coinciden, Kubernetes rechaza el manifiesto.

**`spec.template`**: Es la plantilla del Pod que el Deployment usará para crear réplicas. Tiene la misma estructura que un Pod (`metadata` y `spec`), pero se define dentro del Deployment.

### Diferencia entre Pod y Deployment

| Característica | Pod | Deployment |
|---|---|---|
| API version | `v1` | `apps/v1` |
| Auto-recuperación | No. Si el Pod muere, no se recrea | Sí. Mantiene el número de réplicas deseadas |
| Escalado | No soporta réplicas | Soporta `replicas` para escalar horizontalmente |
| Rolling updates | No | Sí. Permite actualizar la imagen sin downtime |
| Rollback | No | Sí. Permite regresar a una versión anterior |

### Aplicar y verificar

```bash
kubectl apply -f deployment.yaml

# Ver el deployment
kubectl get deployment httpd

# Ver el ReplicaSet creado por el Deployment
kubectl get replicaset

# Ver los pods creados
kubectl get pods

# Ver detalles completos del Deployment
kubectl describe deployment httpd
```

### Comandos imperativos equivalentes

```bash
# Crear el deployment sin manifiesto
kubectl create deployment httpd --image=httpd:latest
```

## Troubleshooting

| Problema | Solución |
|----------|----------|
| Deployment en estado `0/1 available` | Revisar eventos con `kubectl describe deployment httpd` y el estado de los pods con `kubectl get pods` |
| Error `selector does not match template labels` | Verificar que `spec.selector.matchLabels` coincida exactamente con `spec.template.metadata.labels` |
| Pod en estado `ImagePullBackOff` | Verificar que la imagen `httpd:latest` existe y que el nodo tiene acceso al registry |

## Recursos

- [Documentación oficial de Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
