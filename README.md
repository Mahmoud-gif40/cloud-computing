Mahmoud Haskour

# Storingsmelder

Dit is een klein systeem om storingen te melden. Je vult een formulier in, je ziet de open meldingen in een lijst, en je sluit ze af als ze opgelost zijn.

Ik heb de applicatie in containers gezet, samen met een PostgreSQL database. Alles start met een enkel commando en de meldingen blijven bewaard.

## Wat er in de repo staat

```
app.py              de applicatie
requirements.txt    de pakketten die ze nodig heeft
templates/          de HTML van de pagina
Dockerfile          bouwt het image van de applicatie
compose.yaml        start de applicatie en de database samen
.env.example        de namen van de instellingen, met lege waardes
.gitignore          houdt .env uit Git
```

## Wat je nodig hebt

Docker Desktop, en het moet draaien. Git.

## Starten vanaf nul

Clone de repo en ga naar de map.

```
git clone https://github.com/Mahmoud-gif40/cloud-computing.git
cd cloud-computing
```

Maak een eigen .env bestand van het voorbeeld.

```
copy .env.example .env
```

Open .env en vul de drie waardes in. Dat zijn POSTGRES_USER, POSTGRES_PASSWORD en POSTGRES_DB. Kies zelf een wachtwoord. Bijvoorbeeld storing als gebruiker en storingen als databasenaam. Sla het bestand op. Blijft een waarde leeg, dan start de database niet.

Start alles.

```
docker compose up -d --build
docker compose ps
```

Bij de database moet healthy staan. Dat duurt een paar seconden. Open daarna http://localhost:8000 in de browser en meld een storing.

## Controleren

De applicatie heeft een adres dat zegt of alles in orde is.

```
curl.exe http://localhost:8000/health
```

Draait alles, dan krijg je status ok en code 200. Kan de applicatie de database niet bereiken, dan krijg je code 503.

## Stoppen

```
docker compose down
```

Dit verwijdert de containers en het netwerk. De volumes blijven staan, dus de meldingen blijven bewaard. Na een nieuwe start staan ze er nog.

Wil je ook de data wissen, gebruik dan dit.

```
docker compose down -v
```

Dan zijn de meldingen weg.

## Instellingen

Alles wordt geregeld met environment variables. Er staat niets vast in de code.

DATABASE_URL. Is die leeg, dan slaat de applicatie op in een SQLite bestand. Staat er een PostgreSQL adres in, dan gebruikt ze die database. In dit project zet compose.yaml die zelf, op basis van de waardes uit .env.

SQLITE_PAD. De plek van het SQLite bestand. Standaard is dat /data/storingen.db.

APP_TITEL. De titel boven aan de pagina. Standaard is dat Storingsmelder.

PORT. De poort van de applicatie. Standaard is dat 8000.

## Het image

Het image staat publiek op ghcr.io, met een versienummer.

```
docker pull ghcr.io/mahmoud-gif40/storingsmelder:1.0.0
```

## Wat ik gedaan heb

De applicatie draait in een eigen container, gebouwd uit de Dockerfile. De database draait in een tweede container met postgres 18.

Compose maakt zelf een netwerk. Daarom vindt de applicatie de database met de naam db. Met localhost werkt het niet, want in een container is localhost de container zelf. Dan krijg je connection refused. De poort van de database is niet gepubliceerd. Alleen de applicatie moet erbij.

De data staat in twee named volumes, een voor de applicatie en een voor de database. Daardoor blijven de meldingen bewaard als de containers weggaan. Bij postgres 18 hangt het volume aan /var/lib/postgresql, want de data staat daar in een map met het versienummer.

De database start niet meteen klaar. Daarom heeft ze een healthcheck met pg_isready. De applicatie start pas als de database healthy is. Dat doe ik met depends_on en service_healthy. depends_on alleen wacht niet tot de database klaar is. Daarnaast heeft de applicatie restart unless-stopped, voor als ze later om een andere reden crasht.

Het wachtwoord staat in .env en niet in de repo. In .env.example staan dezelfde namen met lege waardes, zodat je weet wat je moet invullen.
dus in kort 
Ik heb de applicatie in een container gezet met de Dockerfile.

Ik heb een tweede container gemaakt met postgres 18 voor de database.

Ik heb de applicatie naar de database laten wijzen met de naam db. Met localhost werkt het niet, want dan zoekt ze de database in haar eigen container.

Ik heb de poort van de database niet gepubliceerd. Alleen de applicatie moet erbij.

Ik heb twee volumes gemaakt, een voor de applicatie en een voor de database. Daardoor blijven de meldingen bewaard. Bij postgres 18 hangt het volume aan /var/lib/postgresql.

Ik heb een healthcheck op de database gezet. De applicatie start pas als de database klaar is.

Ik heb restart unless-stopped op de applicatie gezet, voor als ze later crasht.

Ik heb het wachtwoord in .env gezet. Dat bestand staat niet in de repo. In .env.example staan alleen de namen.