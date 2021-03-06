# Tutoriel mininet/openvswitch

Mininet est un logiciel open source qui est utilisé pour simuler des <i>Software defined network (SDN)</i>. Ce logiciel utilise un système de contrôleurs, de switchs et d'hôtes.

Mininet supporte le protocole OpenFlow. 

## Commandes utiles

Pour éxecuter mininet, il faut utiliser la commande suivante : 
* `sudo mn`

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
* <i>mininet> xterm h1</i> - Ouvre une fenêtre en ligne de commande sur l'hôte h1.

## Miniedit

Mininet fournit un outil qui permet de créer sa propre topologie de manière graphique du nom de miniedit. 

Cette interface s'exécute avec la commande :
* `sudo mininet/examples/miniedit.py`

Par défaut, la topologie donnée par mininet est constituée d'un contrôleur, un switch et deux hôtes. Sur miniedit, cette topologie ressemble à la figure suivante : 

<p align="center">
    <img src="/img/default_topo.PNG" alt="Default topology"/>
</p>

Les différents outils sur la gauche de miniedit permettent de modifier la topologie affichée.

<p align="center">
    <img src="/img/tools_miniedit.PNG" alt="Tools Miniedit" />
</p>

Dans l'ordre d'affichage de la figure précédent :
1. Permet de déplacer les objets dans la topologie.
1. Permet d'ajouter un hôte à chaque clic.
1. Permet d'ajouter un switch à chaque clic.
1. Permet d'ajouter un switch legacy à chaque clic.
1. Permet d'ajouter un routeur legacy à chaque clic.
1. Permet de faire des liens entre les objets de la topologie. Le clic doit être maintenu.
1. Permet d'ajouter un contrôleur à chaque clic.

Lorsque la topologie a été créée avec miniedit, il est possible de l'exporter dans un fichier Python en allant dans <i>File / Export Level 2 Script</i>. Le fichier Python pourra être utilisé comme un fichier Python normal. Le code python offre toutes les fonctionnalités que l'on peut trouver dans le client mininet.

* `sudo python file.py`

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

Reprenons certaines méthodes Python précédentes :

* `addHost(name, cpu=f)` : Ajoute un hôte et permet d'ajouter une contrainte d'utilisation du processeur. Contenue entre 0 et 1, elle correspond au pourcentage f d'utilisation allouée du processeur.
* `addLink(node1, node2, bw=10, delay='5ms', max_queue_size=1000, loss=10, use_htb=True)` : Ajoute un lien bidirectionnel avec une bande passante de 10 Mbits, un délai de 5ms, une limite de 1000 paquets ainsi qu'une perte de 10% de paquets. 

Ensuite, il est possible de tester la connectivité entre deux hôtes grâce à la méthode `net.iperf()`. Cette méthode prend un tuple de noeuds en argument. La méthode `net.get(name)` permet de récupérer un noeud grâce à son nom. 

Le code suivant créé un réseau et effectue un simple test de performance.

```python

#!/usr/bin/python

from mininet.topo import Topo
from mininet.net import Mininet
from mininet.node import CPULimitedHost
from mininet.link import TCLink
from mininet.util import dumpNodeConnections
from mininet.log import setLogLevel

class SingleSwitchTopo( Topo ):
    "Single switch connected to n hosts."
    def build( self, n=2 ):
	switch = self.addSwitch( 's1' )
	for h in range(n):
	    # Each host gets 50%/n of system CPU
	    host = self.addHost( 'h%s' % (h + 1),
		                 cpu=.5/n )
	    # 10 Mbps, 5ms delay, 2% loss, 1000 packet queue
	    self.addLink( host, switch, bw=10, delay='5ms', loss=2,
                          max_queue_size=1000, use_htb=True )

def perfTest():
    "Create network and run simple performance test"
    topo = SingleSwitchTopo( n=4 )
    net = Mininet( topo=topo,
	           host=CPULimitedHost, link=TCLink )
    net.start()
    print "Dumping host connections"
    dumpNodeConnections( net.hosts )
    print "Testing network connectivity"
    net.pingAll()
    print "Testing bandwidth between h1 and h4"
    h1, h4 = net.get( 'h1', 'h4' )
    net.iperf( (h1, h4) )
    net.stop()

if __name__ == '__main__':
    setLogLevel( 'info' )
    perfTest()

```

### Exécuter des programmes sur un hôte

Chaque hôte mininet est un shell bash attaché à une ou plusieurs interfaces réseau. Le plus simple est d'exécuter la commande grâce à la méthode cmd.
Le code suivant montre comment effectuer la commande `ifconfig` sur un hôte et d'afficher son résultat : 

```python
h1 = net.get('h1')	
result = h1.cmd('ifconfig')
print result
```

Il est également possible de démarrer une commande au premier plan avec `sendCmd()` et d'attendre qu'elle se termine plus tard dans le code en utilisant `waitOutput()` :

```python
for h in hosts:
    h.sendCmd('sleep 20')
…
results = {}
for h in hosts:
    results[h.name] = h.waitOutput()
```

### Système de fichier partagé

Par défaut, les hôtes mininet partage le système de fichier racine de la machine utilisée. En revanche, il est possible de déterminer un dossier privé par hôte à la création de l'hôte. 

`h = Host( 'h1', privateDirs=[ '/some/directory' ] )`

### Configurer un hôte 

Les hôtes possèdent des méthodes de configuration permettant de configurer leur adresse IP, leur adresse MAC...

* `host.IP()` : Retourne l'adresse IP de l'hôte.
* `host.MAC()` : Retourne l'adresse MAC de l'hôte.
* `host.setARP()` : Ajoute une entrée ARP statique au cache ARP de l'hôte.
* `host.setIP()` : Assigne une adresse IP spécifique à un hôte.
* `host.setMAC()` : Assigne une adresse MAC spécifique à un hôte.

### Command line interface (CLI)

Lors de l'exécution de mininet via la commande `sudo mn`, une interface ligne de commande est offerte à l'utilisateur. Il est possible d'avoir cette interface sur un script Python. Il suffit d'ajouter la ligne `CLI(net)`. Le code ressemblerait à ceci :

```python
from mininet.topo import SingleSwitchTopo
from mininet.net import Mininet
from mininet.cli import CLI

net = Mininet(SingleSwitchTopo(2))
net.start()
CLI(net)
net.stop()
```

Ajouter cette commande peut être utile pour débugger en temps réel un réseau.


## Utiliser POX

POX est installé de base avec la version complète de mininet. POX permet d'ajouter un contrôleur externe au réseau qui gère tous les switchs.

Pour effectuer les prochaines étapes, le code suivant sera utilisé : 

```python
from mininet.topo import Topo
from mininet.net import Mininet
from mininet.node import OVSSwitch, Controller, RemoteController

class RectTopo( Topo ):
    "Rectangular topology example."

    def __init__( self ):
        "Create custom topo."

        # Initialize topology
        Topo.__init__( self )

        # Add hosts and switchesovs-ofctl
        h1 = self.addHost( 'h1' )
        h2 = self.addHost( 'h2' )
        h3 = self.addHost( 'h3' )
        h4 = self.addHost( 'h4' )
        s1 = self.addSwitch( 's1' )
        s2 = self.addSwitch( 's2' )
        s3 = self.addSwitch( 's3' )
        s4 = self.addSwitch( 's4' )

        # Add links
        self.addLink( h1, s1 )
        self.addLink( h2, s2 )
        self.addLink( h3, s3 )
        self.addLink( h4, s4 )

        self.addLink( s1, s2 )
        self.addLink( s2, s3 )
        self.addLink( s3, s4 )
    #	self.addLink( s4, s1 )

topos = { 'recttopo': ( lambda: RectTopo() ) }
```

A noter la dernière ligne, qui permet de lancer la topologie via ligne de commande. 

Ce code python démarre un simple réseau en rectangle, qui ressemble à ceci : 

<p align="center">
    <img src="/img/topo_rectangle.PNG" alt="Rectangle topology"/>
</p>

La liaison s4, s1 est commentée dans le code, ce qui est normal.

### Forwarding hub

POX fournit des exemples de code pour aider à se familiariser avec cet outil. Tout d'abord, nous allons commencer avec `pox/pox/forwarding/hub.py`. Ce code exemple transforme les switchs de la topologie en hubs.

Pour lancer ce code exemple, rentrer la commande suivante en ligne de commande : 

`pox/pox.py --verbose forwarding.hub`

Une fois lancée, dans un autre terminal, il faut lancer la topologie Mininet en rectangle : 

`sudo mn --custom rectangle.py --topo recttopo --controller remote --switch ovsk --mac`

L'option `--mac` permet de faire en sorte que les adresses MAC soient des integers facile à lire, par exemple 00:00:00:00:00:01.

POX indique qu'il a pris le contrôle des switchs. 

<p align="center">
    <img src="/img/pox_connected.PNG" alt="Pox connected"/>
</p>


Sur le terminal Mininet, vérifier que le réseau fonctionne :

* <i>mininet> h1 ping h4</i>

Tout fonctionne correctement. Maintenant, il faut mettre la liaision entre les switchs 1 et 4. Quitter et décommenter la ligne dans `rectangle.py`.

Relancer Mininet (POX est toujours actif). Et renouveller le ping entre h1 et h4.

La commande ping donne un résultat qui ressemble à ceci :

`6 packets transmitted, 6 received, +134913 duplicates, 0% packet loss, time 5016ms`

Cela prouve que les switchs ont bien été transformés en hubs.

### Analyse du code

Le code du fichier `hub.py` contient une fonction du nom de `launch`. Un composant POX doit toujours posséder une fonction `launch`. Cette fonction est appelée par POX et est utilisée afin d'initialiser le module. Elle permet également de passer directement des arguments via ligne de commande.

Par exemple, pour la commande utilisée pour lancer le hub POX, il aurait pu être précisé que le hub soit reactive.

`pox/pox.py --verbose forwarding.hub --reactive=True`

La première ligne du code qui nous intéresse est celle-ci :

```python
core.openflow.addListenerByName("ConnectionUp", _handle_ConnectionUp)
```

`addListener()` est une méthode qui permet la gestion d'évenement dans POX. Ici, lorsque l'évenement "ConnectionUp" arrive, la méthode `_handle_ConnectionUp()` est appelée.

Dans la méthode `_handle_ConnectionUp()`, on retrouve le code suivant : 

```python
msg = of.ofp_flow_mod()
msg.actions.append(of.ofp_action_output(port = of.OFPP_FLOOD))
event.connection.send(msg)
log.info("Hubifying %s", dpidToStr(event.dpid))
```

La première ligne indique que l'on veut créer un nouveau message. La classe `ofp_flow_mod()` est utilisée et c'est elle qui permet de créer ces messages. 

Ensuite, il est indiqué que l'on veut effectuer une action. Envoyer le message. Plus précisement ce message sera floodé dans le réseau. 

Le message est finalement envoyé.


## Learning switch

Un switch n'est pas censé fonctionner comme un hub. Le fichier `pox/misc/of_tutorial.py` a été créé afin d'apprendre les bases d'OpenFlow. Il est configuré de base pour que les switchs fonctionnent comme des hub. Le code utilise la fonction `act_like_hub()` mais possède la fonction `act_like_switch()`. Le but est d'utiliser cette dernière et de faire en sorte que le switch fonctionne comme un switch.

La fonction `act_like_switch` possède déjà des élements de code pour pouvoir assurer le fonctionnement du switch.

La première étape est de vérifier que le code du hub fonctionne correctement. 

Pour exécuter le fichier, il faut utiliser la commande suivante : 

`pox/pox.py --verbose misc.of_tutorial` puis utiliser la topologie `rectangle.py` utilisée auparavant.

Commencer par effectuer la partie sans avoir de flow. Il suffit d'apprendre le port selon l'adresse MAC source puis, si l'adresse se trouve dans la table, le renvoyer.

Pour tester si le réseau est fonctionnel, la commande `pingall` ainsi que la commande `iperf` peuvent être utiles. 

Une fois que la partie d'apprentissage fonctionne, il est intéressant de faire un flow. Le flow permet d'augmenter fortement la vitesse d'envoi de paquet. Les deux images suivantes montrent cette différence. 

<p align="center">
    <img src="/img/iperf_noflow.PNG" alt="Iperf noflow"/>
    <img src="/img/iperf_flow.PNG" alt="Iperf flow"/>
</p>



La correction de cet exercice se trouve dans le dossier `codes` de ce repo.




## References

https://github.com/mininet/mininet/wiki/Introduction-to-Mininet

https://noxrepo.github.io/pox-doc/html

https://github.com/mininet/openflow-tutorial/wiki/Create-a-Learning-Switch

http://pld.cs.luc.edu/courses/netmgmt/sum17/notes/mininet_and_pox.html

https://ryu.readthedocs.io/en/latest/writing_ryu_app.html

