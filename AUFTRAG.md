# Backend

1. Erstellen Sie aus dem Code ein Image für das Backend mit dem Namen counter-backend. Geben Sie dem Image eine aussagekräftige Version, z.B. v1, sodass der Tag lautet: counter-backend:v1.

```
docker build -t counter-backend:v1 .
```

2. Pushen Sie das erstellte Image auf die Ihre Github Container Registry.

```
docker push ghcr.io/jonathanxdr/counter-backend:v1
```

3. Erstellen Sie ein Deployment mit dem Label app=counter-backend welches das erstellte Image in einem Pod in Betrieb nimmt.

```
kubectl create deployment counter-backend --image=ghcr.io/jonathanxdr/counter-backend:v1
```

4. Das Deployment counter-backend darf dabei nicht mit Image-Tag latest laufen.

```
kubectl get deployment counter-backend -o yaml
```

5. Das Deployment muss die Konfiguration für die Datenbankverbindung beinhalten. Verwenden Sie dazu das bereits erstellte Secret von counter-database und eine neue ConfigMap für die Umgebungsvariable DB_HOST.

```
kubectl create configmap counter-backend-config --from-literal=DB_HOST=counter-database
```

6. Erstellen Sie einen Service mit dem Namen counter-backend, welcher mit dem Backend-Pod interagiert.

```
kubectl expose deployment counter-backend --port=8080 --target-port=8080 --name=counter-backend
```

7. Erstellen Sie eine Route für den Service counter-backend.

```
oc create route edge counter-backend --service=counter-backend --port=8080
```

# Frontend

1. Erstellen Sie aus dem Code ein Image für das Frontend mit dem Namen counter-frontend. Geben Sie dem Image eine aussagekräftige Version, z.B. v1, sodass der Tag lautet: counter-frontend:v1.

```
docker build -t counter-frontend:v1 .
```

2. Erstellen Sie ein weiteres Deployment im mit der Bezeichnung app=counter-frontend welches einen Pod in Betrieb hat. Als Image soll counter-frontend verwendet werden.

```
kubectl create deployment counter-frontend --image=counter-frontend:v1
```

3. Das Deployment muss die Konfiguration für die Backendverbindung beinhalten. Erstellen Sie dafür eine neue ConfigMap für den REACT_APP_BACKEND_URL und dem URL der Backends (siehe Route des Backends).

```
kubectl create configmap counter-frontend-config --from-literal=REACT_APP_BACKEND_URL=http://counter-backend-<project>.<cluster>.<domain>
```

4. Das Deployment counter-frontend darf dabei nicht mit Image-Tag latest laufen.

```
kubectl get deployment counter-frontend -o yaml
```

5. Erstellen Sie einen Service mit dem Namen counter-frontend.

```
kubectl expose deployment counter-frontend --port=80 --target-port=80 --name=counter-frontend
```

# HorizontalPodAutoscaling (HPA)

- Erstellen Sie mindestens ein HorizontalPodAutoscaling mit dem Namen counter-backend mit den Werten maxReplicas=2 und minReplicas=2.

```
kubectl autoscale deployment counter-backend --min=2 --max=2
```

- Erstellen Sie mindestens einen weiteren HPA mit dem Namen counter-frontend im Namespace mit den Werten maxReplicas=4 und minReplicas=2.

  ```
  kubectl autoscale deployment counter-frontend --min=2 --max=4
  ```

# HTTPS Redirects in den Routes

- Stellen Sie sicher, dass die Route des counter-backend von http auf https umleitet.

  ```
  oc edit route counter-backend
  ```

- Stellen Sie sicher, dass die Route des counter-frontend von http auf https umleitet.

  ```
  oc edit route counter-frontend
  ```

# Zusätzliche Dienste

- Erstellen Sie mindestens einen weiteres Deployment mit dem Namen counter-postgresadmin.

  ```
  kubectl create deployment counter-postgresadmin --image=postgresadmin/postgresadmin
  ```

- Verwenden Sie als Image dpage/pgadmin4.

  ```
  kubectl create deployment counter-postgresadmin --image=dpage/pgadmin4
  ```

# Abgabe

Exportieren Sie die Applikation, erstellen Sie ein ZIP der beiden Files und laden sie das Ergebnis hier hoch:

```
oc get all > overview.txt
oc get all -o yaml > project.yml
```

# Secret

## Counter Database

```
kubectl create secret generic counter-database \
    --from-literal=database-name=counter-database \
    --from-literal=database-user=counter-database \
    --from-literal=database-password='S!B\*d$zDsb='
```

## Counter Postgresadmin

```
kubectl create secret generic counter-postgresadmin \
    --from-literal=pgadmin-default-email=admin@admin.com \
    --from-literal=pgadmin-default-password=root \
```
