<h1>🧭 Policy-Based Routing (PBR) Lab – Cisco Network Redirection</h1>

 ### <h2>🧠 Objective</h2>

<p>This lab demonstrates how to implement **Policy-Based Routing (PBR)** to override standard routing decisions made by a router’s routing table. The goal is to redirect specific traffic to a different path based on defined policies, rather than relying purely on the shortest OSPF-learned path.</p>

<h2>🧪 Lab Overview</h2>
<div style="text-align:center">
<p align="center">
  <img src="https://i.imgur.com/Hp0Ixmm.png" width="80%" height="80%" alt="PBR_Digram">
</p>
</div>  
<p>The network topology consists of four routers (R1, R2, R3, and R4). Dynamic routing using OSPF is configured across all routers, and DHCP is used to assign IP addresses to end devices connected to R1 and R3.</p>

- R1 and R3 serve as edge routers, each connected to a PC.
- R2 and R4 serve as transit routers, with R4 being the default OSPF path from R1 to R3.
- The PC connected to R1 is assigned IP 192.168.0.2.via DHCP
- The PC connected to R3 receives IP 192.168.1.2 via DHCP.

Initially, traffic from the PC on R1 to the PC on R3 traverses the OSPF-learned default path:
<p align="center">
<img src="https://i.imgur.com/7kXp3Uo.png" height="80%" weight="80%" alt="Initial Traffic Path">
</br><i>R1 → R4 (203.0.113.2) → R3 (93.0.113.2) → 192.168.1.2</i>
</p>
<p>However, I implemented a policy on R1 to force selected traffic to instead take an alternate path via R2, bypassing OSPF preference, even though this link has lower bandwidth (10 Mbps, half-duplex) compared to the OSPF route (100 Mbps, full-duplex).</p>

</br>

<h2>🧪 Result</h2>
<p>After applying the policy route on R1, traffic from 192.168.0.2 to 192.168.1.2 was successfully rerouted through 203.0.113.6 via R2, as verified using traceroute. The PBR forced the router to ignore OSPF's preferred path (R4) and use a manually defined path (R2), demonstrating fine-grained traffic control using PBR.</p>

<p align="center">
<img src="https://i.imgur.com/QCVGIZA.png" height="80%" width="80%" alt="finalPath">
</p>
</br>
<h2>⚙️ Configuration Summary</h2>

 <h3>R1 Configuration</h3>
 <div>
   
     hostname R1

      ip dhcp pool R1
       network 192.168.0.0 255.255.255.0
       default-router 192.168.0.1
       dns-server 8.8.8.8
      
      interface FastEthernet0/0
       ip address 192.168.0.1 255.255.255.0
       ip policy route-map PC-TO-REMOTE
      
      interface FastEthernet0/1
       ip address 203.0.113.5 255.255.255.252
      
      interface FastEthernet1/0
       ip address 203.0.113.1 255.255.255.252
      
      router ospf 1
       router-id 1.1.1.1
       log-adjacency-changes
       network 192.168.0.0 0.0.0.255 area 0
       network 203.0.113.0 0.0.0.15 area 0
      
       Policy-Based Routing Configuration
      access-list 100 permit ip host 192.168.0.2 host 192.168.1.2
      
      route-map PC-TO-REMOTE permit 10
       match ip address 100
       set ip next-hop 203.0.113.6
      
    
 </div>

<h3> R2 Configuration</h3>
<div>
  <p>
        interface FastEthernet0/0
     ip address 203.0.113.6 255.255.255.252
    
    interface FastEthernet0/1
     ip address 103.0.113.1 255.255.255.252
    
    router ospf 1
     router-id 2.2.2.2
     log-adjacency-changes
     network 103.0.113.0 0.0.0.3 area 0
     network 203.0.113.4 0.0.0.3 area 0

  </p>
</div>

<h3> R3 Configuration</h3>
<div>
  <p>
    hostname R3
    
    ip dhcp pool R3
     network 192.168.1.0 255.255.255.0
     default-router 192.168.1.1
     dns-server 8.8.8.8
    
    interface FastEthernet0/0
     ip address 103.0.113.2 255.255.255.252
    
    interface FastEthernet0/1
     ip address 93.0.113.2 255.255.255.252
    
    interface FastEthernet1/0
     ip address 192.168.1.1 255.255.255.0
    
    router ospf 1
     router-id 3.3.3.3
     log-adjacency-changes
     network 93.0.113.0 0.0.0.3 area 0
     network 103.0.113.0 0.0.0.3 area 0
     network 192.168.1.0 0.0.0.255 area 0
  </p>
</div>

<h3>R4 Configuration</h3>
<div>
  <p>
        interface FastEthernet0/0
     description Connection to R1
     ip address 203.0.113.2 255.255.255.252
     speed 100
     full-duplex
    
    interface FastEthernet0/1
     description Connection to R3
     ip address 93.0.113.1 255.255.255.252
    
    router ospf 1
     router-id 4.4.4.4
     log-adjacency-changes
     network 93.0.113.0 0.0.0.3 area 0
     network 203.0.113.0 0.0.0.3 area 0

  </p>
</div>






