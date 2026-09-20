# Études techniques et arbitrages — état initial

> Étude préliminaire : aucune mesure sur la carte finale, aucune validation du modèle d'écran, du lecteur SD ni des débits obtenus.

## 1. Pourquoi ESP32-WROOM-32 peut convenir au MVP
Le module ESP32-WROOM-32 classique offre CPU double cœur, périphériques I²S/SPI/I²C et une SRAM limitée, sans PSRAM garantie selon la variante. Il convient à la lecture WAV, au mixage simple et à une analyse BPM pré-calculée si les fichiers sont traités en flux et si l'interface reste légère. Il ne faut pas promettre une analyse avancée multi-algorithme ou un time-stretch de qualité avant tests de performance.

**Risques :** manque de RAM avec framebuffer graphique + décodeur + buffers audio, latence SD partagée avec écran, interruptions de l'encodeur, bruit d'alimentation, blocage par mises à jour GUI. Réponses : buffers bornés, rendu partiel, mesures de heap minimale, priorité audio, arbitrage du bus et tests de stress.

## 2. Comparaison des chaînes audio
| Option | Atout | Limite | Décision |
|---|---|---|---|
| Module MP3 autonome type DFPlayer | lecture simple commandable en série | audio/mixage/analyse/synchronisation moins directement pilotables depuis ESP32 | non retenu pour le MVP |
| microSD → décodage ESP32 → mixage → PCM5102A | même horloge pour musique et annonces; contrôle des volumes/phase | consommation CPU/RAM et nécessité de gérer SD | architecture cible |

## 3. Analyse BPM : difficultés connues
L'estimation des battements à partir d'énergie et d'attaques donne des hypothèses, pas une garantie de mesure ni du premier temps. Cas problématiques : intro silencieuse, contretemps, triolets, tempos variables, percussion faible, signatures différentes, estimation double/demi tempo et morceaux avec swing. L'écran et l'encodeur fournissent une correction humaine explicite et contrôlée. La précision s'évalue sur des morceaux-tests, pas seulement à partir de la valeur BPM affichée.

## 4. Alignement : étude d'ergonomie
Affichage d'une enveloppe compacte préparée lors de l'analyse, zoom contextuel, grille 1–2–3–4, déplacement d'un marqueur de premier temps au doigt puis finement à l'encodeur. Le clic de l'encodeur sélectionne le paramètre (phase, BPM, zoom); éviter les commandes cachées compliquées. Possibilité future d'un bouton « TAP » pour marquer un temps pendant écoute. Tester que l'utilisateur puisse corriger un morceau en moins de quelques manipulations, sans connaître la théorie du signal.

## 5. Écran capacitif 3,5" : choix non résolu
Le composant est déjà acheté mais **non identifié**. Déterminer interface TFT (SPI ou bus parallèle), contrôleur tactile (I²C ou autre), RAM graphique nécessaire, broches partagées et lecteur SD éventuel. Un écran complet ESP32 intégré n'est pas équivalent à un écran nu raccordé au WROOM. Sans ces données il serait dangereux de publier un câblage final ou de choisir une bibliothèque graphique définitive.

## 6. Performance et critères de décision
Mesurer en priorité débit SD effectif, durée maximale entre fourniture de blocs audio, fréquence des underruns, heap libre/minimale, CPU par tâche, temps de rendu UI, latence voix-vers-sortie, décalage sur 5–10 minutes et comportement après coupure SD. Si lecture + écran échouent, réduire fréquence de rafraîchissement/résolution graphique, augmenter les buffers dans la limite de RAM et revoir le partage de bus avant d'envisager du matériel supplémentaire.

## Références techniques primaires à consulter lors de l'implémentation
- ESP32-WROOM-32 datasheet : https://www.espressif.com/sites/default/files/documentation/esp32-wroom-32_datasheet_en.pdf
- Espressif ESP-IDF, périphérique I²S : https://docs.espressif.com/projects/esp-idf/en/stable/esp32/api-reference/peripherals/i2s.html
- Espressif ESP-IDF, SPI master : https://docs.espressif.com/projects/esp-idf/en/stable/esp32/api-reference/peripherals/spi_master.html
- TI PCM5102A (datasheet et alimentation suivant le montage) : https://www.ti.com/product/PCM5102A

Les documents fournisseur concernent les composants, pas automatiquement le brochage des modules marchands. Contrôler la révision et la compatibilité framework utilisées.
