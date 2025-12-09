# MikroTik RouterOS Basic Network Configuration

## Project Description
Dokumentasi konfigurasi dasar MikroTik RouterOS untuk simulasi jaringan sederhana menggunakan VirtualBox. Project ini mencakup setup dual interface (INTERNET dan CLIENT) dengan konfigurasi routing, DHCP server, dan firewall NAT untuk koneksi internet sharing.

---

## Network Topology

```
INTERNET (192.168.1.0/24)
        |
   [ether1-INT] 192.168.1.200/24
        |
    RouterOS (MikroTik)
        |
   [ether2-CLIENT] 10.10.10.1/24
        |
  CLIENT NETWORK (10.10.10.0/24)
```

---

## Screenshots

### RouterOS Console View
![RouterOS Console](./assets/routeros-console.png)
*Console RouterOS menampilkan konfigurasi interface dan hasil ping test*

### WinBox Interface Configuration
![WinBox Configuration](./assets/winbox-interface.png)
*Tampilan WinBox menampilkan konfigurasi interface, IP address, routing, dan DHCP server*

---

## Interface Configuration

### Interface List
| Interface Name | IP Address | Network | Gateway | Purpose |
|---------------|------------|---------|---------|---------|
| **ether1-INT** | 192.168.1.200/24 | 192.168.1.0/24 | 192.168.1.1 | Internet/WAN |
| **ether2-CLIENT** | 10.10.10.1/24 | 10.10.10.0/24 | - | Local Client/LAN |

### Network Details
- **INTERNET Network**: 192.168.1.0/24
  - RouterOS IP: 192.168.1.200
  - Gateway: 192.168.1.1 (ke Internet)
  
- **CLIENT Network**: 10.10.10.0/24
  - RouterOS IP: 10.10.10.1
  - DHCP Pool: 10.10.10.10 - 10.10.10.24

---

## Configuration Steps

### 1. Interface Configuration
```routeros
# Set IP untuk interface INTERNET
/ip address add address=192.168.1.200/24 interface=ether1-INT

# Set IP untuk interface CLIENT
/ip address add address=10.10.10.1/24 interface=ether2-CLIENT

# Rename interfaces (optional)
/interface set ether1 name=ether1-INT
/interface set ether2 name=ether2-CLIENT
```

![Interface Setup](./assets/routeros-console.png)

### 2. Routing Configuration
```routeros
# Add default route ke gateway internet
/ip route add dst-address=0.0.0.0/0 gateway=192.168.1.1
```

**Verification:**
```routeros
/ip route print
```

### 3. DHCP Server Configuration
```routeros
# Setup DHCP Server untuk CLIENT network
/ip pool add name=dhcp_pool0 ranges=10.10.10.10-10.10.10.24

/ip dhcp-server add name=dhcp1 interface=ether2-CLIENT address-pool=dhcp_pool0

/ip dhcp-server network add address=10.10.10.0/24 gateway=10.10.10.1 dns-server=8.8.8.8
```

![DHCP Configuration](./assets/winbox-interface.png)
*DHCP Server terlihat aktif dengan pool address 10.10.10.10-10.10.10.24*

### 4. Firewall NAT Configuration
```routeros
# Enable NAT masquerade untuk client bisa akses internet
/ip firewall nat add chain=srcnat out-interface=ether1-INT action=masquerade
```

**View di WinBox:**  
![Firewall NAT](./assets/winbox-interface.png)
*NAT rule untuk masquerade sudah terkonfigurasi*

---

## Testing & Verification

### Check Interface Status
```routeros
/interface print
/ip address print
```

![IP Address List](./assets/winbox-interface.png)
*Daftar IP address yang sudah dikonfigurasi*

### Check Routing Table
```routeros
/ip route print
```

![Route List](./assets/winbox-interface.png)
*Routing table menampilkan default gateway 192.168.1.1*

### Check DHCP Server
```routeros
/ip dhcp-server print
/ip dhcp-server lease print
```

### Ping Test
```routeros
# Ping ke gateway
ping 192.168.1.1 count=5

# Ping ke internet
ping 8.8.8.8 count=5
```

![Ping Test Result](./assets/routeros-console.png)
*Hasil ping test menunjukkan koneksi berhasil dengan 0% packet loss*

---

## VirtualBox Network Settings

### Adapter Configuration
- **Adapter 1 (ether1-INT)**: 
  - Type: Bridged Adapter atau NAT
  - Purpose: Koneksi ke Internet/Host network
  
- **Adapter 2 (ether2-CLIENT)**: 
  - Type: Internal Network atau Host-only
  - Purpose: Network untuk client devices

---

## Features Implemented

✅ Dual Interface Setup (WAN & LAN)  
✅ Static IP Configuration  
✅ Default Gateway Routing  
✅ DHCP Server untuk Client Network  
✅ NAT/Masquerade untuk Internet Sharing  
✅ Basic Firewall Configuration  

---

## System Requirements

- **VirtualBox**: Version 6.1 or higher
- **RouterOS**: Version 6.49+ (tested with v6.49.15)
- **WinBox**: Version 3.x or higher (v3.43 dalam screenshot)
- **RAM**: Minimum 256MB (Recommended 512MB-1GB)
- **Storage**: Minimum 128MB (Recommended 1GB)

---

## Troubleshooting

### Problem: RouterOS tidak bisa ping ke internet
**Solution**: 
- Check default route: `/ip route print`
- Check DNS: `/ip dns print`
- Test ping ke gateway dulu: `ping 192.168.1.1`

### Problem: Client tidak dapat IP dari DHCP
**Solution**:
- Check DHCP server status: `/ip dhcp-server print`
- Check DHCP pool: `/ip pool print`
- Check interface binding: `/ip dhcp-server print detail`

### Problem: Client dapat IP tapi tidak bisa internet
**Solution**:
- Check NAT rule: `/ip firewall nat print`
- Check routing: `/ip route print`
- Pastikan masquerade rule sudah benar

---

## Project Structure

```
routeros-basic-network/
├── README.md                 # Dokumentasi ini
├── assets/
│   ├── routeros-console.png  # Screenshot RouterOS console
│   └── winbox-interface.png  # Screenshot WinBox configuration
└── readme.txt                # Network configuration summary
```

---

## Additional Resources

- **MikroTik Wiki**: https://wiki.mikrotik.com
- **MikroTik Forum**: https://forum.mikrotik.com
- **Download WinBox**: https://mikrotik.com/download
- **RouterOS Manual**: https://help.mikrotik.com/docs/

---

## Configuration Summary (Quick Reference)

```
Interface Configuration:
- ether1-INT: 192.168.1.200/24 (INTERNET)
- ether2-CLIENT: 10.10.10.1/24 (CLIENT)

Routing:
- Default Gateway: 192.168.1.1 via ether1-INT

DHCP Server:
- Pool: dhcp_pool0 (10.10.10.10 - 10.10.10.24)
- Network: 10.10.10.0/24
- Gateway: 10.10.10.1
- DNS: 8.8.8.8

Firewall NAT:
- Chain: srcnat
- Out-Interface: ether1-INT
- Action: masquerade
```

---

## Author & License

**Created for**: Learning & Lab Environment  
**Purpose**: Basic RouterOS Network Configuration Practice  
**License**: Free to use for educational purposes  

---

## Changelog

**v1.0** (December 2025)
- Initial setup with dual interface
- DHCP server configuration
- Basic NAT/Firewall setup
- Documentation created with screenshots

---

## Notes

⚠️ **Important**: 
- Konfigurasi ini untuk **lab/learning environment**
- Untuk production, tambahkan firewall rules yang lebih kompleks
- Backup konfigurasi secara berkala: `/system backup save name=backup`
- Change default password setelah first login!

📝 **Best Practices**:
- Selalu set password untuk user admin
- Enable firewall untuk security
- Monitor system logs: `/log print`
- Update RouterOS secara berkala

---

*Documentation with screenshots captured from actual working configuration*
