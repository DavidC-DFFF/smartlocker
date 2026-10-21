# Casiers connectés - dossier documentaire

Source : https://speed-up.ai/project/71c84930acbb/workspace

Capture du contenu visible : 17 octobre 2026.


## Finalité

Maquette de deux casiers pour E2, E31 et E32 du Bac Pro CIEL, avec une architecture envisagée de 30 casiers : six rangées de cinq cartes. Chaque carte détecte un objet coupant un faisceau IR et commande une LED. Un seul mécanisme assure le verrouillage global.


## Chaîne d’information

Objet → barrière IR → phototransistor → comparateur à hystérésis → PIOB du DS2413 → bus 1-Wire → ESP32 → Wi-Fi / HTTP-JSON → Flask sur Raspberry Pi. La LED est commandée en retour par PIOA.


## Chaîne d’énergie

Source externe 5 V → diode série → VLOGIC → POWER des rangées et régulateur 3,3 V. DATA est tiré au 3,3 V et alimente le DS2413 en parasite. Source 12 V distincte → protections → solénoïde → MOSFET → PGND.


## Données et appairage

L’identité ROM usine ne donne pas l’emplacement physique. Le système associe numéro, bus et ROM dans la NVS de l’ESP32. Autorisation locale, numéro choisi sur le web, transition absent vers présent unique stable 300 ms, expiration 60 s, enregistrement atomique et confirmation LED.


## Lecture et erreurs

Le bit 2 représente PIOB : haut = faisceau libre, bas = coupé. Vérifier d’abord getPIO()!=0xFF et l’intégrité du statut. Après trois échecs, publier inconnu/non joignable. Une perte de POWER locale peut laisser la ROM joignable et rendre la détection trompeuse.


## Budgets annoncés

Carte : environ 22 mA typiques, provision 35 mA. Deux cartes et centrale : environ 0,30 A, provision ≤0,60 A. Trente cartes et centrale : environ 0,93 A, provision ≤1,60 A sur 5 V ; source choisie 5 V / 3 A. Le Pi et le verrou ne sont pas inclus dans ce budget.


## Protocole proposé

POST /api/v1/telemetry chaque seconde ; champs de version, identification, séquence, appairage, verrou et casiers. Données périmées après 5 s côté Flask. PUT /api/v1/lock idempotent et authentifié ; ordre accepté distinct de position confirmée. POST /api/v1/pair refusé sans autorisation locale.


## Essais avant réalisation

Mesurer les seuils et marges optiques, la portée, la lumière ambiante, les niveaux 1-Wire et les temps de récupération. Tester 1, 2, 5 puis 10 cartes sur nappe réelle (10 en surcharge), trafic Wi-Fi et durée 3 h. Injecter les pannes de bus, POWER, Pi, contact et appairage. Mesurer les températures et transitoires du verrou.


## Statut du dossier

Reconstitution documentaire du contenu visible de Speed-up au 17 octobre 2026. Les notices d’origine sont conservées et séparées des fiches explicatives. Aucun export KiCad natif, nomenclature exhaustive, routage PCB, firmware validé ou essai matériel n’est revendiqué. Les calculs sont ceux du projet et n’ont pas fait l’objet d’un audit indépendant des fiches techniques.


# Architecture générale


## Perimetre et principe

Maquette pedagogique pour E2, E31 et E32, seances de trois heures. Le projet contient une centrale et deux instances de la meme carte de casier, CASIER_A et CASIER_B. Les 28 cartes supplementaires sont des repetitions de cette carte reference, raccordees par le faisceau externe : cinq cartes par rangee sur chacun des six bus. Cette extension est un objectif d'essai, pas une capacite certifiee. Les deux cartes de demonstration occupent la premiere rangee. Les adresses usine DS2413P+ sont conservees ; aucune adresse n'est attribuee selon l'ordre de recherche.

CONTROL centralise les etats par ESP32 et communique en Wi-Fi avec une application Flask sur Raspberry Pi. BUS distribue les six lignes independantes et protege leurs alimentations. POWER fournit le 3,3 V depuis une alimentation externe regulee 5 V. Chaque casier associe le DS2413, la signalisation et une barriere optique a seuils resistifs. LOCK pilote un unique dispositif mecanique assurant le verrouillage global ; ce n'est pas une commande individuelle de 30 serrures. Le Raspberry Pi, ses alimentations, le point d'acces Wi-Fi, le faisceau et le mecanisme sont externes au projet.


## Partition fonctionnelle et masses

POWER garde la regulation et ses condensateurs dans une meme responsabilite de stabilite. CONTROL possede le traitement, l'appairage et la programmation. BUS possede les interfaces longues, leurs tirages et leur distribution, separables du logiciel par les six lignes de GPIO. Le comparateur, le recepteur et ses seuils restent avec le DS2413 de chaque casier pour conserver la responsabilite du bruit et de l'hysteresis locale. LOCK garde la boucle inductive, sa protection et le retour de position. GND est le retour logique/optique commun, suppose calme grace a l'absence de convertisseur a decoupage sur cette maquette. PGND est le retour de puissance du verrou ; LOCK possede son unique liaison par resistance nulle avec GND. Ne pas ajouter de liaison ailleurs, notamment par les alimentations ou le chassis. Deux alimentations externes flottantes sont requises, le raccordement de leurs retours etant assure uniquement ici.


## Hypotheses explicites et informations manquantes

Utilisation en interieur sec, 10 a 35 degres C, sans exigence IP ni certification de securite ou d'antivol. Alimentation logique certifiee TBTS, regulee 5 V +/-2%, 3 A, limitee en courant ; alimentation du verrou separee TBTS 12 V +/-5%, au moins 1,5 A, limitee en courant. Aucune charge de telephone. Les dimensions, la largeur du faisceau, les longueurs de nappes et sections, l'ambiance lumineuse et le dispositif de verrouillage restent a fournir. Hypotheses initiales : nappe d'une rangee <=3 m, derivations <=0,10 m, barriere <=100 mm avec tubes opaques ; toutes ces valeurs sont a verifier. Un guide mecanique oblige tout telephone depose a couper le faisceau. Le resultat signifie objet present, jamais identification d'un telephone.

Hypothese du verrou : solenoide 12 V, <=1 A continu, verrouille lorsqu'alimente, deverrouille lorsqu'il ne l'est plus, contact de position sec isole. LOCK est dimensionne seulement pour ce cas. Courant d'appel, inductance, energie et temps de relachement doivent etre mesures avant raccordement du verrou reel. Une motorisation reversible, une serrure bistable ou 30 electroaimants imposent une revision. Une liberation mecanique manuelle est necessaire ; cette maquette n'est pas un equipement de securite.


## Faisceau, tirage et bilan d'energie

Une nappe principale continue DATA/POWER/GND dessert cinq casiers, chaque carte etant connectee par une seule derivation. Aucun trajet serie a travers les cartes : retirer une carte ne coupe pas la nappe. Aucun terminateur en bout. DATA est tire vers 3,3 V, non vers POWER ; le DS2413 est alimente uniquement de facon parasite par DATA. POWER transporte la tension logique issue du 5 V pour les emetteurs IR, comparateurs et LED. Les sorties PIO sont a drain ouvert ; PIOA absorbe le courant de signalisation et le verrou de PIOB reste toujours a 1 pour l'entree. Le comparateur attaque PIOB par une resistance limitant une contention accidentelle.

Le tirage initial de chaque bus est 2,2 kohms, tolerance 1%, conforme a la plage recommande par le fabricant du DS2413. Le GPIO fonctionne exclusivement en drain ouvert ; aucun niveau haut force. La resistance serie du depart est limitee a 33 ohms. A 3,217 V minimum, la marge sur la tension parasite minimale 2,8 V est d'environ 0,42 V avant chute de ligne : elle doit etre controlee a l'oscilloscope avec cinq cartes et Wi-Fi actif. Standard speed seulement, repos haut et recuperation initiale de 20 us entre bits, extensible. Avec une hypothese de 100 pF/m et 200 pF pour protections et interface, la nappe seule donne environ 0,5 nF et une constante RC de 1,1 us ; ceci exclut les effets dynamiques des esclaves et n'est pas une preuve de capacite. Les 800 pF du DS2413 decrivent le comportement initial de son alimentation parasite, pas une capacite permanente a multiplier par cinq. Valider la charge reelle et le rechargement avant d'augmenter le nombre de cartes.

Budget de conception par carte : environ 22 mA habituel, 35 mA de provision maximale, emetteur IR permanent et signalisation allumee. Le casier ne consomme pas son courant optique sur DATA. Deux cartes : environ 0,30 A a 5 V avec centrale a 0,25 A moyen, et <=0,60 A avec provision de 0,50 A pour la centrale. Trente cartes : environ 0,93 A moyen, <=1,60 A en provision simultanee ; choisir 5 V/3 A, soit 15 W disponibles pour environ 8 W de budget maximal. Chaque rangee reserve 0,175 A pour cinq cartes et possede un fusible rapide de 0,5 A. Le 3,3 V reserve 0,55 A incluant radio et six bus. POWER utilise un regulateur lineaire : environ 0,4 W moyen et <1 W a 0,5 A, sous reserve du cuivre et de la mesure thermique. Une diode d'entree et les pertes du faisceau diminuent POWER ; objectif de 3,8 V minimum sur la carte la plus eloignee, condition d'acceptation a mesurer et non garantie sans longueur/section reelles. Le verrou ajoute au plus 12,6 W sur son alimentation distincte ; le Pi n'est pas inclus.

Le fusible de rangee limite un defaut grave de POWER, sans isoler instantanement un casier. Un court-circuit DATA rend toute sa rangee indisponible, mais les autres bus restent independants. Un contact DATA/POWER peut injecter une tension dangereuse pour l'ESP32 : les protections ESD ne constituent pas une protection garantie contre cette erreur de cablage. Connecteurs etiquetes, test de continuite et de polarite avant alimentation, raccordement hors tension uniquement.


## Proprietaires uniques des acces externes

| Acces | Proprietaire | | Alimentation logique 5 V et retour | POWER | | Six departs de rangee DATA/POWER/GND | BUS | | Derivation de la premiere carte | CASIER_A | | Derivation de la seconde carte | CASIER_B | | Adaptateur UART 3,3 V, reset et boot | CONTROL | | Commutateur d'appairage du panneau, contact sec vers masse | CONTROL | | Alimentation 12 V et retour de puissance | LOCK | | Deux conducteurs du solenoide et contact sec de position | LOCK | Les bornes du Raspberry Pi ne sont pas raccordees aux nappes ; la liaison est radio. Emetteur et recepteur optiques sont les composants de chaque carte de reference ; leur placement final peut necessiter une carte longue ou un montage mecanique de leurs pattes, a confirmer avant implantation, sans changer les valeurs supposees. Les deux cartes possedent leur propre connecteur de derivation ; cela n'est pas un second proprietaire des connecteurs de depart de BUS.


## Logiciel ESP32 et appairage propose

Un gestionnaire par bus serie les transactions, une tache de detection echantillonne chaque casier au moins deux fois par seconde, un filtre temporel exige 300 ms de stabilite, une tache reseau publie les etats sans bloquer les creneaux 1-Wire. Ne pas intercaler journalisation ou appel Wi-Fi dans une transaction. La recuperation de 20 us doit etre appliquee dans le moteur bas niveau apres chaque bit, en adaptant ou choisissant une implementation OneWire qui expose ce reglage ; un simple delai entre commandes ne suffit pas. Les timings fixes de la bibliotheque candidate ne sont pas supposes satisfaire cette condition sans examen et mesure. La bibliotheque TechchatCom OneWire-DS2413-Arduino-Library est candidate, non validee sur ESP32 ; compiler et tester avec versions figees du coeur Arduino ESP32 et de OneWire. getPIO() vaut 0xFF en erreur sans actualiser isError() : tester directement le retour ; le bit 2 represente l'etat physique de PIOB. Pour PIOA actif bas, writePIO(0x02) allume et writePIO(0x03) eteint tout en maintenant PIOB libere. Initialiser chaque module avec 0x03, verifier l'ACK et la coherence du statut. Verifier le CRC de chaque ROM, sa famille et l'integrite du statut/complement ; trois echec consecutifs rendent l'etat inconnu/non joignable, jamais absent. Reessayer avec temporisation et rescan periodique.

Le commutateur autorise un appairage local, le numero 1 a 30 est selectionne dans l'interface web ; conserver l'autorisation locale comme garde. Distinguer inventaire temporaire et table persistante numero/bus/ROM. Observer les modules non enregistres, exiger une transition stable absent vers present unique pendant 300 ms ; si plusieurs candidats, attendre que tous soient revenus absents puis une nouvelle transition unique. Expiration apres 60 s, aucune modification sur erreur ou expiration. Confirmation lumineuse apres ecriture atomique en NVS d'une table versionnee avec controle d'integrite ; verifier l'association au redemarrage. Le remplacement requiert une action explicite sur un numero existant, preservation de l'ancien enregistrement jusqu'a validation unique et sauvegarde du nouveau ; journaliser ancien/nouveau ROM et bus. Ne jamais reaffecter automatiquement sur disparition ou ordre de decouverte.


## HTTP/JSON et verrouillage

Proposition : ESP32 publie chaque seconde POST /api/v1/telemetry vers Flask, avec version, device_id, boot_id, seq, uptime_ms, pairing, lock_command, lock_feedback, lockers contenant number, bus, rom, present (true/false/null), reachable et age_ms. Flask acquitte les sequences, ignore les doublons et marque la centrale hors ligne apres 5 s ; aucune valeur stale n'est presentee comme fraiche. Flask envoie une commande idempotente authentifiee PUT /api/v1/lock sur ESP32, avec request_id, desired (locked/unlocked) et expiration courte. Reponse accepted distincte de confirmed par le contact. Deux secondes sans retour conforme donnent un defaut etat inconnu ; l'absence du contact ne prouve pas une panne identifiable. API d'appairage POST /api/v1/pair avec numero, operation et jeton, refusee sans commutateur. Le reseau local est isole, WPA2, secret partage et filtrage IP ; HTTP clair ne convient pas a un reseau hostile, TLS est a ajouter pour exploitation hors maquette.

Le verrou reste a la derniere consigne locale pendant une perte Wi-Fi tant que l'ESP32 est sain, sans deverrouillage automatique a distance. Au demarrage, reset, watchdog ou perte d'alimentation, la commande materielle est inactive et le mecanisme suppose deverrouille. Reprendre une consigne seulement apres initialisation, pas depuis une ancienne commande reseau. Afficher separement demande, retour et etat inconnu. Le contact sec ferme uniquement en position verrouillee ; contact ouvert signifie non confirme, y compris fil coupe.


## Essais et criteres avant extension

Mesurer niveaux optiques, seuils montants/descendants et temporisation sur deux casiers, faisceau libre et coupe, tailles/orientations de telephone, objets autres, soleil et eclairage secteur, alignement et temperature. Exiger des marges de tension de 0,3 V de part et d'autre des seuils ; les seuils calcules ne garantissent pas le bilan optique.

Sur une nappe reelle, tester successivement un, deux, cinq puis dix modules sur un meme bus en essai de surcharge, sans annoncer dix comme capacite normale. Mesurer haut parasite, bas, temps de montee au seuil utile, recuperation apres zero et reset, erreurs de lecture/ecriture, CRC et reconnexion, avec Wi-Fi intense et LED actives ; viser aucun faux etat sur trois heures et publier le nombre de transactions et erreurs brutes. Si cinq modules ne passent pas, allonger la recuperation, reduire longueur/capacite, essayer 1,5 kohms dans les limites de courant, puis envisager un maitre actif avant de modifier l'architecture.

Injecter absence de carte, DATA bloque bas, POWER coupe, fusible ouvert, reboot pendant appairage, doublon de commande, perte Pi/Wi-Fi, contact verrou coupe et defaut de charge. Un bus en panne ne doit pas bloquer les cinq autres ; les commandes de verrou ne doivent pas dependrere d'une presence erronee. Verifier une disparition comme inconnue et la reprise sans nouvelle affectation. Tester courant d'appel, temperature LDO/transistor, tension minimale au cinquieme casier, transitoires du verrou et retour de position. Dix cartes physiques d'essai sont une etape future ; les deux cartes de cette maquette ne suffisent pas a la realiser.


# BUS - Distribution des six rangées


## Six departs independants

J20 a J25 sont les uniques connecteurs de depart de rangee. Chaque connecteur porte DATA en1, POWER en2 et GND en3. La nappe principale continue est externe et dessert cinq derivations courtes, dont deux cartes de demonstration sur OW1. Les cartes ne portent jamais la continuite de la nappe. Pas de terminaison en bout et raccordement hors tension uniquement. Les six lignes sont independantes, mais partagent les alimentations et GND.


## Dimensionnement des lignes

R20 a R25 tirent DATA vers 3V3, pas vers VLOGIC. R26 a R31 sont des resistances serie de33 ohms entre GPIO et nappe, sans conversion de niveau. Les GPIO sont exclusivement drain ouvert, relaches pour lire/recharger le DS2413. Un seul tirage par bus, aucun tirage sur les cartes. Maximum par ligne a3,3825V avec resistance minimale2178ohms :1,553mA, soit9,32mA pour six lignes bloquees basses. Puissance tirage<5,3mW,0603 largement suffisant. Le33ohms ajoute environ51mV au niveau bas pour1,55mA ; verifier VOL total  <seuil bas et haut> =2,8V au DS le plus eloigne. Alimentation3V3 minimum3,2175V laisse0,4175V pour charges/chutes au repos.

R de tirage2,2k est le point initial recommande ; recovery initial20us, standard speed seulement. D20 a D25 apportent20pF typiques par depart selon le datasheet, plus20pF par carte ; aucune capacitance0,3pF catalogue n'est retenue. Hypothese de cable100pF/m,3m et ensemble d'environ200pF hors cable donne C 0,5nF etRC 1,1us, avant comportements dynamiques des esclaves. Pour un simple RC de0,5nF, un depart de0V a3,2175V atteint2,8V en environ2,25us, mais cette estimation ne garantit pas la recharge parasite ni le seuil d'echantillonnage. Les800pF du DS2413 sont un effet initial, jamais multiplies comme charge permanente. Oscilloscope requis au debut et a la fin du cable, un/deux/cinq puis dix modules en surcharge. Si necessaire augmenter la recuperation, reduire le cable puis tester1,5k dans les limites de sink ; pas de tirage actif ajoute ici.


## Distribution et pannes

F20 a F25 sont des fusibles rapides0,5A/32V sur POWER, provisionnormale0,175A par rangee. Resistance froide typique0,870ohm a25C etfaible courant, chute estimee0,152V a0,175A, pas de maximum garanti. La table de derating indique environ0,49A a35C, donc marge confortable sur charge normale. Le budget local3,8V est une condition d'acceptation a mesurer, avec longueurs et section reelles : ni la chute des fusibles ni celle du cable ne sont bornees sans ces donnees. Les0,5A declares sont le calibre de chaque depart, pas un regulateur de courant ni une promesse3A simultanes ; aggregate normal1,05A, capacite VLOGIC2A.

Les fusibles limitent les consequences d'un defaut grave POWER si le courant de source permet leur ouverture. MeltingI2t typique0,0068A2s au test5A et pouvoir de coupure50A/32V ; comparer les impulsions d'inrush des cartes, aucun temps de coupure systeme garanti. L'alimentation limitee en courant reste necessaire. DATA bloque bas affecte toute sa rangee sans couper les cinq autres, un court-circuit POWER peut faire tomber la tension commune avant fusible. Une permutation DATA/POWER peut endommager l'ESP32 : les TVS ESD ne la rendent pas sure. Etiquetage et detrompage du faisceau, continuite/polarite hors tension avant raccordement.

D20 a D25 sont des protections de manipulation, non une garantie de tenue des circuits a toute decharge ; clamp10Vtyp a9A8/20us est au-dessus de l'absmax GPIO. Ne pas annoncer immunite certifiee. C20 est un reservoir de VLOGIC uniquement ; aucun condensateur sur DATA. Essai trois heures sous trafic Wi-Fi, LED et verrou commutes, mesure des niveaux, recovery et erreurs brutes ; criteres aucun fauxetat, disparition signalee inconnue, reprise sans reappairage et pas de blocage des autres bus.


# LOCK - Verrouillage global


## Hypothese mecanique et commande

Un seul solenoide externe assure le verrouillage global mecanique,12V<=1A continu, energise=verrouille. Ce circuit ne pilote ni moteur reversible ni serrure bistable. J4 recoit l'alimentation flottante12V et son retourPGND. F1 puisD3 protegent le depart LOCK_PLUS contre defaut grave et inversion. Q1 commute LOCK_RETURN versPGND, commande LOCK_CMD activehaut parR6. R7 maintient la grille inactive pendant reset/perte3V3. La premiere action logicielle doit etre consignebas ; reset etwatchdog relachent le verrou. Ne pasrejouer une anciennecommande reseau au demarrage.

D4 recircule le courant inductif a l'arret, avecboucle solenoide/diodela pluscourte possible ; sa cathode rejoint LOCK_PLUS et son anode LOCK_RETURN. Cette roue libre ralentit le relachement : verifier letemps compatibleavec mecanisme ettimeoutdeposition2s. D5 traite les transitoires d'alimentation, non l'energie principale du solenoide. Aucun calculd'energie fiable sansinductance etfrequence ; ces informations etlecourantd'appel sont manquantes. Ne raccorder le verroureel qu'apres leurmesure.


## Marges et retours

Q1 est30V avecRdsmax48mohms aVgs2,5V25C. Lelectrique gate>=3,217V en reposnormal estau-dessus dupointdequalification2,5V ; absence de valeur exacte a3,3V ne justifie pas un meilleurRds. Hypothese conservative chaud0,1ohm : P<=0,1W a1A ethausse~9degC autheta89degC/W de la table, a verifier sur cuivre. En roue libre normale Vds<=12,6+0,55=13,15V environ. Le clamp24,4V duTVS laisse encore la diode et lesinductances de piste a inclure sous30V ; mesurer pour viserVds<26V lorsdes perturbations. D3 peutdissiper0,55W a1A conservativement, D4 porte1A initial a chaque arret avecpertes dependantes deL etfrequence. Borniers8A catalogue etpistes>=1,5A continu. F1 calibre1,5A/32V rapide ; une source limitee peut nepas donner lecourantnecessairepourlefairefondre, donc la limitation amont reste necessaire etaucun temps de coupuregaranti n'est revendique.

PGND contient tout le retourdu solenoide ; R10 estl'unique bond nul avecGND, placepresdelasourceQ1 maisen dehorsdu trajetdecourantdulock. L'alimentation5V,12V etlescontacts doiventetre flottants/non reliesau chassisailleurs. Le courantduverrou ne traversepas la masseoptique/logique ; pasd'autrebondpossible via cablefeedback ou alimentation externe.


## Retour de position

J6 accueille uncontactsecdedie qui ferme LOCK_CONTACT versGND uniquement en positionverrouillee. R8/R9 etC10 forment tirage etfiltrage environ1ms ; niveauferme~0,3V a3,3V, filtre logiciel50ms. LOCK_SENSE basconfirmelaposition ; ouvert n'est paspreuve de deverrouillage etfilcoupeindiscernable. D6 protege contre manipulationESD mais pasinjection12V ; contactsecisole<=1m exclusivement. Si alimentation12V manque, lefeedback peutrester utilisable mais la commande ne fournitaucunenergie. Si3V3 manque, touteinterpretationlogicielle estinvalide. Publier demande/confirmation/etat inconnu distincts. Tester touteslescombinaisons d'alimentation, reset souslock, perteWi-Fi, tension/courant/flyback etcontactcoupe avantusagepedagogique. Liberation mecanique manuelle requise.


# POWER - Alimentation logique


## Fonctionnement

J1 est l'unique entree logique du projet. D1 bloque une inversion de polarite, au prix de la chute de tension de VLOGIC ; il ne protege pas d'une tension d'entree excessive. La source externe doit etre 5 V +/-2%, flottante, limitee a 3 A. U1 genere 3V3 avec reservoir amont et decouplage aval ceramique. Aucun chemin d'alimentation par le connecteur de programmation.


## Dimensionnement

Le budget simultane de trente casiers et centrale est <=1,60 A en entree, dont <=0,55 A sur 3V3. Avec chute de diode conservatrice 0,55 V a 3 A, VLOGIC reste >=4,35 V pour VIN5=4,9 V, avant pistes. La marge LDO reste >=4,3-(3,3825+0,4)=0,5175 V ; 0,4 V est le maximum de dropout donne a 1 A utilise conservativement. U1 fournit 1 A catalogue mais seulement 0,7 A de capacite sont engagees ici, charge prevue 0,55 A. C1 doit conserver >=33 uF a 5,1 V afin d'etre au moins aussi grand que les capacites nominales aval de regulation et radio, C2 >=5 uF a 3,4 V, exigences d'approvisionnement a confirmer. La stabilite demande >=1 uF effectif en sortie et ESR<=1 ohm ; les ceramiques choisies satisfont cette intention sous reserve de leurs courbes de biais.

Puissance LDO conservatrice a 0,55 A : (5,1-3,2175)*0,55+0,002 <1,04 W. Au theta de test fabricant 62 degres C/W, jonction environ100 degres C a35 degres C ambiant ; a0,25 A moyen environ65 degres C. Ce n'est pas une garantie pour notre cuivre : grande plage GND sur la languette, vias thermiques, viser theta<=75 degres C/W et mesurer pour rester sous115 degres C en essai. D1 peut dissiper0,88 W avec 1,60 A et0,55 V, necessitant cuivre sur ses deux bornes et controle thermique ; viser jonction<125 degres C. Pas de convertisseur ni surveillance additionnelle pour cette maquette.


## Acceptation

Verifier inversion avec source limitee, rampe, tension 3V3 et creux pendant emission Wi-Fi, temperature apres trois heures au budget maximal. Un bloc secteur non regule ou12 V sur J1 n'est pas couvert. Les condensateurs sont non polarises ; la masse reste GND et n'est pas reliee a PGND dans ce module.


# CONTROL - Centrale ESP32


## Centrale et interfaces

U2 est un ESP32-WROOM-32E a antenne integree. Les six lignes OW_GPIO1, OW_GPIO2, OW_GPIO3, OW_GPIO4, OW_GPIO5 et OW_GPIO6 utilisent GPIO25/26/27/18/19/21, configurees uniquement en drain ouvert, sans sortie haute active. Les tirages etprotections sont dans BUS. GPIO32 porte LOCK_CMD, GPIO33 lit LOCK_SENSE. GPIO34 entree seule lit CTRL_PAIR depuis le commutateur central externe sur J3 ; R3 est indispensable car pas de tirage interne surGPIO34. Avec contact ferme, diviseur R3/R4 donne environ0,30V a3,3V, sousle seuil bas ESP32 de0,25Vdd. C7 filtre environ1ms et lelogiciel impose50ms d'antirebond avant autoriser l'appairage. D2 traite les manipulations ESD, pas une tension externe maintenue.


## Alimentation, demarrage et programmation

C4 etC5 sont au plus pres du module, capacite effective C4>=10uF a3,4V. POWER reserve>=0,55A, dont au moins0,5A disponible pour la radio ; ne pas confondre provision etconsommation moyenne0,25A estimee. R1/C6 donnent retard EN environ10ms ; verifier au moins50us de rails stables avant EN etreset>=50us. BOOT_MODE normalement haut parR2, GPIO2 bas parR5. J2 est le seul acces UART0 etreset/boot, broches1et6 masse,2 entreeUART_RX,3 sortieUART_TX,4 RESET_EN,5 BOOT_MODE. Adaptateur3,3V uniquement, carte alimentee par POWER ; aucune borne d'alimentation sur J2. Reset etboot doivent etre pilotes a drainouvert ou parcontact versGND, pas a5V. Maintenir bootbas pendantreset puis1ms apres liberation pour entrer en telechargement. Les broches non utilisees, ycompris NC, sont intentionnellement libres. Respecter la zone sanscuivre etdegagee de l'antenne 32E, au bord de carte, loin du verrou/cables.


## Structure du programme et acceptance

La specification de taches, appairage, table NVS atomique etHTTP/JSON est portee par le contrat d'ensemble. Initialiser LOCK_CMD bas avant mode sortie, configurer les six DATA en relachement, attendre stabilisation optique300ms puis parcourir ROM CRC/famille ; ne pas activer verrou avant initialisation. La bibliotheque candidate doit etre compilee puis mesuree sous chargeWi-Fi ; isError() ne validepas getPIO(), tester0xFF etbit2. Aucune ecriturePIOA sans PIOB latch1. Serialiser les transactions parbus, appliquer recuperation20us initiale extensible, tache reseau separee.

Une coupure POWER locale ne peut pas etre diagnostiquee par ce circuit reserve a2PIO ; l'optique peut etre fausse meme si ROM repond, limitation a documenter dans la supervision d'essai. Lecontactdeverrou ouvert signifie nonconfirme, pas preuve de position ouverte. Des donnees stale ou erreurs de statut deviennent inconnues. Appairage exigecommutateur actif, numeroexplicite, transition absentverspresent unique etstable, expiration60s, ecritureatomique etconfirmationLED. PerteWi-Fi conserveconsigne locale, reset/watchdog deverrouille dans l'hypothese du mecanisme. Tester reset, NVS interrompue, sixbus souserreurs, compatibiliteWi-Fi, programmeur et3V3 transitoire. Aucun firmware compile ou essai physique n'est revendique par ces schemas.


# CASIER_A - Carte de référence


## Carte reference et detection

J101 est une derivation unique, jamais un passage serie de la nappe. U101 est alimente uniquement par OW1 ; ses deux masses sont raccordees, sa broche non connectee reste ouverte. PIOA absorbe le courant de D102, allumage actif bas. PIOB reste libere, verrou de sortie1, et lit U102 par R107. Initialiser a0x03 ;0x02 allume la signalisation sans fermer PIOB. Le bit2 du statut physique vaut1 faisceau libre,0 faisceau coupe ; donc present=(statut&4)==0, apres validation de lecture et stabilite300ms. Un module non joignable reste inconnu.

D101 est un emetteur IR permanent. Q101 abaisse A_OPT lorsqu'il recoit la lumiere ; R103 le ramene vers ROW1_POWER quand le faisceau est coupe. U102 est un comparateur CMOS push-pull a entree rail-a-rail avec retour positif sur A_REF. C103 filtre les variations rapides de lumiere (tau de charge environ0,47ms), sans rendre l'optique insensible au soleil. Le guide du casier doit assurer la coupure, tubes noirs autour du recepteur ; portee100mm a tester, pas garantie. Emetteur/recepteur montes sur les composants de la carte reference : leur espacement et orientation restent a confirmer avant implantation.


## Courants et contraintes

R101 donne Iir=(Vpower-Vf)/R ; estimation20mA a4,7 V avec Vf1,1 V, cette Vf est une hypothese non garantie a20mA. Sans minimum Vf garanti, borne sure Iir<=5,1/178,2=28,62mA ; c'est bien sous100mA maximum TSAL6200 a25C et compatible avec le derating a35C. P de R101 <=5,1^2/178,2=0,146 W, boitier1206 de0,25 W adequat pour prototype, mesure thermique. Au minimum3,8 V le courant et la portee diminuent ; ne pas annoncer une portee constante. R102 limite la signalisation a<=5,1/990=5,16mA meme en court-circuit LED ; normalement environ2,7mA a4,7 V etVf2 V. Provision35mA par carte, aucun de ces courants n'est tire sur DATA. R107 limite une sortie PIOB accidentellement basse a environ5mA ; ce n'est pas un mode normal, detecter et reinitialiser le latch. Comparateur a faible charge de retour, PIOB fuite<=1uA retenue donc chute serie<=1mV.


## Seuils reels du reseau

Vref=(Vpower/R104+Vout/R106)/(1/R104+1/R105+1/R106). Avec47k/47k/100k, Vref=0,404858 Vpower+0,190283 Vout. Lorsque A_OPT monte, la sortie actuellement haute bascule basse au seuil haut ; lorsqu'il redescend, la sortie basse bascule haute au seuil bas. Inclure niveaux de sortie, offset et hysteresis interne, pas seulement un diviseur ideal : seuil haut environVref(Voh)+Vh/2+Vos, seuil bas environVref(Vol)-Vh/2+Vos selon convention d'offset.

A5,000 V, sorties ideales0/5 V, Vh typique3,3mV etVos nul : haut2,97736 V, bas2,02264 V, largeur0,95472 V. Avec hypothese realiste Vol0,05 V etVoh4,95 V : haut2,96784 V, bas2,03215 V, largeur0,93569 V. A4,7 V avec sorties ideales : haut2,79882 V, bas1,90118 V. Les valeurs varient avec l'alimentation locale ; ce ne sont pas des references absolues. Les resistances sont1%. Un calcul conservateur a5 V utilisant sorties Voh4,8..5 V etVol0..0,2 V, offset+/-7mV etVh<=6,5mV donne enveloppes d'environ2,903..3,010 V haut et1,990..2,097 V bas. Ces limites d'offset/hysteresis sont tabulees a25C etmode commun0 V, pas garanties a notre mode commun ni sur10..35C : enveloppes d'etude, non garantie industrielle. Mesurer les vrais seuils sur la carte.

A5 V et47k, il faut typiquement Iphoto>63,4uA pour passer absent etIphoto<43,0uA pour passer present ; pour marge optique0,3 V viser A_OPT<1,72 V libre et>3,28 V coupe, soit environ Iphoto>70uA libre et<36,6uA coupe. Adapter ces criteres aux seuils mesures et a la tension3,8..5,1 V. La fuite noire200nA maximum a25C au test20 V produit seulement9,4mV dans47k, mais la lumiere ambiante et la temperature ne sont pas bornees par ce chiffre. Les caracteristiques a1mW/cm2 et les intensites emetteur a100mA ne garantissent pas notre lien a20mA.


## Protection et verification

D103 protege OW1 contre manipulation ESD ; capacite reelle typique20pF, pas0,3pF du catalogue. Aucun condensateur de decouplage n'est ajoute a DATA. C101 etC102 decouplent seulement ROW1_POWER. Une coupure POWER peut laisser U101 joignable avec detection non fiable ; marquer les etats suspects et tester cette panne, aucune mesure d'alimentation locale n'est disponible. Le numero du casier est enregistre dans l'ESP32 via ROM et bus, jamais grave par ordre de decouverte. Tester niveaux, polarite, seuils, lumiere, alignement, perturbations, et coupure de la nappe hors tension avant une seance de trois heures.


# CASIER_B - Seconde carte


## Seconde carte de demonstration

Identique electriquement a CASIER_A, avec ROM propre et references distinctes. J201 est sa seule derivation a la nappe continue, jamais en serie. U201 n'a pas de Vcc : alimentation parasite OW1 et deux masses GND. PIOA absorbe la signalisation active bas, PIOB reste latch1 et lit la sortie de U202 a travers R207. Initialiser0x03, allumer par0x02 ; getPIO()==0xFF signifie erreur, pas absence. Etatphysique bit2 haut=absent etbas=objetpresent apres300ms stables. L'association numero/bus/ROM est sauvegardee sur ESP32 independamment de l'inventaire.

Q201 tire B_OPT bas en lumiere, R203 remonte quand le faisceau est coupe ; U202 compare au seuil B_REF avec retour positif. Aucun filtrage ou decouplage ajoute sur DATA. Les capacites d'alimentation sont uniquement sur ROW1_POWER. C203 avec R203 donne environ0,47ms de filtrage. Guide d'insertion et tubes opaques obligatoires ; 100mm est une hypothese d'essai, pas une portee certifiee. Emetteur etrecepteur sont des composants de cette carte, leur disposition mecanique reste a confirmer.


## Dimensionnement et seuils

Iir=(Vpower-Vf)/R201, environ20mA sous4,7V avec hypothese Vf1,1V. Comme aucun minimum Vf n'est garanti a20mA, Iir<=5,1/178,2=28,62mA etP de R201<=0,146W ;1206 de0,25W pour prototype. R202 borne le courant signalisation <=5,16mA meme avec diode court-circuitee. Provision35mA par carte, typique22mA. R207 limite une contention accidentelle a environ5mA, ne jamais utiliser PIOB comme sortie.

Vref=(Vpower/R204+Vout/R206)/(1/R204+1/R205+1/R206)=0,404858 Vpower+0,190283 Vout. Seuil montant du collecteur avec sortiehaute et descendant avec sortie basse. Inclure Vos etVh interne : haut=Vref(Voh)+Vh/2+Vos, bas=Vref(Vol)-Vh/2+Vos selon convention d'offset. A5V, sorties ideales0/5V, Vh3,3mV typique etVos0 : haut2,97736V, bas2,02264V ; avec Vol0,05V etVoh4,95V supposes : haut2,96784V, bas2,03215V. A4,7V sorties ideales : haut2,79882V, bas1,90118V. Resistances1%, niveauxVoh>=Vpower-0,2V etVol<=0,2V a2mA, offset+/-7mV etVhmax6,5mV de la table25C/Vcm0 donnent enveloppes d'etude a5V environ2,903..3,010V haut et1,990..2,097V bas. Le mode commun reel etla plage10..35C ne sont pas couverts par ces garanties ; mesurer les deux seuils avec alimentation locale etverifier marges0,3V.

A5V, viser B_OPT<1,72V libre (Iphoto>70uA) et>3,28V coupe (Iphoto<36,6uA), a recalculer pour3,8..5,1V. Darkcurrent200nA max a25C/20V donnerait9,4mV dans47k, mais ne borne ni soleil ni chaud. Les intensites IR a100mA etcourants photo a1mW/cm2 ne garantissent pas l'alignement/distance a20mA.


## Conditions et essais

La TVS D203 apporte20pF typiques selon son datasheet, non0,3pF catalogue ; manipulation ESD seulement, pas protectiongarantie contre inversion DATA/POWER. Deconnexion hors tension. Une coupure POWER peut laisser DS joignable avec detectionindeterminee : absence de monitoring local, ne pas promettre diagnostic de cette panne. Evaluer avec la premiere carte lumiereambiante, alignement, alimentationminimale, essais3h et remplacements ; extension aux autres rangees en copiant cette carte et choisissant le depart par le faisceau, sans changer le numero de ROM.