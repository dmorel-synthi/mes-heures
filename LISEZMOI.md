# Mes heures — appli de saisie des heures (Sinthylène)

Webapp installable (PWA) : chaque salarié saisit ses heures du jour sur son téléphone, revoit ses mois passés et envoie sa feuille du mois par email. **Aucun serveur, aucune base de données** : les heures restent dans le téléphone.

## Contenu

| Fichier | Rôle |
|---|---|
| `index.html` | L'appli complète (saisie, mois, réglages, espace responsable) |
| `manifest.webmanifest` | Nom, icône, couleur : permet « Ajouter à l'écran d'accueil » |
| `sw.js` | Fait fonctionner l'appli sans réseau |
| `icon-192.png`, `icon-512.png`, `apple-touch-icon.png` | Icônes |

## Mise en ligne (une seule fois, ~15 min)

L'appli doit être servie en **HTTPS** (obligatoire pour l'installation et le mode hors ligne). Hébergement statique gratuit, au choix :

1. **GitHub Pages** : nouveau dépôt (ex. `mes-heures`), déposer les 6 fichiers à la racine, Settings → Pages → Branch `main` / root. Adresse obtenue : `https://<compte>.github.io/mes-heures/`.
   ⚠ Avec un compte GitHub gratuit, le dépôt doit être public pour activer Pages. Le code ne contient aucune donnée personnelle (les prénoms et horaires de l'espace responsable sont stockés dans votre navigateur, pas dans le fichier), mais vérifiez avant de publier.
2. **Netlify Drop** (app.netlify.com/drop) : glisser le dossier, adresse immédiate.
3. Un sous-domaine du site Sinthylène (ex. `heures.sinthylene.fr`) si votre hébergeur web le permet.

Pour une mise à jour : remplacer `index.html` et changer `CACHE = 'mes-heures-v1'` en `v2` dans `sw.js`.

## Déploiement auprès des salariés

1. Ouvrir `https://…/index.html#admin` sur votre ordinateur → **Espace responsable**.
2. Renseigner l'adresse de réception, puis l'horaire prévu de chacun (lun → dim, ex. `9h45`, `0` si non travaillé).
3. **Copier le lien** (ou **SMS**) de chaque salarié et le lui envoyer.
4. Le salarié ouvre le lien → son téléphone est configuré → il installe l'appli :
   - **Android (Chrome)** : menu ⋮ → « Installer l'application ».
   - **iPhone (Safari)** : Partager → « Sur l'écran d'accueil ». Sur iPhone, l'appli installée a un stockage séparé de Safari : à la première ouverture, coller le lien dans « J'ai reçu un lien » (copier le lien depuis le SMS).
5. Changement d'horaire (contrainte de production, avenant) : modifier l'horaire, choisir la **date d'effet**, renvoyer le lien. Les heures déjà saisies sont conservées et les jours antérieurs gardent l'ancien horaire.

## Règles de calcul

- Le salarié saisit les **heures réellement travaillées** (pré-remplies avec l'horaire prévu) ou une absence : Congé payé, Maladie, Absence non payée, Férié, Autre.
- H+ / H− = écart avec l'horaire prévu du jour. Une absence non payée compte l'horaire prévu en H−.
- **Mois de paie** = semaines complètes du lundi au dimanche dont le dimanche tombe dans le mois (la semaine à cheval sur la fin du mois part sur la paie suivante). Ex. octobre 2026 = du 28/09 au 25/10.
- Par semaine : heures au-delà de 35 h ventilées en tranche 35→39 h et au-delà de 39 h. Seules les heures travaillées comptent (congés et absences = 0 h).

## Envoi de la feuille du mois

Bouton **Envoyer ma feuille** dans « Mon mois » :
- Sur la plupart des téléphones : partage natif → choisir Mail/Outlook/Gmail → le fichier `Heures_2026-10_Prenom.csv` est joint et le récapitulatif écrit dans le message.
- Sinon : email pré-rempli (destinataire, objet, récapitulatif + bloc `#DONNEES`) et bouton pour télécharger le CSV.

## Regroupement des feuilles reçues

Espace responsable → **Regrouper les feuilles reçues** : sélectionner les CSV reçus (ou coller le texte des emails). L'appli affiche par salarié : jours saisis / non saisis, heures travaillées, H+, H−, HS 35→39, HS >39, jours de CP, maladie, absence. Boutons pour copier la synthèse ou le détail et les coller dans Excel.

Format CSV (séparateur `;`, décimale `,`, UTF-8) :
`Salarié;Mois de paie;Semaine;Date;Jour;Type;Prévu;Travaillé;H+;H−;Observation`

## Limites à connaître

- Données uniquement sur le téléphone : téléphone perdu ou appli réinitialisée = heures non envoyées perdues. La feuille envoyée chaque mois sert de copie ; Réglages → Sauvegarde permet aussi d'exporter / restaurer.
- Pas de rappel automatique quotidien (impossible de façon fiable sans serveur, surtout sur iPhone).
- Pas de validation par le responsable dans l'appli : le contrôle se fait à réception, comme aujourd'hui.
