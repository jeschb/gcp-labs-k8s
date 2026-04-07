# Laboratorio 015 — Kubernetes: Despliegue de aplicación frontend

**Curso:** GCP Fundamentos 2026 — FORMATIVA  
**Docente:** Ing. Jesús A. Chávez Becerra  
**Laboratorio N.°:** 015  
**Duración estimada:** 40 min

---

## Descripción

Este repositorio contiene los manifiestos YAML para desplegar la aplicación frontend **Nave Aurora** en un clúster Kubernetes local. Se practican los recursos fundamentales de Kubernetes: Namespace, ConfigMap, Secret, Deployment y Service.

La aplicación es una página HTML estática servida por **nginx-unprivileged**, con diseño espacial animado, accesible vía NodePort.

---

## Estructura del repositorio

```
gcp-labs-k8s/
└── manifests/
    ├── 00-namespace.yaml      # Namespace gcplab
    ├── 01-configmap.yaml      # Config de app + nginx.conf + index.html
    ├── 02-secret.yaml         # Credenciales dummy en Base64
    ├── 03-deployment.yaml     # 2 réplicas nginx con probes y resource limits
    └── 04-service.yaml        # NodePort 30080 → contenedor 8080
```

---

## Prerrequisitos

- Clúster Kubernetes local activo (minikube, kind, k3s, Docker Desktop, etc.)
- `kubectl` instalado y configurado apuntando al clúster local
- Acceso a internet para descargar la imagen `nginxinc/nginx-unprivileged:1.27-alpine`

Verifica que el clúster esté activo:

```bash
kubectl cluster-info
kubectl get nodes
```

---

## Despliegue — Orden de aplicación

Aplica los manifiestos **en orden numérico**. El Namespace debe existir antes que cualquier otro recurso.

### Opción A — Aplicar uno por uno (recomendado para el laboratorio)

```bash
# 1. Crear el Namespace
kubectl apply -f manifests/00-namespace.yaml

# 2. Crear el ConfigMap (nginx.conf + index.html + variables)
kubectl apply -f manifests/01-configmap.yaml

# 3. Crear el Secret (credenciales dummy)
kubectl apply -f manifests/02-secret.yaml

# 4. Crear el Deployment (2 réplicas nginx)
kubectl apply -f manifests/03-deployment.yaml

# 5. Crear el Service (NodePort 30080)
kubectl apply -f manifests/04-service.yaml
```

### Opción B — Aplicar todos a la vez

```bash
kubectl apply -f manifests/
```

> **Nota:** Kubectl aplica los archivos en orden alfabético cuando se apunta a un directorio. El prefijo numérico `00-`, `01-`, etc. garantiza el orden correcto.

---

## Verificación del despliegue

```bash
# Ver todos los recursos del namespace gcplab
kubectl get all -n gcplab

# Ver el estado de los pods (esperar STATUS = Running)
kubectl get pods -n gcplab -w

# Ver detalles del deployment
kubectl describe deployment nave-aurora -n gcplab

# Ver el ConfigMap
kubectl get configmap nave-aurora-config -n gcplab -o yaml

# Ver el Secret (valores en Base64)
kubectl get secret nave-aurora-secret -n gcplab -o yaml

# Ver el Service y el NodePort asignado
kubectl get service nave-aurora-svc -n gcplab
```

---

## Acceso a la aplicación

Una vez que los pods estén en estado `Running`, abre el navegador:

```
http://<IP-del-nodo>:30080
```

### Obtener la IP del nodo según el tipo de clúster

| Clúster        | Comando para obtener IP                        |
|----------------|------------------------------------------------|
| **minikube**   | `minikube ip`                                  |
| **kind**       | `kubectl get nodes -o wide`                    |
| **k3s**        | `kubectl get nodes -o wide`                    |
| **Docker Desktop** | `localhost` o `127.0.0.1`               |

**Endpoint de salud:**

```
http://<IP-del-nodo>:30080/health
```

Respuesta esperada:
```json
{"status":"ok","app":"nave-aurora","version":"v1.0"}
```

---

## Comandos útiles durante el laboratorio

```bash
# Ver logs de un pod
kubectl logs -l app=nave-aurora -n gcplab

# Entrar a un pod (shell interactivo)
kubectl exec -it <nombre-del-pod> -n gcplab -- sh

# Ver variables de entorno dentro del pod
kubectl exec -it <nombre-del-pod> -n gcplab -- env | grep APP

# Escalar el deployment a 3 réplicas
kubectl scale deployment nave-aurora --replicas=3 -n gcplab

# Ver el historial de rollouts
kubectl rollout history deployment/nave-aurora -n gcplab
```

---

## Limpieza de recursos

Para eliminar todos los recursos del laboratorio:

```bash
# Eliminar todos los recursos del namespace
kubectl delete -f manifests/

# O eliminar directamente el namespace (elimina todo lo que contiene)
kubectl delete namespace gcplab
```

---

## Sobre el Secret

Los valores del Secret están codificados en Base64 (**no cifrados**). Son valores dummy para propósitos educativos.

Para generar tus propios valores:

```bash
echo -n "mi-valor-secreto" | base64
```

Para decodificar un valor:

```bash
echo "dG9rX2djcF9mdW5kYW1lbnRvc19qZXNjaGJfMjAyNg==" | base64 --decode
```

> En producción se recomienda usar **GCP Secret Manager** integrado con **Workload Identity**.

---

## Créditos

| Campo       | Valor                                  |
|-------------|----------------------------------------|
| Docente     | Ing. Jesús A. Chávez Becerra           |
| Curso       | GCP Fundamentos 2026                   |
| Institución | FORMATIVA                              |
| Laboratorio | LAB-015                                |
| Edición     | Marzo/Abril 2026                       |
