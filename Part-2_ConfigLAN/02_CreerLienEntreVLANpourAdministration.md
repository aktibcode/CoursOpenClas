# Chapitre : Créez des liens entre vos VLAN pour l’administration

Dans ce chapitre, vous allez continuer l’administration de votre réseau local en connectant les VLANs à Internet et en créant des routes entre ces VLANs. Ces routes vous permettront, en tant qu’administrateur, d’avoir accès aux autres VLANs, mais aussi de créer un lien entre deux groupes qui en auraient le besoin.


### Managez vos switchs en vous attribuant un VLAN

<iframe id="video_Player_1" src="https://player.vimeo.com/video/577680038?color=7451eb" frameborder="0" class="video js-claire-video" data-src-origine="https://vimeo.com/577680038" title="Video" webkitallowfullscreen="" mozallowfullscreen="" allowfullscreen="" data-ready="true"></iframe>

Le réseau du datacenter est maintenant configuré pour les clients, mais pas pour vous ! En effet, les VLANs permettent de segmenter le réseau et donc de le sécuriser.

La conséquence ?

Vous vous retrouvez dans l’incapacité de vous connecter à vos propres switchs depuis le réseau !

Lorsque l’on se connecte sur un appareil dans Cisco Packet Tracer, il simule une connexion physique directe, ce qui ne sera pas le cas dans la réalité. Vous n’irez pas vous connecter à chaque appareil en vous baladant dans l’entreprise, mais prendrez contrôle à distance par des moyens que nous verrons lors du chapitre sur la sécurité.

#### Ajoutez un VLAN pour l’administration

Ajoutez donc le VLAN 99 et donnez-lui pour nom : « administration ».

C'est depuis ce VLAN que vous administrerez les switchs et le routeur du datacenter ainsi que les autres serveurs du datacenter.

Pour vous connecter à un switch, il vous faut :

* 1 PC avec le protocole telnet (terminal network), c’est un protocole vous permettant de vous connecter à un autre appareil (PC, switch, routeur) à distance.
* Une adresse sur laquelle vous connecter.

C’est ce deuxième point qu’il vous manque. Tous les PC ont telnet par défaut, par contre votre switch n’a pas d’adresse IP !

#### Ajoutez une interface virtuelle pour le management

Pour vous connecter au switch, vous allez lui ajouter une interface virtuelle (ou VLAN interface). Ceci vous permettra d’ajouter une adresse à votre switch et donc de vous y connecter.

<pre id="r-7351632" data-claire-element-id="31871451" class=""><samp>switch2(config)#interface vlan 99
switch2(config-if)# ip address 192.168.99.2 255.255.255.0
switch2(config-if)#no shutdown
switch2(config-if)#end</samp></pre>

Vérifiez avec la commande : **`show running-config`** . Vous devriez avoir une interface de plus avec ce nom et cette adresse. Maintenant, le switch est disponible depuis le VLAN 99. Tentez un ping depuis le poste de l’administrateur que vous avez ajouté. Si c’est ok, c’est que vous pourrez vous y connecter.

Faites la même chose depuis le switch 1 :

* Ajoutez une interface VLAN 99 avec l’adresse 192.168.99.1/24 ;
* N’oubliez pas d’ajouter le VLAN 99 au trunk de chaque switch ;
* Ajoutez le VLAN 99 au switch 1 (sans lui attribuer de port) ;
* Faites un ping pour vérifier.

Vous pouvez ainsi configurer tous vos matériels depuis votre poste d’administrateur. En revanche, si un client vous demande de redémarrer son serveur, vous ne pouvez le faire que physiquement à ce stade. Cela risque de vous ralentir considérablement dans votre administration d’un datacenter. Pour éviter cela, il va falloir créer des routes entre le VLAN 99 et les autres VLANs.


### Managez les serveurs depuis le poste de l’administrateur

<iframe id="video_Player_2" src="https://player.vimeo.com/video/577680089?color=7451eb" frameborder="0" class="video js-claire-video" data-src-origine="https://vimeo.com/577680089" title="Video" webkitallowfullscreen="" mozallowfullscreen="" allowfullscreen="" data-ready="true"></iframe>

Vous avez dû remarquer qu'aucun serveur n'avait accès au routeur, et donc à Internet ! C’est la première étape : nous allons d’abord les relier au serveur et depuis celui-ci créer la route entre le VLAN 99 et les autres.

#### Reliez vos VLANs au routeur

Effectivement, ils ne le sont pas. Entre le routeur et le switch, il n'y a qu'un câble. Cela vous oblige à configurer un trunk. Cependant, vous n'avez qu'une interface du côté du routeur et plusieurs VLANs à configurer.

Aucun problème pour l'administrateur que vous êtes, car il existe une solution. Vous allez créer une sub-interface pour chaque VLAN sur le routeur. Ces sub-interfaces seront reliées à l'interface physique du routeur, celle où le câble est branché, et fonctionneront de la même manière que n'importe quelle interface physique.

#### Créez des sub-interfaces et reliez vos VLANs à Internet

La sub-interface que vous allez créer sera dépendante de l’interface reliant le routeur au switch 1, c'est-à-dire l’interface XXX. Elle en portera d’ailleurs le nom, suivi d’un point et d’un chiffre.

![Les sous-interfaces](https://user.oc-static.com/upload/2021/07/21/16268629609322_SOUS%20INT.png)
Les sous-interfaces

Par exemple si je veux créer des sub-interfaces dépendantes de l’interface GigabitEthernet 0/0/1, je vais créer les interfaces :

* GigabitEthernet 0/0/1.1
* GigabitEthernet 0/0/1.2
* GigabitEthernet 0/0/1.3 ...

Pour créer vos sub-interfaces, rien de plus simple, il vous suffit de taper ces quelques commandes :

<pre id="r-5306767" data-claire-element-id="31871468" class=""><samp>Router1#configure terminal

Router1(config)#int gigabitEthernet 0/0/0 
Router1(config-if)#no shut
Router1(config-if)#exit

!-- On créé la sub-interface et on la déclare comme étant le VLAN 1 et le VLAN natif

Router1(config)#int gigabitEthernet 0/0/0.1

Router1(config-subif)#encapsulation dot1Q 1 native

Router1(config-subif)#ip address 192.168.1.254 255.255.255.0
Router1(config-subif)#exit

!-- On créé le sub-interface et on la déclare comme étant le VLAN 2

Router1(config)#int fastEthernet 0/0/0.2

Router1(config-subif)#encapsulation dot1Q 2

Router1(config-subif)#ip address 192.168.2.254 255.255.255.0
Router1(config-subif)#exit

!-- On créé le sub-interface et on la déclare comme étant le VLAN 99

Router1(config)#int fastEthernet 0/0/0.99

Router1(config-subif)#encapsulation dot1Q 99

Router1(config-subif)#ip address 192.168.99.254 255.255.255.0
Router1(config-subif)#exit
</samp></pre>

Une fois que vous avez configuré l’interface du switch 1 en trunk pour tous les VLANs, vous pouvez pinger un serveur du client d’e-commerce depuis le routeur.

Allez, on vérifie.

<pre id="r-5306770" data-claire-element-id="10145065" class=""><samp>Routeur1#ping 192.168.2.1</samp></pre>

Si votre routeur peut pinger votre serveur, c’est que ce dernier sera en mesure d’avoir Internet une fois que le WAN sera configuré !

Vos VLANs sont maintenant connectés au routeur, ce qu'il leur donne accès à Internet une fois que le routeur y est connecté.

Il ne vous manque plus qu’à créer la route entre le VLAN 99 et les autres.

#### Créez une route entre le VLAN 99 et les autres

Vous pouvez, pour commencer, faire un  **`show ip interface brief` ** . Cela vous permet de visualiser les interfaces et les adresses IP.

<pre id="r-5306777" data-claire-element-id="10145072" class=""><samp>routeur1#sh ip int brief

Interface                  IP-Address      OK? Method Status                Protocol

GigabitEthernet0/0         unassigned      YES NVRAM  up           up

GigabitEthernet0/0.1       192.168.1.254   YES manual up                 up

GigabitEthernet0/0.2       192.168.2.254   YES manual up                 up

GigabitEthernet0/0.3       192.168.3.254   YES manual up                 up

GigabitEthernet0/0.99      192.168.99.254  YES manual up                 up

GigabitEthernet0/1         unassigned      YES NVRAM  administratively down down

GigabitEthernet0/2         unassigned      YES NVRAM  administratively down down

GigabitEthernet0/3         unassigned      YES NVRAM  administratively down down
</samp></pre>

Il vous faut ensuite ajouter le**`gateway`** aux PC comme ceci :

* Administrateur = IP 192.168.99.99 MASQUE 255.255.255.0 PASSERELLE 192.168.99.254
* Client d’e-commerce= IP 192.168.2.1 MASQUE 255.255.255.0 PASSERELLE 192.168.2.254
* Autres clients= IP 192.168.3.1 MASQUE 255.255.255.0 PASSERELLE 192.168.3.254

Vérifiez en faisant un ping du poste de l’administrateur vers un des serveurs.

Ça fonctionne et c’est tant mieux. Vérifiez si les serveurs ont accès au poste de l’administrateur… Ça fonctionne aussi, et ça, ce n’est pas bon mais chaque chose en son temps. Nous verrons cela dans la partie 4 de ce cours, consacrée à la sécurité justement.

Maintenant que votre LAN est configuré et que vos VLANs ont accès aux routeurs, il ne vous manque plus qu’à ajouter certains services vous permettant de vous faciliter l’administration de votre datacenter. Il vous faut aussi rendre les serveurs de vos clients accessibles depuis le NET. Vous découvrirez tout cela dans le prochain chapitre.

### **En résumé**

* En ajoutant des VLANs interfaces au switch et aux routeurs, vous pouvez les administrer depuis n’importe quel poste :

  * * switch(config)#interface vlan 99

  switch(config-if)# ip address adresse-ip masque
  switch(config-if)#no shutdown
  switch(config-if)#end
* Pour relier vos VLANs aux routeurs, vous devez créer des **sub-interfaces** et leur attribuer un VLAN et une adresse IP. Cette interface deviendra le **gateway** du VLAN :

  * Router(config)#int fastEthernet 0/0.X
  * Router(config-subif)#encapsulation dot1Q X
  * Router(config-subif)#ip address adresse-ip masque
  * Router(config-subif)#exit
* Une fois vos VLANs reliés au routeur, le routeur crée les routes permettant la communication entre eux.
