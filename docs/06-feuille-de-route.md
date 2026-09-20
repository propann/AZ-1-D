# Feuille de route et portes de validation

Chaque étape doit livrer une preuve mesurée avant d'ajouter la suivante.

## Phase 0 — Identification matériel / câblage
- [ ] Photos/référence de l'écran 3,5", de son contrôleur tactile et de son éventuel lecteur SD.
- [ ] Référence exacte DevKit, variante PCM5102A, alimentation, broches disponibles.
- [ ] Tableau complet GPIO sans conflits ni straps sensibles; schéma de masse et niveaux logiques.
**Validation :** alimentation vérifiée, démarrage et programmation ESP32 fiables.

## Phase 1 — Banc audio minimal
- [ ] Firmware de base, SD, lecture WAV PCM 16 bits par blocs.
- [ ] I²S PCM5102A, sortie ligne stéréo, arrêt/pause sans pop excessif.
- [ ] Mélange d'un échantillon « 1 » WAV à une musique test.
**Validation :** lecture longue sans underruns mesurés ni clipping anormal.

## Phase 2 — Interface et commandes
- [ ] Écran tactile + bibliothèque + transport.
- [ ] KY-040 anti-rebond + navigation + réglages; écran non bloquant.
**Validation :** navigation intensive pendant lecture sans coupures sonores.

## Phase 3 — Analyse unique et stockage
- [ ] Enveloppe sonore compacte et estimation BPM hors lecture.
- [ ] Hypothèses demi/double tempo, niveau de confiance indicatif.
- [ ] Métadonnées versionnées par morceau; analyse relancée uniquement sur demande ou fichier modifié.
**Validation :** résultats reproductibles pour un jeu de morceaux de référence.

## Phase 4 — Éditeur de synchronisation
- [ ] Grille 4/4, marqueur de temps 1 tactile, zoom, incrément fin KY-040.
- [ ] Comptage voix 1–4 calé sur l'horloge audio et volumes séparés.
- [ ] Confirmation de phase/BPM et rechargement exact des réglages.
**Validation :** début/milieu/fin écoutables et alignés sur morceaux tests.

## Phase 5 — Fiabilisation et boîtier
- [ ] Mesures charge CPU/RAM, latence, coupures SD et reprise sécurisée.
- [ ] Schéma définitif, nomenclature, photos du montage et guide utilisateur.
**Validation :** séance complète de cours simulée, pas d'interruption audio.

## Extensions différées
MP3 avec index audio exact, 1–8, multilingue, boucle A/B, mode TAP, tempo variable, time-stretch à hauteur constante (à considérer uniquement après benchmark).

**Ne pas coder l'UI définitive ni figer les GPIO avant identification de l'écran.**
