# Restaurant Order Automation

Système d'automatisation de prise de commande vocale pour restaurants utilisant **n8n**, **Vapi** et **HubRise**.

## Fonctionnalités

- 📞 **Prise de commande vocale** via téléphone avec Vapi
- 🍽️ **Synchronisation menu** depuis HubRise
- 📝 **Création automatique** des commandes dans le POS
- 📊 **Historique** des commandes et statistiques
- 🕐 **Informations restaurant** (horaires, adresse, services)

## Architecture

```
Client (Téléphone) → Vapi (Voice AI) → n8n (Workflows) → HubRise (API) → POS
```

## Structure du projet

```
restaurant-order-automation/
├── n8n-workflows/
│   ├── 1-hubrise-get-catalog.json      # Récupération du menu
│   ├── 2-hubrise-create-order.json     # Création de commandes
│   ├── 3-hubrise-restaurant-info.json  # Infos restaurant
│   ├── 4-hubrise-order-history.json    # Historique commandes
│   └── 5-vapi-webhook-handler.json     # Handler principal Vapi
├── vapi-config/
│   ├── assistant-config.json           # Configuration assistant
│   ├── system-prompt.md                # Prompt système
│   └── tools-definition.json           # Définition des fonctions
└── docs/
    ├── GUIDE-INSTALLATION.md           # Guide complet d'installation
    ├── GUIDE-VAPI-CONFIGURATION.md     # Configuration Vapi détaillée
    └── ARCHITECTURE.md                 # Architecture technique
```

## Prérequis

- Compte [HubRise](https://hubrise.com) avec restaurant configuré
- Compte [n8n Cloud](https://n8n.io/cloud/)
- Compte [Vapi](https://vapi.ai)

## Installation rapide

1. **HubRise** : Récupérez vos credentials (account_id, location_id, access_token)
2. **n8n** : Importez les workflows et configurez les credentials
3. **Vapi** : Créez l'assistant avec le system prompt et les tools
4. **Test** : Appelez le numéro Vapi pour tester

👉 Consultez le [Guide d'installation complet](docs/GUIDE-INSTALLATION.md)

## Configuration requise

### Variables n8n

| Variable | Description |
|----------|-------------|
| `HUBRISE_ACCOUNT_ID` | ID compte HubRise |
| `HUBRISE_LOCATION_ID` | ID établissement |
| `HUBRISE_CATALOG_ID` | ID catalogue |

### Credentials n8n

- **HubRise API Token** : Header Auth avec `X-Access-Token`

## Workflows n8n

| Workflow | Webhook | Description |
|----------|---------|-------------|
| Catalogue | `/webhook/get-catalog` | Menu complet |
| Commandes | `/webhook/hubrise-create-order` | Création commande |
| Infos | `/webhook/restaurant-info` | Infos restaurant |
| Historique | `/webhook/order-history` | Liste commandes |
| Vapi Handler | `/webhook/vapi-handler` | Point d'entrée Vapi |

## Fonctions Vapi

| Fonction | Description |
|----------|-------------|
| `get_menu` | Récupère le menu |
| `add_to_order` | Ajoute un article |
| `remove_from_order` | Retire un article |
| `get_order_summary` | Récapitulatif |
| `submit_order` | Valide la commande |
| `get_restaurant_info` | Informations |
| `check_availability` | Disponibilité article |

## Documentation

- [Guide d'installation](docs/GUIDE-INSTALLATION.md)
- [Configuration Vapi](docs/GUIDE-VAPI-CONFIGURATION.md)
- [Architecture](docs/ARCHITECTURE.md)

## Support

- [Documentation HubRise](https://www.hubrise.com/developers)
- [Documentation n8n](https://docs.n8n.io/)
- [Documentation Vapi](https://docs.vapi.ai/)

## Licence

MIT
