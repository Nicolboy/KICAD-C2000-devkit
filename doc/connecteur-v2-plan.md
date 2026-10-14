# Plan v2 du connecteur devkit — proposition non construite

**Statut : proposition, rien n'est construit.** Ni `devkit_B_F28P551.kicad_sch`/
`.kicad_pcb`, ni `devkit_A_F280037`, ni la bibliothèque
`lib/C2000_Devkit_Connectors.kicad_sym`, ni le connecteur miroir côté shield
(`KICAD-C2000-powershield/connecteur_devkit.kicad_sch`) n'ont été modifiés pour
ce plan. Ce document capture une décision de conception prise par discussion,
à construire lors d'une prochaine révision matérielle — `devkit_B_F28P551`
actuel (routé, prêt pour production) n'est pas concerné tant que ce plan n'est
pas implémenté.

## Pourquoi un nouveau connecteur

La décision #13 (voir `decisions.md`) a fait passer la rangée A de
`devkit_B_F28P551` de dix masses pour dix-huit signaux à **six masses pour
vingt-deux** : `Vin`/`Vout` et `Iin`/`Iout` se retrouvent groupés par paire
sans masse entre eux. C'est le point de départ de cette réflexion : comment
retrouver une masse proche de chaque signal, sans reproduire ce
rétrécissement à la prochaine extension.

## Principe retenu : deux connecteurs 1×24 par domaine, déphasés

Au lieu d'une seule rangée alternant signal/masse le long de sa longueur (ce
que font déjà les quatre nappes shield↔carte de puissance, `nappes-shield.md`),
**chaque domaine (A-analogique, B-numérique) est porté par deux connecteurs
1×24 côte à côte, à pas 2,54 mm entre les deux rangées** : un connecteur
« signal-phase », un connecteur « masse-phase », en opposition stricte
position par position. Résultat : chaque signal a une masse des deux côtés
dans sa propre rangée **et** une masse en face, sur la position jumelle de
l'autre connecteur — proche du damier idéal, avec de simples connecteurs
génériques 2,54 mm (aucune pièce exotique).

**Risque non résolu, à traiter avant de router.** Les deux connecteurs
jumeaux d'un même domaine (`A-sig`/`A-gnd`, `B-sig`/`B-gnd`) sont de taille
identique entre eux (24/24) — rien n'empêche de les brancher à l'envers l'un
par rapport à l'autre, ce qui inverserait toutes les phases signal/masse
d'un coup, y compris le +5V du domaine B sur une broche censée être une
masse. C'est exactement le scénario que la décision #11 avait dû corriger
pour les deux rangées du connecteur actuel — il n'est **pas encore réglé**
ici. Piste envisagée et non retenue : réduire d'une broche le connecteur
masse-phase de chaque domaine (ex. 24/23) pour recréer une asymétrie — perd
une broche de réserve, pas encore décidé si c'est le bon compromis.

## Brochage proposé

Position 1 = extrémité commune aux deux connecteurs d'un domaine.
`(réserve)` = broche physiquement présente, non connectée.

### Domaine A — analogique (48 broches : 23 signaux/+ commandes CLB, 23 masses, 2 réserves)

| Pos | A-SIG | A-GND |
|---|---|---|
| 1 | Vin | GND |
| 2 | GND | Vout |
| 3 | Iin | GND |
| 4 | GND | Iout |
| 5 | V1 | GND |
| 6 | GND | Temp1 |
| 7 | Temp2 | GND |
| 8 | GND | ADC_8 |
| 9 | ADC_9 | GND |
| 10 | GND | ADC_10 |
| 11 | ADC_11 | GND |
| 12 | GND | ADC_12 |
| 13 | I_SHUNT1_CMP | GND |
| 14 | GND | I_SHUNT1_MES |
| 15 | I_SHUNT2_CMP | GND |
| 16 | GND | I_SHUNT2_MES |
| 17 | Stage1_EN | GND |
| 18 | GND | Stage2_EN |
| 19 | HV_EN | GND |
| 20 | GND | Discharge |
| 21 | ~{FAULT_MCU} | GND |
| 22 | GND | VREF_ADC |
| 23 | VREFLO_SENSE | GND |
| 24 | (réserve) | (réserve) |

`~{FAULT_MCU}` (ex-`GPIO_1`, nFAULT) est déplacé du domaine B vers le domaine
A, position 21 — décision explicite, pas une correction : ça rééquilibre les
deux domaines (B passe de zéro réserve à une, A passe de quatre réserves à
deux). Seule réserve : la position 21 se retrouve adjacente à `VREF_ADC`/
`VREFLO_SENSE`, les deux lignes les plus sensibles du connecteur — acceptable
tant que `~{FAULT_MCU}` reste un simple niveau logique (pas de commutation
rapide), à revoir si ça change.

### Domaine B — numérique (48 broches : 23 signaux + 5V, 24 masses, 1 réserve)

| Pos | B-SIG | B-GND |
|---|---|---|
| 1 | +5V | GND |
| 2 | GND | ~{RESET} |
| 3 | BOOT_SEL | GND |
| 4 | GND | PWM1_A |
| 5 | PWM1_B | GND |
| 6 | GND | PWM2_A |
| 7 | PWM2_B | GND |
| 8 | GND | PWM3_A |
| 9 | PWM3_B | GND |
| 10 | GND | PWM4_A |
| 11 | PWM4_B | GND |
| 12 | GND | UART_TX |
| 13 | UART_RX | GND |
| 14 | GND | I2C_SCL |
| 15 | I2C_SDA | GND |
| 16 | GND | SPI_CLK |
| 17 | SPI_SIMO | GND |
| 18 | GND | SPI_SOMI |
| 19 | SPI_STE | GND |
| 20 | GND | CAN_TX |
| 21 | CAN_RX | GND |
| 22 | GND | CMP_OUT |
| 23 | CLB_OUT | GND |
| 24 | GND | (réserve) |

B reste à zéro broche de réserve côté signal (23 signaux + 5V = 24/24 sur les
deux connecteurs combinés, une seule réserve côté masse) — un futur signal
numérique obligera à rouvrir ce connecteur.

## Ce qui ne change pas

MCU, alimentation, JTAG, LED, découplage : aucun de ces blocs ne bouge. Seule
la zone du connecteur et ses pistes de raccordement seraient reprises lors de
la construction — pas une reprise du PCB entier.

## Ce qui reste à faire avant de construire

1. Trancher le risque de détrompage intra-domaine (ci-dessus).
2. Mettre à jour `lib/C2000_Devkit_Connectors.kicad_sym`, `gen_devkit.py`,
   `spec-devkit.md`, `brochage-devkit.csv` avec le brochage final retenu.
3. Répercuter sur `KICAD-C2000-powershield/connecteur_devkit.kicad_sch`
   (symboles et labels réels, même reséquencement, pour que les connecteurs
   mâle/femelle restent en accord — piège déjà rencontré à la décision #20
   côté shield).
4. Redessiner la zone connecteur de `devkit_B_F28P551.kicad_pcb` (et, si le
   CLB devient commun aux deux MCU, `devkit_A_F280037`).
