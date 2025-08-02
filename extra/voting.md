Voici la traduction française du texte au format Markdown :


--[ 1 - Une Genèse

Nous sommes en juin 2024. Un groupe d'hommes riches et puissants est assis dans un salon perdu dans les montagnes de San Bernardino, à 130 km à l'est de Los Angeles. Une épaisse fumée de cigare âcre envahit la pièce. Mais ce qui me soulève le cœur, c'est l'odeur putride d'une nation entière dont on vole la direction. Ces hommes discutent et marchandent pour décider quel candidat sera élu président, sénateur, etc.

Le désordre actuel aux États-Unis a été amorcé il y a 24 ans, lors de l'élection Bush contre Gore en 2000. Il a fallu plus d'un mois pour déclarer un vainqueur en raison d'un litige sur le décompte des voix en Floride. George Bush a finalement remporté la Floride par 537 voix, soit 0,009% [1]. Une controverse immense a entouré les bulletins confus, les erreurs des cartes perforées et les anomalies du recomptage.

Par la suite, des gens bien intentionnés ont réclamé l'informatique pour résoudre les problèmes de vote des États-Unis. Après tout, les ordinateurs ont simplifié tous les autres aspects de la vie. Mais ces personnes ont fait preuve d'une certaine arrogance ; elles ne comprenaient pas la technologie. Elles banquaient avec l'ordinateur, discutaient avec l'ordinateur, achetaient avec l'ordinateur, donc sûrement on pouvait lui faire confiance pour le vote aussi. Mais elles ne comprenaient pas la profondeur des problèmes de sécurité informatique, ni pourquoi le vote est fondamentalement différent de toutes ces autres tâches. Les experts en sécurité, presque unanimement contre le vote électronique, ont été rejetés comme paranoïaques.

En réponse à la demande publique, le Congrès a adopté le *Help America Vote Act* visant à remplacer les machines à levier et à cartes perforées [38]. Et c'est ainsi que nos élections sont désormais décidées au gré de groupes puissants contrôlant les serveurs électoraux. Cet article disséquera les problèmes qui ont affligé le vote par internet dès son origine.

--[ 2 - Pourquoi les Gens Veulent le Vote par Internet

Avant d'examiner sérieusement les défauts inhérents au vote par internet, il faut se demander : pourquoi les gens le veulent-ils ? La réponse est : 1) l'engagement civique, 2) l'argent, 3) la soif de pouvoir, et 4) la technophilie.

Certains militants croient que le vote par internet augmentera la participation et donc l'engagement civique. Cela mène à la question : "Le vote par internet augmente-t-il significativement la participation ?" En 2002, certaines élections locales au Royaume-Uni ont utilisé un pilote de vote par internet, entraînant une augmentation de 3,5% de la participation [6]. Il est cependant impossible de prouver que cela était dû au vote par internet [6]. Même si l'augmentation était de 50%, accroître la participation au prix d'élections dignes de confiance n'est pas un plan sage. Aux États-Unis, tout le monde peut voter par correspondance, en renvoyant un bulletin qu'on lui envoie. Si participer à la démocratie n'est pas assez important pour envoyer un bout de papier, doit-on vraiment se plier en quatre pour leur étendre la démocratie ?

L'argent est un problème inhérent au vote en ligne car il y a beaucoup d'argent à gagner dans les systèmes de vote. Aux États-Unis, les solutions open source sont rarement adoptées par le gouvernement. Si le vote par internet était sérieusement légiféré ici, les entreprises déverseraient toutes sortes d'exagérations sur la sécurité de leurs systèmes pour obtenir des contrats lucratifs. De plus, dans le cas des machines à voter électroniques, les entreprises ont longtemps fait pression pour garder leur code source secret. Que nous confiions l'intégrité de notre démocratie à cela était sans importance pour leurs avocats en brevets [7].

Certains argumentent aussi que le vote par internet économisera les coûts d'organisation des élections. Bien que cela soit possible, il n'est pas clair que le coût de maintenance et de développement soit réellement moins cher que les bulletins papier. Surtout, le but d'une élection n'est pas de la faire au moindre coût, mais d'obtenir des résultats fiables. Cela n'a pas de sens de compromettre les élections pour économiser de l'argent.

La raison pour laquelle ceux qui cherchent le pouvoir veulent du vote par internet est une question ancienne. Boss Tweed, le politicien corrompu de New York estimé avoir volé entre 1 et 8 milliards de dollars (valeur 2010) [8], disait : "Tant que c'est moi qui compte les votes, qu'allez-vous y faire ?" [35]. Contrôler les officiels électoraux qui comptent les votes était (et reste) un des moyens les plus simples de truquer une élection. Cette fraude est commise à l'échelle locale, car aux États-Unis, truquer une élection fédérale comté par comté est considéré comme relativement irréaliste.

Bien sûr, cela pourrait arriver aux États-Unis, et c'est certainement arrivé ailleurs. Songez à l'élection russe de 2011, rapportée comme présentant de nombreuses et graves irrégularités au niveau national [39]. Au Ghana aussi, des plaintes ont fait état de fraudes généralisées lors de l'élection de 2012 [40].

Même dans les pays où c'est possible, cela demande une coordination et un travail énormes, nécessitant des appareils politiques loyaux (ou de sérieuses menaces de violence). Le vote par internet, cependant, facilite grandement la fraude en permettant d'attaquer des points de défaillance uniques – un serveur central de comptage, ou un logiciel fonctionnant sur de nombreux serveurs de bureaux de vote. Qui ne voudrait pas contrôler le logiciel comptabilisant les votes ? Au pire, les résultats d'un pays entier pourraient être manipulés, et même si chaque région avait son propre système, des groupes pourraient influencer fortement une élection régionale.

Enfin, les technophiles peuvent être un facteur moteur. Ce sont des gens qui aiment la nouvelle technologie pour elle-même. Moi-même, j'aime les derniers produits. Mais dans certains cas, comme le vote par internet, nous devons nous assurer que la technologie améliore réellement la situation. Aux technophiles, bien qu'ils soient de bonne foi, je demande de faire preuve de retenue et de considérer les conséquences avant de sauter sur l'occasion.

--[ 3 - L'Évolution du Comptage des Voix

Avant la Révolution américaine, les électeurs votaient généralement à voix haute, un greffier enregistrant leur choix [2]. Cela facilitait la vérification, mais offrait évidemment des opportunités de représailles, d'achat de votes, etc. À l'époque de la Révolution, Américains et Français exploraient le vote secret. La constitution française de 1795 stipulait : "toutes les élections se font au scrutin secret" [2].

Bien sûr, avec l'adoption du bulletin secret est apparu le bourrage d'urne. En 1856, un comité de vigilance de San Francisco découvrit une urne à double fond dissimulant des bulletins. Elle paraissait vide avant le vote, et après la fermeture, les faux bulletins pouvaient être mélangés. Une des premières technologies pour contrer ces astuces fut simple : en 1858, Alan Cummings et Samuel Jollie brevetèrent des urnes transparentes. Un globe de verre dans un cadre en bois permettait de voir les bulletins du début à la fin. Ce principe est toujours utilisé, bien que le plastique ait souvent remplacé le verre [41].

Vingt ans avant ces urnes, la *People's Charter* de 1838 en Grande-Bretagne décrivait déjà une machine à voter. Je vous encourage à regarder l'image dans [42] : une boule en laiton tombait dans un trou correspondant à un candidat, enregistrant un vote sur un cadran.

En 1892, la *Myers Automatic Voting Booth* fut introduite aux États-Unis [43]. Selon Douglas W. Jones de l'Université de l'Iowa, ces machines étaient à la pointe dans les années 1890, avec un nombre impressionnant de pièces mobiles. Elles ne fournissaient pas de trace par électeur, mais simplement un compteur pour chaque candidat.

La fraude électorale était déjà un problème majeur, et n'est pas nouvelle avec le vote par internet ; mon inquiétude porte sur son ampleur. En 1934, Joseph P. Harris publia un rapport sur la fraude électorale aux États-Unis [44]. Il résumait les types de fraude :

*   **Fraudes à l'inscription** : Inscrire des électeurs morts ou fictifs. Les votes sont ensuite émis sous ces noms.
*   **Vote multiple** : Des personnes votent dans plusieurs bureaux sous des noms fictifs ou réels.
*   **Bourrage d'urne** : Les officiers chargés bourrent l'urne. Pour éviter des anomalies évidentes, ils cochent le nom d'un électeur absent pour chaque bulletin frauduleux.
*   **Bulletins en chaîne** : Un bulletin marqué est donné à un électeur le matin. Il doit déposer ce bulletin et rendre le bulletin vierge reçu au bureau. Il est payé une fois le bulletin vierge rendu. Ce processus continue toute la journée. Harris note un manque de preuves sur sa fréquence.
*   **Assistance aux électeurs** : Les électeurs demandent de l'aide pour voter. C'est un moyen facile de briser le secret et de s'assurer qu'ils votent "correctement", volontairement ou sous intimidation.
*   **Intimidation et violence** : Chicago, IL est une ville tristement corrompue. Harris nota des quartiers entiers terrorisés par "les fusillades de gangsters". Des enlèvements avaient même été utilisés pour écarter des observateurs déterminés.
*   **Altération des bulletins** : Si un électeur n'a pas voté pour tous les candidats, un officier peut ajouter des marques. Des marques excessives peuvent aussi invalider un bulletin pour un candidat indésirable.
*   **Substitution de bulletins** : Des bulletins légitimes peuvent être jetés et remplacés.
*   **Faux décompte et fausses déclarations** : Il est plus facile de truquer le décompte que d'altérer les bulletins. Parfois, les bulletins ne sont pas comptés du tout, et les résultats sont fabriqués. Les votes peuvent aussi être mal lus ou enregistrés par des préposés.
*   **Altération des résultats** : Les résultats des bureaux peuvent être altérés par des officiels.

Spécifique aux machines à levier, une fraude signalée consistait à casser les dents d'un pignon du mécanisme de comptage d'un candidat, empêchant l'enregistrement d'un vote par cycle. En 1978 à Philadelphie, lors d'une élection sur un second mandat du maire, les machines tombèrent en panne, curieusement dans des districts qui lui étaient hostiles. Aucun rapport satisfaisant ne fut produit [2].

La prochaine évolution fut la carte perforée. Comme lors de l'élection présidentielle américaine de 2000, elles sont sensibles aux "chads" (petits carrés) pas entièrement détachés, créant des controverses. Parallèlement, les machines à lecture optique se popularisèrent. L'électeur remplit une bulle avec un crayon.

La technologie suivante, se rapprochant du vote par internet, fut la machine à voter électronique à enregistrement direct (DRE). Ce sont des ordinateurs où les votes sont saisis et comptés électroniquement. Bien que plus efficaces que le papier, elles sont criblées de graves problèmes de sécurité [47]. J'aimerais explorer ces problèmes, mais cet article se concentre sur le vote par internet.

--[ 4 - Où le Vote par Internet est-il Testé et Utilisé ?

Maintenant que nous comprenons pourquoi les gens veulent le vote par internet et son histoire, il est important de savoir où il est déjà utilisé. Je commence par Washington D.C., un cas rare où le public a pu tester la pénétration du système lors d'une élection simulée.

En 2010, Washington D.C. lança un projet pilote permettant de voter en ligne lors d'élections locales. En septembre 2010, avant les vrais votes, le Bureau des Élections permit au public de tester la sécurité. Une attaque d'une équipe de l'Université du Michigan fit annuler le projet. Les chercheurs prirent le contrôle des serveurs, dévoilèrent des bulletins secrets et altérèrent les résultats. Voici un résumé de leurs découvertes [9] :

Le système utilisait Ruby on Rails, Apache et MySQL. Un serveur frontal HTTPS relayait les requêtes vers un serveur d'applications stockant les bulletins. Des pare-feux bloquaient les connexions sortantes. Le système de détection d'intrusion (IDS) ne déchiffrait pas le trafic HTTPS contenant leur attaque.

Pour se connecter, l'électeur utilisait un ID, son nom, son code postal et un code PIN hexadécimal à 16 caractères envoyés par courrier.

Les bulletins étaient des PDF remplis par l'électeur et téléversés. Pour garantir le secret, ils étaient chiffrés avec une clé publique des officiels. Après l'élection, ils étaient transférés vers une machine hors ligne (détenant la clé privée) pour déchiffrement et comptage. Réfléchissez-y : on prend la peine de garder la machine de comptage hors ligne mais on lui permet d'ouvrir des PDF arbitraires. :>

Voici quelques attaques trouvées par l'équipe :
1.  Vol de la clé publique (qui devrait rester secrète ici car elle permet au serveur de chiffrer de faux bulletins). Ils l'ont utilisée pour remplacer tous les bulletins par des faux votant comme ils le voulaient.
2.  Remplacement de la fonction de traitement des bulletins par une version modifiée qui remplaçait chaque bulletin et traquait chaque électeur.
3.  Stockage non chiffré de chaque bulletin dans /tmp par le plugin Rails PaperClip *avant* chiffrement, permettant de corréler les fichiers avec les logs pour identifier les électeurs.
4.  Identifiants de base de données trouvés dans l'historique bash.
5.  Un PDF de 937 pages contenant *tous* les identifiants des électeurs pour la *vraie* élection trouvé dans /tmp. Ils auraient pu voter à la place des citoyens.

Après l'attaque, ils nettoyèrent les logs et supprimèrent leurs fichiers. Pour marquer leur territoire, ils programmèrent la page de confirmation pour jouer l'hymne de l'Université du Michigan après chaque vote. Malgré cet appel musical, il a fallu 36 heures aux officiels pour détecter l'attaque (un autre testeur a demandé sur une liste de diffusion quelle chanson était jouée, éveillant les soupçons).

D'autres exemples existent :

*   **Canada** : Utilisé dans certaines municipalités (système Intelivote). En 2010, le système a planté en Ontario [46]. La société a invoqué une demande élevée et une panne matérielle, affirmant que l'intégrité n'était pas compromise. Troublant : aucune mention d'audit externe indépendant.
*   **New Jersey** : Après l'ouragan Sandy en 2012, les résidents déplacés purent voter par e-mail ou fax [17][18]. Cela brise le secret (votre e-mail est lié à votre bulletin) et est vulnérable à des attaques basiques. De plus, les électeurs devaient aussi envoyer une copie papier, ce que beaucoup ignoraient [36]. Si on compte les papiers, le vote émail est inutile. Si on ne les compte pas, les résultats ne sont pas fiables.
*   **Arizona** : La primaire présidentielle démocrate de 2000 fut la première élection majeure par internet [19]. La société Election.com a rapporté aucun piratage.
*   **États-Unis** : Autorise le vote en ligne pour les militaires déployés.
*   **Estonie** : Premier pays à utiliser le vote par internet à l'échelle nationale en 2005. Utilise une carte d'identité nationale à puce et un lecteur (7$). Un schéma à double enveloppe cryptographique est censé garantir le secret [20]. Cependant, des observateurs de l'OSCE/ODIHR ont trouvé des problèmes majeurs en 2007 [2] : le chef de projet pouvait pousser des modifications logicielles à volonté, aucun audit de code n'a été produit, et aucune politique n'existait pour invalider les votes internet en cas de problème.
*   **Autriche** : Utilisé en 2009 pour des élections étudiantes (fournisseur : Scytl) [48]. Nécessitait la carte d'identité nationale et un lecteur.
*   **Finlande** : Testé lors d'élections municipales en 2008 via des kiosks dans les bureaux de vote (pas à domicile) [48]. Un bug a empêché le comptage de certains votes, forçant une reprise dans les zones concernées. Le projet a été abandonné. Le choix des kiosks visait à prévenir l'intimidation et l'achat de votes à domicile.
*   **France** : Utilisé depuis 2001 dans des pilotes locaux (kiosks) [49]. En 2009, le Ministère des Affaires Étrangères l'a mis en place pour les citoyens français à l'étranger (310 000 utilisateurs). Fournisseurs : Scytl et Atos Origin. Audit rapporté par "Opida" (probablement Oppida).
*   **Suisse** : Trois cantons (Genève, Neuchâtel, Zurich) offrent le vote par internet [49]. Genève possède et gère son système. Les électeurs reçoivent une "Carte de vote" avec les infos de connexion et un code PIN pour valider. Le Conseil d'État genevois a imposé 11 exigences strictes (par exemple : "Les votes ne peuvent être interceptés ni modifiés", "Le secret du vote est garanti", "Le système sera auditable") [49]. Je trouve ces exigences curieuses car théoriquement impossibles à garantir dans un système informatisé ("ne peuvent", "prouver"). Par exemple, "ne peuvent être interceptés" suppose probablement SSL, mais SSL peut être attaqué. Les Suisses utilisent aussi la cryptographie quantique pour transférer les résultats [24][54].
*   **Royaume-Uni** : Plus de 30 pilotes entre 2002 et 2007 [48]. En 2002 à Liverpool, le vote par SMS était possible (fournisseur : Election.com). Le format SMS : `<PIN> <PASSWORD> <CANDIDATE NUMBER>`. J'ai de grandes inquiétudes : SMS utilise le chiffrement A5 cassé et n'est pas fiable (retards, non-livraison). En 2002 à Liverpool : 59.4% en personne/courrier, 16.4% internet, 17.4% téléphone, 6.7% SMS.
*   **Nouvelle-Galles du Sud (Australie)** : Système "iVote" en 2011 pour les électeurs handicapés, illettrés ou éloignés [48]. Fournisseur : Everyone Counts [51]. Les électeurs s'inscrivaient, recevaient un numéro iVote et un PIN. 2 259 votes par téléphone, 44 605 par internet. Un rapport post-élection par PwC affirme qu'aucune falsification n'a été détectée via des "vérifications d'intégrité cryptographiques" [52], mais c'est trop vague. Un rapport de PwC liste des incidents, d'erreurs d'envoi de numéros à une panne de 8 minutes sans cause identifiée [52].

--[ 5 - Autres Problèmes Liés à l'Internet

La cyberguerre est devenue une affaire sérieuse. Par exemple, le 20 mars 2013, des réseaux TV et banques sud-coréens furent paralysés par une attaque attribuée à la Corée du Nord [11]. Le gouvernement américain semble paranoïaque face aux attaques venant d'Iran et de Chine [29]. Si une élection nationale se tenait en ligne, un pays étranger aurait un fort intérêt à la perturber ou la contrôler.

Une autre menace serait une attaque par hameçonnage (phishing) et/ou désinformation. Par exemple, en 2012 à Madison (Wisconsin), le parti républicain envoya des instructions d'inscription erronées dans des zones démocrates [30]. On peut imaginer des e-mails envoyant les électeurs sur un faux site de vote pour jeter leur vote ou voler leurs identifiants. Cela pourrait priver certains électeurs de leurs droits et influencer une course serrée.

Il serait aussi possible de récolter des identifiants avant l'élection via des e-mails demandant de "vérifier leur compte". Ces identifiants serviraient ensuite à voter le jour J. Impact limité mais potentiel dans une élection serrée.

Une autre attaque réelle est le rootkit navigateur, installant secrètement une extension modifiant le comportement des pages. Le système de vote Helios [32] (open source, conçu pour le vote secret vérifiable) utilise massivement JavaScript côté client et Java (JVM) pour le chiffrement (ElGamal exponentiel) [34]. JavaScript et JVM... meilleur vecteur d'attaque ? :> Helios permet aux candidats de fournir un PDF (autre vecteur d'attaque). Des chercheurs [31] ont exploité une faille PDF pour installer un rootkit dans une extension existante (Firefox ou IE). Ce rootkit espionne le trafic et agit quand l'utilisateur visite le site de vote. Il change silencieusement le vote (ex: Bart -> Alice) tout en affichant un faux écran de confirmation montrant Bart. Il modifie aussi la fonction de vérification pour toujours afficher "Chiffrement vérifié". Seul un candidat ou un admin pouvait uploader le PDF piégé.

Cette attaque montre que même avec un serveur sécurisé, le poste client (chaque ordinateur domestique) est une faille majeure. Un système bien conçu peut être compromis car le "kiosk de vote" n'est pas sécurisé.

Un autre problème lié au vote à domicile est la résurgence de la fraude par intimidation ou achat de votes. Actuellement difficile car le vote est secret dans un isoloir. Avec le vote par internet, on peut simplement regarder l'électeur voter ou organiser un "événement communautaire de vote" pour faire pression.

N'oublions pas non plus la menace interne. Comme le disait Boss Tweed, contrôler ceux qui comptent les votes est primordial. Il serait facile pour un programmeur du logiciel de vote d'insérer du code pour altérer l'élection (cf. l'incident estonien où le chef de projet pouvait pousser des changements à volonté). Pensez à l'exemple du film "Office Space" avec la petite fraction détournée de chaque transaction.

--[ 6 - Systèmes de Vote par Internet Vérifiables de Bout en Bout

Un système vérifiable cryptographiquement, Helios, a déjà été mentionné. Ces systèmes tentent de compenser les problèmes des réseaux non fiables. Cependant, le rootkit navigateur l'a compromis. D'autres systèmes utilisent des reçus physiques cryptographiquement signés, imprimés par des machines spécialisées (plus proches des DRE mais potentiellement connectées à Internet pour l'agrégation).

Un des plus connus est le système "Secret Ballot Receipts" de David Chaum [60]. Les électeurs reçoivent un reçu physique composé de deux couches laminées séparément. Ensemble, elles forment le bulletin lisible. Séparément, elles semblent aléatoires. L'électeur choisit quelle moitié garder comme preuve (après impression) et détruit l'autre sur place. Le matériel cryptographique dans la couche permet ensuite aux trustees de comptabiliser, et les électeurs vérifient sur un tableau public.

Je ne connais pas de faille dans le schéma cryptographique de Chaum, mais comme le soulignent Karlof et al. [61], l'implémentation présente de nombreuses opportunités d'erreurs ou d'attaques d'ingénierie sociale. Les citoyens ordinaires ne comprennent pas assez la cryptographie pour détecter même une altération mineure du protocole. Par exemple, si la machine demande quelle partie garder *avant* d'imprimer le reçu signé, elle peut construire les deux parties pour qu'elles décryptent vers un bulletin arbitraire choisi par l'attaquant [61].

C'est précisément mon problème avec ces systèmes. Un principe central des élections démocratiques est que les citoyens ordinaires comprennent le processus et aient confiance dans les résultats. Les citoyens ordinaires (moi y compris) ne comprennent pas ces systèmes assez en profondeur pour surveiller l'élection et avoir confiance. Pire, quelle que soit la solidité mathématique, l'implémentation des primitives cryptographiques doit être absolument parfaite. Un État-nation pourrait exploiter la faille statistique la plus subtile dans la génération de nombres aléatoires (s'il n'avait pas déjà backdooré le matériel générant les clés).

Les citoyens peuvent voir les bulletins mis dans les urnes puis comptés. Ils peuvent voir les urnes emmenées dans une arrière-salle secrète et en ressortir. Ils ne peuvent pas regarder un reçu cryptographique et dire "Ah, la génération aléatoire est défectueuse !". Un schéma cryptographique complexe et mal compris n'est pas la voie pour la confiance démocratique.

--[ 7 - Contre-Réaction

Malgré l'adoption de pilotes, il y a eu un rejet du vote électronique dans certains pays.

*   **Pays-Bas (2007)** : Interdiction des machines Nedap [58] en raison de l'absence de trace papier.
*   **Irlande (2009)** : Abandon de l'e-voting en raison du coût élevé et du manque de confiance [57].
*   **Allemagne (2009)** : Interdiction des machines électroniques (pas internet) par la Cour constitutionnelle fédérale. Ses conclusions coïncident avec mes critiques : le citoyen moyen ne peut pas comprendre ce que fait la machine ("boîte noire"), et les manipulations/fraudes dans un logiciel sont plus faciles à insérer et plus difficiles à détecter que dans un système traditionnel [59].

--[ 8 - Mais On Utilise Internet pour [Foo]

Un sophisme courant est : "Si on utilise internet pour des activités importantes comme la banque ou le commerce, pourquoi pas pour le vote ?" Deux réponses principales : la banque en ligne n'est pas secrète, et la fraude bancaire peut être "recouverte" avec de l'argent.

Supposons que j'envoie 1000€ à mon propriétaire en ligne. Le propriétaire verra la réception, je verrai le débit, la banque aura des relevés. Je peux appeler pour confirmer. S'il ment, je peux prouver le paiement. Si une erreur survient (2000€ envoyés), je le vois sur mon relevé et peux demander un remboursement. Avec le vote, secret oblige, cette vérification est impossible. Je sais que j'ai voté, mais je ne sais pas si mon vote a été compté pour mon candidat, ni même s'il a été compté. Ce serait comme voir un montant mystère débité, ne pas connaître mon solde, et ni le destinataire ni la banque n'ayant de trace.

L'autre problème est le "recouvrement" de la fraude. Quand une entreprise évalue une technologie, elle demande si les économies dépassent les coûts (dont la fraude accrue). Les banques remboursent les victimes car elles économisent globalement. Cela ne fonctionne pas avec le vote. On ne peut pas "recouvrir" une élection volée - elle est truquée et la confiance du pays est ruinée (si quelqu'un s'en aperçoit).

En e-commerce, il est courant de laisser un conjoint ou un enfant utiliser ses identifiants. C'est généralement illégal pour le vote. Avec le vote par internet, c'est impossible à contrôler. Imaginez un site type "Silk Road" [15] vendant des identifiants de vote contre des Bitcoins.

--[ 9 - Imaginer un Système de Vote par Internet Plus Sûr

Le livre *Broken Ballots* [2] mentionne un brevet américain de 1875 (Henry Spratt) pour une machine à voter. Il prétendait permettre : "le vote secret sans aide de boules, tickets, passes, lettres, chiffres, tampons officiels ou urnes ; le secret absolu (impossible de découvrir pour qui l'électeur a voté) ; la satisfaction des parties que l'électeur a voté ; la connaissance instantanée du résultat à la fermeture ; un contrôle complet empêchant toute falsification".

Cette revendication est remarquable car elle reste l'objectif central, toujours non résolu 140 ans plus tard.

Matt Bishop décrit les propriétés académiques qu'un système de e-voting doit satisfaire [56] :
1.  Ne pas pouvoir associer un vote à un électeur.
2.  Empêcher un électeur de voter plus qu'autorisé.
3.  Permettre à l'électeur de vérifier son bulletin avant de le soumettre.
4.  Comptabiliser les votes avec précision (pas d'enregistrement erroné).
5.  Permettre un audit du décompte déclaré, via un mécanisme hors-bande (un recomptage logiciel ne sert à rien si le serveur est bogué).

J'ajouterais une sixième exigence :
6.  **Confiance** : La population doit pouvoir croire que les votes ou le décompte n'ont pas été modifiés.

La question est : pourrait-on concevoir un tel système ? Comme Helios l'a montré, des modèles mathématiques existent. Mais nos ordinateurs regorgent de vecteurs d'exploitation. Ce n'est pas réalisable avec nos connaissances actuelles en sécurité, et j'espère que les exemples présentés vous en ont convaincu.

--[ 10 - Conclusion

Cet article a discuté de l'usage du vote par internet et de ses lacunes techniques. Je conclurai sur la sociologie de la démocratie. Je crois que :

1.  Le vote par internet est incompatible avec la démocratie.
2.  Aucune technologie ne peut changer cela.
3.  Le choix de vote doit rester secret.
4.  Le fait d'avoir voté ne doit *pas* être secret – il doit être connu le plus largement possible.
5.  Les personnes comptant les votes, et la manière, ne doivent *certainement pas* être secrètes.

Comme mentionné, en 1856, un comité de San Francisco découvrit une urne à double fond. Depuis, on essaie de contrer la fraude par la technologie [2].

La démocratie est miraculeuse car elle transfère le pouvoir sans violence, car les gens acceptent que le gagnant ait une légitimité. Si les gens ne croient pas en la légitimité des votes, ils ne croient pas en la légitimité du dirigeant, et le chaos social peut s'ensuivre.

Compliquant les choses : les votes doivent être secrets, sinon les citoyens peuvent être contraints ou achetés. Comme je ne peux pas vérifier dans une base de données que mon vote pour Bob a été enregistré, je dois faire confiance au processus de comptage. Je dois croire que le gouvernement résiste aux intérêts extérieurs voulant truquer le résultat.

Pendant des siècles, nous avons utilisé le papier. Loin d'être parfait, il a vu des fraudes locales. Mais il n'a pas de point de défaillance unique où une élection nationale entière pourrait être compromise, surtout dans des pays peu corrompus. Les préposés vérifient l'identité des votants (souvent reconnus localement). Les bulletins sont comptés par des gens, devant d'autres gens, dans chaque bureau. Les résultats sont ensuite agrégés. C'est un système distribué, tolérant aux pannes, reposant sur la confiance humaine en un processus humain, observable et compréhensible.

Avec le vote par internet, un simple bug logiciel pourrait affecter des bureaux, régions ou pays entiers, difficilement détectable. Un bug malveillant aurait le même effet. Il est très difficile pour les humains de savoir ce qu'un ordinateur fait exactement, surtout quand chaque ordinateur est un kiosk potentiel.

Le vote par internet n'est donc pas une modernisation de la démocratie par la technologie. C'est une technologie qui sape la confiance en un processus qui *doit* être cru pour que les gouvernements élus réussissent. Je suis un électeur heureux de continuer à utiliser des bulletins papier.

--[ 11 - Remerciements

Merci à Twiga pour son temps et ses conseils inestimables. Daw a apporté un éclairage précieux et des lectures sur le vote internet vérifiable de bout en bout.

--[ 12 - Références
*(Conservées en anglais comme dans l'original, car contenant des URLs et références techniques)*
[1] http://en.wikipedia.org/wiki/United_States_presidential_election_in_Florida,_2000
[2] Broken Ballots: Will Your Vote Count? Douglas W. Jones & Barbara Simons. 2012.
[6] http://www.emeraldinsight.com/journals.htm?articleid=863987
[7] http://www.cbsnews.com/8301-505124_162-57545531/ohio-faces-controversy-over-voting-machines/
[8] http://en.wikipedia.org/wiki/William_M._Tweed
[9] https://jhalderm.com/pub/papers/dcvoting-fc12.pdf
[11] http://www.zdnet.com/probe-says-north-korea-behind-south-korean-hack-7000013784/
[15] http://en.wikipedia.org/wiki/Silk_Road_(marketplace)
[17] http://allthingsd.com/20121105/after-sandy-new-jersey-becomes-an-unwilling-test-case-for-internet-voting/
[18] http://www.njelections.org/2012-results/directive-email-voting.pdf
[19] http://en.wikipedia.org/wiki/Electronic_voting_examples#2000_Arizona_Democratic_presidential_primary_Internet_election
[20] http://www.vvk.ee/public/dok/Internet_Voting_in_Estonia.pdf
[21] http://www.cse.wustl.edu/~jain/cse571-07/ftp/ballots.pdf
[26] http://www.poemhunter.com/poem/the-brus-book-i/
[29] http://online.wsj.com/article/SB10001424127887324345804578424741315433114.html
[30] "Election Board Warns About Confusing Mailers." http://www.channel3000.com/news/Elections-board-warns-about-confusing-mailers/-/1648/16903214/-/2jq57j/-/index.html
[31] http://static.usenix.org/event/evtwote10/tech/full_papers/Estehghari.pdf
[32] http://heliosvoting.org/
[33] https://github.com/benadida/helios-server
[34] http://www.win.tue.nl/~berry/papers/euro97.pdf
[35] https://www.schneier.com/essay-101.html
[36] http://www.politico.com/news/stories/1112/84202.html
[38] http://en.wikipedia.org/wiki/Help_America_Vote_Act
[39] http://en.wikipedia.org/wiki/Russian_legislative_election,_2011#Electoral_irregularities_and_assessment
[40] http://www.bbc.co.uk/news/world-africa-20660228
[41] http://en.wikipedia.org/wiki/Ballot_box
[42] http://www.bl.uk/onlinegallery/takingliberties/staritems/159peoplescharterpic.html
[43] http://homepage.cs.uiowa.edu/~jones/voting/pictures/
[44] http://www.nist.gov/itl/vote/upload/chapter9.pdf
[45] http://demo.intelivote.com/WEBDEMO/
[46] http://www.recorder.ca/2010/10/27/technical-snags-wont-be-repeated-intelivote
[47] http://en.wikipedia.org/wiki/DRE_voting_machine
[48] A Survey of Internet Voting: http://www.eac.gov/assets/1/Documents/SIV-FINAL.pdf
[49] http://www.systematic-paris-region.org/en/members/oppida
[50] https://en.wikipedia.org/wiki/Short_Message_Service
[51] http://www.everyonecounts.com
[52] http://www.elections.nsw.gov.au/__data/assets/pdf_file/0007/93481/iVote_Audit_report_PIR_Final.pdf
[53] http://www.youtube.com/watch?v=ZVWIOwSkMew
[54] http://spectrum.ieee.org/computing/networks/geneva-vote-will-use-quantum-cryptography
[56] Bishop, Matt. "An Overview of Electronic Voting and Security." Department of Computer Science. University of California, Davis.
[57] http://www.thedailybeast.com/newsweek/2009/05/23/we-do-not-trust-machines.html
[58] http://www.theregister.co.uk/2007/10/01/dutch_pull_plug_on_evoting/
[59] http://www.edri.org/edri-gram/number7.5/no-evoting-germany
[60] http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.71.9418&rep=rep1&type=pdf
[61] http://naveen.ksastry.com/papers/cryptovoting-usenix05.pdf


**Notes du traducteur :**
*   Les termes techniques spécifiques (DRE, SSL, chiffrement quantique, etc.) et les noms propres ont été conservés ou adaptés aux usages francophones courants.
*   Les références et URLs sont conservées en anglais.
*   Le ton critique, parfois sarcastique (ex: ":>", commentaires sur les PDF) a été préservé dans la mesure du possible.
*   Les structures longues et complexes ont été adaptées pour une lecture fluide en français tout en conservant la précision technique.
*   Les citations (Boss Tweed, constitution française) ont été traduites fidèlement.
*   Les exigences suisses et les propriétés académiques ont été traduites de manière précise et littérale.