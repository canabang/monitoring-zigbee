# 🧪 Templates de Debug - Monitoring Zigbee

> **Usage** : Copier le contenu d'un bloc de code (en cliquant sur l'icône 📋) et le coller dans **Home Assistant > Outils de développement > Modèle**

---

## 1. Vérification Globale du Système

Aperçu rapide de l'état complet du monitoring.

```jinja2
{% set master = 'sensor.z2m_battery_devices' %}
{% set alerts_sensor = 'sensor.zigbee_battery_alerts' %}
{% set monitor = 'sensor.z2m_network_monitor' %}

## 🔍 Diagnostic Complet

### Capteurs
| Capteur | État |
|---------|------|
| {{ master }} | {{ states(master) }} appareils sur batterie |
| {{ alerts_sensor }} | {{ states(alerts_sensor) }} alertes |
| {{ monitor }} | {{ states(monitor) }} offline (réseau) |

### Attributs clés
- **raw_devices**: {{ 'OK (' ~ (state_attr(master, 'raw_devices') | length) ~ ' appareils)' if state_attr(master, 'raw_devices') else '❌ VIDE' }}
- **devices**: {{ 'OK (' ~ (state_attr(master, 'devices') | length) ~ ' batteries)' if state_attr(master, 'devices') else '❌ VIDE' }}
- **last_seen_registry**: {{ (state_attr(master, 'last_seen_registry') | length) ~ ' entrées' if state_attr(master, 'last_seen_registry') else '❌ VIDE' }}
- **last_check**: {{ state_attr(master, 'last_check') | as_timestamp | timestamp_custom('%d/%m %H:%M') if state_attr(master, 'last_check') else 'N/A' }}
```

---

## 2. Vérification de raw_devices (FIX V4)

Vérifie que l'inventaire est bien stocké et allégé.

```jinja2
{% set raw = state_attr('sensor.z2m_battery_devices', 'raw_devices') %}

## 📦 Raw Devices

- **Existe**: {{ raw is not none }}
- **Longueur**: {{ raw | length if raw else 'N/A' }}

{% if raw and raw | length > 0 %}
### 3 premiers appareils
{% for d in raw[:3] %}
- {{ d.friendly_name }} ({{ d.power_source | default('?') }})
{% endfor %}

### Champs présents (1er appareil)
{{ raw[0].keys() | list if raw else 'N/A' }}
{% endif %}
```

---

## 3. Vérification des Batteries (FIX V5, V6, V7)

Vérifie la détection des entités et valeurs de batterie.

```jinja2
{% set devices = state_attr('sensor.z2m_battery_devices', 'devices') %}

## 🔋 Devices Enrichis

{% if devices %}
| Appareil | Status | Batterie | Entity Debug |
|----------|--------|----------|--------------|
{% for d in devices[:10] %}
| {{ d.name }} | {{ '🟢' if d.status == 'online' else '🔴' }} | {{ d.battery }}% | {{ d.entity_debug }} |
{% endfor %}
{% if devices | length > 10 %}
| ... | ... | ... | ({{ devices | length - 10 }} de plus) |
{% endif %}
{% else %}
❌ Attribut devices vide ou inexistant
{% endif %}
```

---

## 4. Vérification du Moniteur Réseau

Vérifie le last_seen registry et la détection des appareils offline.

```jinja2
{% set master = 'sensor.z2m_battery_devices' %}
{% set monitor = 'sensor.z2m_network_monitor' %}
{% set registry = state_attr(master, 'last_seen_registry') | default({}) %}

## 📡 Moniteur Réseau

### Last Seen Registry ({{ registry | length }} entrées)
{% if registry | length > 0 %}
| Appareil | Dernier signal | Il y a |
|----------|----------------|--------|
{% for name, ts in registry.items() %}
{% set delta = ((as_timestamp(now()) - as_timestamp(ts)) / 3600) | round(1) %}
| {{ name }} | {{ ts | as_timestamp | timestamp_custom('%d/%m %H:%M') }} | {{ delta }}h {{ '🔴' if delta > 25 else '🟢' }} |
{% endfor %}
{% else %}
⚠️ Registry vide - Le trafic MQTT n'a pas encore été capturé
{% endif %}

### Appareils Offline
{% set offline = state_attr(monitor, 'offline_list') | default([]) %}
{% if offline | length > 0 %}
{% for d in offline %}
- **{{ d.name }}** : {{ d.hours_ago }}h sans signal
{% endfor %}
{% else %}
✅ Aucun appareil offline (seuil: 25h)
{% endif %}
```

---

## 5. Debug d'un Appareil Spécifique

⚠️ **Remplace `salle_de_bain` par le nom à tester** (en minuscule, avec underscores)

```jinja2
{% set device_name = 'salle_de_bain' %}

## 🔎 Recherche "{{ device_name }}"

### Capteurs batterie (device_class=battery)
{% set batt_sensors = states.sensor 
  | selectattr('attributes.device_class', 'defined') 
  | selectattr('attributes.device_class', 'eq', 'battery')
  | selectattr('entity_id', 'search', device_name)
  | list %}
{% for s in batt_sensors %}
- **{{ s.entity_id }}** = `{{ s.state }}`
{% endfor %}
{% if batt_sensors | length == 0 %}Aucun avec device_class=battery{% endif %}

### Tous les sensors contenant "{{ device_name }}"
{% set all_sensors = states.sensor 
  | selectattr('entity_id', 'search', device_name)
  | list %}
{% for s in all_sensors %}
- **{{ s.entity_id }}** = `{{ s.state }}`
{% endfor %}
{% if all_sensors | length == 0 %}Aucun sensor trouvé{% endif %}
```

---

## 6. Vérification des Alertes

Liste les appareils en alerte et pourquoi.

```jinja2
{% set alerts = state_attr('sensor.zigbee_battery_alerts', 'alert_devices') | default([]) %}

## 🚨 Alertes

{% if alerts | length > 0 %}
| Appareil | Status | Batterie | Raison |
|----------|--------|----------|--------|
{% for d in alerts %}
| {{ d.name }} | {{ d.status }} | {{ d.battery }}% | {{ '🔴 Offline' if d.status == 'offline' else '⚠️ Batterie faible' if d.battery | string | replace('~','') | float(100) < 15 else '❓ Inconnue' }} |
{% endfor %}
{% else %}
✅ Aucune alerte active
{% endif %}
```
