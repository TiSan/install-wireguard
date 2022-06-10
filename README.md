# Installation von Wireguard

**Schritt 1:** Ubuntu Server installieren (auf VM, wird hier nicht näher erläutert)

**Schritt 2:** Installieren vom Wireguard-Paket mit apt

    apt update && apt install wireguard -y
    sudo su && cd /etc/wireguard

**Schritt 3:** Generieren von Public-Private-Keypairs

    wg genkey | tee privatekey | wg pubkey > publickey
    

Dieser Schritt muss einmal für den Server gemacht werden, sowie für jeden User, der sich verbinden will. In dem oberen Befehl können *"privatekey"* und 
*"publickey"* durch andere Filenames ersetzt werden, damit mehrere Keypairs parallel im gleichen Ordner liegen können. 
Zusätzlich muss der Befehl `wg genkey > presharedkey_client` für jeden User ausgeführt werden. **Dieser preshared Key muss sowohl auf dem Server als auch auf dem 
Client für ein User gleich sein.**

**Schritt 4:** Anlegen einer Wireguard-Config

    nano wg0.conf

Beispielkonfiguration des Servers: 

    [Interface]
    PrivateKey = <PRIVATE KEY DES SERVERS>
    ListenPort = 51821
    Address = 10.1.6.1/32
    PostUp=iptables -t nat -A POSTROUTING -s 10.1.6.0/24 -o <NETZWERKKARTEN-NAME> -j MASQUERADE
    PostDown=iptables -t nat -D POSTROUTING -s 10.1.6.0/24 -o <NETZWERKKARTEN-NAME> -j MASQUERADE
    
    [Peer]
    # User 1
    PublicKey = <PUBLIC KEY DES CLIENT 1>
    PresharedKey = <PRESHARED KEY FÜR CLIENT 1>
    AllowedIPs = 10.1.6.2/24
    
    [Peer]
    # User 2
    PublicKey = <PUBLIC KEY DES CLIENT 2>
    PresharedKey = <PRESHARED KEY FÜR CLIENT 2>
    AllowedIPs = 10.1.6.3/24

**Schritt 5:** IP-Forwarding in Ubuntu aktivieren
Damit die Weiterleitung der VPN-Netzwerkpakete ins Netzwerk auch klappt, muss das IPv4-Forwarding aktiviert werden.

    nano /etc/sysctl.conf
Dort `net.ipv4.ip_forward = 1` hinzufügen (entsprechendes `#` vor der Zeile entfernen).
Anschließend mit `sysctl -p` die Datei neu einlesen. Fertig.

**Schritt 6:** Wireguard Autostart

    sudo systemctl enable wg-quick@wg0.service
    sudo systemctl daemon-reload

**Schritt 7:** Wireguard starten

    wg-quick up wg0
Falls Wireguard beendet werden soll, kann dies analog mit dem Befehl `wg-quick down wg0` geschehen.

**Schritt 8:** Config für die Clients anlegen
Die Clients benötigen ebenfalls eine Wireguard Config, die sie dann einfach importieren können. Diese Datei legen wir mit einem einfachen Texteditor auf einem 
System deiner Wahl an und nennen sie entsprechend mit `*.conf`.

Beispielkonfiguration:

    [Interface]
    PrivateKey = <PRIVATE KEY DES CLIENT 1>
    Address = 10.1.6.2/32
    
    [Peer]
    PublicKey = <PUBLIC KEY DES SERVERS>
    PresharedKey = <PRESHARED KEY DES CLIENT 1>
    AllowedIPs = 10.1.6.1/32, <NETZWERK AUF DAS ZUGEGRIFFEN WERDEN SOLL, z.B. 192.168.2.1>24
    Endpoint = unifi.evk-o.de:51821

HINWEIS: Soll die Konfiguration auf einem Android-Telefon verwendet werden, müssen die Netzwerkdefinitionen in `AllowedIPs` mit einer 0 enden, da Android gültige 
Netzwerkadressen erfordert. Ansonsten wird eine Verbindung mit `bad address` beendet und eine Verbindung ist unmöglich.

**Schritt 9:** Verbinden
Wireguard gibts für verschiedenste Plattformen. Dort die eben erstellte Config importieren und verbinden. Bitte dran denken, den entsprechenden Port im Router 
freizugeben, den man in der Serverconfig gewählt hat (UDP). 

# FERTIG! Viel Spaß beim VPNen!

