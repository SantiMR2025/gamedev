# 02_entrevista_requeriments

## Fase 02 — Entrevista de requeriments

**Joc:** The Dark of Shadows  
**Client:** Santiago Moreno  
**Motor:** GB Studio 4.3.2 per a Windows  
**Objectiu de la fase:** concretar els requeriments del joc a partir del brief aprovat i de les respostes del client.

## 1. Objectiu del jugador i com comença i acaba el joc

| Pregunta | Resposta del client | Decisió confirmada |
|---|---|---|
| Hi haurà alguna escena abans de donar el control al jugador? | Sí. Hi haurà una escena amb un diàleg abans de donar el control a Henry. | El joc començarà amb una escena de diàleg abans del control del jugador. |
| Com coneixerà el jugador l'objectiu inicial de buscar ajuda? | Mitjançant un diàleg. | L'objectiu inicial es comunicarà mitjançant diàleg. |
| Com es comunicarà l'objectiu general d'escapar i sobreviure? | Mitjançant un altre diàleg. | L'objectiu es comunicarà mitjançant diàleg. |
| Què passa després de trencar el cristall màgic? | Hi haurà un pla per darrere de Henry mirant un paisatge. | Després de trencar el cristall hi haurà una cinemàtica final amb Henry observant un paisatge. |
| La cinemàtica final serà el final definitiu? | Sí. | La cinemàtica final serà el final definitiu del joc. |

## 2. Accions del jugador i progressió

| Pregunta | Resposta del client | Decisió confirmada |
|---|---|---|
| Quines accions de moviment tindrà Henry? | Només caminar i córrer. | No hi haurà altres accions de moviment dins del MVP. |
| El jugador podrà explorar lliurement les zones? | Sí. El jugador podrà explorar lliurement les zones, però el canvi de zona estarà bloquejat. | Les zones seran explorables lliurement i la progressió entre zones estarà bloquejada fins a complir els requisits necessaris. |
| Com recollirà el jugador els objectes? | Els haurà de recollir explícitament amb una interacció, com ara obrir un calaix, armari, bagul o cadàver. | Els objectes no es recolliran automàticament; caldrà interactuar amb l'element que els conté. |
| Els puzles seran obligatoris? | Sí. S'hauran de resoldre obligatòriament per continuar avançant. | La resolució dels puzles forma part de la progressió principal. |
| El joc donarà pistes sobre els puzles? | El jugador haurà de descobrir-ho explorant i llegint/veient elements de l'entorn. | No es proporcionarà una guia directa dels puzles; el jugador haurà de descobrir-ne la solució mitjançant l'exploració i l'entorn. |

## 3. Condicions de victòria i derrota

| Pregunta | Resposta del client | Decisió confirmada |
|---|---|---|
| Què passa quan un enemic atrapa Henry? | Hi haurà una escena breu abans de tornar al checkpoint. | Henry morirà i, després d'una escena breu, tornarà a l'últim checkpoint. |
| Quines altres formes de derrota hi haurà? | Només pels enemics. | No hi haurà altres formes de mort o derrota. |
| Què passa si el jugador fa una acció incorrecta en un puzle? | Simplement no passarà res. | Les accions incorrectes dels puzles no provocaran la mort ni una derrota. |
| Quina és la condició de victòria? | Trobar el cristall, trencar-lo, escapar de l'abisme i veure la cinemàtica final. | Aquesta serà la condició de victòria definitiva. |
| Quan s'actualitzaran els checkpoints? | Cada vegada que el jugador completa algun puzle important. | Els checkpoints seran automàtics i s'actualitzaran després de completar puzles importants. |

## 4. Personatges, objectes i escenes

| Pregunta | Resposta del client | Decisió confirmada |
|---|---|---|
| Quins personatges apareixeran? | Hi haurà un dels experiments, que serà pacífic però només comentarà la història. | Hi haurà un experiment pacífic que servirà per explicar part de la història. |
| Com es diferenciaran els dos enemics? | L'únic canvi serà que l'enemic ràpid serà més negre. | Els dos enemics seran genèrics; l'enemic ràpid tindrà una aparença més negra i serà més ràpid. |
| Quins objectes importants hi haurà? | No n'hi haurà cap més dels ja definits. | Els objectes importants seran els ja establerts: claus, codis, targetes de diferents nivells, fusibles i pedres màgiques. |
| Hi haurà altres escenes narratives? | S'afegiran diàlegs i notes. | Hi haurà diàlegs i notes com a contingut narratiu. |
| L'entorn explicarà visualment la història dels experiments? | No. | L'entorn no serà el principal sistema per explicar la història dels experiments. |

## 5. Interaccions clau

| Pregunta | Resposta del client | Decisió confirmada |
|---|---|---|
| Com sabrà el jugador què necessita per obrir una porta bloquejada? | Es mostrarà un missatge o diàleg, per exemple: «Necessito una targeta de nivell 2». | Les portes bloquejades comunicaran mitjançant missatges o diàlegs quin requisit necessita el jugador. |
| Les claus i targetes serviran per a diverses portes? | No. S'utilitzaran per a portes concretes. | Cada clau o targeta estarà vinculada a una porta concreta. |
| Com s'utilitzaran els objectes en els mecanismes? | El jugador haurà d'interactuar directament amb el mecanisme per utilitzar-los. | Els objectes necessaris s'utilitzaran mitjançant una interacció directa amb el mecanisme corresponent. |
| Les notes i els diàlegs seran obligatoris per acabar el joc? | No. El jugador podrà acabar el joc encara que no els llegeixi tots. | Les notes i els diàlegs narratius no seran necessaris per completar el joc. |
| Hi haurà combat? | No. | No hi haurà atacs ni combat. Quan un enemic detecti Henry, el jugador haurà de fugir/córrer. |

## 6. Abast mínim

| Pregunta | Resposta del client | Decisió confirmada |
|---|---|---|
| Què és imprescindible per al MVP? | Pròleg, oficines, laboratoris i abisme, amb exploració, puzles, objectes, dos enemics de patrulla, checkpoints, portal, cristall i cinemàtica final. | Aquest conjunt constitueix el mínim imprescindible del joc. |
| El sistema de jumpscare i amagar-se de The Shadow of God és imprescindible? | No. Queda confirmat que pot esperar i només s'implementarà si queda temps. | El sistema específic de jumpscare i amagar-se queda fora de la prioritat inicial del MVP. |
| L'experiment pacífic és imprescindible? | Sí. Explica part de la història i és imprescindible per entendre el joc. | L'experiment pacífic forma part del contingut imprescindible. |
| Els diàlegs i les notes són una part important del MVP? | No. | Els diàlegs i les notes no formen una part important del MVP, tot i que es podran incloure. |
| Quina funcionalitat es prioritza si falta temps? | Puzles/progressió. | La prioritat de desenvolupament serà garantir els puzles i la progressió principal. |

## Requeriments confirmats resultants de l'entrevista

### Inici

1. El joc començarà amb una escena de diàleg.
2. Després del diàleg, el jugador controlarà Henry.
3. El primer objectiu es comunicarà mitjançant diàleg.

### Progressió

1. Henry només podrà caminar i córrer.
2. Les zones es podran explorar lliurement.
3. El canvi de zona estarà bloquejat fins a complir els requisits corresponents.
4. Els objectes s'hauran de recollir mitjançant una interacció.
5. Els puzles seran obligatoris per continuar.
6. El jugador haurà de descobrir les solucions explorant i observant l'entorn.
7. Les claus i targetes estaran associades a portes concretes.
8. Els mecanismes requeriran una interacció directa per utilitzar els objectes.

### Enemics

1. Hi haurà dos enemics genèrics.
2. Un enemic serà lent.
3. L'altre serà ràpid i visualment més negre.
4. No hi haurà combat.
5. El jugador haurà de fugir dels enemics.
6. Ser atrapat provocarà la mort de Henry.

### Checkpoints

1. Els checkpoints seran automàtics.
2. S'actualitzaran quan el jugador completi puzles importants.
3. Després de morir hi haurà una escena breu.
4. Després de l'escena Henry tornarà a l'últim checkpoint.

### Victòria

1. Henry haurà d'arribar a l'abisme.
2. Haurà d'explorar-lo per trobar el cristall màgic.
3. Haurà d'interactuar amb el cristall per trencar-lo.
4. Henry escaparà de l'abisme.
5. Es mostrarà una cinemàtica final.
6. La cinemàtica mostrarà Henry d'esquena observant un paisatge.
7. Aquesta cinemàtica serà el final definitiu del joc.

### Narrativa

1. Hi haurà un experiment pacífic que explicarà part de la història.
2. Hi haurà diàlegs i notes.
3. El jugador no haurà de llegir tots els diàlegs o notes per acabar el joc.

## Qüestions obertes

- Definir el contingut exacte dels diàlegs inicials.
- Definir el contingut dels diàlegs de l'experiment pacífic.
- Definir el contingut i la ubicació de les notes.
- Definir quins puzles concrets activaran els checkpoints.
- Definir el disseny concret de cada puzle.
- Definir el disseny visual exacte dels dos enemics.
- Definir el disseny concret del laberint de l'abisme.
- Definir el contingut exacte de la cinemàtica final.
- Verificar tècnicament les interaccions, checkpoints, enemics i altres mecàniques amb GB Studio 4.3.2.

## Decisions pendents

- Contingut dels diàlegs.
- Contingut i ubicació de les notes.
- Puzles concrets i la seva distribució.
- Punts exactes dels checkpoints.
- Disseny visual dels enemics.
- Estructura del laberint de l'abisme.
- Contingut definitiu de la cinemàtica final.
- Verificació tècnica de les mecàniques a GB Studio 4.3.2.

## Documents que cal revisar

- Document de disseny de nivells: caldrà concretar la distribució de les zones, els bloquejos i el laberint de l'abisme.
- Document de puzles/progressió: caldrà definir els puzles obligatoris i els objectes necessaris.
- Document de personatges/enemics: caldrà concretar l'experiment pacífic i els dos enemics.
- Document de narrativa: caldrà concretar els diàlegs, les notes i la cinemàtica final.
- Document tècnic de GB Studio 4.3.2: caldrà verificar la viabilitat de les mecàniques confirmades.
