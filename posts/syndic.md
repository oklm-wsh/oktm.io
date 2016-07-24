---
title: Syndic ou comment fermer les portes de l'enfer
author: dinosaure (Romain Calascibetta)
tags: ocaml,syndic,atom,rss,type,web
---

### Des nuages, de l'informatique et pas de cloud

Syndic est un projet  qui date de 2014.  On va considérer qu'il  est né comme ça
et éviter de  raconter  l'histoire  de  [Cumulus][cumulus]  (un  projet comme un
autre).  Il reste la  référence  dès  qu'il  s'agit  de  manipuler des documents
RSS   ou    Atom   en   OCaml    -   pour   preuve,    il    est   utilisé   par
[ocaml.org](https://ocaml.org/).  C'est à  l'aide de Christophe  Troestler et de
moi-même que nous avons réussi à finaliser le projet.

Il peut paraître simple de lire des documents RSS ou Atom mais en vérité,  c'est
difficile et je vais vous expliquer pourquoi.  D'ailleurs, une grosse difficulté
a été  résolu par [Daniel  Bünzli](http://erratique.ch/contact) avec  son projet
[xmlm][xmlm] que je remercie au passage.

### Respecter la RFC

Au tout début de ce travaille,  j'ai  pris comme référence [la RFC d'Atom][atom]
et la spécification d'[RSS 1.0][rss1] et [2.0][rss2].

C'est pas  toujours la joie  de lire des  RFCs.  Ces documents sont  longs et il
s'agit  de  faire  un  travail  précis  pour  éviter  les  aller/retour  sur  la
spécification.   Le  travail,   en  général,  n'est  pas  si  intellectuellement
intéressant.  Il  s'agit  surtout  d'appliquer  ce  qui  est  dit  bêtement.  La
réflexion  se  porte   surtout   sur   l'interface   qu'on   souhaite  offrir  à
l'utilisateur, le coeur du logiciel reste [petit][syndic#generate].

Mais  voilà,  respecter la  spécification demande  quand même  un travaille.  Un
travail bête mais un travail quand même.  Comme il a été dit,  il s'agit surtout
d'être précis.  Dans Syndic  par  exemple,  on  retrouve  des  paragraphes de la
spécification  en   commentaire  afin  de   comprendre  pourquoi  on   fait  tel
traitement sur tel élément.

Et donc arrive le moment fatidique.  Est  ce que mon logiciel fonctionne avec le
monde réel.  Est  ce que je  peux manipuler des  documents RSS  et Atom  - c'est
quand  même son  objectif initial  au lieu  de me  faire consommer  une quantité
hallucinante de café.

### Le oueb

Si il y a  bien un domaine que je déteste,  c'est le  web.  Il décrit de manière
non-exhaustive l'interaction possible avec le monde extérieur.  Bien entendu, ce
qu'il  décrit,  et  c'est  bien  dans  son  essence  même,  n'est  pas  parfait,
standardisé et correct.  C'est  un monde fluctuant que je  décris comme étant le
tartare  de tout  informaticien -  car  au  fond,  quoi  de  plus  difficile que
d'imaginer toutes les interactions *chaise-clavier*.

Le web  en est l'image  parfaite dans le sens  où son contenu  se rapproche tout
autant de la perfection que du  chaos que l'homme est capable de générer.  Alors
des gens bien  sympathique ont mis en  place des règles - comme  des règles dans
une société.  Ces règles permettent de  définir des moyens de communication afin
qu'on puisse tous se comprendre - comme notre langue.

La  particularité  -  et  c'est  le point  de  Syndic  -  est  que  ces  flux de
communication peuvent autant  être générer par un ordinateur  que par un homme.
Les flux  RSS et Atom  n'échappent pas à la  règle - pendant  un temps,  le flux
Atom d'ocsigen était  géré  par  Vincent  Balat  lui  même.  Et vous auriez beau
respecter la RFC sur tout ses points,  vous trouverez sûrement un flux incorrect
dans ce tartare.

![syndic error](https://oktm.io/imgs/syndic.png)

Le premier lancement de Syndic a donc  été un échec presque complet dans le sens
où nous  avions pas  mal de  flux que  nous invalidions.  C'est  à ce  moment où
Christophe  a fait  un très  bon travail  pour  essayer  de  rendre  Syndic plus
*résiliant*.

### Fermer les portes de l'Enfer

Le problème  est que nous  utilisons OCaml et  l'état flou de  la quasi-totalité
des flux RSS et  Atom ne peux être traiter par la  rigidité du type dans Syndic.
Philipe Wang,  programmeur dans  l'équipe [BeSport][besport],  m'a signalé cette
erreur:

> J'ai  essayé d'utilisé  Syndic mais  tu invalides  presque quasiment  tout les
> documents alors j'ai  regardé  sur  le  site  de  la  [W3C][w3c] et en vérité,
> Syndic a raison d'invalider les documents.

Alors qui a raison ? Syndic qui  implémente scrupulesement le standard ou le web
qui se contre fiche  dans une certaine mesure de ces  règles ? L'extrémise n'est
jamais bon  et je  dirais que  c'est un  peu des  deux.  Ainsi,  Syndic **doit**
pouvoir gérer des flux invalides.  Sauf qu'on en revient sur le problème de base
qui est celui de la sévérité des types en OCaml<sup>β</sup>.

<div class="Notes">

β:  dans d'autres langages, on n'a pas forcément ce problème
comme en  PHP où  l'on peut autant  considérer "1"  un nombre,  une  *string* ou
encore autre chose que j'ai pas forcément envie de savoir.

On pourrait  croire que c'est pratique,  mais  ça ne  l'est que  pour l'analyse.
Dans l'utilisation,  c'est juste  très  pénible  de  faire  des tests dynamiques
supplémentaires pour tester si on a bien une donnée du type `int` par exemple.

</div>

Nous pourrions remplacer  tout  les  types  qui  composent le [*record*][record]
avec un `'a option` et donc être  totalement résiliant à tout contenu mais on en
revient  à  un  problème  d'interface  inutilisable  (même  si  on  retrouve des
librairies  pour  faciliter  la  manipulation  du  type  `option`,  il n'empêche
que non, ce n'est pas utilisable).

### Le type a toujours raison

En vérité,  le  problème n'est pas  lié à la  rigidité des  types en  OCaml mais
plutôt  à la  manière dont  on  traite  l'information.  C'est  à  dire  que nous
pourrions   considérer  comme   *customizable*   le   traitement   de  certaines
informations  dont nous  définirions un  traitement  par  défaut  (respectant la
RFC).

Pierre Chambart m'a donc conseillé une  manière d'exprimer ce problème en OCaml.
Vous pouvez voir l'avancement [ici][syndc#pr]  qui concerne au moment où j'écris
ces lignes  uniquement le  RSS 2.0.  On y  a rajouté  une structure  `Relax` qui
définit le traitement que vous souhaitez et  qui permet de créer le *record* que
vous souhaitez (en tout cas,  de pouvoir spécifier les types que vous attendez -
par  exemple,  vous pourriez  considérer que  les  dates  peuvent  être  du type
`Ptime.t` par exemple, comme du type `string`).

Il  s'agit   ensuite  de  définir   des  contraintes  fortes   par  rapport  aux
*conveniences  functions* disponiblent  dans Syndic.  L'exemple  de la  date est
révélateur d'une  chose,  si vous  considériez la  date comme  une `string`,  un
traitement tel  qu'un réordonnancement  selon la date  de publication  n'est pas
possible. Mais Christophe et moi-même allons travailler sur la question !

### Conclusion

Je pense que  Syndic  révèle  un  problème  assez  commun  dans le traitement de
l'information standardisé  - mais dans  ce cas,  l'échec de  traitement est bien
plus prononcé il me semble.

Il s'agit  de bien  faire les  choses car  ce qu'on  retiendra reste  avant tout
l'interface disponible  pour l'utilisateur.  Cette interface  doit cacher autant
les  traitements chiants  que d'offrir  des  *convenience  functions*  autour de
l'information.

Ce n'est pas  un travail très très  plaisant mais parfois il faut  le faire pour
le bien  de la communauté.  Syndic  est encore un  projet jeune que  j'ai initié
pendant mes études et qui a sûrement quelques problèmes dans l'API.  Les retours
sont les bienvenues :) !

[atom]: https://tools.ietf.org/html/rfc4287
[rss1]: http://web.resource.org/rss/1.0/spec
[rss2]: http://www.rssboard.org/rss-specification
[syndic#generate]: https://github.com/Cumulus/Syndic/blob/master/lib/syndic_common.ml#L19
[syndic#pr]: https://github.com/Cumulus/Syndic/pull/66
[cumulus]: https://github.com/Cumulus/Cumulus
[xmlm]: https://github.com/dbuenzli/xmlm
[besport]: https://www.besport.com/
[w3c]: https://validator.w3.org/feed/
[record]: https://github.com/Cumulus/Syndic/blob/master/lib/syndic_rss2.mli#L218
