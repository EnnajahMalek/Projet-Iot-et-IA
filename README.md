# Projet IoT et IA - Maison intelligente

Système de maison intelligente qui collecte les données de capteurs (température, humidité, gaz, mouvement, portes), les analyse à l'aide de modèles d'apprentissage automatique pour détecter des anomalies (incendie, comportement anormal des portes) et les expose à travers une application mobile avec notifications en temps réel.

## Table des matières

- [Architecture](#architecture)
- [Structure du dépôt](#structure-du-dépôt)
- [Prérequis](#prérequis)
- [Configuration](#configuration)
- [Installation et lancement](#installation-et-lancement)
- [API REST (Spring Boot)](#api-rest-spring-boot)
- [Modules d'intelligence artificielle](#modules-dintelligence-artificielle)
- [Technologies utilisées](#technologies-utilisées)
- [Auteur](#auteur)

## Architecture

```
Capteurs simulés (Python)
        |  MQTT (Mosquitto, topic home/+)
        v
Passerelle (mqtt_listener) ---> MongoDB
                                   |
                  +----------------+----------------+
                  v                                 v
       Backend IA (Flask)                 Backend API (Spring Boot)
  surveillance de la base, modèles ML,    authentification JWT, contrôle des
  alertes par pièce                       équipements, notifications Firebase
                  |                                 ^
                  +---------- alertes ------------->|
                                                    |
                                       Application mobile (React Native / Expo)
```

## Structure du dépôt

La branche de travail est `vf`.

```
Projet-Iot-et-IA/
|-- Smart-Home-Sensors/      Couche capteurs
|   |-- sensors/             Capteurs simulés (température, humidité, gaz, virtuel, anomalies)
|   |-- mqtt/                Publication des mesures sur le broker MQTT
|   `-- gateway/             Écoute MQTT et insertion dans MongoDB
|-- Backend/
|   |-- Ai Endpoint/         Service Flask : traitement des données et modèles ML
|   `-- Api Endpoint/        Service Spring Boot : API REST, sécurité, notifications
`-- Frontend/                Application mobile React Native (Expo)
```

## Prérequis

- Git
- Python 3.13 (les fichiers compilés du dépôt ciblent cette version) et `pip`
- Java 21 et Maven (ou le wrapper `mvnw` fourni)
- Node.js et npm
- MongoDB (instance locale ou cluster Atlas)
- Broker MQTT Mosquitto
- Un projet Firebase pour les notifications

## Configuration

Les fichiers de configuration contenant des secrets ne doivent pas être publiés. Chaque composant doit être configuré avec ses propres valeurs.

**Backend Spring Boot** : `Backend/Api Endpoint/src/main/resources/application.properties`

```properties
spring.data.mongodb.uri=mongodb://localhost:27017/smarthome
spring.data.mongodb.database=smarthome
server.port=8080
jwt.secret=<à définir>
jwt.expiration=<durée en ms>
mongodb.sensors.uri=<URI de la base des capteurs>
device.user.id=<identifiant>
allowed.device.id=<identifiant>
device.secret=<à définir>
```

Placer également le fichier de compte de service Firebase dans `src/main/resources/` et renseigner `firebase-config.json`.

**Backend Flask** : `Backend/Ai Endpoint/.env`

```env
MONGO_URI=mongodb://localhost:27017/smarthome
```

**Capteurs** : `Smart-Home-Sensors/.env`

```env
MONGODB_URI=mongodb+srv://<utilisateur>:<mot_de_passe>@<hote>/
```

Les documents des pièces (climatisation, éclairage, portes, fenêtres) doivent être créés manuellement dans la collection prévue de MongoDB.

## Installation et lancement

```bash
git clone https://github.com/EnnajahMalek/Projet-Iot-et-IA.git
cd Projet-Iot-et-IA
git checkout vf
```

### 1. Capteurs et passerelle

```bash
cd Smart-Home-Sensors
python -m venv venv
source venv/bin/activate        # Windows : venv\Scripts\activate
pip install paho-mqtt pymongo python-dotenv

sudo systemctl enable mosquitto
sudo systemctl start mosquitto
```

Lancer ensuite la passerelle (`gateway/mqtt_listener.py`) puis les capteurs simulés (`sensors/`).

### 2. Backend IA (Flask, port 5000)

```bash
cd "Backend/Ai Endpoint"
pip install -r requirements.txt
python run.py
```

### 3. Backend API (Spring Boot, port 8080)

```bash
cd "Backend/Api Endpoint"
./mvnw spring-boot:run
```

### 4. Application mobile (Expo)

```bash
cd Frontend
npm install
npx expo start
```

Dans `Frontend/services/api.ts`, adapter l'adresse IP du backend à celle de la machine qui l'héberge.

## API REST (Spring Boot)

Les routes ci-dessous sont regroupées par préfixe.

| Préfixe | Rôle |
| --- | --- |
| `/api/auth` | Inscription (`/register`) et connexion (`/login`) avec jeton JWT |
| `/api/room` | État d'une pièce, climatisation, éclairage, portes et fenêtres, activation et désactivation des équipements, préférences, graphiques |
| `/api/temperature` | Données de température et préférences |
| `/api/humidity` | Données d'humidité |
| `/api/gaz` | Données et alertes de gaz |
| `/api/device` | Notifications (envoi, non lues, marquage comme lues) |

## Modules d'intelligence artificielle

Le service Flask surveille la base de données, traduit les documents entrants vers un format interne puis applique un traitement par pièce (chambre, cuisine, salon, toilettes).

- **Détection d'incendie** (`ml/FireDetection`) : génération de données, entraînement et détecteur sérialisé avec scikit-learn.
- **Surveillance des portes** (`ml/doorMonitoring`) : détection d'anomalies sur l'utilisation des portes.
- **Services** : génération d'alertes, détection de mouvement, gestion des préférences de température, envoi des alertes vers l'API Spring Boot.

## Technologies utilisées

| Couche | Technologies |
| --- | --- |
| Capteurs | Python, paho-mqtt, Mosquitto |
| Base de données | MongoDB |
| Backend IA | Flask, scikit-learn, pandas, numpy, joblib |
| Backend API | Spring Boot (Java 21), Spring Security, JWT, Spring Data MongoDB |
| Notifications | Firebase |
| Application mobile | React Native, Expo, Expo Router, react-native-chart-kit |

## Auteurs

Malek Ennajah, Doaue Sbayi, Nissrine EL Asri , Yahia Morabet, Ilyas Elhoudaigui , Youssef Bouhdyd
