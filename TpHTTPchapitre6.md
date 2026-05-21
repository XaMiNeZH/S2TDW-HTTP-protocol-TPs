# Chapitre 6 : Le Protocole HTTP - Travaux Pratiques
**Étudiant :** *EZ-ZAHERY Ahmed Amine*  
**Date :** Mai 2025  
**Enseignant :** Abdelaziz DAAIF  

---

## TP 1 : Exploration avec les DevTools

### 1.2 – Observer une requête simple (`https://httpbin.org/get`)

Après avoir ouvert les DevTools (F12 → onglet Network) et navigué vers `https://httpbin.org/get` :

- **Code de statut :** `200 OK`
- **Headers de requête envoyés (principaux) :**
  - `Host: httpbin.org`
  - `User-Agent: Mozilla/5.0 ...`
  - `Accept: text/html,application/xhtml+xml,...`
  - `Accept-Language: fr-FR,fr;q=0.9`
  - `Connection: keep-alive`
- **Content-Type de la réponse :** `application/json`

---

### 1.3 – Tester différentes méthodes via la Console

#### Requête GET

```javascript
fetch('https://httpbin.org/get')
  .then(r => r.json())
  .then(console.log);
```

**Résultat observé :** Réponse JSON contenant `args`, `headers`, `origin`, `url`. Code `200 OK`.

#### Requête POST avec JSON

```javascript
fetch('https://httpbin.org/post', {
  method: 'POST',
  headers: {'Content-Type': 'application/json'},
  body: JSON.stringify({name: 'John', age: 30})
})
  .then(r => r.json())
  .then(console.log);
```

**Résultat observé :** Réponse JSON avec le champ `json: { name: "John", age: 30 }` confirmant la bonne réception du body. Code `200 OK`.

---

### 1.4 – Codes de statut observés

| URL | Code observé | Signification |
|-----|-------------|---------------|
| `httpbin.org/status/200` | `200 OK` | Succès |
| `httpbin.org/status/404` | `404 Not Found` | Ressource introuvable |
| `httpbin.org/status/500` | `500 Internal Server Error` | Erreur serveur |
| `httpbin.org/redirect/3` | `302 → 302 → 302 → 200` | 3 redirections successives puis succès |

---

### Exercice – Tableau récapitulatif

| URL | Méthode | Code | Content-Type |
|-----|---------|------|--------------|
| `httpbin.org/get` | GET | 200 | application/json |
| `httpbin.org/post` | POST | 200 | application/json |
| `httpbin.org/status/201` | GET | 201 | text/plain |

---

## TP 2 : Maîtrise de cURL

### 2.1 – Différence entre `-i` et `-v`

- **`-i`** : Affiche les **headers HTTP de la réponse** suivis du corps. Utile pour voir rapidement le code de statut et les headers sans détails sur la négociation de connexion.
- **`-v`** : Mode **verbeux complet** — affiche les headers de requête ET de réponse, les détails de connexion TLS, et les métadonnées de transfert. Idéal pour le débogage.

```
# Exemple -i (résultat simplifié)
HTTP/2 200
content-type: application/json
...
{ "args": {}, ... }

# Exemple -v (résultat complet)
* Connected to httpbin.org
> GET /get HTTP/2
> Host: httpbin.org
< HTTP/2 200
< content-type: application/json
...
```

---

### 2.2 – Requêtes POST

```bash
# Form data
curl -X POST -d "name=John&email=john@example.com" \
  https://httpbin.org/post

# JSON
curl -X POST \
  -H "Content-Type: application/json" \
  -d '{"name": "John", "email": "john@example.com"}' \
  https://httpbin.org/post
```

**Différence :** Avec `form data`, le serveur reçoit les données dans `form`; avec `JSON`, elles apparaissent dans `json`. Le Content-Type détermine comment le serveur interprète le body.

---

### 2.3 – Headers personnalisés

```bash
curl -H "Authorization: Bearer mon-token-secret" \
  -H "Accept: application/json" \
  https://httpbin.org/headers
```

**Résultat :** Le JSON de réponse confirme que `Authorization` et `Accept` ont bien été transmis dans les headers.

---

### 2.4 – Suivre les redirections

```bash
# Sans -L : s'arrête à la 1ère redirection (302)
curl https://httpbin.org/redirect/3
# Réponse : 302 Found avec header Location

# Avec -L : suit toutes les redirections jusqu'au 200 final
curl -L https://httpbin.org/redirect/3
# Réponse finale : 200 OK avec le JSON habituel
```

---

### 2.5 – Télécharger un fichier

```bash
# Sauvegarder sous un nom spécifique
curl -o image.png https://httpbin.org/image/png

# Garder le nom original du fichier
curl -O https://example.com/fichier.pdf
```

---

### Exercice avancé – Commande cURL complète

```bash
curl -X POST \
  -H "Content-Type: application/json" \
  -H "X-Custom-Header: MonHeader" \
  -d '{"action": "test", "value": 42}' \
  -i \
  https://httpbin.org/post
```

**Explication des options :**
- `-X POST` : méthode HTTP POST
- `-H "Content-Type: application/json"` : type du body
- `-H "X-Custom-Header: MonHeader"` : header personnalisé
- `-d '...'` : body JSON envoyé
- `-i` : affiche les headers de réponse

---

## TP 3 : API REST avec JavaScript

### 3.1 – GET basique

```javascript
// Avec .then()
fetch('https://jsonplaceholder.typicode.com/users')
  .then(response => response.json())
  .then(users => console.log(users))
  .catch(error => console.error('Erreur:', error));

// Avec async/await
async function getUsers() {
  try {
    const response = await fetch('https://jsonplaceholder.typicode.com/users');
    const users = await response.json();
    console.log(users);
  } catch (error) {
    console.error('Erreur:', error);
  }
}
```

**Résultat :** Retourne un tableau de 10 utilisateurs au format JSON avec leurs propriétés (`id`, `name`, `email`, `address`, etc.).

---

### 3.2 – POST – Créer une ressource

```javascript
async function createPost(data) {
  const response = await fetch('https://jsonplaceholder.typicode.com/posts', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(data)
  });

  if (!response.ok) {
    throw new Error(`HTTP ${response.status}`);
  }

  return response.json();
}

createPost({
  title: 'Mon article',
  body: "Contenu de l'article",
  userId: 1
}).then(console.log);
```

**Résultat :** Code `201 Created`. Le JSON retourné inclut un `id: 101` (nouvel identifiant simulé par l'API).

---

### 3.3 – PUT – Modifier une ressource

```javascript
async function updatePost(id, data) {
  const response = await fetch(`https://jsonplaceholder.typicode.com/posts/${id}`, {
    method: 'PUT',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(data)
  });
  return response.json();
}

// Exemple d'utilisation
updatePost(1, { title: 'Titre modifié', body: 'Nouveau contenu', userId: 1 })
  .then(console.log);
```

**Résultat :** Code `200 OK`. L'objet retourné reflète les nouvelles données (mais la modification n'est pas persistée, c'est une API fictive).

---

### 3.4 – DELETE – Supprimer une ressource

```javascript
async function deletePost(id) {
  const response = await fetch(`https://jsonplaceholder.typicode.com/posts/${id}`, {
    method: 'DELETE'
  });
  return response.ok;
}

deletePost(1).then(ok => console.log('Supprimé :', ok)); // true
```

**Résultat :** Code `200 OK`, corps `{}`. La fonction retourne `true`.

---

### Exercice pratique – `fetchWithRetry`

```javascript
async function fetchWithRetry(url, options = {}, maxRetries = 3) {
  let lastError;

  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      const response = await fetch(url, options);

      // Si erreur 5xx → on réessaie
      if (response.status >= 500 && attempt < maxRetries) {
        console.warn(`Tentative ${attempt} échouée (${response.status}), nouvelle tentative dans 1s...`);
        await new Promise(resolve => setTimeout(resolve, 1000));
        continue;
      }

      return response; // Succès ou erreur non-5xx
    } catch (error) {
      lastError = error;
      if (attempt < maxRetries) {
        console.warn(`Tentative ${attempt} : erreur réseau, réessai dans 1s...`);
        await new Promise(resolve => setTimeout(resolve, 1000));
      }
    }
  }

  throw lastError || new Error(`Échec après ${maxRetries} tentatives`);
}

// Utilisation
fetchWithRetry('https://jsonplaceholder.typicode.com/posts/1', {}, 3)
  .then(r => r.json())
  .then(console.log)
  .catch(console.error);
```

**Fonctionnement :**
1. Tente la requête jusqu'à `maxRetries` fois
2. En cas de code 5xx ou d'erreur réseau, attend 1 seconde et réessaie
3. Retourne la réponse finale ou lève une erreur si toutes les tentatives échouent

---

## TP 4 : Analyse des Headers de Sécurité

### 4.1 – Vérification avec cURL

```bash
# Voir tous les headers de réponse
curl -I https://google.com

# Filtrer les headers de sécurité de GitHub
curl -s -D - https://github.com -o /dev/null | grep -i "strict\|x-frame\|x-content\|content-security\|referrer"
```

**Résultat pour github.com :**

```
strict-transport-security: max-age=31536000; includeSubdomains; preload
x-frame-options: deny
x-content-type-options: nosniff
referrer-policy: origin-when-cross-origin, strict-origin-when-cross-origin
content-security-policy: default-src 'none'; base-uri 'self'; child-src ...
```

---

### Headers de sécurité – Référentiel

| Header | But | Valeur recommandée |
|--------|-----|--------------------|
| `Strict-Transport-Security` | Forcer HTTPS | `max-age=31536000; includeSubDomains` |
| `X-Frame-Options` | Anti-clickjacking | `DENY` ou `SAMEORIGIN` |
| `X-Content-Type-Options` | Anti-MIME sniffing | `nosniff` |
| `Content-Security-Policy` | Sources autorisées | `default-src 'self'` |
| `Referrer-Policy` | Contrôle du referrer | `strict-origin-when-cross-origin` |

---

### Exercice – Analyse de 3 sites

| Site | HSTS | X-Frame-Options | CSP | Note |
|------|------|-----------------|-----|------|
| `github.com` | ✅ `max-age=31536000; includeSubdomains; preload` | ✅ `deny` | ✅ Complète et stricte | A+ |
| `google.com` | ✅ `max-age=31536000` | ❌ Non présent | ⚠️ Partielle | B |
| `wikipedia.org` | ✅ `max-age=106384710` | ✅ `SAMEORIGIN` | ⚠️ Présente mais permissive | B+ |

**Observations :**
- GitHub a la configuration la plus rigoureuse avec HSTS preload et une CSP très détaillée.
- Google omet `X-Frame-Options` mais bénéficie d'une protection via CSP (`frame-ancestors`).
- Wikipedia applique les bases mais sa CSP pourrait être renforcée.

---

## TP 5 : Cache HTTP

### 5.1 – Observer le cache

```bash
# Première requête – observer les headers de cache
curl -i https://httpbin.org/cache/60
```

**Headers de cache attendus :**
```
Cache-Control: public, max-age=60
ETag: "some-etag-value"
Last-Modified: Thu, 21 May 2025 10:00:00 GMT
```

**Interprétation :** La ressource peut être mise en cache pendant 60 secondes. Après ce délai, le client doit revalider.

---

### 5.2 – Requête conditionnelle avec ETag

```bash
# 1. Obtenir l'ETag
curl -i https://httpbin.org/etag/test123
# → Réponse : 200 OK + ETag: "test123"

# 2. Requête conditionnelle
curl -i -H "If-None-Match: test123" https://httpbin.org/etag/test123
# → Réponse : 304 Not Modified (pas de body)
```

**Fonctionnement :**  
Le client envoie l'ETag reçu précédemment. Si la ressource n'a pas changé, le serveur répond `304 Not Modified` sans retransmettre le corps — économie de bande passante.

---

### 5.3 – Cache dans le navigateur

Observations avec DevTools → Network :
- **Chargement initial :** toutes les ressources sont récupérées depuis le réseau (statut `200`).
- **Rechargement F5 :** les ressources en cache apparaissent avec le statut `200 (from disk cache)` ou `200 (from memory cache)`, avec un temps de chargement quasi nul.
- **Ctrl+Shift+R (hard reload) :** ignore le cache, toutes les ressources sont re-téléchargées depuis le serveur.

---

### Exercice – Page HTML avec stratégie de cache

```html
<!DOCTYPE html>
<html lang="fr">
<head>
  <meta charset="UTF-8">
  <title>Cache HTTP Demo</title>
  <link rel="stylesheet" href="styles.css">
</head>
<body>
  <h1>Démonstration Cache HTTP</h1>
  <img src="logo.png" alt="Logo">
  <script src="app.js"></script>
</body>
</html>
```

**Configuration serveur (ex: Apache `.htaccess`) :**

```apache
# Images — cache long (contenu statique rarement modifié)
<FilesMatch "\.(png|jpg|gif|svg|ico)$">
  Header set Cache-Control "public, max-age=2592000, immutable"
</FilesMatch>

# CSS — cache moyen avec revalidation
<FilesMatch "\.css$">
  Header set Cache-Control "public, max-age=86400, must-revalidate"
</FilesMatch>

# JavaScript — cache court avec revalidation
<FilesMatch "\.js$">
  Header set Cache-Control "public, max-age=3600, must-revalidate"
</FilesMatch>
```

**Justification :**
- **Images** : rarement modifiées → `max-age` long (30 jours) + `immutable` pour éviter les revalidations inutiles.
- **CSS** : peut changer lors des mises à jour UI → 1 jour avec `must-revalidate`.
- **JS** : logique applicative susceptible de changer → 1 heure avec revalidation.

---

## Exercices Récapitulatifs

### Exercice 1 : Client HTTP minimaliste

```html
<!DOCTYPE html>
<html lang="fr">
<head>
  <meta charset="UTF-8">
  <title>Client HTTP</title>
  <style>
    body { font-family: sans-serif; max-width: 800px; margin: 2rem auto; padding: 0 1rem; }
    input, select, textarea { width: 100%; margin: .4rem 0 1rem; padding: .5rem; box-sizing: border-box; }
    button { padding: .6rem 1.5rem; background: #2563eb; color: #fff; border: none; cursor: pointer; border-radius: 4px; }
    pre { background: #f4f4f4; padding: 1rem; overflow-x: auto; white-space: pre-wrap; }
  </style>
</head>
<body>
  <h1>🌐 Client HTTP Minimaliste</h1>

  <label>URL :</label>
  <input type="text" id="url" value="https://jsonplaceholder.typicode.com/posts/1">

  <label>Méthode :</label>
  <select id="method">
    <option>GET</option><option>POST</option><option>PUT</option><option>DELETE</option>
  </select>

  <label>Body (JSON) :</label>
  <textarea id="body" rows="4" placeholder='{"key": "value"}'></textarea>

  <button onclick="sendRequest()">Envoyer</button>

  <h2>Statut : <span id="status">—</span></h2>
  <h3>Headers de réponse :</h3>
  <pre id="headers">—</pre>
  <h3>Corps de la réponse :</h3>
  <pre id="response">—</pre>

  <script>
    async function sendRequest() {
      const url    = document.getElementById('url').value.trim();
      const method = document.getElementById('method').value;
      const body   = document.getElementById('body').value.trim();

      const options = { method, headers: {} };
      if (body && method !== 'GET') {
        options.headers['Content-Type'] = 'application/json';
        options.body = body;
      }

      try {
        const response = await fetch(url, options);
        document.getElementById('status').textContent = `${response.status} ${response.statusText}`;

        // Headers
        const hdrs = [...response.headers.entries()]
          .map(([k, v]) => `${k}: ${v}`)
          .join('\n');
        document.getElementById('headers').textContent = hdrs;

        // Body
        const text = await response.text();
        try {
          document.getElementById('response').textContent =
            JSON.stringify(JSON.parse(text), null, 2);
        } catch {
          document.getElementById('response').textContent = text;
        }
      } catch (err) {
        document.getElementById('status').textContent = 'Erreur réseau';
        document.getElementById('response').textContent = err.message;
      }
    }
  </script>
</body>
</html>
```

---

### Exercice 2 : Questions théoriques

**1. Quelle est la différence entre `no-cache` et `no-store` ?**

- **`no-cache`** : Le navigateur peut stocker la réponse en cache, mais il **doit revalider** auprès du serveur avant de l'utiliser (envoi d'une requête conditionnelle avec `ETag` ou `Last-Modified`). Si le serveur confirme que la ressource n'a pas changé (`304 Not Modified`), le contenu local est utilisé.  
- **`no-store`** : La réponse **ne doit jamais être stockée** en cache, ni en mémoire ni sur disque. Chaque requête force un téléchargement complet depuis le serveur. Utilisé pour des données confidentielles (ex : pages bancaires).

---

**2. Pourquoi POST n'est-il pas idempotent ?**

Une méthode est **idempotente** si l'envoyer plusieurs fois produit le même résultat que l'envoyer une seule fois. POST **crée une nouvelle ressource** à chaque appel : envoyer deux fois le même POST peut créer deux entrées distinctes dans la base de données. Par opposition, PUT est idempotent car mettre à jour une ressource avec les mêmes données plusieurs fois ne change pas l'état final.

---

**3. Que se passe-t-il si le serveur renvoie un code 301 ?**

Un code **`301 Moved Permanently`** indique que la ressource a été **déplacée définitivement** vers l'URL fournie dans le header `Location`. Le navigateur :
1. Redirige automatiquement vers la nouvelle URL.
2. **Mémorise** cette redirection afin de l'utiliser directement lors des prochaines visites (sans re-contacter l'ancienne URL).
3. Les moteurs de recherche transfèrent le "link juice" (référencement) vers la nouvelle URL.

---

**4. À quoi sert le header `Origin` ?**

Le header `Origin` est envoyé automatiquement par le navigateur lors des **requêtes cross-origin** (CORS) et des formulaires POST. Il indique l'**origine de la page** qui a déclenché la requête (schéma + domaine + port), sans inclure le chemin. Le serveur l'utilise pour décider s'il doit autoriser la requête via les headers `Access-Control-Allow-Origin`. Il protège contre les attaques **CSRF** en permettant au serveur de vérifier la provenance des requêtes.

---

**5. Pourquoi utiliser `HttpOnly` sur les cookies de session ?**

Le flag `HttpOnly` empêche JavaScript d'accéder au cookie via `document.cookie`. Sans ce flag, une attaque **XSS** (Cross-Site Scripting) permettrait à un script malveillant injecté dans la page de voler le cookie de session et d'usurper l'identité de l'utilisateur. Avec `HttpOnly`, le cookie est uniquement transmis dans les en-têtes HTTP et reste invisible au code JavaScript — même si du code XSS s'exécute, il ne peut pas extraire le cookie.

---

*Fin des Travaux Pratiques.*
