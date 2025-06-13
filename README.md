# Contenedores-Practicas

## Orden de ejecucion 

kubectl apply -f https://github.com/cafc79/Contenedores-Practicas/blob/loadBalancer/{file.yaml}

1. splunk-secret.yaml  
2. kApplication.yaml
3. kService.yaml
4. https://litmuschaos.github.io/litmus/litmus-operator-v1.13.8.yaml
5. chaos-experiment.yaml


# Prueba de Chaos Engineering con Load Balancer en Kubernetes

Este proyecto demuestra una prueba de Chaos Engineering para evaluar el comportamiento de un Load Balancer en Kubernetes cuando se eliminan pods.

## Prerrequisitos

- Cluster Kubernetes (EKS, GKE, AKS o Minikube)
- `kubectl` configurado
- CLI de Splunk (opcional para métricas avanzadas)
- `hey` para pruebas de carga (`go get -u github.com/rakyll/hey`)

#### En una terminal
Obtener la IP externa del LoadBalancer:
kubectl get svc loginapi-service -w

## Ejecutar prueba manual:
####  En una terminal
kubectl get pods -w

### Iniciar prueba de carga continua (en otra terminal):
Se utilizara HEY (https://github.com/rakyll/hey) para enviar una peticion de carga a una web app 
#### En otra terminal 
hey -z 10m -q 10 http://<load-balancer-ip>  
o  
export LB_IP=$(kubectl get svc loginapi-service -o jsonpath='{.status.loadBalancer.ingress[0].ip}')  
hey -z 10m -q 10 http://$LB_IP


Obtener la IP del Load Balancer:


export LB_IP=$(kubectl get svc webapp-service -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
Iniciar prueba de carga:


hey -z 5m -q 10 -o csv http://$LB_IP > load_test.csv &
Eliminar un pod manualmente:


kubectl delete pod $(kubectl get pods -l app=webapp -o jsonpath='{.items[0].metadata.name}')
Monitorizar en Splunk:

Buscar en el índice k8s_metrics los eventos de Kubernetes

Filtrar por deployment.environment="chaos-testing"

Prueba automatizada con Litmus
Ejecutar el experimento de caos:


kubectl create -f chaos-experiment.yaml
Monitorizar progreso:


kubectl get chaosengine -w
kubectl describe chaosresult pod-delete-chaos
Métricas Clave en Splunk
Tiempo de respuesta durante failover:


index="k8s_metrics" metric_name="http.response.time" | timechart avg(value)
Disponibilidad de pods:


index="k8s_metrics" metric_name="k8s.pod.phase" | stats count by phase
Errores durante la prueba:


index="k8s_metrics" (metric_name="http.request.error" OR metric_name="k8s.pod.notready") | stats count by metric_name
Limpieza
bash
kubectl delete -f deployment.yaml
kubectl delete -f service.yaml
kubectl delete -f chaos-experiment.yaml
kubectl delete -f splunk-secret.yaml
Análisis de Resultados



#### En tercera terminal (para eliminar un pod)
kubectl get pods  
kubectl delete pod <nombre-pod-1>  
kubectl get events --sort-by='.metadata.creationTimestamp' -w

## Para el escenario de fallo total:  
kubectl scale deployment loginapi-deployment --replicas=0  
### Observar el comportamiento del LoadBalancer
El Load Balancer debería devolver errores 502/503

## Para métricas más detalladas, considera agregar:  
### Prometheus Operator:  
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts  
helm install prometheus prometheus-community/kube-prometheus-stack  
### Grafana para visualización:  
kubectl port-forward svc/prometheus-grafana 3000:80

## Ejemplo de métricas esperadas
|Escenario   |Latencia promedio  | Tasa de error  | Tiempo de recuperación  |  
|---|---|---|---|
| Normal  (2 pods) | 50-100ms  | 0%  | -  |   
| Durante failover | 200-500ms  | 5-10%  | 2-10 segundos  |   
| Ambos pods caídos   | N/A  | 100%  | Hasta nuevo despliegue  |   

## Automatizar la prueba:
### Script de ejemplo
#!/bin/bash  
echo "Iniciando prueba..."  
hey -z 30s -q 5 http://$LB_IP &  
sleep 10  
kubectl delete pod $POD1  
sleep 20  
kubectl delete pod $POD2  

#### Considerar usar herramientas profesionales:  
Chaos Mesh  
Litmus Chaos  
Gremlin
