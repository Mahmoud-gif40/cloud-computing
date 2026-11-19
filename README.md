Mahmoud Haskour

# Storingsmelder

Dit is een klein systeem om storingen te melden. Je vult een formulier in, je ziet de open meldingen in een lijst, en je sluit ze af als ze opgelost zijn.

De applicatie draait op een Kubernetes cluster, samen met een PostgreSQL database. De cluster haalt het image op uit ghcr.io.

## Wat er in de repo staat

```
app.py                  de applicatie
requirements.txt        de pakketten die ze nodig heeft
templates/              de HTML van de pagina
Dockerfile              bouwt het image van de applicatie
compose.yaml            de opzet met Docker Compose uit opdracht 1
k8s/deployment.yaml     de applicatie, twee replicas
k8s/service.yaml        de Service van de applicatie
k8s/db-deployment.yaml  de database, een replica
k8s/db-service.yaml     de Service van de database
```

## Wat je nodig hebt

Een draaiende Kubernetes cluster en kubectl die naar die cluster wijst. Internet, want de cluster haalt het image op bij ghcr.io. Het image is publiek.

Ik heb dit getest op Docker Desktop met kubeadm.

## Starten op de cluster

Clone de repo en ga naar de map.

```
git clone https://github.com/Mahmoud-gif40/cloud-computing.git
cd cloud-computing
```

Maak een namespace en zet alles op.

```
kubectl create namespace proef
kubectl apply -f k8s/ -n proef
kubectl get pods -n proef
```

Wacht tot alle pods Running zijn. Dat duurt een minuut of twee, want de images moeten worden opgehaald. De applicatie kan een keer herstarten omdat de database nog opstart. Dat is normaal, Kubernetes start haar zelf opnieuw.

## Controleren

Kijk of de Services hun pods gevonden hebben. Bij elke Service moeten IP adressen staan.

```
kubectl get endpoints -n proef
```

Maak de applicatie bereikbaar.

```
kubectl port-forward service/storingsmelder 8000:80 -n proef
```

Open http://localhost:8000 in de browser en meld een storing. In een tweede venster kan je dit doen.

```
curl http://localhost:8000/health
```

Daar moet PostgreSQL bij opslag staan. Dan praat de applicatie met de database.

## Opruimen

```
kubectl delete namespace proef
```

## Wat ik gedaan heb

Het image van de applicatie staat publiek op ghcr.io, met het versienummer 1.0.0 en niet latest.

De applicatie is een Deployment met twee replicas. De Service storingsmelder luistert op poort 80 en stuurt door naar poort 8000 van de pods. Het label op de pods is gelijk aan de selector, zo vindt de Service haar pods.

De database is een aparte Deployment met een replica. Drie replicas zouden drie aparte databases geven, want elke pod schrijft naar zijn eigen opslag.

De Service van de database heet db. Die naam staat in DATABASE_URL, en niet localhost. In een pod is localhost de pod zelf. Kubernetes heeft een eigen DNS die de naam van de Service omzet naar het juiste adres. De Service is van het type ClusterIP, dus de database is niet van buitenaf bereikbaar. Alleen de applicatie komt erbij.

Het wachtwoord staat nu nog in het manifest. Dat los ik op in opdracht 3.

De database heeft nog geen volume. De meldingen verdwijnen als de database pod verdwijnt.