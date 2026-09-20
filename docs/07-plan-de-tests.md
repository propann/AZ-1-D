# Plan de tests — protocole de validation

## Matrice de morceaux de test (libres ou générés pour les tests)
- Métronome synthétique 60/90/120/160 BPM, premier temps connu.
- Musique 4/4 à tempo régulier avec silence initial et accent à contretemps.
- Morceau à swing, pulsation faible ou syncopée; intro sans percussion.
- Cas volontairement hors périmètre : tempo variable et 3/4; signaler l'incertitude et permettre correction manuelle.

## Mesures audio
- Tracer les index d'échantillons d'annonce et du fichier source; vérifier intervalles sans accumulation d'erreur d'arrondi.
- Mesurer décalage entre attaque vocale en sortie ligne et repère de référence, y compris en début/milieu/fin.
- Compter les underruns de buffers I²S, la heap minimale, les temps max d'accès microSD.
- Test 30 à 60 minutes avec déplacements tactiles et rotation rapide de l'encodeur; aucune coupure ni plantage attendu.
- Vérifier écrêtage en sommant musique et voix forte; écouter les transitoires après pause/seek.

## Persistance et robustesse
- Sauvegarder paramètres, redémarrer, vérifier identité des réglages et absence de réanalyse.
- Modifier/remplacer le fichier audio : invalider les métadonnées, demander une réanalyse.
- Retirer la SD, fournir WAV corrompu, nom long, fichier vide, coupure durant sauvegarde : erreur claire, pas de crash ni de perte des fichiers originaux.
- Tester alimentation/bruit, connexion enceinte active, absence de 5 V sur les GPIO.

## Condition de publication
Aucune valeur de précision, latence ou compatibilité n'est annoncée avant mesure sur la carte réelle et conservation des résultats de test datés dans le dépôt.
