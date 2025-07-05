
### **1. Executive Summary**

This project outlines the design, implementation, and verification of a secure multi-branch enterprise network using a hub-and-spoke topology. The network consists of a Headquarter (HQ) office and two Branch offices, all interconnected via a simulated public network represented by an ISP router. The core objective was to establish secure, encrypted communication between the Headquarter and each branch using Site-to-Site IPsec VPN tunnels. The project was simulated in GNS3, and this document includes detailed configurations and a step-by-step troubleshooting guide for addressing common implementation challenges.

### **2. Network Topology**

The network topology follows a hub-and-spoke model, with the Headquarter router (`R-HQ`) acting as the hub. All branch routers (`R-Branch1`, `R-Branch2`) connect to the HQ via a central ISP router, simulating an internet connection.

### **3. IP Addressing Scheme**

The following table details the IP addressing used for all devices and interfaces in the topology.
### **3. IP Addressing Scheme**

The following table details the IP addressing used for all devices and interfaces in the topology.

| Device          | Interface            | IP Address        | Subnet Mask       | Role                    |
|-----------------|----------------------|-------------------|-------------------|-------------------------|
| **R-HQ** | `FastEthernet0/0`    | `192.168.1.1`     | `255.255.255.0`   | LAN Interface           |
|                 | `FastEthernet1/0`    | `203.0.113.2`     | `255.255.255.252` | WAN Interface (to ISP)  |
| **PC-HQ** | `(connected to Fa0/0)` | `192.168.1.10`    | `255.255.255.0`   | Host in HQ LAN          |
| **R-Branch1** | `FastEthernet0/0`    | `192.168.2.1`     | `255.255.255.0`   | LAN Interface           |
|                 | `FastEthernet1/0`    | `203.0.113.6`     | `255.255.255.252` | WAN Interface (to ISP)  |
| **PC-Branch1** | `(connected to Fa0/0)` | `192.168.2.10`    | `255.255.255.0`   | Host in Branch1 LAN     |
| **R-Branch2** | `FastEthernet0/0`    | `192.168.3.1`     | `255.255.255.0`   | LAN Interface           |
|                 | `FastEthernet1/0`    | `203.0.113.10`    | `255.255.255.252` | WAN Interface (to ISP)  |
| **PC-Branch2** | `(connected to Fa0/0)` | `192.168.3.10`    | `255.255.255.0`   | Host in Branch2 LAN     |
| **R-ISP** | `FastEthernet0/0`    | `203.0.113.1`     | `255.255.255.252` | WAN Interface (to R-HQ) |
|                 | `FastEthernet1/0`    | `203.0.113.5`     | `255.255.255.252` | WAN Interface (to R-Branch1) |
|                 | `FastEthernet1/1`    | `203.0.113.9`     | `255.255.255.252` | WAN Interface (to R-Branch2) |

### **4. Router Configurations**

Here are the complete, final configurations for each router after all troubleshooting steps were applied.

#### **R-HQ Configuration**

```cisco
!
hostname R-HQ
!
crypto isakmp policy 10
  encr aes
  authentication pre-share
  group 5
  hash sha
!
crypto isakmp key ciscovpn address 203.0.113.6
crypto isakmp key ciscovpn address 203.0.113.10
!
crypto ipsec transform-set MY_TRANSFORM_SET esp-aes esp-sha-hmac
!
crypto map MY_CRYPTO_MAP 10 ipsec-isakmp
  set peer 203.0.113.6
  set transform-set MY_TRANSFORM_SET
  match address 100
!
crypto map MY_CRYPTO_MAP 20 ipsec-isakmp
  set peer 203.0.113.10
  set transform-set MY_TRANSFORM_SET
  match address 101
!
interface FastEthernet0/0
  ip address 192.168.1.1 255.255.255.0
  duplex auto
  speed auto
!
interface FastEthernet1/0
  ip address 203.0.113.2 255.255.255.252
  duplex auto
  speed auto
  crypto map MY_CRYPTO_MAP
!
ip route 0.0.0.0 0.0.0.0 203.0.113.1
!
access-list 100 permit ip 192.168.1.0 0.0.0.255 192.168.2.0 0.0.0.255
access-list 101 permit ip 192.168.1.0 0.0.0.255 192.168.3.0 0.0.0.255
!
end
```

#### **R-Branch1 Configuration**

```cisco
!
hostname R-Branch1
!
crypto isakmp policy 10
  encr aes
  authentication pre-share
  group 5
  hash sha
!
crypto isakmp key ciscovpn address 203.0.113.2
!
crypto ipsec transform-set MY_TRANSFORM_SET esp-aes esp-sha-hmac
!
crypto map MY_CRYPTO_MAP 10 ipsec-isakmp
  set peer 203.0.113.2
  set transform-set MY_TRANSFORM_SET
  match address 100
!
interface FastEthernet0/0
  ip address 192.168.2.1 255.255.255.0
  duplex auto
  speed auto
!
interface FastEthernet1/0
  ip address 203.0.113.6 255.255.255.252
  duplex auto
  speed auto
  crypto map MY_CRYPTO_MAP
!
ip route 0.0.0.0 0.0.0.0 203.0.113.5
!
access-list 100 permit ip 192.168.2.0 0.0.0.255 192.168.1.0 0.0.0.255
!
end
```

#### **R-Branch2 Configuration**
```cisco
!
hostname R-Branch2
!
crypto isakmp policy 10
  encr aes
  authentication pre-share
  group 5
  hash sha
!
crypto isakmp key ciscovpn address 203.0.113.2
!
crypto ipsec transform-set MY_TRANSFORM_SET esp-aes esp-sha-hmac
!
crypto map MY_CRYPTO_MAP 10 ipsec-isakmp
  set peer 203.0.113.2
  set transform-set MY_TRANSFORM_SET
  match address 100
!
interface FastEthernet0/0
  ip address 192.168.3.1 255.255.255.0
  duplex auto
  speed auto
!
interface FastEthernet1/0
  ip address 203.0.113.10 255.255.255.252
  duplex auto
  speed auto
  crypto map MY_CRYPTO_MAP
!
ip route 0.0.0.0 0.0.0.0 203.0.113.9
!
access-list 100 permit ip 192.168.3.0 0.0.0.255 192.168.1.0 0.0.0.255
!
end
```
#### **R-ISP Configuration**

```cisco
!
hostname R-ISP
!
interface FastEthernet0/0
  ip address 203.0.113.1 255.255.255.252
  duplex auto
  speed auto
!
interface FastEthernet1/0
  ip address 203.0.113.5 255.255.255.252
  duplex auto
  speed auto
!
interface FastEthernet1/1
  ip address 203.0.113.9 255.255.255.252
  duplex auto
  speed auto
!
! Static routes to ensure return traffic from branches can reach the HQ LAN
ip route 192.168.1.0 255.255.255.0 203.0.113.2
ip route 192.168.2.0 255.255.255.0 203.0.113.6
ip route 192.168.3.0 255.255.255.0 203.0.113.10
!
end
```
### **5. Implementation & Verification**

1.  **Build the topology** in GNS3 and load the configuration files onto each router.
2.  **Verify Layer 3 WAN Connectivity** by pinging from `R-HQ` to `R-Branch1` and `R-Branch2`'s WAN IP addresses (`ping 203.0.113.6` and `203.0.113.10`).
3.  **Initiate the VPN tunnel** by sending "interesting traffic" (e.g., pinging from a PC in the HQ LAN to a PC in a branch LAN).
4.  **Verify the VPN tunnel status** using `show crypto isakmp sa` and `show crypto ipsec sa`. The `QM_IDLE` state confirms successful establishment.
5.  **Confirm end-to-end connectivity** by pinging from `PC-HQ` to `PC-Branch1` (`192.168.2.10`) and `PC-Branch2` (`192.168.3.10`).

### **6. Troubleshooting & Lessons Learned**

During the implementation, the VPN tunnels initially failed to establish. The systematic troubleshooting process revealed two distinct problems:

1.  **IKE Policy Mismatch:** The initial `show crypto isakmp sa` output showed `MM_NO_STATE`, indicating a failure in the initial negotiation. The root cause was a missing `hash sha` command in the `crypto isakmp policy`, which was not auto-negotiated by the routers.
2.  **Layer 3 Routing Failure:** Even after correcting the VPN policy, the tunnel did not establish. This led to a crucial Layer 3 check. A ping from `R-HQ`'s WAN interface to `R-Branch1`'s WAN IP failed with "Destination Unreachable" (`UUUUU`), revealing that the ISP router was not routing traffic correctly. The `FastEthernet1/0` interface on the ISP router was found to be administratively down and unconfigured, breaking the connectivity to the branch.
