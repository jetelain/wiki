# 📘 Réseau Tailscale

## 🔹 Étape 1 : Installation sur les clients

1. **Aller sur https://tailscale.com/ et se connecter avec Microsoft, Google ou autre**

2. **Télécharger le client Tailscale sur une des machines clientes (PC portable, smartphone)**

3. **Après installation, se connecter avec le même compte**

Répéter l'opération pour chaque machine cliente à connecter

## 🔹 Étape 2 : Installation sur le serveur

1. **Ouvrir une session SSH vers le serveur**

2. **Lancer l'installation avec le script officiel**

```bash
curl -fsSL https://tailscale.com/install.sh | sh
```

3. **Démarrer le réseau**

```bash
sudo tailscale up
```

4. **Ouvrir le lien affiché avec un navigateur web, et se connecter avec le même compte qu'à l'étape 1**

5. **Activer l'exposition HTTPS sur la console d'administration de Tailscale**

Aller dans [l'onglet **DNS**](https://login.tailscale.com/admin/dns), dans la section **HTTPS Certificates**, cliquer sur **Enable https...**

6. **Exposer Open WebUI en HTTPS**

```bash
sudo tailscale serve --bg localhost:3000
```

7. **Vérifier le fonctionnement depuis une des machines clientes**

Récupérer le nom du serveur Linux depuis la console d'administration, et taper son nom dans la barre d'adresse d'un navigateur depuis une des machines clientes. Par exemple `https://aiserver.tail123456789.ts.net/`


## 🔹 Étape 3 : Renforcer la sécurité

Cette opération vise à autoriser uniquement le traffic nécessaire au sein du réseau Tailscale.

1. **Créer une étiquette (tag) nommée `server`**

Aller dans [l'onglet **Access control** puis **Tags**](https://login.tailscale.com/admin/acls/visual/tags). Cliquer sur **Create tag**, et saisir `server` et valider.

2. **Assigner l'étiquette `server` au serveur Linux**

Aller dans [l'onglet **Machines**](https://login.tailscale.com/admin/machines), au niveau du serveur Linux, cliquer sur **...** puis **Edit ACL tags...**. Saisir `server` et valider.

3. **Modifier les règles par défaut pour autoriser uniquement le traffic vers le serveur**

Retourner dans [l'onglet **Access control**](https://login.tailscale.com/admin/acls/visual/general-access-rules). Sur la seule règle, cliquer sur **...** puis **Edit**.

Supprimer la valeur dans **destination**, et saisir `tag:server` à la place.

Supprimer la valeur dans **Port and protocol**, et saisir `tcp:443`, puis `tcp:22` puis `tcp:11434` à la place.

Le résultat attendu est :
```json
// Allow access only to servers (ssh, https and ollama)
{
	"src": ["*"],
	"dst": ["tag:server"],
	"ip":  ["tcp:443", "tcp:11434", "tcp:22"]
}
```

Enregistrer avec **Save grant**