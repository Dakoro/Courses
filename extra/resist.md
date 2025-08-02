date : 2007
Oubliez tout ce que vous savez sur les révolutions. Tout est faux.

Mener une guerre conventionnelle dans une nation industrialisée est suicidaire. Même si vous pouviez déployer une force militaire capable de vaincre les forces gouvernementales, le champ de ruines qui en résulterait ne vaudrait pas la peine. Pensez aux obus de mortier tombant sur des usines chimiques. Aux déversements massifs de déchets toxiques. Aux nuages de produits toxiques dérivant au gré des vents. Livrer une guerre dans son propre pays est tout simplement stupide. Remarquez comment les superpuissances se combattent par guerres interposées dans d'autres pays.

Bien sûr, former une milice et jouer aux soldats avec vos amis dans l'Idaho pourrait sembler amusant. Vous avez des fusils d'assaut en tir automatique ? Peut-être même des mortiers, des mitrailleuses lourdes et des canons anti-aériens ?

Pensez-vous qu'ils pourraient détruire un AC-130 tirant des obus d'artillerie depuis 20 km ? Une escadrille d'A-10 crachant des obus à uranium appauvri de la taille de votre poing à un rythme qui fait ressembler le canon à une moto-cross à régime maxi ? Une guerre ouverte contre un gouvernement moderne est un raccourci vers l'anéantissement.

La plupart des coups d'État sont accomplis (ou déjoués) par une manipulation habile de l'information. Il y a eu plusieurs pays où des tyrans (et des dirigeants légitimes) ont été renversés par de très petits groupes utilisant efficacement les communications de masse.

La méthode typique consiste à bloquer toutes (ou la plupart) des sources d'information contrôlées par le gouvernement, et à fournir une alternative qui diffuse votre message. Généralement, vous annoncez simplement le changement de gouvernement, dites à tout le monde qu'il est en sécurité et imposez un couvre-feu pendant une courte période pour consolider votre contrôle. Annoncez que le pays, la police et l'armée sont sous votre contrôle, et répétez-le sans cesse. Saturez les ondes de votre message, tout en empêchant toute propagation de messages contradictoires.

Pratiquement tous les médias de diffusion utilisent le réseau téléphonique (PSTN - Réseau Téléphonique Commuté) pour acheminer le contenu de leurs studios à leurs émetteurs. Les réseaux utilisent des satellites et le PSTN pour distribuer le contenu aux stations locales, qui utilisent ensuite le PSTN pour l'acheminer au site de l'émetteur.

Détourner ces connexions téléphoniques permet d'atteindre les deux objectifs : priver les médias "officiels" d'accès et diffuser votre propre message.

Dans les cas où vous ne pouvez pas détourner les émetteurs, couper le PSTN sera efficace. La police et l'armée utilisent également le PSTN pour relier les centres de commandement aux tours d'émission. Récemment, beaucoup ont installé des systèmes de secours sans fil (micro-ondes).

Couper physiquement le PSTN juste avant vos diffusions peut être très efficace. Cela s'accomplit le plus facilement par des dommages physiques aux installations des opérateurs télécoms, mais il existe aussi des moyens techniques non physiques pour le faire à grande échelle. Les détailler ici ne ferait que permettre de combler ces failles, mais si vous avez des personnes possédant les compétences pour le faire, c'est préférable aux moyens physiques car vous pourrez exploiter l'avantage d'utiliser ces ressources de communication au fur et à mesure de l'avancement de votre plan.

Exploiter Internet

La plupart des FUD (Peur, Incertitude, Doute) produits sur l'insurrection et Internet se concentrent sur la "destruction" d'Internet. Ce n'est probablement pas l'utilisation la plus efficace des ressources techniques. Une insurrection bénéficierait davantage de l'utilisation du net. Un usage est la communication de masse. Faites passer votre message aux masses et recrutez de nouveaux membres.

Un autre usage est la communication au sein de votre groupe. C'est là que les choses se compliquent. La plupart des gouvernements ont la capacité de surveiller et d'intercepter le trafic Internet de leurs citoyens. Les gouvernements qui méritent le plus d'être renversés sont probablement aussi les plus efficaces en matière de surveillance électronique.

Le gouvernement infiltrera également votre groupe, donc les forums ne seront pas le meilleur moyen de communiquer des stratégies et tactiques. Les forums peuvent être utiles pour des discussions générales, comme les déclarations de mission, les objectifs et le recrutement. Méfiez-vous de l'analyse de trafic et du reniflage (sniffing). TOR peut être utile, surtout si votre serveur n'est accessible que sur le réseau TOR.

Le chiffrement est votre meilleur ami, mais peut aussi être votre pire ennemi. Gardez à l'esprit que le chiffrement ne vous achète que du temps. Un chiffrement solide et efficace ne sera probablement pas lu en temps réel par votre adversaire, mais finira par être cassé. Le facteur important ici est qu'il ne soit pas cassé avant qu'il ne soit trop tard pour qu'il soit utile.

Un chiffre à masque jetable (One Time Pad - OTP) est la meilleure solution. Générez des données aléatoires et écrivez-les sur 2, et seulement 2, DVD. Transportez physiquement les DVD à chaque point de communication. Ne les laissez jamais hors de votre contrôle direct. Ne les envoyez pas par la poste. N'envoyez pas de clés via SSH ou SSL. Remettez physiquement le DVD à votre correspondant à l'autre bout. Ne réutilisez jamais une partie de la clé.

Voici une bonne façon d'utiliser votre OTP :

Générez un bon OTP (K), trouvez un message alternatif suspect (M), et connaissant votre texte secret (P), vous calculez (où "+" = addition modulo 26) :
K' = M + K
K'' = P + K
C = K' + P

Enfermez K'' dans un coffre-fort, et cachez K' dans un autre lieu sécurisé hors site. Gardez C à portée de main avec de grands panneaux "Attention aux systèmes de cryptage". Lorsque la coercition physique commence ("rubber hose cryptanalysis"), encaissez au moins deux bonnes raclées, puis donnez la clé du coffre-fort. Ils obtiennent K'' et calculent :
K'' + C = M
ce qui leur donne le message bidon, protégeant ainsi votre vrai texte.

Sécurité Opérationnelle

La configuration "cellulaire" classique est la plus sécurisée contre l'infiltration et la compromission. Une cellule typique ne doit pas compter plus de 5 à 10 membres. Un leader, 2 membres qui savent chacun comment contacter un membre d'une cellule "en amont", et 2 membres qui savent chacun comment contacter un membre d'une cellule "en aval". Personne, y compris le leader, ne doit savoir comment contacter plus d'une personne en dehors de sa propre cellule.

N'utilisez jamais votre vrai nom, et n'utilisez jamais votre pseudonyme organisationnel dans un autre contexte.

Les communications électroniques entre les membres doivent être réduites au strict minimum. Lorsque c'est nécessaire, elles ne doivent être menées que via le chiffrement OTP. De préférence, ces communications ne devraient guère dépasser l'organisation d'une rencontre physique. Rencontrez-vous à un lieu pré-arrangé, puis allez dans un autre lieu, non annoncé, où la surveillance est difficile, pour discuter des questions opérationnelles.

Ne portez pas de téléphone. Même un téléphone éteint peut être pisté, et la plupart peuvent être utilisés pour écouter les discussions même éteints. Retirer la batterie n'est que marginalement plus sûr, car des équipements de pistage/écoute peuvent être intégrés au bloc-batterie. Si vous vous retrouvez coincé avec un téléphone lors d'une réunion, retirez la batterie et placez le téléphone ET la batterie dans une boîte métallique, puis éloignez-la de la zone immédiate de conversation.

Cela ne fait jamais de mal de générer du trafic bidon. Charabia, données aléatoires, histoires anodines, etc., tout cela sert à générer du bruit dans lequel mieux cacher vos vraies communications.

La stéganographie peut être utile combinée à un chiffrement solide. Chiffrez et cachez (stego) de petits messages dans quelque chose comme un film entier en AVI, et distribuez-le à beaucoup de monde via un torrent. Seul votre destinataire prévu aura la clé pour déchiffrer le message dissimulé. Assurez-vous de cacher (stego) du bruit purement aléatoire dans d'autres films, et de les diffuser également en torrent.

J'espère que vous trouverez ce document utile comme point de départ pour des discussions et des affinements ultérieurs. Il n'est pas censé être définitif et n'est sûrement pas exhaustif. N'hésitez pas à le copier, ajouter, éditer ou modifier comme bon vous semble.

PSTN : réseaux téléphones fixes