# Architecture du Système

## Vue d'ensemble

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   Client    │────▶│    Vapi     │────▶│    n8n      │────▶│  HubRise    │
│ (Téléphone) │◀────│  (Voice AI) │◀────│ (Workflows) │◀────│   (API)     │
└─────────────┘     └─────────────┘     └─────────────┘     └─────────────┘
                           │                   │                   │
                           ▼                   ▼                   ▼
                    ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
                    │  11Labs     │     │   Redis*    │     │    POS      │
                    │  (TTS/STT)  │     │  (Cache)    │     │ Restaurant  │
                    └─────────────┘     └─────────────┘     └─────────────┘

                    * Optionnel pour production
```

## Flux de données

### 1. Appel entrant

```
Client appelle → Vapi reçoit → Assistant démarre → Premier message envoyé
```

### 2. Prise de commande

```
Client parle → Deepgram transcrit → GPT-4 analyse → Function call déclenché
     ↓
n8n webhook → HubRise API → Données récupérées → Réponse formatée
     ↓
Vapi reçoit → 11Labs synthétise → Client entend la réponse
```

### 3. Validation commande

```
Client confirme → submit_order() appelé → n8n prépare commande
     ↓
HubRise POST /orders → Commande créée → Confirmation envoyée
     ↓
POS reçoit → Cuisine notifiée → Préparation démarre
```

## Composants

### Vapi (Interface Vocale)

| Composant | Rôle |
|-----------|------|
| Transcriber | Convertit la voix en texte (Deepgram) |
| LLM | Comprend l'intention et génère les réponses (GPT-4) |
| Voice | Synthétise la voix (11Labs) |
| Tools | Appelle les fonctions externes (n8n) |

### n8n (Orchestration)

| Workflow | Fonction |
|----------|----------|
| vapi-webhook-handler | Point d'entrée principal, route les requêtes |
| hubrise-get-catalog | Récupère le menu du restaurant |
| hubrise-create-order | Crée une nouvelle commande |
| hubrise-restaurant-info | Informations du restaurant |
| hubrise-order-history | Historique des commandes |

### HubRise (Middleware)

| API | Usage |
|-----|-------|
| GET /catalogs | Menu et produits |
| GET /locations | Informations restaurant |
| POST /orders | Création de commande |
| GET /orders | Historique commandes |

## Sécurité

### Authentification

```
Vapi → n8n : Webhook secret (optionnel)
n8n → HubRise : X-Access-Token header
```

### Données sensibles

- Les tokens sont stockés dans les credentials n8n
- Aucune donnée client n'est logguée
- Les enregistrements Vapi peuvent être désactivés

## Scalabilité

### Production recommandée

1. **Cache Redis** pour les paniers en cours
2. **Base de données** pour l'historique des appels
3. **Monitoring** (Datadog, Sentry)
4. **Load balancer** si plusieurs instances n8n

### Limites actuelles

| Service | Limite |
|---------|--------|
| Vapi | Dépend du plan |
| n8n Cloud | 10k exécutions/mois (starter) |
| HubRise | 1000 requêtes/heure |
