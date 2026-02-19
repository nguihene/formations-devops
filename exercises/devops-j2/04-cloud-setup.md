# 🎯 Exercice 04 : Cloud Setup

> 🟢 Niveau : Débutant | ⏱️ Durée : 45 min

## Objectif

Créer votre premier compte cloud et provisionner une VM.

## Prérequis

- Compte Denv-r
- Navigateur web

## Instructions

Denv-r (Token fourni)

### Étape 1 : Configurer le token (5 min)

```bash
# Token fourni par le formateur
export DENVR_API_TOKEN="votre-token-ici"

# Vérifier
curl -H "apikey: $DENVR_API_TOKEN" https://api.denv-r.com/v1/compute/virtual-machines
```

### Étape 2 : Créer une VM via l'API (20 min)

Voir le dossier `terraform/` du repo pour utiliser Terraform avec Denv-r.

---

## 🧪 Validation

✅ Vous avez réussi si :
- [ ] Vous avez un compte cloud actif
- [ ] Une VM est en cours d'exécution
- [ ] Vous pouvez vous y connecter en SSH
- [ ] Vous avez nettoyé après l'exercice

---

## 💡 Indice

Si la connexion SSH échoue :
1. Vérifiez que la VM est bien démarrée
2. Vérifiez les règles de firewall (port 22 ouvert)
3. Vérifiez que vous utilisez la bonne clé privée

---
