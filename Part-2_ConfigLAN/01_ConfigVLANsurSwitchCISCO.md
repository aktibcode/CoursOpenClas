# Chapitre :  Configurez des VLAN sur un switch CISCO

Dans la première partie, nous avons vu ensemble comment importer des routeurs et switchs CISCO et comment utiliser le CLI CISCO. Dans cette nouvelle partie, vous apprendrez à administrer la partie LAN du réseau d’une entreprise, en d’autres termes, la partie privée d’une entreprise. Vous serez par exemple, capables de :

* relier tous les ordinateurs d’une entreprise,
* les faire communiquer entre eux,
* les relier à internet.

Dans ce premier chapitre, je vous propose d’apprendre à segmenter votre réseau à l’aide de VLAN. Cela vous permettra de séparer différentes utilisations d’un même réseau, pour configurer de la voix sur IP (VoIP) ou  créer une DMZ (zone démilitarisée) entre autres. Ceci vous permettra d’ajouter de la sécurité à votre réseau, mais aussi de l’optimiser.

Allez, on commence tout de suite.

### Segmentez votre réseau

Le VLAN pour Virtual Local Area Network (réseau local virtuel en français), est comme dit dans l’introduction, une façon de segmenter le réseau non pas physiquement mais logiquement.

![Illustration de plusieurs VLAN sur un switch](https://user.oc-static.com/upload/2021/07/21/1626860401332_VLAN.png)

Cette segmentation est la solution à plusieurs problèmes :

* Problème de sécurité : elle permet d’isoler certaines parties du réseau, comme les serveurs, sans recours au routeur ;
* Problème d’optimisation : la segmentation étant logique, on peut créer plusieurs réseaux avec le même nombre de switch et de câble ;
* Problème de qualité de service : il est possible de réserver de la bande passante pour la VoIP par exemple (la téléphonie par voie IP).

Prenons par exemple le cas d’un datacenter regroupant un grand nombre de serveurs de différents clients. Les clients se connectant au datacenter doivent avoir accès uniquement à leurs serveurs et non pas ceux des autres clients. Un des moyens de faire serait de séparer physiquement les réseaux à partir du routeur donc d’avoir deux interfaces réseau du côté du LAN et de continuer ainsi avec, pour chaque réseau, ses câbles et ses switchs.

![](https://user.oc-static.com/upload/2021/07/21/16268613638992_vlan%20reseau.PNG)Cette solution est en fait viable et fonctionne très bien, mais il est peu probable qu’un datacenter ait pour objectif de n’avoir que deux clients, mais plutôt plusieurs centaines ou milliers. Il faudrait alors que le datacenter prévoit des milliers d’interfaces réseaux, suivi de centaines de milliers de câbles et de switch, car croyez-moi, il est inconcevable d’ajouter tous ces composants à chaque nouveau client, étant donné le coût que cela représente.

Une des façons de faire est donc de n’avoir qu’un seul réseau physique où tous les switchs et câbles seraient déjà installés est de les segmenter à l’aide des VLAN. De cette façon, lorsqu’un nouveau client demande un serveur, par exemple, au datacenter, il vous suffira de lui ajouter un VLAN et de brancher son serveur à ce VLAN. De cette façon, il aura accès uniquement à son serveur et à rien d’autre, bien qu’étant sur le même réseau physique que d’autres personnes (peut-être sur le même switch qu’un autre serveur), de la même manière personne d’autre n’aura accès à son serveur. C’est une façon simple et fiable d’ajouter de la sécurité dans un réseau LAN.

### Parcourez la trame d’un VLAN

Pour ceux qui ne se rappellent pas, c’est l’occasion de relire comment se décompose une trame Ethernet car une trame de VLAN est sensiblement la même avec un petit ajout. N’hésitez donc pas à revoir le cours « [Apprenez le fonctionnement des réseaux TCP/IP](https://openclassrooms.com/courses/apprenez-le-fonctionnement-des-reseaux-tcp-ip/faire-communiquer-les-machines-entre-elles-la-couche-2) ».

Cet ajout est normalisé par le protocole 802.1q (vous entendrez souvent « dot one Q » en anglais). Voilà comment elle se décompose :

![tableau 802.1Q](https://user.oc-static.com/upload/2021/07/21/16268614087517_Sch%C3%A9ma%20r%C3%A9seau-Page-2.png)
tableau 802.1Q

Elle commence de la même façon avec :

* Le préambule ;
* L’adresse de destination ;
* L’adresse source ;

Puis, le protocole dot1q ajoute deux fois 4 octets (deux pour le VLAN et deux pour l’Ether Type) à cet endroit de la trame :

* Tout d’abord, le champ Ether Type où est renseigné le protocole utilisé ici 0X8100 (pour 802.1q). Dans wireshark, vous verrez directement 802.1Q car il vous facilite la lecture ;
* Puis, les deux octets 802.1q décomposés comme ceci :
  * 3 bits pour la priorité (donc 8 priorités), 111 étant la priorité la plus haute ;
  * 1 bit appelé CFI qui doit être à 0 (pour des problèmes de compatibilité) ;
  * puis 12 bits pour le VLAN ID ce qui fait 4096 VLAN possibles dans un réseau ;
  * suit un autre Ether Type ;
  * puis les données.

Maintenant que nous avons vu ce qu’était un VLAN, voyons comment le créer sur votre switch CISCO, afin de placer le nouveau client d’e-commerce sur un réseau sécurisé, auquel personne d’autre n’aura accès.

### Créez un VLAN sur votre switch CISCO


<iframe id="video_Player_2" src="https://player.vimeo.com/video/577679969?color=7451eb" frameborder="0" class="video js-claire-video" data-src-origine="https://vimeo.com/577679969" title="Video" webkitallowfullscreen="" mozallowfullscreen="" allowfullscreen="" data-ready="true"></iframe>

Ajoutez ces adresses aux PC avant de commencer la configuration des VLAN.

![Maquette du data center](https://user.oc-static.com/upload/2021/07/21/16268614784817_VLAN%20SCHEMA%201.PNG)
Maquette du data center

En tant qu’administrateur·rice du datacenter, il vous faut ajouter un premier client au réseau et donc un premier VLAN.

Pour créer un VLAN sur votre switch, il vous faut tout d’abord entrer en mode configuration globale :

<pre id="r-7351600" data-claire-element-id="31871339" class=""><samp>switch1# configure terminal</samp></pre>

#### Ajoutez un VLAN

Puis entrez cette commande :

`switch1(config)# vlan { vlan-id | vlan-range }`

* Le pipe  `|`  entre “vlan-id” et “vlan-range” et un "ou", ce qui signifie que vous pouvez utiliser la commande vlan soit en ajoutant un seul vlan-id ou en ajoutant un vlan-range.
* Les crochets  `{}`  signifient qu’un de ces arguments est obligatoire après la commande vlan.

Pour notre datacenter qui prévoit d’avoir 1000 clients cette année, la commande serait :

<pre id="r-7351601" data-claire-element-id="31871347" class=""><samp>switch1(config)# vlan 1-1001</samp></pre>

Le vlan 0 représentant ce que l’on appelle l’access c’est-à-dire l’absence de VLAN.

Le vlan 1 est le vlan natif, c’est-à-dire le vlan par défaut, si vous ne configurez par vos interfaces, elles seront toutes sur ce VLAN. Nous reviendrons sur ce concept juste après.

La commande   `no vlan` permet de supprimer un VLAN dans le cas où vous vous seriez trompés. Elle fonctionne de la même manière que  `vlan`  .

<pre id="r-7351602" data-claire-element-id="31871350"><samp>switch1(config)# no vlan 1</samp></pre>

Cette commande supprime le vlan1

#### Configurez votre VLAN

Votre VLAN étant créé, il ne vous manque plus qu’à le configurer. La commande pour entrer en mode configuration vlan est… exactement la même. En fait, cette commande créée un VLAN s’il n'existe pas déjà. Dans le cas contraire, il passe en mode configuration sur celle-ci.

Si vous voulez configurer le VLAN 2 :

<pre id="r-5306572" data-claire-element-id="10144792"><samp>switch1(config)# vlan 2 (pour entrer en mode configuration vlan)

switch1(config-vlan)# name e-commerce (pour lui donner un nom)

switch1(config-vlan)# state active

switch1(config-vlan)# no shutdown (pour l’activer)
</samp></pre>

Ces commandes fonctionnent aussi pour des ranges comme à la création d’ID.

#### Attribuez un port au VLAN

C’est la dernière étape avant que votre VLAN ne soit configuré. Cette étape consiste à attribuer votre VLAN à un port de votre switch. De cette façon, vous allez pouvoir construire son réseau tout en l’optimisant.

En effet, votre VLAN n’est actuellement disponible sur aucun port de vos switchs et donc de votre réseau. Vous pouvez voir l’association d’un port à votre VLAN comme le fait de brancher un câble sur votre réseau physique. Pour brancher le serveur du nouveau client, il va falloir associer tous les ports qui partent du routeur et qui mènent à lui. Dans notre exemple c’est le port Gi0/0/0 qui est relié au serveur d’e-commerce. Nous allons donc associer ce port au VLAN 2 que nous venons de créer.

Voici les commandes :

<pre id="r-5306578" data-claire-element-id="31871356" class=""><samp>switch1(config)# interface gigabitethernet 0/0/0 (on passe sur l’interface voulue)

switch1(config-if)# switchport access vlan 2 (on ajoute le VLAN)
</samp></pre>

#### Vérifiez votre configuration

Ne jamais oublier de vérifier votre configuration et pour cela il existe des commandes très simples :

<pre id="r-5306651" data-claire-element-id="10144891" class=""><samp>switch1# show vlan id 2 (pour voir un vlan en particulier)

switch1# show vlan summary (pour avoir un résumé des VLANs existants)

switch1# show running-config vlan 2 (on spécifie le VLAN ou le range que l’on veut vérifier)

switch1# show vlan (pour voir tous les VLAN existants)

VLAN Name                             Status    Ports

---- -------------------------------- --------- -------------------------------

1    default                          active    Gi0/0, Gi0/2, Gi0/3, Gi1/0

Gi1/1, Gi1/2, Gi1/3, Gi2/0

Gi2/1, Gi2/2, Gi2/3, Gi3/0

Gi3/1, Gi3/2, Gi3/3

2   e-commerce                       active    Gi0/1 (On voit ici que le VLAN 2 est créé et qu’il est associé au port Gi0/1)

1002 fddi-default                     act/unsup

1003 token-ring-default               act/unsup

1004 fddinet-default                  act/unsup

1005 trnet-default                    act/unsup
</samp></pre>

On s'aperçoit de plus qu’il existe un VLAN 1.

Ça me rappelle quelque chose ?

Eh oui, dans la partie précédente nous activons le protocole spanning-tree pour le VLAN 1.

#### Changez le VLAN ID du VLAN natif

Ce VLAN 1 s’appelle le VLAN natif. En fait, sur un switch CISCO (même chez d’autres constructeurs) par défaut, tous les ports sont configurés sur le VLAN natif c’est-à-dire le VLAN 1. Du coup, lorsque l’on branche un ordinateur sur un port dont le VLAN n’est pas configuré, cet ordinateur est en fait sur le VLAN 1.

Il est possible et même recommandé de changer l’ID du VLAN natif. Pas d’inquiétude, chaque chose en son temps on en reparle juste après un nouveau concept.

#### Vérifiez que vous n’avez pas accès au serveur d’e-commerce

Pour vérifier votre configuration, je vous propose de tester la connexion vers le serveur du client d’e-commerce depuis un autre poste ne se trouvant pas sur le même VLAN.

Pour cela, configurez le deuxième PC (appelé autre_client sur la maquette) avec l’adresse 192.168.2.2/24. Sans VLAN, ces deux serveurs devraient pouvoir communiquer, mais ils ne se trouvent plus sur le même VLAN. C’est comme s’ils n’étaient plus sur le même réseau...

Tentez de faire un PING entre les deux… ça ne fonctionne pas et c’est tant mieux. Bien qu’étant sur le même réseau (192.168.2.X/24), ils ne peuvent pas communiquer, car ils sont sur un VLAN différents.

Configurez maintenant cette interface au VLAN 2 et refaites le test.

Magie, ça fonctionne !!!

Vous avez noté que le réseau est en 192.168.2 ?

C’est une bonne pratique pour s’y retrouver dans les VLAN que de faire correspondre le 3e octet (ici le 2) avec l’ID du VLAN (2 aussi). Bien évidemment, tous les VLAN pourraient être configurés avec la même plage réseau, cela ne changerait rien.


### Faites passer plusieurs VLAN sur le même câble


<iframe id="video_Player_3" src="https://player.vimeo.com/video/577680007?color=7451eb" frameborder="0" class="video js-claire-video" data-src-origine="https://vimeo.com/577680007" title="Video" webkitallowfullscreen="" mozallowfullscreen="" allowfullscreen="" data-ready="true"></iframe>

Le datacenter grandit et il se retrouve maintenant avec plusieurs clients, plusieurs serveurs et donc plusieurs switchs. Et à ce moment, le client de la société d’e-commerce pour laquelle vous avez attribué le VLAN 2 vous demande un accès à un second serveur, cependant sur le switch auquel son premier serveur est branché, il n’y a plus de place.

Vous le branchez donc sur un autre switch comme ceci :

![Evolution du data center](https://user.oc-static.com/upload/2021/07/21/16268622140535_VLAN%20SCHEMA%202.PNG)
Evolution du data center

Changez l’adresse de l’autre client en 192.168.3.1/24 et placez-le sur le VLAN 3 et ajoutez un deuxième identique sur le switch 0.

Et n’oubliez pas d’ajouter le nouveau serveur sur le VLAN2.

#### Faites passer plusieurs VLAN dans le trunk

Une fois la nouvelle interface du nouveau switch configurée, tentez de faire un ping entre les deux serveurs… ça ne fonctionne pas. En effet, pour que cela fonctionne, il faudrait que le câble qui relie les deux switchs soit aussi sur le VLAN 2 mais dans ce cas, les switchs du VLAN 3 ne pourraient pas communiquer entre eux !

Et pour que deux VLAN ou plus partagent le même câble, on configure les interfaces de ce câble en “ **trunk** ”.

C’est quoi un trunk ?

Nous l’avons déjà vu, mais le trunk est un moyen du protocole dot1q. Il permet de faire communiquer deux appareils (des switchs en général) et leurs VLAN respectifs. De cette façon, le VLAN 2 du switch 1 et du switch 2 peuvent communiquer ensemble.

Attention, cela ne veut pas dire que des VLAN de différents ID peuvent communiquer ! Le VLAN 2 ne peut toujours pas communiquer avec le VLAN 3 !

La configuration se passe sur l’interface qui relie les deux switchs, comme ceci :

<pre id="r-7351608" data-claire-element-id="31871373" class=""><samp>switch1# configure terminal
switch1(config)# interface GigabitEthernet 0/1
switch1(config-if)#switchport trunk encapsulation dot1q
switch1(config-if)# switchport mode trunk
switch1(config-if)# switchport trunk allow vlan 2-3
</samp></pre>

* La commande  `switchport trunk encapsulation dot1q`  force l’interface à passer en dot1q (sinon elle est en auto).
* La commande  `switchport mode trunk`  passe le lien en trunk.
* La commande  `switchport trunk allow vlan 2-3`  ajoute les VLAN 2 et 3 au trunk (en plus du VLAN natif)

On fait la même opération sur l’interface du switch2. Et on retente le ping entre les deux serveurs du VLAN2… heureusement cela fonctionne comme on le veut.

Vous avez maintenant un moyen très utile pour segmenter et sécuriser vos réseaux privés. Cela vous sera très utile pour l’administration de votre LAN. Étrangement, vous découvrirez dans le prochain chapitre comment relier ces VLAN entre eux, et saurez bien évidemment pourquoi vous devez le faire.


### **En résumé**

* Vous pouvez segmenter un réseau physique grâce au VLAN sans modifier le câblage.
* Les VLANs sont standardisés par le protocole **802.1q** appelé aussi ** Dot1q** .
* Les commandes à retenir sont :
  * `<strong>vlan</strong><span> </span>{ vlan-id | vlan-range }`, pour la création de VLAN ;
  * **`name vlan-name`** , pour lui donner un nom ;
  * **`switchport access vlan vlan-id`** , pour assigner un VLAN à une interface (une fois que vous êtes sur la configuration de l’interface).
* Il est possible de vérifier vos VLAN avec les commandes :
  * `<strong>show running-config vlan</strong><span> </span>[ vlan_id | vlan_range ]`;
  * **`show vlan`** [ brief | id [ vlan_id | vlan_range ] | name name | summary  ].
* Pour faire passer plusieurs VLANs sur une interface il faut la passer en mode** trunk** (une fois que vous êtes sur la configuration de l’interface) :
  * **`switchport mode trunk`  ** ;
  * **`switchport trunk allowed vlan`** { vlan-list all | none [ add |except | none | remove { vlan-list }]}
