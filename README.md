# AZ-1-D — lecteur musical pédagogique pour la danse

Projet de machine autonome pour enseignants de danse : sélectionner une musique sur microSD, estimer son tempo une seule fois, ajuster à l'oreille et à l'écran le premier temps, puis superposer un comptage vocal synchronisé (1-2-3-4 ou 1-8). Les réglages sont sauvegardés par morceau.

> **État : étude et conception, aucun firmware ni câblage validé.** Le modèle exact de l'écran capacitif 3,5" reste à identifier. Les GPIO proposés sont provisoires. La détection BPM ne garantit pas la reconnaissance automatique du début de mesure.

## Matériel déjà indiqué
- ESP32-WROOM-32 DevKit (modèle exact de la carte à confirmer).
- Écran tactile capacitif 3,5" déjà acheté (référence, contrôleur et brochage inconnus).
- DAC stéréo I²S PCM5102A (variante et alimentation à confirmer).
- Encodeur incrémental KY-040 avec poussoir.
- microSD : emplacement et interface à confirmer sur l'écran ou avec lecteur séparé.

## Architecture envisagée
`microSD → décodage/lecture ESP32 → mélange numérique musique + échantillons vocaux → I²S → PCM5102A → sortie ligne → enceinte active / console`.

Le mode **préparation** (lecture séquentielle et calcul BPM/enveloppe, édition de l'alignement) est distinct du mode **cours** (lecture fluide et comptage); aucun algorithme d'analyse lourd ne tourne pendant la lecture normale. Le comptage doit être synchronisé à l'horloge des échantillons audio, pas aux rafraîchissements de l'écran.

## Démarrer
1. Lire [le cahier des charges](docs/01-cahier-des-charges.md).
2. Vérifier [le matériel et le câblage provisoire](docs/02-materiel-cablage.md) **avant toute connexion**.
3. Étudier [l'architecture audio et la synchronisation](docs/03-architecture-audio.md).
4. Voir [l'étude de faisabilité et des risques](docs/04-etudes-techniques.md), [l'interface](docs/05-interface-pedagogique.md), [la feuille de route](docs/06-feuille-de-route.md) et [les tests](docs/07-plan-de-tests.md).

## Principes de conception
- MVP : lecture WAV PCM 16 bits, mesure 4/4, tempo quasi constant, voix WAV préenregistrée, décalage manuel, mémorisation par morceau.
- MP3, métriques complexes, tempo variable et time-stretch sans changement de hauteur : extensions après mesures sur la carte réelle.
- Aucune musique ni voix propriétaire n'est redistribuée dans ce dépôt.
- Documenter les mesures et références exactes du matériel avant de figer le schéma.

Licence et choix de framework non arrêtés.
