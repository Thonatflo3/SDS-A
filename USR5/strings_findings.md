# Analyse sécurité de strings.xml
**Application :** Nickel (`com.fpe.comptenickel`)  
**Date d'analyse :** 2026-04-29  

---

## Findings

### [CRITICAL] F-01 — Clé API Google exposée

| Champ | Valeur |
|---|---|
| `google_api_key` | `AIzaSyBTgztvImsUfMWDa41PCrDWAj7dmyIDhUg` |
| `google_crash_reporting_api_key` | `AIzaSyBTgztvImsUfMWDa41PCrDWAj7dmyIDhUg` |

**Extrait :** `AIzaSyBTgztvImsUfMWDa41PCrDWAj7dmyIDhUg` (même clé pour deux usages)

**Risque :** Clé Google API en clair dans le binaire. Un attaquant peut l'extraire et l'utiliser pour consommer des quotas, accéder aux services Google associés au projet, ou effectuer des appels frauduleux facturés au compte Nickel.

---

### [HIGH] F-02 — Firebase Database URL exposée

| Champ | Valeur |
|---|---|
| `firebase_database_url` | `https://application-client-nickel.firebaseio.com` |

**Extrait :** URL Firebase Realtime Database de production exposée.

**Risque :** Permet à un attaquant de cibler directement la base de données Firebase. Si les règles de sécurité Firebase sont mal configurées, un accès en lecture/écriture non authentifié est possible.

---

### [HIGH] F-03 — Identifiants de projet Firebase en clair

| Champ | Valeur |
|---|---|
| `google_app_id` | `1:717748501407:android:e0cdeb712e27275d` |
| `gcm_defaultSenderId` | `717748501407` |
| `GCM_PROJECT_NUMBER` | `246766419677` |
| `project_id` | `application-client-nickel` |
| `google_storage_bucket` | `application-client-nickel.appspot.com` |

**Extrait :** Project ID Firebase `application-client-nickel` et sender ID `717748501407` exposés.

**Risque :** Ces identifiants permettent de reconstituer la configuration Firebase complète. Un attaquant peut tenter d'envoyer des notifications push frauduleuses (via GCM/FCM) ou d'accéder au bucket de stockage.

---

### [HIGH] F-04 — Endpoints API de production exposés

| Champ | Valeur |
|---|---|
| `ACCOUNT_ENDPOINT` | `https://api.nickel.eu/customer-banking-api` |
| `CUSTOMER_AUTHENTICATION_ENDPOINT` | `https://api.nickel.eu/customer-authentication-api` |
| `PERSONAL_SPACE_API_ENDPOINT` | `https://api.nickel.eu/personal-space-api` |
| `ENVIRONMENT` | `production` |

**Extrait :** 3 endpoints API bancaires de production exposés, environnement confirmé `production`.

**Risque :** Cartographie complète de l'architecture API bancaire. Facilite les attaques ciblées (brute-force, fuzzing, IDOR) directement sur les endpoints de production.

---

### [MEDIUM] F-05 — Pushwoosh App ID exposé

| Champ | Valeur |
|---|---|
| `PUSHWOOSH_APPID` | `DDF11-D1BF9` |

**Extrait :** Identifiant Pushwoosh `DDF11-D1BF9` (service de push notifications tiers).

**Risque :** Permet d'identifier le compte Pushwoosh de l'application. Pourrait permettre l'envoi de fausses notifications push si l'API Pushwoosh n'est pas correctement sécurisée côté serveur.

---

### [MEDIUM] F-06 — Google OAuth Client ID exposé

| Champ | Valeur |
|---|---|
| `default_web_client_id` | `717748501407-v621tmfvqd7etdouc3df5vuv31scle0g.apps.googleusercontent.com` |

**Extrait :** Client ID OAuth Google `717748501407-v621...` exposé dans le binaire.

**Risque :** Peut être utilisé pour des attaques de type OAuth token hijacking ou pour identifier les scopes d'autorisation Google autorisés par l'application.

---

### [LOW] F-07 — Crashlytics Build ID exposé

| Champ | Valeur |
|---|---|
| `com.crashlytics.android.build_id` | `9ad6d741-4e10-4abd-9f12-163a0e3fb0b2` |

**Extrait :** UUID de build Crashlytics `9ad6d741-4e10-4abd-9f12-163a0e3fb0b2`.

**Risque :** Faible. Permet d'identifier la version exacte du build. Combiné à d'autres informations, peut faciliter le ciblage d'une version spécifique avec des vulnérabilités connues.

---

### [INFO] F-08 — Présence de code React Native / debug

**Extrait :** Strings `catalyst_debugjs`, `catalyst_remotedbg_error`, `Enable Hot Reloading` présentes.

**Risque :** L'application est construite avec React Native et contient des strings de debug. Indique une possible surface d'attaque via le remote debugger JS si activé en production.

---

## Résumé des risques

| ID | Sévérité | Titre |
|---|---|---|
| F-01 | 🔴 CRITICAL | Clé API Google exposée |
| F-02 | 🟠 HIGH | Firebase Database URL exposée |
| F-03 | 🟠 HIGH | Identifiants Firebase en clair |
| F-04 | 🟠 HIGH | Endpoints API bancaires de production |
| F-05 | 🟡 MEDIUM | Pushwoosh App ID exposé |
| F-06 | 🟡 MEDIUM | Google OAuth Client ID exposé |
| F-07 | 🔵 LOW | Crashlytics Build ID |
| F-08 | ⚪ INFO | Présence de code React Native debug |

---

## Recommandations

1. **Ne jamais stocker de clés API dans les ressources compilées.** Utiliser un backend proxy ou Google Secret Manager.
2. **Sécuriser les règles Firebase** (Realtime Database + Storage) avec des règles d'authentification strictes.
3. **Externaliser les endpoints** via une configuration serveur dynamique plutôt qu'en dur dans l'APK.
4. **Révoquer et régénérer** immédiatement la clé `AIzaSyBTgztvImsUfMWDa41PCrDWAj7dmyIDhUg`.
5. **Désactiver le mode debug React Native** en production.
