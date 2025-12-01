# 🌐 Comprehensive Guide – Turning a Vodafone Station Wi‑Fi 6 into a Pure Modem (Bridge‑Mode)

Dive into setting up and configuring your modem for internet browsing and voice services on the Vodafone network, with all the necessary technical specifications streamlined for clarity and ease of understanding.

## 🛠️ Minimum Technical Requirements

Your device must support specific protocol levels to ensure seamless internet service operation, from physical modem compatibility to the adaptation protocol/encapsulation and proper IP session establishment.

### Connection Speed Standards:

#### Up to 20Mbps:
- **Physical standard**: ADSL 2+
- **References**: ITU G.992.1 and G.992.5 over POTS
- **Session protocol**: PPPoE Routed (RFC2516)

#### Up to 200Mbps:
- **Physical standard**: VDSL2
- **References**: VDSL2 profiles 8b, 12a, 17a, 35b with Retransmission (ITU-T G.998.4), Vectoring (ITU-T G.993.5), and SRA (ITU-T G.993.2) recommended
- **Session protocol**: PPPoE Routed (RFC2516)

#### Up to 2.5Gbps (FTTH*):
- **Physical standard**: FTTH*
- **References**: GE WAN Interface (IEEE 802.1q)
- **Session protocol**: PPPoE Routed (RFC2516)

*For FTTH architecture, ensure your router has a Gigabit Ethernet WAN interface supporting Ethernet 802.1q protocol, connected to the Vodafone-provided ONT with at least a Category 6 Ethernet cable.

### Voice Service Requirements:

Your modem must support:
- **DNS SRV queries** for automatic service IP configuration (RFC 2782)
- **Codecs**: G.729, G.711 A-law, T.38

## 🔧 Configuration Parameters

### Data and Voice Service on an Alternative Modem:

#### ADSL2+ (Up to 20Mbps):
- **Encapsulation**: ATM LLC
- **VPI/VCI**: 10/36
- **Username/Password**: vodafoneadsl/vodafoneadsl
- **NAT**: Active

#### VDSL2 (Up to 200Mbps):
- **Encapsulation**: PTM
- **VLAN**: 1036
- **Username/Password**: vodafoneadsl/vodafoneadsl
- **NAT**: Active

#### FTTH* (Up to 2.5Gbps):
- **Encapsulation**: VLAN Ethernet 802.1q
- **VLAN**: 1036
- **Username/Password**: vodafoneadsl/vodafoneadsl
- **NAT**: Active

### Voice Service via Your Modem:

- **SIP Domain**: ims.vodafone.it
- **SIP Protocol**: UDP Port 5060
- **Voice Codecs** (priority order): G.711 A-law, G.729
- **Fax Codecs**: G.711 A-law, T.38

## ⚙️ Additional Recommended Settings (If Available):

- **Packetization Time**: 20ms
- **Expire Time**: 3600
- **DSCP Marking**: 34 (decimal)
- **Register fetch/Fetching bindings**: NO
- **DTMF Tones Support**: RFC 2833, InBand, SIP INFO
- **Voice Activity Detection (VAD)**: Disabled
- **100rel Message Support (PRACK)**: Enabled per RFC3262
- **UPDATE Support**: Enabled per RFC3311

## 📡 Using Your Modem with Vodafone Station for Data and Voice Services

### Configuring Vodafone Station for Alternative Modem Use:

1. **Expert User Mode**: Enable it.
2. **WiFi Settings**: Disable Vodafone Station WiFi.
3. **DNS&DDNS**: Disable Secure DNS.
4. **Sharing Settings**: Disable UPnP, DLNA/Twonky, SAMBA, FTP.
5. **Internet - Firewall**: Disable Vodafone Station firewall.
6. **Vodafone Secure Network**: Disable if available.
7. **IPv4 Settings**: Set a new LAN DATA subnet different from 192.168.1.1 (e.g., 192.168.10.1).
8. **WAN Ethernet Connection**: Connect your alternative router's WAN port to a LAN port on the Vodafone Station. For non-WAN Ethernet routers, ensure WAN connection via a LAN port.
9. **Host Nat Static**: Set static NAT for the connected router using its LAN IP.
10. **Home Phone Connection**: Connect it to the FXS1 port on the Vodafone Station.

*Procedures are valid for Vodafone Station Revolution and Vodafone Power Station.

## ⚠️ Service Restrictions with Non

-Vodafone Modems:

Using an alternative modem limits access to specific Vodafone services, including Mobile Backup, Vodafone Station App, Secure Network, advanced WiFi management, and remote maintenance.

The responsibility for any issues related to your modem's installation, configuration, and operation lies with you or the device's supplier, excluding remote assistance.

## 🎉 You’re Done!

Your Vodafone Station now behaves purely as a modem/bridge, handing all routing, firewall, Wi‑Fi, and advanced services to your 3rd party router. Enjoy the flexibility, better performance, and the ability to manage your home‑lab network exactly the way you want. 🚀
