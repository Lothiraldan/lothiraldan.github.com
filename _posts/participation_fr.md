Préparation pour le Google Summer Of Code 2011
==============================================

date  
2011-04-04

category  
GSOC

lang  
fr

tags  
GSOC, Python

slug  
gsoc-2011-preparation

Un petit article pour vous parler de mon projet pour cet été. J'ai en effet prévu de participer au Google Summer Of Code. Pour ceux qui ne connaissent pas, c'est une programme financé par Google pour faire avancer les projets open-source. Pour résumer, google paye des étudiants pour travailler sur des projets open-source.

Pour un passionné de technologies open-source, c'est vraiment LE truc à faire pendant l'été.

Parlons maintenant un peu du projet auquel je vais essayer de participer : PYpi Testing Infrastructure (PYTI pour les intimes). Ce projet a été proposé il y quelques années déjà, il a été choisi par l'un des étudiants de l'édition précédente, mais il reste du travail à faire pour le finaliser.

La problématique qui a motivé ce projet est la suivante : comment détecter les distributions nuisibles et les distributions buguées envoyées sur PYPI ? L'idée pour répondre à ça est de lancer des tests dans un environnement contrôlé pour permettre la détection des comportement suspects. Ces tests iraient de l'installation de la distribution aux détections de comportements suspects en passant par les test unitaires et les indicateurs de qualités. En ce qui concerne l'environnement contrôlé, nous avons décidé de partir sur des Machines Virtuelles plutôt que de faire du sandboxing, car si la machine virtuelle est compromise, il suffira de la récréer; si le sandboxing ne suffit pas à confiner les distributions nuisibles, ce serait la machine hôte qui serait directement compromise, ce qui risque d'être plus problématique. De plus, les VMs n'auront pas d'accès au réseau, ainsi les codes nuisibles ne pourront pas faire grand chose. On peut remarquer que la détection de ces comportements peut s'apparenter à de la détection de virus, ainsi ce sujet sera relativement complexe à mettre en œuvre et formera le cœur du projet.

Je ne vais pas détailler les différents points qui seront abordés pendant ce projet, je vous invite à regarder ma proposition ici : <http://www.google-melange.com/gsoc/proposal/review/google/gsoc2011/lothiraldan/1>.

Si vous voulez avoir votre mot à dire, vous pouvez contacter la mailing-liste suivant : <the.fellowship.of.the.packaging@librelist.com>. Nous avons aussi besoin de votre aide pour recueillir des statistiques sur les dépendances de vos distributions python et les indicateurs que vous aimeriez avoir sur ceux-ci, merci de prendre 2 minutes pour remplir le google moderator suivant : <http://www.google.com/moderator/#16/e=62dc1>.

PS : Pourquoi avoir fait un si petit article pour ne rien dire, en fait, je vais devoir bloguer pendant le gsoc (cela fait partie de mon travail), mais comme la plupart des participants parleront anglais, je vais sûrement bloguer en anglais et si vous êtes développeur français parlant uniquement la belle langue de Shakespeare, ça serait dommage de vous laisser à l'écart de ce projet qui devrait contribuer à toute la communauté, donc j'essayerait aussi de faire le point sur mon travail en langue française (sûrement moins souvent par contre).
