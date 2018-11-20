# Tutoriel mininet/openvswitch

Mininet est un logiciel open source qui est utilisé pour simuler des <i>Software defined network (SDN)</i>. Ce logiciel utilise un système de contrôleurs, de switchs et d'hôtes.

Mininet supporte le protocole OpenFlow. 

## Commandes utiles

Pour éxecuter mininet, il faut utiliser la commande suivante : 
* <i>sudo mn</i>

De nombreuses options sont à disposition, elles peuvent être affichées en ajoutant le flag <i>-h</i>

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

![Default topology](/img/default_topo.PNG)

Les différents outils sur la gauche de miniedit permettent de modifier la topologie affichée.

![Tools Miniedit](/img/tools_miniedit.PNG)

Dans l'ordre d'affichage de la figure précédent :
1. Permet de déplacer les objets dans la topologie.
1. Permet d'ajouter un hôte à chaque clic.
1. Permet d'ajouter un switch à chaque clic.
1. Permet d'ajouter un switch legacy à chaque clic.
1. Permet d'ajouter un routeur legacy à chaque clic.
1. Permet de faire des liens entre les objets de la topologie. Le clic doit être maintenu.
1. Permet d'ajouter un contrôleur à chaque clic.

Lorsque la topologie a été créée avec miniedit, il est possible de l'exporter dans un fichier Python en allant dans <i>File / Export Level 2 Script</i>. Le fichier Python pourra être utilisé comme un fichier Python normal. Le code python offre toutes les fonctionnalités que l'on peut trouver dans le client mininet.

* <i>sudo python file.py</i>

## Utiliser Mininet en Python

Mininet peut être utilisé en Python pour créer et faire les tests sur les topologies. C'est ce que miniedit fait lorsque l'on exporte le script. 

### Créer des topologies

Soit le code python suivant : 

```python

#!/usr/bin/python                                                                            
                                                                                             
from mininet.topo import Topo
from mininet.net import Mininet
from mininet.util import dumpNodeConnections
from mininet.log import setLogLevel

class SingleSwitchTopo(Topo):
    "Single switch connected to n hosts."
    def build(self, n=2):
        switch = self.addSwitch('s1')
        # Python's range(N) generates 0..N-1
        for h in range(n):
            host = self.addHost('h%s' % (h + 1))
            self.addLink(host, switch)

def simpleTest():
    "Create and test a simple network"
    topo = SingleSwitchTopo(n=4)
    net = Mininet(topo)
    net.start()
    print "Dumping host connections"
    dumpNodeConnections(net.hosts)
    print "Testing network connectivity"
    net.pingAll()
    net.stop()

if __name__ == '__main__':
    # Tell mininet to print useful information
    setLogLevel('info')
    simpleTest()

```
Plusieurs informations importantes sont à retenir de ce code. 

* `build()` : La méthode à surcharger pour créer des topologies personnalisées.
* `addSwitch()` : Ajoute un switch et retourne son nom.
* `addHost()` : Ajoute un hôte et retourne son nom.
* `addLink()` : Ajoute un lien bidirectionnel entre les composants passés en argument.
* `start()` : Démarre le réseau.
* `pingAll()`: Effectue l'équivalent de pingall sur le réseau.
* `stop()` : Arrête le réseau.
* `net.hosts`: Liste de tous les hôtes du réseau.
* `dumpNodeConnections()` : Affiche les connexions entre les noeuds. 

### Performances d'un réseau

Par défaut, Mininet utilise un réseau qui est "parfait". Il est possible d'ajouter des indicateurs de performance sur les liens et par exemple sur les hôtes. 

