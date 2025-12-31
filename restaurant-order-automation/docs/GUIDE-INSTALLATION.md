# Guide d'Installation - Automatisation Commande Restaurant

Ce guide vous accompagne dans la mise en place complète du système de prise de commande vocale avec n8n, Vapi et HubRise.

## Table des matières

1. [Prérequis](#prérequis)
2. [Configuration HubRise](#configuration-hubrise)
3. [Configuration n8n Cloud](#configuration-n8n-cloud)
4. [Configuration Vapi](#configuration-vapi)
5. [Test du système](#test-du-système)
6. [Dépannage](#dépannage)

---

## Prérequis

Avant de commencer, assurez-vous d'avoir :

- [ ] Un compte [HubRise](https://www.hubrise.com/) avec un restaurant configuré
- [ ] Un compte [n8n Cloud](https://n8n.io/cloud/)
- [ ] Un compte [Vapi](https://vapi.ai/)
- [ ] Vos credentials HubRise (account_id, location_id, access_token)

---

## Configuration HubRise

### 1. Récupérer vos identifiants

1. Connectez-vous à [HubRise Back Office](https://manager.hubrise.com/)
2. Allez dans **Paramètres** > **Développeur**
3. Notez les informations suivantes :
   - **Account ID** : `xxxx-xxxx`
   - **Location ID** : `xxxx-xxxx`
   - **Catalog ID** : `xxxx` (visible dans Catalogs)
4. Créez un **Access Token** avec les permissions :
   - `location[orders.write]`
   - `location[orders.read]`
   - `location[catalog.read]`
   - `location[customer_list.read]`

### 2. Vérifier votre catalogue

Assurez-vous que votre catalogue HubRise contient :
- Des catégories (Entrées, Plats, Desserts, Boissons...)
- Des produits avec prix
- Des options si applicable (cuisson, accompagnements...)

---

## Configuration n8n Cloud

### 1. Créer les credentials HubRise

1. Dans n8n, allez dans **Credentials** > **Add Credential**
2. Sélectionnez **Header Auth**
3. Configurez :
   - **Name**: `HubRise API Token`
   - **Header Name**: `X-Access-Token`
   - **Header Value**: `votre_access_token_hubrise`

### 2. Configurer les variables d'environnement

Dans n8n Cloud, allez dans **Settings** > **Variables** et ajoutez :

| Variable | Valeur | Description |
|----------|--------|-------------|
| `HUBRISE_ACCOUNT_ID` | `xxxx-xxxx` | ID de votre compte HubRise |
| `HUBRISE_LOCATION_ID` | `xxxx-xxxx` | ID de votre établissement |
| `HUBRISE_CATALOG_ID` | `xxxx` | ID de votre catalogue |

### 3. Importer les workflows

1. Allez dans **Workflows** > **Import from File**
2. Importez dans l'ordre :
   - `1-hubrise-get-catalog.json`
   - `2-hubrise-create-order.json`
   - `3-hubrise-restaurant-info.json`
   - `4-hubrise-order-history.json`
   - `5-vapi-webhook-handler.json`

3. Dans chaque workflow, mettez à jour :
   - Les références aux credentials HubRise
   - Vérifiez les URLs des webhooks

### 4. Activer les workflows

1. Activez chaque workflow (toggle en haut à droite)
2. Notez les URLs des webhooks :
   - `https://votre-instance.app.n8n.cloud/webhook/vapi-handler`
   - `https://votre-instance.app.n8n.cloud/webhook/hubrise-create-order`
   - etc.

---

## Configuration Vapi

### 1. Créer un compte Vapi

1. Inscrivez-vous sur [vapi.ai](https://vapi.ai)
2. Accédez au Dashboard

### 2. Configurer les providers

#### Voix (11Labs ou autre)
1. Allez dans **Providers** > **Voice**
2. Ajoutez vos credentials 11Labs (ou utilisez les voix Vapi par défaut)

#### Transcription (Deepgram)
1. Allez dans **Providers** > **Transcription**
2. Configurez Deepgram ou utilisez le provider par défaut

### 3. Créer l'assistant

1. Allez dans **Assistants** > **Create Assistant**
2. Configurez l'assistant avec les paramètres de `vapi-config/assistant-config.json`
3. Copiez le contenu de `vapi-config/system-prompt.md` dans le champ **System Prompt**

#### Configuration détaillée :

```
Nom: Assistant Commande Restaurant
Première phrase: Bonjour et bienvenue ! Je suis votre assistant...
Voice: 11Labs - Voix française
Model: GPT-4 Turbo
Transcriber: Deepgram Nova 2 (French)
```

### 4. Ajouter les outils (Tools)

1. Dans l'assistant, allez dans **Tools**
2. Pour chaque fonction dans `vapi-config/tools-definition.json` :
   - Cliquez **Add Tool** > **Function**
   - Copiez la définition de la fonction
   - **Server URL**: Votre webhook n8n (`https://xxx.app.n8n.cloud/webhook/vapi-handler`)

### 5. Configurer le webhook serveur

1. Dans **Assistant Settings** > **Server**
2. **Server URL**: `https://votre-instance.app.n8n.cloud/webhook/vapi-handler`
3. Optionnel: Ajoutez un **Secret** pour sécuriser les appels

### 6. Obtenir un numéro de téléphone

1. Allez dans **Phone Numbers**
2. Achetez ou importez un numéro français
3. Associez-le à votre assistant

---

## Test du système

### 1. Test des workflows n8n

#### Test récupération catalogue
```bash
curl -X GET "https://votre-instance.app.n8n.cloud/webhook/get-catalog"
```

#### Test informations restaurant
```bash
curl -X GET "https://votre-instance.app.n8n.cloud/webhook/restaurant-info"
```

### 2. Test de l'assistant Vapi

1. Dans le Dashboard Vapi, utilisez le **Test Call**
2. Testez les scénarios :
   - "Bonjour, je voudrais voir le menu"
   - "Je voudrais commander une pizza"
   - "Quels sont vos horaires ?"

### 3. Test complet par téléphone

1. Appelez le numéro Vapi attribué
2. Passez une commande test complète
3. Vérifiez dans HubRise que la commande apparaît

---

## Dépannage

### Problèmes courants

#### "Erreur 401 Unauthorized" sur HubRise
- Vérifiez votre access token
- Assurez-vous que le token a les bonnes permissions

#### L'assistant ne répond pas
- Vérifiez que le workflow n8n est actif
- Vérifiez les logs dans n8n > Executions
- Vérifiez la configuration du webhook dans Vapi

#### Le menu ne se charge pas
- Vérifiez que le catalogue HubRise n'est pas vide
- Vérifiez le CATALOG_ID dans les variables n8n

#### Commandes non créées dans HubRise
- Vérifiez les logs d'exécution n8n
- Vérifiez le format de la commande dans les logs

### Logs et debugging

#### n8n
- Allez dans **Executions** pour voir l'historique
- Cliquez sur une exécution pour voir le détail de chaque nœud

#### Vapi
- Allez dans **Call Logs** pour écouter les enregistrements
- Consultez les transcripts pour identifier les problèmes

#### HubRise
- Vérifiez les **Logs API** dans le back-office HubRise

---

## Support

Pour toute question :
- [Documentation HubRise](https://www.hubrise.com/developers)
- [Documentation n8n](https://docs.n8n.io/)
- [Documentation Vapi](https://docs.vapi.ai/)
