# Matériel, interfaces et pré-câblage

## Inventaire déclaré
| Élément | État | Informations à confirmer |
|---|---|---|
| ESP32-WROOM-32 DevKit | Possédé | variante exacte, broches réellement exposées, flash, alimentation |
| Écran capacitif 3,5 pouces | Possédé | modèle/photo avant-arrière, référence dalle, contrôleurs graphique/tactile, résolution, interface, tensions, microSD intégrée |
| PCM5102A I²S | Possédé | variante du module, régulateur présent, tensions autorisées, sorties |
| KY-040 | Possédé | brochage/qualité des résistances et rebonds |
| MicroSD | À confirmer | lecteur intégré à l'écran ou séparé, bus partagé ou SDMMC |
| Alimentation et boîtier | À définir | consommation écran/rétroéclairage, masse analogique, sorties et isolation mécanique |

## Schéma fonctionnel (non électrique)
`SD → ESP32 (lecteur/analyse/mixeur) → I²S → PCM5102A → sortie ligne`
`Tactile/écran ↔ ESP32 ; KY-040 → GPIO ESP32`.

## Affectation de broches de travail — NON DÉFINITIVE
| Signal | GPIO ESP32 envisagé | Remarques |
|---|---:|---|
| DAC BCK | 26 | I²S, à vérifier selon écran |
| DAC LRCK / LCK / WS | 25 | I²S, à vérifier selon écran |
| DAC DIN | 22 | I²S, à vérifier selon écran |
| KY-040 CLK | 32 | entrée avec tirage adapté |
| KY-040 DT | 33 | entrée avec tirage adapté |
| KY-040 SW | 27 | entrée avec anti-rebond |
| écran / tactile / SD | À affecter | réserver les GPIO et bus seulement après identification de l'écran |

Les fonctions I²S de l'ESP32 peuvent être mappées sur plusieurs GPIO; ce tableau n'est ni un câblage validé ni une promesse de compatibilité avec le futur écran.

## Vérifications électriques avant branchement
- **Masse commune** entre modules, signaux GPIO **3,3 V uniquement** ; ne pas faire sortir 5 V du KY-040 vers l'ESP32.
- Déterminer le **schéma du module PCM5102A concret** : ses broches VIN/3V3 et ses niveaux de signaux ne sont pas interchangeables entre toutes les cartes; ne pas deviner sa tension d'alimentation.
- Vérifier tensions et consommation de l'écran, besoin éventuel de régulateur et contraintes du rétroéclairage.
- Éviter les GPIO de straps au démarrage (GPIO 0, 2, 4, 5, 12, 15 selon carte/usage), GPIO 6 à 11 dédiés à la flash et GPIO 34–39 sans pull-up/down interne. Vérifier également les broches UART utilisées pour programmer/déboguer.
- Un écran SPI avec SD sur bus partagé impose des CS distincts, arbitrage de bus et priorisation de la lecture audio; un écran parallèle peut consommer trop de GPIO. Ne rien figer avant son modèle exact.
- Le PCM5102A fournit une **sortie ligne**, prévoir une enceinte active, un ampli ou une console, et une sortie jack correctement câblée.

## Décisions encore ouvertes
Référence exacte écran/tactile, organisation microSD, budget GPIO, alimentation USB, boîtier, sorties audio, éventuelle carte SDMMC, configuration et mesure de bruit analogique.

**Blocage de la révision finale du câblage : photo ou lien fiche technique de l'écran 3,5 pouces et identification de chaque variante de module.**
