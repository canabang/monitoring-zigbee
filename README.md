# Surveillance des Batteries Zigbee (Zigbee2MQTT)

Ce projet permet de surveiller l'état de santé de tous vos appareils Zigbee sur batterie. Il croise les données de **Zigbee2MQTT** (pour les métadonnées comme les dates de changement de pile) avec les états de **Home Assistant** (pour le niveau de pile et la disponibilité).

## 📂 Structure du Projet
Les fichiers sont situés dans le dossier `/mnt/Data/Github/unvalaible-device/`.

- `zigbee_sensors.yaml` : Contient les capteurs template.
- `README.md` : Ce fichier de documentation.

## 🛠️ Fonctionnement Technique

### 1. Le Capteur Maître (`sensor.z2m_battery_devices`)
Ce capteur est **déclenché par MQTT**. Il ne se met à jour que lorsque le bridge Zigbee2MQTT publie la liste de ses appareils (`zigbee2mqtt02/bridge/devices`).

- **État** : Nombre total d'appareils sur batterie détectés.
- **Attributs** : Une liste `devices` contenant pour chaque appareil :
    - `name` : Friendly name Z2M.
    - `status` : `online` ou `offline` (basé sur l'entité de batterie HA).
    - `battery` : Niveau en % (récupéré de HA).
    - `maintenance` : Date extraite de la description Z2M (formant "pile JJ/MM/AAAA").
    - `entity_debug` : L'ID de l'entité Home Assistant liée (pour vérification).

### 2. Le Capteur d'Alertes (`sensor.zigbee_battery_alerts`)
Ce capteur filtre la liste du capteur maître pour ne sortir que les appareils nécessitant une intervention humaine.

**Critères d'alerte :**
- Appareil marqué `offline`.
- Niveau de batterie `< 15%`.
- Niveau de batterie inconnu (`?`).

## 📋 Comment tenir à jour les dates ?
Pour que la date de changement de pile s'affiche :
1. Allez dans l'interface **Zigbee2MQTT**.
2. Cliquez sur un appareil > **Settings** (Paramètres).
3. Dans le champ **Description**, écrivez par exemple : `pile 02/02/2026`.
4. Le capteur se mettra à jour automatiquement à la prochaine publication du bridge.

## 🚀 Prochaines Étapes
- [ ] Créer une automatisation déclenchée par `sensor.zigbee_battery_alerts` pour envoyer une notification via K-2SO.
- [ ] Ajouter une carte sur le Dashboard pour visualiser la liste `alert_devices`.
