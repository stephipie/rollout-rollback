# Kubernetes Rollout und Rollback Demonstration

## Aufgabe und Implementation

Dieses Projekt demonstriert die Durchführung von Rolling Updates und Rollbacks in Kubernetes mit einem NGINX-Webserver. Es zeigt die Funktionsweise von Deployments, Services und Pod-Management in Kubernetes.

### Stack-Komponenten
- NGINX Webserver (Blau/Grün Versionen)
- Kubernetes Deployment
- Kubernetes Service (NodePort)

### Deployment-Prozess

1. Initiales Deployment (Version 1.0.0):
```bash
kubectl apply -f kubernetes/nginx-deployment.yaml
kubectl apply -f kubernetes/nginx-service.yaml
```

2. Version A: Rolling Update zu Version 2.0.0:
```bash
# Update image in deployment
kubectl set image deployment/my-nginx-deployment my-nginx=stephipie/my-nginx:2.0.0
```
oder

2. Version B: Innerhalb der nginx-deployment.yaml das Image manuell abändern:
```bash
# Update image within deployment.yaml
containers:
      - name: my-nginx
        image: stephipie/my-nginx:2.0.0
# geänderte Deployment-Definition anwenden
kubectl apply -f nginx-deployment.yaml
```     
3. Rollout Prozess beobachten:
```bash
kubectl rollout status deployment/my-nginx-deployment
kubectl get pods -l app=my-nginx -w
```
Wenn der Rollout abgeschlossen ist (Status successfully rolled out), rufe die Adresse im
Browser neu auf (http://<Node-IP>:<NodePort>). Du solltest nun die Nginx-Seite deiner
Version 2 sehen.

4. Rollback zu Version 1.0.0:
```bash
kubectl rollout undo deployment/my-nginx-deployment
```

5. Aufräumen
```bash
kubectl delete -f nginx-deployment.yaml,nginx-service.yaml
#Überprüfung ob alle Ressourcen entfernt wurden
kubectl get deployment,service,pods -l app=my-nginx
```
Stoppe dein lokales Cluster (optional, aber empfohlen): minikube stop oder Docker Desktop
K8s deaktivieren.

## Deployment-Beweise

### Version 1.0.0 (Initial)
[Hier Screenshot von Version 1.0.0 einfügen](/screenshots/Version1.0.0.png)

### Version 2.0.0 (Nach Rolling Update)
[Hier Screenshot von Version 2.0.0 einfügen](/screenshots/Version2.0.0.png)
[Hier Screenshot vom Rollout](/screenshots/rolledOut.png)

### Version 1.0.0 (Nach Rollback)
[Hier Screenshot vom Rollback einfügen](/screenshots/rolledBack.png)
[Hier Screenshot von Version 1.0.0 nach Rollback einfügen](/screenshots/Version1.0.0nachRollback.png)

### Kubernetes Ressourcen Status
```bash
kubectl get deployment,service,pods -l app=my-nginx
```
[Hier Output einfügen](/screenshots/RessourcenStatus.png)

### Kuberntes Ressourcen Status nach dem Clean-Up
```bash
kubectl get deployment,service,pods -l app=my-nginx
```
[Hier Output einfügen](/screenshots/StatusCleanup.png)


## Reflexion (Kurzfassung)

### 🔬 Warum ist ein Deployment in Kubernetes nicht einfach nur eine etwas andere Version von docker run mit --restart=always?
Ein Deployment ist ein intelligenter Controller, der den gewünschten Zustand überwacht, Updates und Rollbacks steuert und Skalierung ermöglicht. Es ist viel mehr als ein simpler Neustart-Mechanismus – es ist ein Orchestrator für ganze Anwendungen.

### 👨‍🔧 Was passiert, wenn ein Pod verschwindet? Ist das Magie?
Nein, das ist keine Magie! Das Deployment merkt sofort, wenn ein Pod fehlt, und startet automatisch einen neuen. Es sorgt immer dafür, dass die gewünschte Anzahl an Pods läuft – wie ein Gärtner, der fehlende Pflanzen sofort ersetzt.

### 🎬 Was passiert beim Rolling Update? Wie wird ein Ausfall verhindert?
Action! Neue Pods werden schrittweise gestartet, während alte noch laufen. Erst wenn die neuen bereit sind, werden die alten beendet. So bleibt der Service immer erreichbar – kein kompletter Ausfall, sondern ein fließender Wechsel.

### 👽 Wie findet der Service immer den richtigen Pod (NodePort & Update)?
Stell dir vor, jeder Pod sendet ein Label-Signal. Der Service sucht immer nach Pods mit dem passenden Label (z.B. `app: my-nginx`). Egal wie viele sich austauschen – der Service leitet den Traffic immer an die richtigen, aktuellen Pods weiter.

### 📄 Welche Angaben in der Deployment-YAML betreffen die Pod-Vorlage, welche das Deployment-Verhalten?
- 🎯 `spec.template`: Pod-Vorlage (Container, Image, Ports)
- 🔄 `spec.replicas`, `spec.strategy`: Verhalten des Deployments (Skalierung, Update-Strategie)
- 📦 `metadata.labels`, `spec.selector`: Zuordnung von Pods zum Deployment

### Was passiert mit den Pods, wenn das Deployment gelöscht wird?
Alle zugehörigen Pods werden automatisch entfernt. Das ist logisch, weil das Deployment der "Besitzer" der Pods ist – verschwindet der Besitzer, verschwinden auch seine Ressourcen. So bleibt der Cluster sauber.
