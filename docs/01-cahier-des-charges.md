# Cahier des charges — AZ-1-D

## Public et usage
Appareil autonome et simple pour professeurs de danse, utilisable en cours sans ordinateur ni réseau. Une musique personnelle sur microSD est lue sur une enceinte active ou console via la sortie ligne du DAC. La voix pédagogique peut compter 1–2–3–4, et plus tard 1–8, sans modifier le fichier source.

## Parcours utilisateur cible
1. Sélectionner un morceau sur la carte et lancer « Analyser » lors du premier usage.
2. Consulter BPM estimé, grille rythmique et enveloppe de forme d'onde.
3. Écouter avec voix/clic, corriger la phase du temps 1 au tactile et à l'encodeur; ajuster éventuellement le BPM.
4. Valider et sauvegarder les repères associés au morceau.
5. Aux usages suivants, charger directement ces données sans réanalyse, tout en permettant une nouvelle analyse manuelle.
6. Régler indépendamment le volume musique et celui de la voix; choisir comptage actif/muet.

## Fonctions MVP (à valider sur matériel)
- WAV PCM 16 bits mono/stéréo, échantillonnage pris en charge explicitement; démarrer à 44,1 kHz.
- Lecture stable depuis microSD avec transport lecture/pause/retour début.
- Pré-analyse séquentielle, estimation BPM sur musique à tempo stable et enveloppe réduite.
- Grille 4/4, position du temps 1 éditable en millisecondes et ajustement BPM fin.
- Voix WAV « un, deux, trois, quatre » mixée numériquement, niveau séparé et protection contre l'écrêtage.
- Interface tactile 3,5" + KY-040 (rotation, clic, appui long si fiable), sauvegarde par morceau.
- Sortie stéréo niveau ligne vers enceinte **active**, pas directement vers haut-parleur passif.

## Hors MVP
- Détermination garantie des mesures et de la signature rythmique pour tout type de morceau.
- Time-stretch de qualité sans modifier la hauteur.
- Détection robuste en temps réel de variations de tempo, MP3 garanti, Bluetooth, Wi-Fi et synchronisation réseau.
- Enregistrement ou redistribution de morceaux commerciaux.

## Critères d'acceptation initiaux
- Lecture complète sans underruns audibles pendant navigation tactile et rotation de l'encodeur.
- Décalage sauvegardé et rechargé sans dérive imprévue du comptage sur plusieurs minutes.
- Correction du « 1 » accessible pendant écoute, avec retour auditif immédiat et sauvegarde explicite.
- En cas de carte absente/fichier invalide : message clair et sortie audio silencieuse.
- Les tolérances chiffrées de précision et de latence seront fixées après mesure de la première maquette.
