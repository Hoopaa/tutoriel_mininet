# Tutoriel mininet/openvswitch

Mininet est un logiciel open source qui est utilisé pour simuler des <i>Software defined network (SDN)</i>. Ce logiciel utilise un système de contrôleurs, de switchs et d'hôtes.

Mininet supporte le protocole OpenFlow. 

## Commandes utiles

Mininet fournit une interface en ligne de commande qui peut être utilisée pour voir l'état du réseau et faire des tests dessus. 

* <i>mininet> nodes </i>  - Affiche les noeuds dans le réseau. h correspond à un host, s a un switch et c a un contrôlleur.
* <i>mininet> net</i> - Affiche la topologie du réseau. Quelle machine est connectée à quel host et le nom de l'interface réseau de connexion.
* <i>mininet> dump</i> - Affiche les adresses IP de chaque machine ainsi que le nom de la carte réseau. 
* <i>mininet> h1 ping h2</i> - Demande à l'hôte h1 d'effectuer un ping sur l'hôte h2.
* <i>mininet> h1 ifconfig -a</i> - Effectue et affiche les résultats de la commande <i>ifconfig</i> sur la machine souhaitée.
* <i>mininet> pingall</i> - Permet de tester la connectivité du réseau. Toutes les machines vont se ping entre elles. 
* <i>mininet> link s1 h1 down</i> - Enlève le lien réseau entre s1 et h1.
* <i>mininet> link s1 h1 up </i> - Remet le lien réseau entre s1 et h1.

## Miniedit

Mininet fournit un outil qui permet de créer sa propre topologie de manière graphique du nom de miniedit. 

Cette interface s'exécute avec la commande :
* <i>sudo mininet/examples/miniedit.py</i>

Par défaut, la topologie donnée par mininet est constituée d'un contrôleur, un switch et deux hôtes. Sur miniedit, cette topologie ressemble à la figure suivante : 




