# Architecture audio, comptage et sauvegarde

## Deux modes exclusifs
**Préparation / analyse :** lecture séquentielle du fichier, décodage PCM par blocs, extraction d'enveloppe, calcul d'onsets et estimation BPM; écran d'édition et sauvegarde. Allouer/réutiliser les buffers pour respecter la RAM du WROOM.

**Cours / lecture :** chargement des métadonnées, décodage musique → ring buffer → mixage échantillons de voix WAV selon grille temporelle → I²S DMA → DAC. Le calcul BPM intensif est arrêté. L'UI ne doit jamais bloquer le chemin audio.

## Horloge et alignement
- Représenter la position musicalement significative en **index d'échantillon audio**, et non via un timer UI.
- BPM constant : intervalle d'échantillons par temps `samples_per_beat = sample_rate * 60 / bpm`, en précision fractionnaire pour éviter une dérive par arrondis.
- Position du temps `n` : `phase_samples + n * samples_per_beat`. Accents et annonces suivent cette grille, avec compensation éventuelle de latence calibrée.
- Lors d'un changement de BPM/phase en cours de lecture, appliquer une transition définie (au prochain bloc ou temps) sans double-déclenchement vocal; enregistrer seulement après validation.
- L'estimation BPM peut tomber à ×2/÷2; proposer une correction simple, puis un repérage manuel du premier temps. Le 4/4 n'est pas déduit de façon garantie par l'algorithme.

## Analyse initiale (hypothèse de travail)
1. Lire et décoder une représentation mono réduite de la musique en streaming.
2. Calculer énergie/flux d'attaque par fenêtres, conserver enveloppe min/max ou RMS réduite pour affichage.
3. Estimer des périodicités d'attaques dans une plage BPM définie; retourner BPM + confiance indicative.
4. Construire une grille provisoire et demander à l'utilisateur de confirmer la phase du temps 1 et le BPM.
5. Tester en écoute en début, milieu et fin de morceau : le BPM faux de 0,1 peut entraîner une dérive sensible.

## Mixage
- Échantillons vocaux mono PCM 16 bits préchargés en RAM **si la mémoire le permet**, sinon lecture par blocs avec cache; mesurer l'espace restant avant de choisir.
- Mixage en accumulateur signé 32 bits, gains musique/voix séparés, saturation douce ou limitation contrôlée vers 16 bits; pas d'addition brute risquant un écrêtage.
- Définir le routage mono/stéréo des annonces; limiter le volume lors de l'allumage et de la pause.
- Audio prioritaire : écran rafraîchi de façon partielle/limitée, tâches SD arbitrées, aucun accès SD bloquant dans une interruption audio.

## Métadonnées par morceau
Créer un fichier associé (ou un index) comprenant version de schéma, chemin relatif SD, taille + empreinte/date si disponible pour détecter les modifications, sample rate, BPM estimé et confirmé, premier temps en échantillons, mode 4/4, gain voix, gain musique, format d'enveloppe, indicateur validation et éventuelles zones de tempo.

Conserver la version des métadonnées pour migration. Sauvegarde atomique par fichier temporaire + renommage lorsque le système de fichiers le permet; ne jamais écraser le fichier musical.

## Formats
MVP : WAV PCM 16 bits documenté. MP3 : analyser après décodage avec compensation de délai d'encodeur/décodeur pour que les repères correspondent aux échantillons réellement écoutés. Modifier la vitesse sans changement de hauteur reste une fonction séparée à évaluer.
