# FaaSt ohne Server - Serverless 101

## Cluster erstellen
```console
k3d cluster create cl2024
``` 
Prüfen, ob Installation erfolgreich war und kubeconfig richtig gesetzt ist:
```console
kubectl cluster-info
```
## OpenFaas-Plattform im Cluster installieren
Namespaces erzeugen
```console
kubectl apply -f https://raw.githubusercontent.com/openfaas/faas-netes/master/namespaces.yml
```
OpenFaas Helm-Repository hinzufügen
```console
helm repo add openfaas https://openfaas.github.io/faas-netes/
```

OpenFaas via Helm installieren
```console
helm repo update \
 && helm upgrade openfaas \
  --install openfaas/openfaas \
  --namespace openfaas
```

Installierte Komponenten anzeigen
```console
kubectl get pods -n openfaas
```

Portforwarding für OpenFaas-Gateway:
```console
kubectl port-forward -n openfaas svc/gateway 8080:8080
```

## OpenFaas Login
OpenFaaS-Admin-Passwort abfragen
```console
PASSWORD=$(kubectl -n openfaas get secret basic-auth -o jsonpath="{.data.basic-auth-password}" | base64 --decode)
```
FaaS-URL als env-Variable setzen:
```console
export OPENFAAS_URL=http://127.0.0.1:8080
```
Login
```console
echo -n $PASSWORD | faas-cli login -g $OPENFAAS_URL -u admin --password-stdin
```

## Demo-Funktion erstellen und deployen
Verfügbare Language-Templates anzeigen
```console
faas-cli template pull
faas-cli new --list
```
Neue Funktion "cl24-sample" mit Node18 erstellen:
```console
faas-cli new --lang node18 cl24-sample
```

Zuerst in der stack-Yaml eigene Registry eintragen (Hinweis: Community Edition unterstützt nur Public-Registries):

cl24-sample.yml
```
version: 1.0
provider:
  name: openfaas
  gateway: http://127.0.0.1:8080
functions:
  cl24-sample:
    lang: node18
    handler: ./cl24-sample
    image: <REGISTRY-HERE>/cl24-sample:latest
```

Funktion bauen
```console
faas-cli build -f cl24-sample.yml
```

Funktion zu Docker-Hub pushen :
```console
docker login -u <user> -p <password oder accesstoken>
```
```console
faas-cli push -f cl24-sample.yml
```

Funktion lokal deployen
```console
faas-cli deploy -f cl24-sample.yml
```

Deployte Funktionen auflisten:
```console
faas-cli list
```

## Funktion aufrufen
via curl
```console
curl http://127.0.0.1:8080/function/cl24-sample -d "Hello Cloudland!"
```
via faas-cli
```console
echo "Hello Cloudland via faas-cli!" | faas-cli invoke  cl24-sample
```
via Dashboard
```console
http://127.0.0.1:8080/ui/
```

Login:
> user: admin
> 
> password auslesen via
```console
echo $PASSWORD
```

## Funktion wieder entfernen
```console
faas-cli remove -f cl24-sample.yml
```

