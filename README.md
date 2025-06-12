# Contenedores-Practicas

Orden de ejecucion 

kubectl apply -f https://github.com/cafc79/Contenedores-Practicas/blob/loadBalancer/{file.yaml}

1. kApplication.yaml
2. kService.yaml
3. kLimitRange.yaml (opcional)
4. metrics-server.yaml
5. aggregated-metrics-reader.yaml
6. auth-delegator.yaml
7. metrics-server-deployment.yaml
8. load-test.yaml

# En una terminal
Obtener la IP externa del LoadBalancer:
kubectl get svc loginapi-service -w

Ejecutar prueba manual:
# En una terminal
kubectl get pods -w

Iniciar prueba de carga continua (en otra terminal):
# En otra terminal 
hey -z 10m -q 10 http://<load-balancer-ip>  
o  
export LB_IP=$(kubectl get svc loginapi-service -o jsonpath='{.status.loadBalancer.ingress[0].ip}')  
hey -z 10m -q 10 http://$LB_IP

# En tercera terminal (para eliminar un pod)
kubectl get pods  
kubectl delete pod <nombre-pod-1>  
kubectl get events --sort-by='.metadata.creationTimestamp' -w

Para el escenario de fallo total:  
kubectl scale deployment loginapi-deployment --replicas=0  
# Observar el comportamiento del LoadBalancer
El Load Balancer debería devolver errores 502/503

Para métricas más detalladas, considera agregar:  
Prometheus Operator:  
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts  
helm install prometheus prometheus-community/kube-prometheus-stack  
Grafana para visualización:  
kubectl port-forward svc/prometheus-grafana 3000:80


# Ejemplo de métricas esperadas
Escenario	Latencia promedio	Tasa de error	Tiempo de recuperación
Normal (2 pods)	50-100ms	0%	-
Durante failover	200-500ms	5-10%	2-10 segundos
Ambos pods caídos	N/A	100%	Hasta nuevo despliegue

Automatizar la prueba:
# Script de ejemplo
#!/bin/bash  
echo "Iniciando prueba..."  
hey -z 30s -q 5 http://$LB_IP &  
sleep 10  
kubectl delete pod $POD1  
sleep 20  
kubectl delete pod $POD2  

Considerar usar herramientas profesionales:  
Chaos Mesh  
Litmus Chaos  
Gremlin
