# 🌐 Lab 3 — IPv6 Address Autoconfiguration (SLAAC + EUI-64)

![Cisco](https://img.shields.io/badge/Cisco-CCNA-blue?style=for-the-badge&logo=cisco&logoColor=white)
![PacketTracer](https://img.shields.io/badge/Packet%20Tracer-8.x-orange?style=for-the-badge&logo=cisco)
![Status](https://img.shields.io/badge/Status-✅%20Completed-brightgreen?style=for-the-badge)
![Lab](https://img.shields.io/badge/101%20Labs-Lab%203-purple?style=for-the-badge)
![SLAAC](https://img.shields.io/badge/SLAAC-Autoconfiguration-red?style=for-the-badge)
![EUI64](https://img.shields.io/badge/EUI--64-Addressing-green?style=for-the-badge)

---

## 📋 Description

Troisième lab du livre **101 Labs Cisco CCNA (200-301)**.
L'objectif est de configurer des adresses **IPv6** sur des
routeurs Cisco en utilisant **SLAAC** (Stateless Address
Autoconfiguration) et **EUI-64** pour générer automatiquement
la partie hôte de l'adresse IPv6.

### Objectifs
- ✅ Configurer les **hostnames** sur R1 et R3
- ✅ Configurer l'interface **F0/0** de R1 avec une adresse IPv6
- ✅ Configurer **SLAAC** sur F0/0 de R3
- ✅ Configurer **EUI-64** sur Loopback0 de R3
- ✅ Vérifier la configuration avec les commandes **show ipv6**

---

## 🖥️ Équipements

| Équipement | Modèle | Nom | Rôle |
|-----------|--------|-----|------|
| 🔀 Routeur | Cisco 2911 | R1 | Annonce préfixe IPv6 |
| 🔀 Routeur | Cisco 2911 | R3 | SLAAC + EUI-64 |

---

## 🗺️ Topologie

```
[R1]────F0/0────────────────F0/0────[R3]
2001:ABCD:ABCD::1/64    SLAAC auto
                         Loopback0 : EUI-64
```

<img width="1920" height="1080" alt="Capture d’écran du 2026-07-12 18-38-08" src="https://github.com/user-attachments/assets/76a90603-02a1-4ab4-9d2a-ac270b0ff227" />


---

## 📊 Plan d'adressage IPv6

| Interface | Adresse IPv6 | Méthode | Routeur |
|-----------|-------------|---------|---------|
| F0/0 | 2001:ABCD:ABCD::1/64 | Manuelle | R1 |
| F0/0 | Automatique | **SLAAC** | R3 |
| Loopback0 | 2001:aaaa:aaaa:aaaa::/64 | **EUI-64** | R3 |

---

## ⚙️ Configuration complète

### 🔧 Task 1 — Hostnames

```cisco
! Sur R1
enable
configure terminal
hostname R1
end

! Sur R3
enable
configure terminal
hostname R3
end
```

---

### 🔧 Task 2 — Configuration R1

```cisco
enable
configure terminal
hostname R1

! Activer le routage IPv6
ipv6 unicast-routing

! Interface F0/0 - adresse manuelle
interface fastEthernet 0/0
ipv6 address 2001:ABCD:ABCD::1/64
no shutdown
exit

end
write
```

---

### 🔧 Task 2 — Configuration R3 (SLAAC + EUI-64)

```cisco
enable
configure terminal
hostname R3

! Activer le routage IPv6
ipv6 unicast-routing

! Interface F0/0 - SLAAC
! R3 obtient automatiquement son IP depuis R1
interface fastEthernet 0/0
ipv6 address autoconfig
no shutdown
exit

! Loopback0 - EUI-64
! La partie hôte est générée depuis l'adresse MAC
interface loopback 0
ipv6 address 2001:aaaa:aaaa:aaaa::/64 eui-64
exit

end
write
```

---

### 🔧 Task 3 — Commandes de vérification

#### 1. Résumé de toutes les adresses IPv6

```cisco
R1# show ipv6 interface brief
R3# show ipv6 interface brief
```

**Résultat attendu R1 :**
```
FastEthernet0/0    [up/up]
    FE80::1        ← Link-local
    2001:ABCD:ABCD::1  ✅ Manuelle
```

**Résultat attendu R3 :**
```
FastEthernet0/0    [up/up]
    FE80::3
    2001:ABCD:ABCD::XXXX  ✅ SLAAC auto
Loopback0          [up/up]
    FE80::3
    2001:AAAA:AAAA:AAAA::XXXX  ✅ EUI-64
```

---

#### 2. Statut des interfaces

```cisco
R1# show interfaces fastEthernet 0/0
R3# show interfaces fastEthernet 0/0
```

**Résultat attendu :**
```
FastEthernet0/0 is up, line protocol is up ✅
```

---

#### 3. Préfixe IPv6 appliqué

```cisco
R1# show ipv6 interface fastEthernet 0/0
R3# show ipv6 interface loopback 0
```

---

#### 4. Table de routage IPv6

```cisco
R1# show ipv6 route
R3# show ipv6 route
```

---

## 🧪 Tests de connectivité

```cisco
! Depuis R3 vers R1
R3# ping ipv6 2001:ABCD:ABCD::1  ✅

! Depuis R1 vers R3 (adresse SLAAC)
R1# show ipv6 neighbors
R1# ping ipv6 [adresse SLAAC de R3]  ✅
```

---

## 📖 Notions importantes

### SLAAC (Stateless Address Autoconfiguration)
```
→ Pas de serveur DHCP nécessaire
→ Le routeur R1 envoie des Router Advertisement (RA)
→ R3 reçoit le préfixe réseau depuis R1
→ R3 complète l'adresse avec son adresse MAC (EUI-64)
→ Résultat : adresse IPv6 complète automatique !
```

### EUI-64
```
→ Méthode pour générer la partie hôte IPv6
→ Basée sur l'adresse MAC de l'interface
→ MAC 48 bits → EUI-64 64 bits

Exemple :
MAC      : AA:BB:CC:DD:EE:FF
EUI-64   : AA:BB:CC:FF:FE:DD:EE:FF
           ↑ FF:FE inséré au milieu
           ↑ 7ème bit inversé
```

### Différences SLAAC vs DHCPv6
```
SLAAC    → Pas de serveur, IP auto depuis RA
DHCPv6   → Serveur distribue tout comme DHCPv4
Stateless → SLAAC + DHCPv6 pour DNS uniquement
```

---

## 💡 Points clés à retenir

| 🔑 Commande | 📖 Rôle |
|-------------|---------|
| `ipv6 unicast-routing` | Active le routage IPv6 |
| `ipv6 address autoconfig` | Active SLAAC sur l'interface |
| `ipv6 address X::/64 eui-64` | Configure EUI-64 |
| `show ipv6 interface brief` | Vue rapide IPv6 |
| `show ipv6 neighbors` | Table des voisins IPv6 |
| `show ipv6 route` | Table de routage IPv6 |
| `ping ipv6 X::X` | Test connectivité IPv6 |

---

## 📊 Comparatif Lab 1, 2 et 3

| | Lab 1 | Lab 2 | Lab 3 |
|---|---|---|---|
| **Protocole** | IPv4 | IPv6 Manuel | IPv6 Auto |
| **Interface** | Serial | Serial | FastEthernet |
| **Méthode IP** | Manuelle | Manuelle | SLAAC + EUI-64 |
| **Clock rate** | ✅ Oui | ✅ Oui | ❌ Non |
| **Loopback** | ✅ Oui | ✅ Oui | ✅ Oui |

---

## 🛠️ Outils

![Cisco Packet Tracer](https://img.shields.io/badge/Cisco%20Packet%20Tracer-8.x-orange?style=flat-square&logo=cisco)
![GNS3](https://img.shields.io/badge/GNS3-2.x-green?style=flat-square)
![Cisco IOS](https://img.shields.io/badge/Cisco%20IOS-15.x-blue?style=flat-square)
![GitHub](https://img.shields.io/badge/GitHub-black?style=flat-square&logo=github)

---

## 👨‍💻 Auteur

**Urbain Sedami Landjidé**
🎓 Étudiant en 2ème année — Licence Professionnelle
📡 Réseaux Informatique Mobilité Sécurité (RMS)
🏫 Cisco Networking Academy
📍 Cotonou, Bénin 🇧🇯

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connecter-blue?style=flat-square&logo=linkedin)](https://www.linkedin.com/in/urbain-sedami-landjide-9b49043a8/)

---

## 📄 Licence

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

Libre d'utilisation pour l'apprentissage et la formation réseau.
