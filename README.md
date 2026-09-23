# Codename Arena Bot

Chaque etudiant lance **son** bot. Quand quelqu'un poste un prompt d'attaque dans le salon commun, tous les bots le recoivent, le soumettent a l'agent Ollama local de leur proprietaire et repondent dans le salon predefini de ce proprietaire, avec un verdict 🚨 FUITE / 🛡️ TENU.

## Installation

1. Prerequis : Node >= 20.12, [Ollama](https://ollama.com) installe, puis `ollama pull granite3.1-moe:3b`.
2. Portail developpeur Discord (https://discord.com/developers/applications) :
   - New Application, puis onglet **Bot** : Reset Token (copie-le).
   - Active **Message Content Intent** (Privileged Gateway Intents).
   - Onglet OAuth2 > URL Generator : scope `bot`, permissions *View Channels*, *Send Messages*, *Add Reactions*, *Manage Messages* et *Read Message History* (pour `!clear`). Ouvre l'URL pour inviter ton bot sur le serveur.
3. Structure des salons : un salon commun `#attaque-all` (cree une fois par l'organisateur, tous les bots l'ecoutent) et un salon par bot (la ou il repond). Recupere les IDs (Parametres > Avance > Mode developpeur, puis clic droit sur le salon > Copier l'identifiant). Pour ton salon, deux options : soit tu le crees et mets son ID dans `RESPONSE_CHANNEL_ID`, soit tu laisses la variable vide et ajoutes la permission *Manage Channels* a ton bot : il creera `#bot-<son nom>` a cote de `#attaque-all`.
4. Dans ce dossier :

```bash
npm install
cp .env.example .env
```

5. Remplis `.env` (token, `ATTACK_CHANNEL_ID`, `RESPONSE_CHANNEL_ID`) et mets ton `Modelfile` a la racine.
6. `npm start`

`{password}` et `{codeword}` dans le Modelfile sont **optionnels** :
- Avec les placeholders et sans rien dans `.env`, le bot genere des valeurs aleatoires a chaque demarrage.
- Avec des mots en dur dans le Modelfile, recopie-les dans `CODEWORD` et `PASSWORD` de `.env` pour avoir le verdict (🚨 codeword, ⚠️ password, 🛡️ tenu). Sans eux, le bot fonctionne mais affiche « ❔ sans verdict ».

Au demarrage, le bot cree le modele Ollama et lance les checks *release* (si codeword et password sont connus) et *utility* (resultat dans la console).

## Les deux modes

| Ou tu ecris | Effet |
|---|---|
| `#attaque-all` | Le prompt part a **tous** les bots, chacun dans une conversation **vierge** (un seul message user, comme dans le sujet). |
| Le salon d'un bot (`#bot-...`) | Tu **continues** la conversation avec ce bot uniquement : il se souvient des tours precedents. Sert a attaquer un bot 1 par 1. |
| `!reset` dans `#attaque-all` | Efface le contexte de **tous** les bots. |
| `!reset` dans le salon d'un bot | Efface le contexte de ce bot seulement. |
| `!clear` dans le salon d'un bot | Supprime **tous les messages** de ce salon (pas le contexte du modele : utilise `!reset` pour ca). Sans effet dans `#attaque-all`. Demande les permissions *Manage Messages* et *Read Message History* au bot. |

Un nouveau message dans `#attaque-all` repart toujours d'un contexte vierge.

## Regles du jeu

- Tout le monde peut attaquer, dans les deux modes. 1500 caracteres max par message.
- Chaque bot ignore les autres bots. Pas de delai d'attente entre deux messages. ❌ = message trop long.
- Le contexte garde les 20 derniers messages (environ 10 tours) : `num_ctx` n'est que de 4096 tokens.
- Les messages passent en file, dans l'ordre : une inference a la fois sur ta machine.

## Securite

- Ne partage jamais ton token et ne commit jamais `.env` (deja dans `.gitignore`).
- Le bot ne donne aucun outil au modele : il ne fait qu'envoyer du texte et poster la reponse.
