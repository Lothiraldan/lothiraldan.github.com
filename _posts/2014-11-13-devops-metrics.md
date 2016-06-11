---
layout: post
title: Métriques DevOps
tag: DevOps
---

La semaine dernière nous avons eut l’occasion de participer à un open-space sur le sujet des métriques. La question posée était : “Quelles sont les métriques à monitorer pour améliorer la collaboration entre les équipes de Développement et les équipes d’Éxploitation ?”.

On as eut pas mal d’idées, divergés sur d’autres sujets mais nous avons notés une liste de métriques intéressantes à mettre en place dans ce but. Sans plus attendre, les voici:

<table>
<colgroup>
<col width="12%" />
<col width="44%" />
<col width="6%" />
<col width="37%" />
</colgroup>
<tbody>
<tr class="odd">
<td>Durée des tests</td>
<td>Tout déploiement devrait d’abord être validé par les tests, un build court profite à tout le monde</td>
<td>Le plus petit possible</td>
<td>Mocker plus de services côté dev ou installer et configurer un serveur de DB en RAM</td>
</tr>
<tr class="even">
<td>Durée de déploiement</td>
<td>Plus un déploiement est long, plus on attendra pour avoir la dernière feature ou le dernier fix en production. Des déploiements plus courts c’est plus de déploiements possibles.</td>
<td>Le plus petit possible</td>
<td>Packager le code applicatif en ZIP / DEB / RPM ou installer un mirroir de package système / python / ruby pour accélerer les déploiements</td>
</tr>
<tr class="odd">
<td>Fréquence de déploiement</td>
<td>Des déploiements plus fréquents, c’est des features qui arrivent plus tôt pour les clients et des commerciaux plus content</td>
<td>Le plus proche du besoin produit</td>
<td>Travailler le workflow, réduire les points de douleur dans le déploiement, côté code applicatif ou côté migration de données</td>
</tr>
<tr class="even">
<td>Taux de réussite du déploiement</td>
<td>Personne n’aime faire d’erreur mais ça arrive, il faut faire en sorte d’apprendre de ses erreurs</td>
<td>100%</td>
<td>Monitorer les déploiements, passer à un outil de gestion de configuration, fixer chaque erreur pour éviter que ça se reproduise</td>
</tr>
<tr class="odd">
<td>Temps moyen pour déployer une nouvelle feature</td>
<td>Souvent les nouvelles features stagnent en attendant une release ou sont reportées parce que les besoins ont étés mal anticipés, nouveau serveur, nouvelle technologie…</td>
<td>Le plus petit possible</td>
<td>Faire participer les équipes d’exploitation aux réunions d’architecture, travailler le workflow, faire collaborer les équipes sur l’écriture de recettes de déploiement</td>
</tr>
<tr class="even">
<td>Pourcentage de serveurs automatisés</td>
<td>Les serveurs non automatisés sont souvent ceux qui nous trahissent au plus mauvais moment</td>
<td>100%</td>
<td>Écrire des recettes de configuration pour ces anciens serveurs</td>
</tr>
<tr class="odd">
<td>Pourcentage de tâches manuelles</td>
<td>Il reste souvent des tâches manuelles à faire lors du déploiement, à la création d’un nouveau client, etc… Ces tâches sont une source probable d’erreurs, on peut les oublier, la personne qui les fait d’habitude peut-être en maladie, en vacances, etc…</td>
<td>0%</td>
<td>Écrire des scripts, intégrer les tâches dans le code applicatif</td>
</tr>
<tr class="even">
<td>Pourcentage d’écarts entre les environnments</td>
<td>On ne peut pas souvent se permettre d’avoir des environnements parfaitement conforme à la production, souvent pour des question de coût. Ça peut-être une version de la DB différente, des règles réseau différentes. Ces différences sont une source de problèmes potentiels</td>
<td>0%</td>
<td>Identifier les différences et les réduire. Donner la responsabilité de l’ensemble des environnements aux deux équipes</td>
</tr>
<tr class="odd">
<td>Temps de livraison d’une nouvelle machine / d’un nouvel environnement</td>
<td>Même si les déploiements sont entièrement automatisés et rapide comme l’éclair, il reste parfois des opérations qu’on fait peut souvent, à la création d’une nouvelle machine ou d’un nouvel environnement, ce qui nous ralentit si elles ne sont pas automatisées</td>
<td>Le plus petit possible</td>
<td>Identifier ces opérations et qualifier le ration temps à automatiser / (temps à faire à la main) * fréquence</td>
</tr>
<tr class="even">
<td>MTTR, mean time to recover</td>
<td>Automatiser le déploiement c’est nécessaire, pouvoir fixer la production rapidement, ça l’est tout autant</td>
<td>Le plus petit possible</td>
<td>Améliorer le monitoring, automatiser la collection d’information, centraliser la collecte des logs, automatiser les actions courantes tel que mettre le site en maintenance, <a href="https://blog.serverdensity.com/guide-handling-incidents-downtime-outages/">écrire et suivre une doc de réponse aux incidents</a></td>
</tr>
</tbody>
</table>

Toutes ces métriques sont utiles pour identifier des améliorations réalisable par les efforts combinés des équipes de dévelopements et les équipes d’exploitation. Néanmoins en fin de session, on s’est demandé si il n’y aurait pas des métriques intéressante pour l’ensemble du cycle de développement et celà s’est révélé plus difficile que pour les autres. Nous en avons réussi à en identifier une qui englobe au final plusieurs citées auparavant :

-   Temps moyen entre l’acceptation d’une feature et son déploiement

Cette métrique est intéressante par elle concerne à la fois la collboration entre le product owner et les développeurs et la collaboration entre les développeurs et les administrateurs système. Et vous avez-vous des idées de métriques que vous avez mis en place ou que vous pensez pertinentes ?

EN’oubliez pas que vous pouvez suivre les actualités de parisdevops sur le channel irc \#parisdevops de freenode, sur le compte twitter [@parisdevops](https://twitter.com/parisdevops) et sur notre site web: [parisdevops.fr](http://parisdevops.fr)

Merci à tous!
