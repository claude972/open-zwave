# Guide de Configuration Vapi - Étape par Étape

## Introduction

Ce guide détaillé vous accompagne dans la configuration complète de votre assistant vocal Vapi pour la prise de commande restaurant.

---

## Étape 1 : Création du compte Vapi

1. Rendez-vous sur [vapi.ai](https://vapi.ai)
2. Cliquez sur **Sign Up**
3. Créez votre compte (email ou Google/GitHub)
4. Confirmez votre email

---

## Étape 2 : Configuration des Providers

### 2.1 Voice Provider (Synthèse vocale)

Vapi propose plusieurs options :

#### Option A : Voix Vapi (par défaut)
- Gratuit, inclus dans votre plan
- Voix de bonne qualité

#### Option B : ElevenLabs (recommandé pour le français)
1. Créez un compte sur [elevenlabs.io](https://elevenlabs.io)
2. Récupérez votre **API Key** dans Settings
3. Dans Vapi Dashboard : **Settings** > **Providers** > **ElevenLabs**
4. Collez votre API Key
5. Sélectionnez une voix française (ex: "Antoni", "Rachel")

### 2.2 LLM Provider

#### Option A : OpenAI (recommandé)
1. Créez un compte sur [platform.openai.com](https://platform.openai.com)
2. Générez une API Key
3. Dans Vapi : **Settings** > **Providers** > **OpenAI**
4. Collez votre API Key

#### Option B : Utiliser le LLM Vapi
- Inclus par défaut, pas de configuration requise

### 2.3 Transcription Provider

#### Deepgram (recommandé)
1. Créez un compte sur [deepgram.com](https://deepgram.com)
2. Créez un projet et une API Key
3. Dans Vapi : **Settings** > **Providers** > **Deepgram**
4. Collez votre API Key

---

## Étape 3 : Création de l'Assistant

### 3.1 Nouveau Assistant

1. Dans le Dashboard, cliquez sur **Assistants**
2. Cliquez sur **Create Assistant**
3. Donnez un nom : `Assistant Commande Restaurant`

### 3.2 Configuration Générale

#### First Message
```
Bonjour et bienvenue chez [Nom du Restaurant] ! Je suis votre assistant vocal. Comment puis-je vous aider ? Souhaitez-vous passer une commande, consulter notre menu, ou obtenir des informations sur le restaurant ?
```

#### System Prompt
Copiez l'intégralité du fichier `vapi-config/system-prompt.md`

### 3.3 Configuration Voice

```json
{
  "provider": "11labs",
  "voiceId": "21m00Tcm4TlvDq8ikWAM",
  "model": "eleven_multilingual_v2",
  "stability": 0.5,
  "similarityBoost": 0.75
}
```

Ou via l'interface :
- **Provider**: ElevenLabs (ou Vapi)
- **Voice**: Choisissez une voix française
- **Model**: eleven_multilingual_v2

### 3.4 Configuration LLM

```json
{
  "provider": "openai",
  "model": "gpt-4-turbo",
  "temperature": 0.7,
  "maxTokens": 500
}
```

Ou via l'interface :
- **Provider**: OpenAI
- **Model**: gpt-4-turbo
- **Temperature**: 0.7
- **Max Tokens**: 500

### 3.5 Configuration Transcriber

```json
{
  "provider": "deepgram",
  "model": "nova-2",
  "language": "fr"
}
```

Ou via l'interface :
- **Provider**: Deepgram
- **Model**: Nova 2
- **Language**: French (fr)

---

## Étape 4 : Configuration des Tools (Fonctions)

### 4.1 Ajouter les Tools

Pour chaque fonction, suivez ces étapes :

1. Dans l'assistant, onglet **Tools**
2. Cliquez **Add Tool**
3. Sélectionnez **Function**
4. Remplissez les champs

### 4.2 Tool : get_menu

```json
{
  "name": "get_menu",
  "description": "Récupère le menu complet du restaurant avec toutes les catégories et articles disponibles",
  "parameters": {
    "type": "object",
    "properties": {
      "category": {
        "type": "string",
        "description": "Catégorie spécifique (optionnel)"
      }
    },
    "required": []
  }
}
```

**Server URL**: `https://votre-instance.app.n8n.cloud/webhook/vapi-handler`

### 4.3 Tool : add_to_order

```json
{
  "name": "add_to_order",
  "description": "Ajoute un article à la commande en cours",
  "parameters": {
    "type": "object",
    "properties": {
      "item_name": {
        "type": "string",
        "description": "Nom de l'article"
      },
      "quantity": {
        "type": "integer",
        "description": "Quantité",
        "default": 1
      },
      "options": {
        "type": "array",
        "description": "Options sélectionnées"
      },
      "notes": {
        "type": "string",
        "description": "Notes spéciales"
      }
    },
    "required": ["item_name"]
  }
}
```

### 4.4 Tool : submit_order

```json
{
  "name": "submit_order",
  "description": "Valide et soumet la commande finale",
  "parameters": {
    "type": "object",
    "properties": {
      "customer_first_name": {
        "type": "string",
        "description": "Prénom du client"
      },
      "customer_phone": {
        "type": "string",
        "description": "Téléphone du client"
      },
      "service_type": {
        "type": "string",
        "enum": ["delivery", "collection"],
        "description": "Livraison ou retrait"
      },
      "delivery_address": {
        "type": "object",
        "description": "Adresse de livraison"
      }
    },
    "required": ["customer_first_name", "customer_phone", "service_type"]
  }
}
```

### 4.5 Tool : get_restaurant_info

```json
{
  "name": "get_restaurant_info",
  "description": "Récupère les informations du restaurant",
  "parameters": {
    "type": "object",
    "properties": {
      "info_type": {
        "type": "string",
        "enum": ["all", "hours", "address", "services"],
        "default": "all"
      }
    },
    "required": []
  }
}
```

Répétez pour tous les tools définis dans `tools-definition.json`.

---

## Étape 5 : Configuration du Serveur (Webhook)

### 5.1 Server URL

1. Dans l'assistant, onglet **Advanced**
2. Section **Server**
3. **Server URL**: `https://votre-instance.app.n8n.cloud/webhook/vapi-handler`

### 5.2 Messages serveur

Cochez les événements à recevoir :
- [x] function-call
- [x] end-of-call-report
- [x] status-update
- [ ] transcript (optionnel)

### 5.3 Secret (optionnel mais recommandé)

1. Générez un secret aléatoire
2. Ajoutez-le dans Vapi
3. Configurez la vérification dans n8n

---

## Étape 6 : Numéro de téléphone

### 6.1 Acheter un numéro

1. Allez dans **Phone Numbers**
2. Cliquez **Buy Number**
3. Sélectionnez :
   - **Country**: France
   - **Type**: Local ou Mobile
4. Confirmez l'achat

### 6.2 Configurer le numéro

1. Cliquez sur le numéro
2. **Inbound Call Settings**
3. **Assistant**: Sélectionnez votre assistant
4. Sauvegardez

---

## Étape 7 : Test

### 7.1 Test dans le Dashboard

1. Dans l'assistant, cliquez **Test**
2. Utilisez le micro de votre navigateur
3. Testez les scénarios :
   - "Bonjour, je voudrais commander"
   - "Quel est votre menu ?"
   - "Je prends une pizza margherita"

### 7.2 Test par téléphone

1. Appelez votre numéro Vapi
2. Vérifiez que l'assistant répond
3. Passez une commande test complète

### 7.3 Vérification des logs

1. Allez dans **Call Logs**
2. Écoutez les enregistrements
3. Lisez les transcripts
4. Identifiez les points d'amélioration

---

## Conseils d'optimisation

### Améliorer la compréhension

1. Ajoutez des **keywords** dans le transcriber pour les noms de plats
2. Ajustez le **temperature** du LLM (plus bas = plus prévisible)
3. Enrichissez le system prompt avec des exemples

### Améliorer la voix

1. Testez différentes voix 11Labs
2. Ajustez **stability** et **similarityBoost**
3. Utilisez des pauses naturelles dans les réponses

### Réduire la latence

1. Utilisez des réponses courtes
2. Activez le **backchannel** (hmm, d'accord...)
3. Optimisez les workflows n8n

---

## Support

- [Documentation Vapi](https://docs.vapi.ai)
- [Discord Vapi](https://discord.gg/vapi)
- [Guide des Tools](https://docs.vapi.ai/tools)
