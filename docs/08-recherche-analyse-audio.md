# Étude des bibliothèques audio et des algorithmes BPM — 20 septembre 2026

> **Statut : veille documentaire, aucune bibliothèque tierce copiée, compilée ou validée sur la carte.** Le projet reste conçu pour ESP32-WROOM-32 DevKit sans PSRAM présumée. Sources et licences à recontrôler à la version exacte utilisée avant intégration.

## Besoin précis
Détecter, **une seule fois à l'import**, le tempo probable (BPM) d'un morceau, les attaques rythmiques et une enveloppe graphique. Le professeur vérifie ensuite au casque/enceinte le placement du premier temps via écran capacitif 3,5 pouces et encodeur KY-040, puis valide BPM et phase. En lecture normale, aucune analyse lourde : les comptes vocaux WAV sont calés sur la position exacte des échantillons transmis au DAC PCM5102A.

**Distinction essentielle** : BPM ≠ positions des battements ≠ premier temps de la mesure (downbeat). Une impulsion de grosse caisse n'est pas forcément le « 1 ». L'estimation peut donner BPM/2 ou BPM×2. Une musique swing, syncopée, avec silence initial ou tempo variable peut invalider une grille fixe. L'édition humaine est une fonction centrale, pas une simple rustine.

## Sources GitHub vérifiées et stratégie de réutilisation

| Référence | Apports à étudier | Contraintes / décision |
|---|---|---|
| [Espressif esp-dsp](https://github.com/espressif/esp-dsp) | FFT optimisée, filtrage, opérations vectorielles adaptées à ESP32; [documentation FFT](https://github.com/espressif/esp-dsp/blob/master/modules/fft/include/dsps_fft2r.h) | Apache-2.0 d'après README; **candidat à intégrer** en dépendance ESP-IDF si FFT/flux spectral bat l'approche énergétique en benchmark. Ce n'est **pas** un détecteur BPM complet. |
| [aubio](https://github.com/aubio/aubio) | Détection d'attaques/onsets, tempo, beat tracking; [tempo.c](https://github.com/aubio/aubio/blob/master/src/tempo/tempo.c), [beattracking.h](https://github.com/aubio/aubio/blob/master/src/tempo/beattracking.h), [exemple aubiotrack.c](https://github.com/aubio/aubio/blob/master/examples/aubiotrack.c) | GPL-3.0-or-later selon dépôt/fichiers : **référence algorithmique et éventuel banc d'essai PC**. Ne pas recopier son code dans un firmware propriétaire sans décision explicite de conformité GPL; intégration sur WROOM non démontrée. |
| [Essentia](https://github.com/MTG/essentia) | [RhythmExtractor](https://github.com/MTG/essentia/blob/master/src/algorithms/rhythm/rhythmextractor.h) expose BPM, positions de battements, paramètres de cadence et hints | AGPL-3.0-or-later selon fichiers; framework C++ important. **Référence comparative hors appareil**, pas un « drop-in » ESP32. |
| [madmom](https://github.com/CPJKU/madmom) | Algorithmes spécialisés de beat/downbeat et modèles appris | Python et modèles de réseau neuronal; code BSD selon README, **modèles/données CC BY-NC-SA 4.0** avec contraintes commerciales. **Analyse comparative sur PC uniquement**, pas embarquement de ses modèles ni présomption de droit commercial. |
| [librosa](https://github.com/librosa/librosa) | [Beat tracking](https://librosa.org/doc/latest/generated/librosa.beat.beat_track.html) pour comparer les sorties d'un prototype sur PC | Python/NumPy/SciPy, licence ISC indiquée dans [LICENSE.md](https://github.com/librosa/librosa/blob/main/LICENSE.md); **banc d'essai PC**, pas moteur ESP32. |
| [arduinoFFT](https://github.com/kosme/arduinoFFT) | FFT alternative si choix Arduino plutôt qu'ESP-IDF | GPL-3.0 selon README et en-têtes, et ne fournit pas seule BPM/downbeat; ne pas confondre disponibilité d'une FFT et capacité à caler « 1 ». |
| [schreibfaul1/ESP32-audioI2S](https://github.com/schreibfaul1/ESP32-audioI2S) | Exemple d'audio SD/MP3 et sortie I²S vers PCM5102A | Documentation actuelle demande **au moins 2 Mo de PSRAM** et limite le WROOM classique; GPL-3.0 : **ne pas sélectionner sans preuve de compatibilité sur notre module**. WAV PCM + mixeur minimal reste le MVP. |

**Licence ≠ simple attribution** : GPL/AGPL imposent potentiellement des obligations fortes pour une distribution du firmware ou du produit. Consigner version exacte, licence, notices, modifications et dépendances transitives avant tout import; aucune copie de source tierce à ce stade. Les idées et formules mathématiques générales peuvent guider une implémentation indépendante, sans reprendre le code d'autrui.

## Proposition de moteur natif AZ-1-D v0 : zéro dépendance lourde
1. **Lire le WAV en streaming** et convertir en mono pour l'analyse; décimation/filtre anti-repliement adaptés si réduction de fréquence; conserver l'index d'échantillons source. Pas de chargement intégral du morceau en RAM.
2. **Extraire une enveloppe compactée** RMS/énergie et min/max par fenêtres, pour l'affichage zoomable sur écran 3,5 pouces. Écrire les points agrégés sur SD; conserver un petit cache en mémoire.
3. **Détecter les attaques** : énergie courte vs bruit de fond mobile et seuil adaptatif, puis comparer avec le flux spectral (FFT esp-dsp) seulement si le premier modèle échoue sur le banc d'essai. Normaliser pour éviter que des passages forts dominent toute l'analyse.
4. **Estimer plusieurs hypothèses de tempo** (par exemple 60–200 BPM puis ×2/÷2), par autocorrélation des attaques et contrôle de régularité. Calculer un indicateur de confiance interne; ne pas présenter une fausse précision au centième.
5. **Phase : proposer, ne pas imposer** le premier battement; l'utilisateur écoute et déplace le marqueur via tactile + KY-040. Sélection « décaler le 1 d'un temps », correction ms, BPM fin, écoute début/milieu/fin pour déceler une dérive.
6. **Sauvegarder** BPM validé, premier temps exprimé en échantillons source, signature 4/4 choisie, forme d'onde réduite, format et identité du morceau. La lecture conserve une horloge à phase fractionnaire pour éviter la dérive par arrondi.
7. **Pendant le cours**, aucune FFT ni autocorrélation. Le mixeur injecte les WAV des nombres avec volume séparé en fonction de l'index audio; priorité au buffer I²S et au chemin SD.

### Recommandation d'architecture logicielle
- Prévoir une interface `IBeatAnalyzer` et deux implémentations conceptuelles : `EnergyOnsetAnalyzer` léger et `SpectralFluxAnalyzer` avec esp-dsp **optionnel** après benchmark.
- Séparer `TrackAnalysis`, `BeatGridEditor`, `TrackMetadataStore` et `AudioBeatScheduler` pour pouvoir remplacer l'algorithme sans refaire l'interface ni le lecteur.
- Pas de modèle neuronal embarqué, de « BPM garanti » ou de reconnaissance fiable des mesures annoncée sans tests.

## Banc d'essai à créer avant intégration
Générer des WAV libres de droits avec attaques connues à 60, 90, 120, 160 BPM; silence initial, intro décalée, batterie syncopée, accent hors temps, passages faibles, tempo qui varie et 3/4 hors périmètre. Comparer BPM/2/BPM×2, erreurs d'attaques, temps d'analyse par minute de musique, heap minimale et qualité de phase après 5–10 min. Une simulation PC peut confronter notre algorithme à aubio/librosa; les mesures finales devront se faire sur le **WROOM et l'écran réels**.

## Priorités et blocages
1. Identifier modèle et brochage exacts de l'écran pour arbitrer RAM et bus SD.
2. Implémenter lecteur WAV + enveloppe énergétique, **sans FFT**, obtenir une mesure de performance sur WROOM.
3. Ajouter autocorrélation + candidats BPM et sauvegarde, puis éditeur phase.
4. Ajouter esp-dsp/flux spectral uniquement si le jeu de tests montre un gain mesuré.

**Aucun dépôt tiers n'a été « récupéré »/vendorié dans AZ-1-D** : étude, liens sources et plan de sélection seulement. Le code tiers n'entrera que lorsque sa licence, sa version et sa compatibilité matérielle seront validées.
