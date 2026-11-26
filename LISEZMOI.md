# Mes heures — appli de saisie des heures (Sinthylène)

Webapp installable (PWA) : chaque salarié saisit ses heures du jour sur son téléphone, revoit ses mois passés et envoie sa feuille du mois par email. **Aucun serveur, aucune base de données** : les heures restent dans le téléphone.

## Contenu

| Fichier | Rôle |
|---|---|
| `.nojekyll` | Fichier vide : GitHub publie les fichiers tels quels |
| `index.html` | L'appli complète (saisie, mois, réglages, espace responsable) |
| `manifest.webmanifest` | Nom, icône, couleur : permet « Ajouter à l'écran d'accueil » |
| `sw.js` | Fait fonctionner l'appli sans réseau |
| `icon-192.png`, `icon-512.png`, `apple-touch-icon.png` | Icônes |

## Mise en ligne (une seule fois, ~15 min)

L'appli doit être servie en **HTTPS** (obligatoire pour l'installation et le mode hors ligne). Hébergement statique gratuit, au choix :

1. **GitHub Pages** : nouveau dépôt (ex. `mes-heures`), déposer les 6 fichiers à la racine, Settings → Pages → Branch `main` / root. Adresse obtenue : `https://<compte>.github.io/mes-heures/`.
   ⚠ Avec un compte GitHub gratuit, le dépôt doit être public pour activer Pages. Les fichiers ne contiennent aucune donnée personnelle : la démo utilise un salarié fictif, et les prénoms et horaires que vous saisissez dans l'espace responsable restent dans votre navigateur. Ne déposez jamais de feuille d'heures ni de CSV reçu dans le dépôt.
2. **Netlify Drop** (app.netlify.com/drop) : glisser le dossier, adresse immédiate.
3. Un sous-domaine du site Sinthylène (ex. `heures.sinthylene.fr`) si votre hébergeur web le permet.

Pour une mise à jour : remplacer `index.html` et augmenter le numéro de `CACHE = 'mes-heures-v4'` dans `sw.js` (v3, v4…).

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
- **Mois du salarié = mois calendaire** : la feuille de novembre couvre du 1er au 30 novembre (jours, heures, absences).
- **Heures supplémentaires = semaine complète** (lundi → dimanche), rattachée à la paie du mois de son dimanche, comme le demande le comptable : la semaine à cheval sur la fin du mois est comptée sur la paie suivante. Ventilation au-delà de 35 h : tranche 35→39 h et au-delà de 39 h. Seules les heures travaillées comptent (congés, absences, fériés = 0 h).
- Pour que chaque feuille se suffise à elle-même, la 1re semaine reprend les jours de fin du mois précédent en lignes « Rappel » (ex. feuille de novembre : 26 → 31 octobre), utilisées seulement pour le calcul des heures sup.

## Saisie en avance et régularisations

- Une journée saisie **avant sa date** (ex. vendredi et lundi saisis le jeudi pour boucler le mois) est enregistrée **en provisoire** : cadre en pointillés, mention « provisoire ».
- Le jour venu, l'appli demande de la **confirmer** (un appui) ou de la corriger. La valeur saisie en avance reste en mémoire.
- À chaque envoi, l'appli retient ce qui a été transmis pour chaque journée. Si une journée d'une feuille déjà envoyée change ensuite (provisoire corrigé, oubli complété, erreur), elle apparaît dans **Régularisations** et part automatiquement avec la feuille suivante : date, mois d'origine, valeur envoyée, valeur réelle, écart.
- Dans le CSV : colonne `Statut` (Confirmé, Provisoire, À venir, Non saisi, Rappel, Régularisation), colonne `Paie HS` (mois de paie où la semaine compte) et, pour les régularisations, `Travaillé transmis` / `Type transmis`.
- Dans l'espace responsable, les régularisations reçues sont listées à part, avec un bouton de copie pour Excel.

## Envoi de la feuille du mois

Bouton **Envoyer ma feuille** dans « Mon mois » :
- Sur la plupart des téléphones : partage natif → choisir Mail/Outlook/Gmail → le fichier `Heures_2026-10_Prenom.csv` est joint et le récapitulatif écrit dans le message.
- Sinon : email pré-rempli (destinataire, objet, récapitulatif + bloc `#DONNEES`) et bouton pour télécharger le CSV.

## Regroupement des feuilles reçues

Espace responsable → **Regrouper les feuilles reçues** : sélectionner les CSV reçus (ou coller le texte des emails). L'appli affiche par salarié : jours saisis / non saisis, heures travaillées, H+, H−, HS 35→39, HS >39, jours de CP, maladie, absence. Boutons pour copier la synthèse ou le détail et les coller dans Excel.

Format CSV (séparateur `;`, décimale `,`, UTF-8) :
`Salarié;Mois;Semaine;Date;Jour;Type;Prévu;Travaillé;H+;H−;Observation;Statut;Travaillé transmis;Type transmis;Paie HS`

## Limites à connaître

- Données uniquement sur le téléphone : téléphone perdu ou appli réinitialisée = heures non envoyées perdues. La feuille envoyée chaque mois sert de copie ; Réglages → Sauvegarde permet aussi d'exporter / restaurer.
- Pas de rappel automatique quotidien (impossible de façon fiable sans serveur, surtout sur iPhone).
- Pas de validation par le responsable dans l'appli : le contrôle se fait à réception, comme aujourd'hui.
