---
title: Promox mit einzel-IP mit PfSense router VM
author: "Jakob Langisch"
date: 2023-08-08 11:33:00 +0800
categories: [Linux, Proxmox]
tags: [Proxmox, PfSense, Linux]
pin: false
math: false
mermaid: false
---


## Einführung

Proxmox VE ist eine Open-Source Virtualisierungsplattform mit Unterstützung für KVM. Die Möglichkeiten der Netzwerkkonfiguration sind vielseitig. Die Grundlagen werden [hier](https://community.hetzner.com/tutorials/install-and-configure-proxmox_ve/de) vermittelt.
Mit diesem Tutorial sind sie in der Lage, einen Proxmox Server so zu konfigurieren, dass Sie mittels einer Router-VM auf dem Proxmox Server den Netzwerktraffic innerhalb eines [RFC1918 Private Network](https://www.rfc-editor.org/rfc/rfc1918.txt) (10.0.0.0/8, 192.168.1.0/24, etc.) aller ihrer VMS managen können. Wir werden hierfür PfSense nutzen, grundsätzlich ist aber jede konfiguration auf der VM möglich (anderes Firewall-OS, Debain + IPtables, etc.)

**Vorraussetzungen**

Wir versuchen, hier auf die vielseitigen konfigurationsmöglichkeiten einzugehen.
Durch verschiedene Hardwarekonfigurationen kann es aber nötig sein einzelne Punkte der Konfiguration anzupassen. (Hauptsächlich Speicherkonfiguration Proxmox, Details des Netzwerkadapters)

* Wie Sie Proxmox installieren, ist für die Netzwerkkonfiguration grundsätzlich egal.
    * Sie können (wie unten beschrieben) das Installimage von Hetzner wählen, könnten Proxmox aber auch manuell installieren, wie [hier](https://community.hetzner.com/tutorials/install-and-configure-proxmox_ve/de) beschrieben.

## Schritt 1 - `<Installation von Promox>`

Zunächst Booten wir den Server in das Hetzener eigene [Rescue-System](https://docs.hetzner.com/de/robot/dedicated-server/troubleshooting/hetzner-rescue-system/) und verbinden uns per SSH.

Mit der Pfeiltaste `(arrow key up)` auf ihrer Tastatur können Sie eine reihe ein praktischen commands abfragen, die Hetzner vorbereitet hat.
Ich möchte an dieser Stelle auch explizit den Speedtest loben.

## Step 2 - `<Installimage>`

Wir rufen nun [installimage](https://docs.hetzner.com/de/robot/dedicated-server/operating-systems/installimage/) mit dem Befehl `installimage` auf.

Unter dem Punkt "other" finden wir eine auswahl der verfügbaren Proxmoxversionen.

Achtung: Hetzner supported die *Konfiguration* der Proxmox images nur durch die [Tutorials](https://community.hetzner.com/tutorials).
Normale Tätigkeiten wie "helping hand" etc. werden selbstverständlich ausgeführt.

![Installimage Other](/assets/posts/proxmox-hetzner/installimage_other.png)

Wir wählen hier "Debian BullsEye"

### Schritt 2.1 - `<installimage/install.conf>`

Im Standard wird ein SWRAID (Softwareraid) erzeugt.

In diesem Tutorial werden wir ein SWRAID 0 konfiurieren und den gesamten Speicher als Lokale Platte in Proxmox mounten.
Basierend auf ihren Zielen kann dies nützlich sein, muss es aber nicht.

Die genaue Konfiguration können Sie selbstverständlich im Texteditor der Install.conf an ihre bedürfnisse anpassen, oder Sie folgen einfach dieser Anleitung.

In der Config sind bereits alle Punkte die Sie brauchen vorhanden, Sie brauchen nur zu scrollen und die Werte anpassen.

```shell
## activate software RAID?  <0 | 1>
SWRAID 1
```
setzen das SWRAIDLEVEL:
```shell
SWRAIDLEVEL 0
```
Nun müssen wir einen Hostname setzen:
```shell
HOSTNAME ax61.hetzner-local.dc
```
Der Hostaname muss immmer ein FQDN sein.

Nun scrollen wir nach unten, bis wir keine # zeichen mehr vor den Zeilen sehen.
Hier begrüßt uns die Partitionierung des Systems.

Eine sehr einfache Variante:

```shell
PART swap swap 16G 
PART /boot ext4 2G
PART / ext4 all
```

Mit F2 + F10 bzw click auf die Tasten im Editor speichern und schließen.
Nun bestätigen wir noch mehrere nachrichten, die aus dem Zusammenhang ersichtlich sein sollten

![Installimage Starting](/assets/posts/proxmox-hetzner/installimage_starting.png)
![Installimage Starting](/assets/posts/proxmox-hetzner/installimage_finished.png)

## Schritt 3 - `<Einrichten von Proxmox>`

Nach dem Reboot können wir die WebUI von Proxmox unter `https://HOSTNAME:8006` aufrufen, oder uns per SSH verbinden.

## Schritt 4 - `<Einrichten einer Router-VM>`

Wie bereits erwähnt, sind die möglichkeiten hier vielseitig. Wir setzen der einfacheit halber aber eine PfSense VM ein. (Netzwerker lachen und weinen bei diesem Satz)

Um die PfSense einzurichten, brauchen wir der einfachheit halber eine VM *mit GUI und Browser*, da Sie vermutlich noch keinen VPN zum Hostsystem aufgebaut haben.

Zunächst brauchen wir also eine Ubuntu & eine PfSense Iso auf dem Hostsystem. (auf dem WebUI: Datacenter -> Hostname -> Local -> Iso Images)

### Schritt 4.1 - `<vmbr WAN>`

Bevor wir diese Images installieren, brauchen wir sogenannte vm-bidges `vmbr`:
(auf dem WebUI: Datacenter -> Hostname -> Network)

Die Zahl der vmbr können Sie frei wählen.

IPv4/CIDR: 10.10.10.1/30


![Create vmbr](/assets/posts/proxmox-hetzner/create_vmbr.png)

### Schritt 4.2 - `<vmbr LAN>`

Zudem brauchen wir eine vmbr, die als LAN Interface für die Router-VM agiert.

Diese Werte entsprechen den Standardwerten des PfSense LAN-Netzwerks:

IPv4/CIDR: 192.168.1.2/24

Diese vmbr kann genauso wie die vmbr für das WAN angelegt werden, mit ausnahme der IP Adresse

### Schritt 4.3 - `<Installation Ubuntu VM>`

Jetzt erstellen wir eine neue VM mit der Ubuntu Iso.
Der Netzwerkadapter sollte mit der vmbr LAN verbunden sein, bedarf aber im Gatsystem keiner weiteren konfiguration, da die PfSense ein DHCP-Paket versendet.
(auf dem WebUI: Datacenter -> Hostname der Ubuntu VM -> VM -> Hardware)
Im zweifel im Gastsystem einstellen: IPv4/CIDR: 192.168.1.3/24 Gateway 192.168.1.1

### Schritt 4.4 - `<Installation PfSense>`

Jetzt erstellen wir eine neue VM mit der PfSense Iso.
Grundsätzlich empfiehelt es sich, der PfSense VM ausreichend Resourcen zur Verfügung zu stellen.
Vor dem Start weisen wir der VM zwei neue Netzwerkadapter als *VirtIO Interfaces* in den eben erstellen vmbridges zu.
(auf dem WebUI: Datacenter -> Hostname -> VM -> Hardware
)
Nun starten und installieren wir die PfSense VM. Die installation ist nach dem Booten der VM selbsterklärend.

Nach dem Reboot weisen wir die erstellten Netzwerkinterfaces zu.
Die MAC-Adressen können im zweifel zum differenzieren der Adapter genutzt werden.
![Create vmbr](/assets/posts/proxmox-hetzner/pfsense_config_interfaces.png)


### Schritt 4.5 - `<Einrichtung PfSense>`

Über eine Ubuntu-VM verbinden wir uns nun per browser mit der IP der Router-VM, 192.168.1.1

user: admin
passwort: pfsense

Die einrichtung ist auch hier bis zum Punkt "Configure WAN Interface" selbsterklärend.

Hier geben wir bei Selected Type "Static" und bei IP configuraton die IP 10.10.10.2 an.
Als Subnetzmaske wählen wir /30.

Als Gateway geben wir die Public IP des Hosts an. Wichtig hierbei ist das setzen des Hakens bei `Use non-local gateway through interface specific route` in den "Advanced" Einstellungen des Gateways. 
(System -> Routing -> Gateways -> Edit)

### Schritt 4.5.1 - `<Hardware checksum offload>`

Wichtig:

Nun ist unbedingt der `hardware checksum offload` zu deaktivieren, da dies bei VirtIO interfaces [zu problemen führt](https://docs.netgate.com/pfsense/en/latest/recipes/virtualize-proxmox-ve.html#disable-hardware-checksums-with-proxmox-ve-virtio)
(System -> Advanced -> Networking -> Network Interfaces -> Disable hardware checksum offload ✅)


### Schritt 5 - `<Netzwerkconfig auf dem Hostsystem>`

Nun müssen wir die Interfaces Datei auf dem Hostsystem (Proxmox) bearbeiten.

Diese Konfigurationsdatei definiert die Netzwerkschnittstellen des Systems und die Einstellungen für jede dieser Schnittstellen.

Die Datei existiert bereits und kann mit `nano /etc/network/interfaces` bearbeitet werden.

Unten finden Sie eine vollständige Interfaces Datei.

### Schritt 5.1 - `<Bearbeiten der Config>`

Idealerweise kopieren Sie die Datei und die beispielkonfiguration, öffnen Sie nebeneinander und übertragen die Details ihrer Datei in das unten abgebildete file. Dieser schritt ist unbedingt nötig, da INTERFACENAME in unserer Datei mit dem Name ihres Netzwerkinterface ersetzt werden muss.


Zu bearbeitende Variablen:
        *IP-HOSTSYSTEM
        *GATEWAY-HOSTSYSTEM
        *INTERFACENAME
        *Auch die namen der von ihnen angelegten `vmbr` können von den unten definierten `vmbr1` & `vmbr99` abweichen.

### Schritt 5.2 - `<Interfaces>`

Die IPv6 Konfiguration (iface INTERFACENAME inet6 static) können Sie aus der vom Host Automatisch generierten Datei übernehmen.


INTERFACENAME: Dies ist die Schnittstelle, die für den WAN-Verkehr (Internet) verwendet wird. Sie ist als statische IP-Adresse konfiguriert und leitet den gesamten Datenverkehr, der nicht für den Port 22 oder 8006 bestimmt ist, an die PfSense VM mit der IP-Adresse 10.10.10.2 weiter.

vmbr1: Dies ist die Bridge für das lokale Netzwerk (LAN). Sie ist als statische IP-Adresse konfiguriert.


vmbr99: Dies ist die Bridge für das WAN Netzwerk zwischen dem Host und der PfSense VM. Sie ist ebenfalls als 
statische IP-Adresse konfiguriert.

### Step 5.3 - `<NAT Regeln>`

Die erste "iptables"-Regel auf der WAN Schnittstelle besagt, dass jeder eingehende TCP-Verkehr, der nicht für die Ports 8006 oder 22 bestimmt ist, an eine interne IP-Adresse 10.10.10.2 weitergeleitet werden soll. Das Ausrufezeichen (!) vor "--dport" bedeutet "nicht", so dass jeder Port außer 8006 und 22 dieser Regel entspricht.

Die zweite "iptables"-Regel besagt, dass jeder eingehende UDP-Verkehr an die gleiche interne IP-Adresse 10.10.10.2 weitergeleitet werden soll. Da es keine Port-Beschränkung gibt, wird diese Regel auf jeden eingehenden UDP-Verkehr zutreffen.

Durch die Weiterleitung des eingehenden Datenverkehrs an eine interne IP-Adresse ermöglicht NAT den Geräten im internen Netz die Kommunikation mit der Außenwelt über die öffentliche IP-Adresse der Netzwerkschnittstelle, die im Parameter "gateway" angegeben ist.

```shell
#network interface settings; JMG-IT 2023
#Please do NOT modify this file directly, unless you really know what
#you're doing.

source /etc/network/interfaces.d/*

auto lo
iface lo inet loopback

iface lo inet6 loopback

auto INTERFACENAME
iface INTERFACENAME inet static
        address IP-HOSTSYSTEM/32
        gateway GATEWAY-HOSTSYSTEM
        post-up echo 1 > /proc/sys/net/ipv4/ip_forward
        post-up sysctl -w net.ipv6.conf.all.forwarding=0
        post-up iptables -t nat -A PREROUTING -i INTERFACENAME -p tcp -m multiport ! --dport 8006,22 -j DNAT --to 10.10.10.2
        post-up iptables -t nat -A PREROUTING -i INTERFACENAME -p udp -j DNAT --to 10.10.10.2
#WAN - All ports that are NOT 22,8006[SSH,PVE] DNAT to PfSense VM on 10.10.10.2

iface INTERFACENAME inet6 static
        address 2a01:xxxx::xxxx::x/64
        gateway fe80::1

auto vmbr99
iface vmbr99 inet static
        address 10.10.10.1/30
        bridge-ports none
        bridge-stp off
        bridge-fd 0
        post-up   iptables -t nat -A POSTROUTING -s '10.10.10.0/30' -o INTERFACENAME -j MASQUERADE
        post-down iptables -t nat -D POSTROUTING -s '10.10.10.0/30' -o INTERFACENAME -j MASQUERADE
        post-up   iptables -t raw -I PREROUTING -i fwbr+ -j CT --zone 1
        post-down iptables -t raw -D PREROUTING -i fwbr+ -j CT --zone 1
#WANBRIDGE

auto vmbr1
iface vmbr1 inet static
        address 192.168.1.2/24
        bridge-ports none
        bridge-stp off
        bridge-fd 0
#LAN Bridge
```

``reboot`

## Fazit

Nun sollten Sie ein LAN haben, das in das Internet geNATet wird. Alle ports ausser 8006 und 22 werden nun vom Proxmox Host auf die PfSense VM geleitet.

##### License: MIT

<!--

Contributor's Certificate of Origin

By making a contribution to this project, I certify that:

(a) The contribution was created in whole or in part by me and I have
    the right to submit it under the license indicated in the file; or

(b) The contribution is based upon previous work that, to the best of my
    knowledge, is covered under an appropriate license and I have the
    right under that license to submit that work with modifications,
    whether created in whole or in part by me, under the same license
    (unless I am permitted to submit under a different license), as
    indicated in the file; or

(c) The contribution was provided directly to me by some other person
    who certified (a), (b) or (c) and I have not modified it.

(d) I understand and agree that this project and the contribution are
    public and that a record of the contribution (including all personal
    information I submit with it, including my sign-off) is maintained
    indefinitely and may be redistributed consistent with this project
    or the license(s) involved.

Signed-off-by: Jakob Langisch, J.Langisch@jmg-it.de

-->
