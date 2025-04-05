Ce document est destiné à vous aider pour configurer le premier cas, une fois ce document lu vous serez normalement capable de configurer n'importe quel autre cas.

## Configuration des Hosts
Vous allez commencer par ajouter 6 ordinateurs (2 ordinateurs par switch) afin de pouvoir tester la connectivité entre switch plus tard)
après avoir cliqué sur le premier host vous tomberez normalement sur la page de configuration du host (cliquez sur config-->interface utilisé) 
ensuite choisissez une adresse ip et un masque (pour des raisons d'efficacité j'ai juste choisi l'adresse 192.168.0.6 et un masque /24 donc 255.255.255.0)

![Exercice 1](hostconfig.png)

Vous faites la meme chose pour les autres hosts.

## Les switchs

Pour les switchs nous allons sur database vlan et nous créeons nos vlans (en leur donnant un nom et un chiffre, dans mon cas vlanA(2) et VlanB(3) le numéro 1 est reservé au Vlan administratif)
![image](https://github.com/user-attachments/assets/f4d60840-ed89-4d1b-bab6-82e4f029349c)

Ensuite il nous suffit de sélectionner l'un des ports relié à l'un des hosts et de définir dans quel vlan on voudrait mettre ce port et sur quel mode (trunk ou access)
![image](https://github.com/user-attachments/assets/0e9dddbc-f47b-4d36-b9da-808868d6e966)

C'est tout pour la partie configuration, vous n'avez pas besoin d'autre choses à part répeter la procédure pour tous les hosts et switchs et changer les modes et liaisons en fonction des cas donnés dans les exercices.
