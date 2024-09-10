# Prise de notes - Test d'intrusion

Cette proposition de méthodologie par la pratique d'un test de pénétration (ou _pentest_ en anglais) est largement décrite par l'[OWASP - The Open Web Application Security Project](https://owasp.org).

## Phases de reconnaissance

Avant de décortiquer l'application ou le serveur web en lui-même, le but est de se renseigner sur la cible, son écosystème, les services exposés, vérifier son chiffrement, etc.

### Reconnaissance passive

En ligne, le site [shodan.io](https://www.shodan.io) propose un moteur de recherche d'équipements et de services vulnérables. Pour chaque adresse IP le site proposera aussi un assortiment de failles potentielles. Pour compléter cette première collecte d'informations, passer la cible au crible de [WHOIS](https://whois.com/whois/) sera également nécessaire.

Ensuite, il est possible d'utiliser des techniques de Google Dork en se basant sur le site [DB Exploit](https://exploit-db.com/) qui pourvoit la [Base de données](https://www.exploit-db.com/google-hacking-database) qui fait office de référence en la matière.

Possible d'utiliser l'outil de récolte de données [theHarvester](https://github.com/laramies/theHarvester) ou encore l'outil proposé à cet effet par l'OWASP dénommé [Amass](https://github.com/OWASP/Amass) dont il faudra éviter d'utiliser l'option *-active* pour ne pas laisser de traces de son passage sur le serveur cible.

Quelques outils pour cette phase de recueil d'informations afin - notamment - de réaliser des énumérations de DNS, de réseaux et d'e-mails :
- whois
- host
- nslookup
- dig
- theHarvester
- traceroute
- netdiscover
- arpscan
- fping --quiet --alive --generate <adresse IP du réseau>

Des _frameworks_ de collecte de données peuvent aussi être utiles pour automatiser une partie des tâches :
- [OSRFframework](https://www.github.com/i3visio/osrframework)
- [Maltego](https://www.maltego.com)
- [spiderfoot](https://www.spiderfoot.net/)
- [Recon-NG](https://www.github.com/lanmaster53/recon-ng)

Arpenter les réseaux sociaux est également enrichissant à cet effet.

La structure d'un site est intéressante à explorer dans la mesure où les fichiers qui le compose sont souvent génériques.
L'aide de quelques commandes Linux est appréciable :
- les liens vers les réseaux sociaux ou les noms et adresses mail qui peuvent être affichées sont à noter
- www.lesite.com/robots.txt : le fichier robots.txt sert a orienter les crawlers des moteurs de recherche
- www.lesite.com/sitemap.xml ou /sitemaps.xml : ce fichier xml décrit l'organisation du site au profit des moteurs de recherche
- la commande `host` permettra de déterminer l'adresse IP du site cible (c'est une méthode parmi d'autres)
- certains add-on de navigateurs sont utiles lors de cette phase : built-in, wappalyzer.
- la commande `whatweb` fournira un descriptif technique du site
- enfin, la copie complète du site web peut être réalisée gràce à HTTrack Website Copier

Une énumération par WHOIS permettra d'agrémenter les données collectées :
- par l'utilisation de la commande `whois` sur Linux
- obtenir le nom de serveur d'un domaine

Le footprinting d'un site peut-être réalisé :
- gràce à l'outil [Netcraft](https://www.netcraft.com) qui permet de connaître les technologies utilisées (SSL/TLS...) et d'autres éléments comme l'appartenance à certains blocs d'adresses IP, les trackers en place (Google Analytics...) ou certaines vulnérabilités
- et complété par DNSRECON qui est un programme en python disponible sur Kali pour obtenir davantage d'informations sur l'environnement immédiat de la cible
- tout comme le site [DNSdumpster](https://www.dnsdumpster.com) qui permet des graphiques lisibles et compréhensibles de l'organisation de la cible

```bash
whois hackerslpoit.org
```

```bash
dnsrecon -d hackersploit.org
```

Un WAF (Web Application Firewall) peut être décelé avec l'outil *wafw00f*. L'objectif serait de déterminer si un site web ou une application est protégée par un service du type CloudFlare et n'affiche pas directement sa propre adresse IP de manière évidente.

```bash
wafw00f -a hackersploit.org
```

L'énumération de sous-domaine peut elle être réalisée par l'outil *sublist3r*. Cet outil n'est pas directement installé sur Kali Linux bien qu'il soit disponible sur le repository de Kali. Il nécessite donc d'être installé. Il est intéressant de savoir que *sublist3r* dispose aussi d'une capacité de bruteforce qui n'est pas recommandée dans la phase de reconnaisse passive.

```bash
sudo apt-get install sublist3r
sublist3r -d ine.com
```

Comme évoqué plus haut, le Google Dorking (ou Google Hacking ou Google Dorks) permet d'obtenir des résultats ciblé en utilisant des mots clés du moteur de recherche.
Ceux-ci peuvent être :
- site: (pour limiter la recherche à un nom de domaine et ses sous-domaines avec par exemple *.ine.com*)
- inurl: (limiter la recherher au contenu de l'URL comme par exemple *admin* ou *auth_user_file.txt* ou *password.txt*)
- intitle: (déterminer un terme contenu dans le titre de la page comme par exemple *admin* ou la faille très réputée *index of*)
- filetype: (limiter les résultats à un type de fichier comme par exemple *pdf* ou *docx* ou *zip*)
- cache: (pour retrouver l'ancienne version d'un site ou une archive tout comme le ferait le site [WayBackMachine](https://archive.org/))

En détail, l'outil [theHarvester](https://github.com/laramies/theHarvester) permet de collecter des adresses mail liées à un domaine internet.

```bash
theHarvester -d hackersploit.org -b google, linkedin, yahoo, dnsdumpster
```
Enfin, pour compléter le tout, l'utilisation d'une base de données regroupant des _leaks_ permettra éventuellement de mettre la main sur des listes de mots de passe ou d'autres données concernant la cible.
Plusieurs sont disponibles :
- [haveIbeenPwned](https://haveibeenpwned.com/)
- [Firefox Monitor](http://monitor.firefox.com/)
- [Dehashed](https://www.dehashed.com/)

### Reconnaissance active

L'outil [Amass](https://github.com/OWASP/Amass) pourra toujours être utilisé pour renforcer la connaissance de la cible en activant l'option `-active` dans la formulation des requêtes.

Réaliser un scan de ports avec les outils dont le plus connu est [Nmap](https://nmap.org/).

**Attention** car l'utilisation de cet outil peut ne pas être très discrète en fonction des options utilisées. Un scan "silencieux" pourra être réalisé grâce aux options `-sS` ; c'est le type de scan réalisé par défaut.

L'attribution des ports est réalisée par l'[IANA](https://www.iana.org/assignments/service-names-port-numbers/service-names-port-numbers.xhtml) donc le résultat attendu est supposé conforme aux recommandations de cette autorité de l'Internet.

La découverte du système d'exploitation de la cible peut être réalisée grâce à la commande suivante : 

```bash
nmap -sS -O scanme.namp.org
```

Pour obtenir davantage d'informations, il est possible de cherche à définir les versions des services qui tournent sur le terminal cible :

```bash
nmap -sV scanme.nmap.org
```

Pour forcer la découverte de type ICMP :

```bash
nmap -PE -sn -n <adresse IP du réseau>
```

Quelques outils supplémentaires pour cette phase de recueil d'informations visant à énumérer différent protocoles (SMB, NFS, SMTP, SNMP, LDAP, Active Directory) ainsi que des bases de données et des serveurs web :
- Nmap (sans le scan de ports avec l'option -sn en premier lieu)
- netdiscover
- enum4linux
- smbmap
- CrackmapExec
- SQLmap
- [Nikto](http://www.cirt.net/Nikto2)
- dirb
- wfuzz
- gobuster
- wpscan

```bash
nmap -sn -PS // permet de réaliser la découverte des hôtes actifs sans scanner les ports
nmap -Pn -n -sV --script default -pPORT1, PORT2, PORT3 <adresse IP de la cible>
nmap -Pn -sU -F -sV -O -sC //pour un scan discret sur les ports les plus courants avec détection de la version des services et du système d'exploitation utilisés ainsi qu'une application des scripts Nmap par défaut sur les ports ouverts
nmap -T[0-5] -oN doc.txt //pour réguler la vitesse de scan et produire un fichier avec les résultats de la commande (doc.xml)
nmap -Pn -sS -F //pour un stealth scan sur les ports les plus courants
```

Pour utiliser le scripting par défaut de Nmap (NSE : Nmap Scripting Engine) et approfondir la collecte de données concernant la détection de système d'exploitation, la liste des scripts Nmap est disponible ici : /usr/share/nmap/scripts/

```bash
ls -al /usr/share/nmap/scripts/
ls -al /usr/share/nmap/scripts/ | grep -e "http"
nmap -h
nmap -A //AGRESSIVE SCAN peu discret qui est un raccourci pour scanner les services et système d'exploitation
```

Le spoofing d'IP est rendu possible comme suit :
```bash
nmap -Pn -sS -sV -p445,3389 -f --data-lenght 200 -D 192.168.1.1 192.168.1.200 //spoofing de l'adresse 192.168.1.1 (gateway) avec des paquets fragmentés de taille 200
```

Énumération avec Nikto :

```bash
nikto -h <adresse IP de la cible>
```

La commande ci-après qui utilise Gobuster permet d'énumérer un serveur web pour y déceler des répertoires en se basant sur un dictionnaire tout en testant certaines extensions pour recherche d'éventuels fichiers :

```bash
gobuster dir --url <adresse IP de la cible> --wordlist /usr/share/wordlists/directory-list-2.3-medium.txt -x html,php,txt
```

Le _Domain Name Server_ utilise des NS ou _nameservers_ qui sont similaires à des répertoires téléphoniques listant la correspondance entre adresses IP et noms de domaines. Cloudflare (1.1.1.1) et Google (8.8.8.8) possèdent par exemple leurs propres DNS.

Le _DNS Zone Transfer_ est notamment décrit par ici sur [ZoneTransfer](https://digi.ninja/projects/zonetransferme.php). 

L'énumération active de DNS peut être réalisée gràce à la commande *dnsenum* puis appronfondie avec les outils *dig* et/ou *fierce* présents dans Kali Linux.

```bash
dnsenum zonetransfer.me
```

## Identifier les services vulnérables

Une fois que le profil de la cible est un peu plus affiné, il faudra partir en quête des vulnérabilités qui peuvent éventuellement exister sur un service donné dans une version donnée.

A cet effet, outre une recherche Google, il faut consulter les CVE (Common Vulnerability Exposure) qui impactent possiblement les services identifiés :
- [MITRE](https://cve.mitre.org/index.html)
- [CVE Program](https://www.cve.org/)
- [CVE Details](https://www.cvedetails.com/)
- [CERT ANSSI](https://www.cert.ssi.gouv.fr/alerte/)

En guise de ressources, quelques outils :
- [Nessus](https://fr.tenable.com/products/nessus/nessus-essentials)
- OpenVAS
- nikto et Nmap (déjà vus plus haut) s'accompagnent aussi de capacités de détection de vulnérabilités
- WPscan

Un exemple de recherche de vulnérabilités :

```bash
wpscan --url http://<adresse IP ou URL de la cible>/wp
```

Complété par la commande suivante pour lister les utilisateurs enregistrés d'une instance WordPress :

```bash
wpscan --url http://<adresse IP ou URL de la cible>/wp --enumerate u
```

### Vérification de la qualité du chiffrement

Il faut pouvoir jauger :
1. la qualité du certificat utilisé, qui pourrait en faciliter la falsification par un attaquant si insuffisante (la taille de clé du certificat, qui ne doit pas être trop faible i.e. inférieure à  2 048 bits et l'algorithme de hachage de type SHA-256 avec RSA ou supérieur) ;
2. les protocoles proposés par le serveur, qui peuvent contenir des défauts d’implémentation ;
3. les suites de chiffrement proposées, parfois faibles ;
4. la librairie utilisée pour le chiffrement, qui peut comporter des vulnérabilités.

Quelques outils existent pour automatiser ces tâches :
+ [SSLlabs](https://www.ssllabs.com/ssltest/)
+ [SSLscan](https://github.com/rbsec/sslscan)
+ [testssl.sh](https://github.com/drwetter/testssl.sh)

## Exploitation

Après la reconnaissance passive puis active et faisant suite aux scans de réseaux et des vulnérabilités, la phase d'exploitation a pour but de compromettre la cible en s'engouffrant dans une des failles décelées. Il s'agira de mettre en oeuvre un *exploit* combiné à une *payload* pour s'immiscer dans le terminal cible.

Pour ce faire, il faudra sélectionner l'*exploit* idoine dans l'[Exploit Database](https://www.exploit-db.com/) et certainement l'adapter (selon le langage de programmation utilisé) pour y charger la *payload* voulue en fonction de l'effet recherché sur cible (comme par exemple la génération d'un _reverse shell_).

Pour faciliter cette recherche, l'outil suivant en ligne de commande *searchploit* pourra servir :

```bash
searchsploit -www wordpress mail data
```

La commande précédente va rechercher les failles connues du *plugin* WordPress Mail Masta et donnera l'URL correspondant sur *Exploit Database* si elles existent avec l'option `-www`.

## Post-exploitation

Une fois l'accès acquis sur une nouvelle machine par le biais d'un profil utilisateur, l'idée sera alors de chercher à acquérir de nouveaux privilèges sur la cible ou de nouveaux accès à travers celle-ci. Une nouvelle phase d'énumération devra être conduite pour obtenir une élévation de privilèges, le maintien de l'accès ou pour réaliser un mouvement latéral (saut vers un autre utilisateur du terminal) voire un *pivoting* (saut vers un autre réseau jusque-là non atteignable).

Les scripts de [PEASS - Privilege Escalation Awesome Scripts SUITE](https://www.github.com/carlospolop/privilege-escalation-awesome-scripts-suite) peuvent aider a atteindre ce but.

De plus, quelques outils supplémentaires aideront à la post-exploitation :
- [GTFOBins](https://gtfobins.github.io/)
- [Hacking Articles](https://www.hackingparticles.in/ctf-challenges-walkthrough)
