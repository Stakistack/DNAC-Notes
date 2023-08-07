# DNAC-Notes
A dump of DNAC notes from CBT Nuggets

# History
DNA Center can deploy Cisco's SDA (Software Defined Access), and is a SDN solution.

What is SDN? 
    History 
        Used to have issues like: 
            Deciding what vendor to go with 
                If using multiple, techs would have to try to make diff. devices work together 
                Generally wanted to go with a single vendor because reduced complexity 
        Idea of SDN was to move the management of devices to a centralized controller. 
            Is standards-based 
                1st SDN protocol was OpenFlow  
    Modern 
        Still have SDN controller 
            Hardware is generally from the same vendor as the controller 
        The goal of SDN is to help org. manage their growing networks 
            Ex. Stretching L2 between two locations greatly increases the broadcast / STP domains. 
            We solve this with SDN via Overlay/Underlays 
        Overlay 
            A set of tunnels as a networking construct in order to connect switches to each other to form the topology we want 
            The overlay is the topology we want the network to look like 
        Underlay 
            The physical hardware. What the network actually looks like. 
        Programmability  
            REST APIs for configuring networking gear 
                Uses structured language like JSON/XML to accomplish that 
            Can use these same elements on the controller itself, and the controller will then push those configurations to the network hardware. 
        Assurance 
            Uses AI machine learning to actively look for problems on the network - even before they are reported yet. 
        Cisco 
            Use a declarative SDN model 
            Controller tells hardware what to do. The network hardware has the intelligence to make that happen. 
        Other vendors 
            Use an imperative SDN model 
            Controller isn't really pushing configurations down. Its playing the role of the control plane  
                It makes all the forwarding decisions and then populates mac tables/ routing tables, etc. 
                Controller is very important. If it goes down, you just lost the control plane.  

    Scaling out L2 networks via ethernet is a pain 
    In most cases our security / QOS policies are deployed in vlans / subnets (just to say L2) 
        In traditional networks, since VLANs need to be stretch to cover as many devices as possible, it means we arent able to deploy L3 on the access layer in many cases. 

# Overlays / Underlays 
    Overlay 
        Basically a bunch of network tunnels that create the architecture you want 
        Like having a piece of paper on top of the physical topology, where you then 'draw' the topology you want 
    Underlay 
        The actual physical topology  
    The underlay would be L3, and the overlay would be L2+L3 using VXLAN 
    VXLAN is a L2/L3 encapsulation protocol 
    First Hop redundancy protocols not needed (like HSRP) 
        Instead you use Anycast Gateways at each of the switches 

# Campus Fabric 
    VXLAN used for the Data plane (the tunnels in the overlay) 
    LISP used for the control plane (where in the network other devices are) 
    CTS (Cisco TrustSec)- use ISE. determine who this person is, who they can speak to, and if it should even be allowed. 
    SDA = Campus fabric + DNAC 
    SDA = used where the users are 
    SD-WAN = used for our WAN circuits  
    ACI = used inside our data centers 
    SDA = VXLAN, LISP, and CTS.

# SDA Layers
SDA has 4 layers (Still using the OSI model. This is just a breakdown of SDA) 
    Physical Layer 
        Network devices 
    Network Layer 
        Overlay (fabric)/ Underlay 
    Controller Layer 
        DNAC / ISE (ISE configured and managed by DNAC) 
    Management Layer 
        Network programmability (REST APIs) 
        DNAC web GUI 

## Physical Layer 
    Physical Roles 
        Networking devices have to act in a specific way, relative to how they interact with the fabric. 
        Fabric edge node 
            Where wired users connect 
        Control plane node 
            Devices (usually routers) that know where everything else on the network is 
            Stores endpoint information 
        Fabric border node 
            Connects to outside 
            Connecting to other parts of the network that arent part of the SD Fabric.  
            Used for advertising fabric routes to the rest of the network and vise versa.  
            Can take the role of control plane nodes as well 
                Not recommended in large networks / networks with lots of user mobility (like roaming between Aps) 
        Fabric WLC 
            WLC that is aware of the SDA Fabric 
            Connects via the underlay 
        Intermediate nodes 
            Just L3 devices that will facilitate the tunnel connections between the other SDA devices. 
            Doesn’t even have to be cisco devices, and isnt aware of the fabric. 
## Physical Devices 
        Talk to cisco rep for most updated info. 
        Ex. Catalyst 9K switches 
            Have a UADP (Unified Access Data Plane) 
                What delivers the benefits of SDA. [how we use VXLAN / LISP] 
                Needed in fabric edge nodes 
        Ex 3650/3850  
            Older switches that have an older version of UADP 
                Programmable, so they can be used in an SDA Fabric 
        Fabric edge nodes 
            9200-9500 
            3650, 3850 
            4500E 
                Needs supervisor 8E or 9E installed (these have the UADP) 
        Fabric border nodes  
            (don’t need the UADP. These are the platforms supported by cisco because they have the appropriate hardware) 
            9300 - 9600 
            3850 
            6807, 6500  
                Needs supervisor 2T or 6T 
            6880-X, 6840-X 
        --Routers below here-- 
            ASR 1000  
            ISR 4300, 4400 
        Control Plane Node 
            Same list as fabric border nodes. 
        Fabric WLC 
            Catalyst 9800 
                Is a switch, but also a WLC. Has UADP. 
            3504, 5520, 8510 
        -Access Points- 
            9100 (generally recommended for greenfield deploy) 
            X700, x800 
            1540,1560 
        Controllers 
            DNAC 
                Physical mainly, but has virtual options now.
            ISE 
                Can be physical or virtual 

## Network Layer 

    Overlay 

    A fabric is an overlay 
        An overlay is built with tunnels over the physical architecture  (underlay). 
        Consists of tunnels that are established between edge devices 
        Use VXLAN (virtual extensible LAN) to accomplish this 
            This turns the edge device into a VTEP (Virtual Tunneling End Point) 

    VTEP 
        When client traffic arrives, it looks up and figures out that the traffic is destined for host B which is on switch 5. It then encapsulates that packet in a VXLAN header and sends it to switch 5. 
        It knows where the other switch was because of the SDA control plane. 
            It relies on the control plane nodes to get that traffic to the destination 
            We use LISP at the control plane. 
                BGP tables were getting large. Goal of LISP is to offload those core devices so they don’t need to know where every device on the internet lives. 
                Within SDA, the underlay only needs to know how to get to the final switching destination. Not to the end user. 
                    LISP lets us do this by offloading those routes from the intermediate nodes. 

    SDA Policy plane 
        Embed security into the solution from the ground up 
        CTS is used for this.  
            Applies tags to traffic called a Scalable Group Tag (SGT). 
            When traffic arrives, it looks at the tag and compares it with the policy associated with the tag. 
        Relies on ISE to build these SGTs. 
            We configure these on DNAC and then it will push those configurations to ISE. 
                ISE will then deploy those to the fabric. 


   Underlay 
        Only cares about getting the traffic from one L3 interface to another L3 interface 
        2 options: 
            Automated 
                Requires DNAC 
                L3 Underlay 
                Only for greenfield deployments 
                    Brand new networks 
                Leverages PNP technology. 
                    Switches will go out to the DNAC and get a config. 
                        These configs will leverage a global IP pool we configure in DNAC. 
            Manual 
                Need to know how to build out this underlay 
                Can be L2 or L3 
                    L3 is recommended  
                    L2 can work if you put everything in the same subnet. Has the issue of STP, etc.  
                L3 needs an IGP 
                    Cisco recommends ISIS. Can work with others though. 
                    ISIS is what the automated deployment would use. 
                Need to increase the MTU by 50 bytes for VXLAN. 
                    Can work without, but will fragment the packets. More overhead. 

## Controller Layer 
    DNAC is the SDN controller 
        Can deploy 1 or 3 of these. (3 is a cluster. Used for redundancy) 
    An Out of Band management system 
        If it goes down, the network would still be functional.  
    2 Sub systems: 
        NCP (Network Control Platform) 
            Modified version of APIC-EM 
            What lets you push those configs 
                Can use CLI, SNMP, or Netconf. 
        NDP (Network Data Platform) 
            Responsible for network assurance 
                Pulls information from devices 
                    Like syslog, SNMP, streaming telemetry  
                All this data goes into the Assurance Engine 
                    This is what lets it warn you about potential problems before they happen 
    ISE 
        Can be controlled via DNAC via REST APIs 
        Subsystems 
            PAN (Policy Administration Node) 
                The REST APIs are directed to PAN, and that is how DNAC communicates with ISE. 

## Management Layer 
    DNA Center 
        2 primary ways to interface with it: 
            Web GUI 
                Anything the web GUI does, it leverages REST APIs to do. 
                REST API on the backend. 
            REST APIs 
                Using applications like POSTMAN to accomplish the same thing the GUI uses 
    4 step workflows for managing our fabric: 
        Design 
            Setting up our network hierarchy  
                Will set up sites, buildings, floors. 
                Will create network settings like IP pools, DNS, DHCP 
                Image management 
                Network profiles 
                    Allows you to standardize network configurations for diff. locations 
        Policy 
            Macro segmentation 
                Keeping separate networks separate from each other 
                    Ex. Two different organizations in a shared space being kept separate 
                        Or two diff. departments, etc. 
                Rough equivalent to a VRF (called VNs in DNAC) 
            Micro segmentation  
                SGTs 
                    Determining whether devices in the same network can speak to one another 
        Provision 
            Define what your devices are and then discover them on the network 
            Connect the policy you defined in the policy workflow and connect them to the sites you designed in the design workflow 
                Connect policies -> sites 
            Deploy services here 
        Assurance 
            Pulling data from network, and NDP is crunching that data and trying to figure out whats going on in the network 
            Health Dashboard 
                Overview of whats happening in your network 
            Trends/insights 
                Deeper look into whats happening in your network 
            360 Views 
                Gives an in-depth view about a particular client/network device 
                    Useful for solving problems 

# SDA Fabric Operation 
Sum: 
    LISP at the Control Plane 
    VXLAN at the Data Plane 
    CTS at the Policy Plane 

## SDA Control Plane - LISP 
    Originally designed for the service provider space 
    LISP is L3 only 
    Without LISP, every single device needs to know every single subnet 
        Takes up a lot of resources 
    An IP can act as both an ID and a locator. 
        With LISP, you just want the IP to be the ID, not the locator. 
    EID (Endpoint Identifier) = our IP address 
    RLOC (Routing Locator) = how to locate our device 
         is a layer 3 interface on the switch you're attached to. 
        Is going to be the switch that connects you to the fabric 
    You will have a mapping server on the network that tracks all of the EID to RLOC mappings 
        When switch needs to forward traffic to a user, it'll check in with the mapping server to get that mapping. 
            It does this by encapsulating the packet and changing the IP to the correct RLOC. 
        Switches in the "middle" only need to know the RLOCs now. They don’t need to know EIDs. 

## SDA Data Plane - VXLAN 
    Tunnel mechanism that is L2 and L3 
    Carries the original ethernet header which allows for L2 transport (can share VLANs) 
    Also has support for L3 segmentation via VRFs 
        Segmentation between diff. networks (like diff. organizations sharing a space) 
    Can also carry SGTs 
        Tweaked by cisco. Called VXLAN-GPO (Group Policy Option) 
    Uses UDP 
    Edge switches called VTEPs 
    
## SDA Policy Plane - CTS 
    Who can talk to who in a network 
    Micro segementation  
        Access control within a network 
            Ex. Within subnets / the same organization  
        Managed with SGs (scalable groups) 
            Carried with traffic in the form of SGTs. 
                Located in the VXLAN header 
    Macro segementation  
        Access control between networks 
            Ex. Between departments / diff. organizations  
        Managed with VNs (similar to VRFs) 
            Identified by VNID (special field in the header that you carry with you when transporting traffic) 
                Located in the VXLAN header          

# User Authentication 
    3 methods: 
        802.1x 
            Seamless login, requires supplicant software 
            3 main components: 
                Supplicant (the end point) 
                Authenticator (the upstream networking device [NAD]) 
                Auth Server (ISE) 
            When a device connects to the network, the authenticator will send an EAP message(Extensible Authentication Protocol). This just requests the creds of the user.  
            When the authenticator receives the credentials it will use the RADIUS protocol to send those credentials to the auth server. 
            If creds good, server will respond with Access-Accept message 
            In an SDA environment, we now also know the SGT / VNID to assign to this user. 
        MAB (MAC authentication bypass) 
            Supports any client, requires database. 
            Going to use this for any device that doesn't support 802.1x [like IP phone] 
            Authenticator will send EAP message, and when the device doesn't respond, it will attempt MAB. 
            The Authenticator will send the MAC address to ISE. In order for this to work, ISE needs the MAC address in its Database. 
            Good backup option 
        Web Auth 
            No supplicant, requires user login 
            Allows the end point to have access to a web site to allow the user to log in. 
            DNS/DHCP will get passed to the host. The next web request on the host will send them to the website. 
            The user enters creds, and if everything is good they are allowed on the network. 
            
Endpoint Onboarding 
    When an end user connects to a fabric edge node, the fabric edge node will send a map-register message to the control plane node that contains the EID of the user. The control plane node will map the RLOC to the EID. RLOC is typically a loopback on the switch. 
    When another host on the network sends a packet to another host with its destination IP. The ITR will send a map-request to the control plane node. Control plane node sends a map-reply to the ITR will the RLOC that the other user is hanging off of. The ITR will store that RLOC - EID mapping in its cache. Will remain for up to 24 hours of inactivity. 
    Core underlay only needs to know how to get to RLOCs, not to EIDs.  

Wireless onboarding  
    Will have WAP attached directly to fabric edge nodes. Node will send that EID to the control plane where it gets mapped. 
    The WAP is only concerned about where the Wireless LAN Controller is. It will form a CAPWAP tunnel to where ever the WLC is. (control plane info still sent via CAPWAP tunnel) 
    The WAP will form a VXLAN tunnel to the fabric edge node. 
    The WLC will send a map-register message with the EID MAC. 
    The fabric edge node will also send a map-register with the EID IP.  
    At this point the users will be dropped onto the wired network after they hit the WAP. 
Sum: 
    ETR sends map-register to CP node. 
    ITR sends map-request to CP node. 
    CP Node sends map-reply to the ITR. 
    ITR caches entry, forwards traffic. 
    WAPs form VXLAN tunnels to Fabric Edge Nodes 

Endpoint Roaming 
    User B moves to a different Fabric edge node. 
    The Fabric Edge Node will send a map-register message to CP node. 
    CP node will update its table with the new RLOC - EID mapping. 
    CP node will send an update to the previous switch telling it user B is no longer on it. 
    Any new request that hits it, it will send a reply with the new edge node user b is attached to. 
        The other switch will then update their cache to the new switch that user B is attached to. 
        The old switch that user B was attached to will forward the traffic to the new switch on behalf of the sending switch until the update is complete. 

Sum: 
    New ETR updates CP node 
    CP Nodes updates original ETR 
    Original ETR updates other ITR dynamically  

 External Networks 
    Use Fabric Border nodes to connect to networks outside of the fabric 
    3 types: 
        Internal Fabric Border Node 
            Connects to other parts of our own network that aren't part of our SDA Fabric 
            With the help of a Fusion router, an INT. Border Node will exchange routes upstream with the internal networks. 
            Responsible for forwarding our SDA routes to the rest of the network 
            Responsible for forwarding external routes into the fabric. 
                Cant just use IGP, because we're not using IGP inside the fabric. 
                We're going to act as if we're a fabric edge node. 
                    Will send a map-register message to the CP node. 
                    The CP Node will map the subnets being advertised to the RLOC of the device. 
        External Border Node 
            Like the default gateway of the fabric. (its also referred to as a default border node) 
            Responsible for forwarding traffic to unknown destinations 
            Also requires a fusion router. 
            When the map-request hits the CP and it doesn’t know where the network is, it will send back a negative map reply. At this point the fabric edge node will send the traffic to the External Border Node. 
        Hybrid 
            Also called an Anywhere border Node. 
            Performs both the Internal and External Border node roles. 
            Better for smaller networks. Requires special configuration for this to happen. 
Sum: 
    Internal border node registers subnets to CP Node 
    External (default) border nodes handle unknown destinations and do not register anything to CP Node 
    Hybrid (Anywhere) border nodes perform both roles 

 The Fusion Router 
    Isn't strictly required in an SDA Fabric, but is recommended we deploy one. 
    Need to do VRF leaking to allow IGP <--> LISP route sharing. 
    If you have different VNs (multiple VRFs) and want them all to accesses shared services, need to do VRF leaking. 
        Doing VRF leaking directly on the border nodes can lead to some configuration challenges. This is where a fusion router comes into play. 
        A fusion router sits in-between an existing network and a border node 
    The fusion router will form an eBGP peer with our border nodes. 
    Uses an IGP with the rest of the network. Will redistribute those routes. 
    Border nodes learn routes via eBGP and registers them with the CP node. 
    Fusion router is NOT managed by DNAC. 
    No specific requirements for Fusion router. Just needs to be L3 and support VRFs / VRF leaking 

Sum: 
    The Fusion Router performs VRF route leaking. 
    The Fusion Router redistributes between IGPs and BGP 
    The Fusion Router is not strictly required, but it Is recommended by Cisco. 
    The Fusion Router is not managed by DNAC. 

Anycast Gateway 
    Because we're using an underlay and the underlay doesn't know where any of the subnets are, we can use the traditional default GW method with First Hop Redundancy Protocols. 
    You can throw the default gateways on your most powerful switch on the network, but that is inefficient because the traffic has to traverse the network to get anywhere. 
    With Anycast, you can put all the gateways on every fab edge node. 
        Results in duplicate IPs on the network. 
        Not an issue because we're using these IP addresses for local communication only (i.e. the subnet you wanted to reach was on the same switch as you) 
        Trying to use for remoting into wouldn't work out though.  

Sum: 
    Each SVI gets assigned to every Fabric Edge Node 
    Local endpoints use the local SVI 
    FHRPs are not required in SDA 


## LISP (non-SDA specific) [Locator ID Separation Protocol] 
    Solves the problem of the Internet routing table size 
    LISP lowers the resources needed on core devices 
    LISP solution = locator/ID separation 
    Underlay only needs to know where the locators are 
LISP Control and Data Planes 
    Control Plane 
        Leverages a pull instead of a push 
        Pulls specific information instead of pushing all information 
        Has a mapping server that tracks all information 
    Data Plane 
        Tunneling mechanism 
            In SDA, we use VXLAN 
            In a regular LISP environment, it has its own tunneling mechanism 
                UDP destination port always 4341 
                Doesn’t use an ethernet header. L3 only 
                In LISP, there is an Instance ID header 
                    Similar to VNIDs in SDA 
        Have ITR (ingress tunneling router) and ETR (egress tunneling router) 
            If speaking generically, we call them XTRs. 
Sum:
    LISP control plane pulls information as needed 
    LISP Data Plane is L3-only. Supports Ipv4/IPv6.  
    LISP tunnels are formed between ITRs and ETRs (XTRs) 

LISP Roles and Terminology  
    EID  
        Typically a Subnet 
    RLOC 
        Typically a loopback 
    ITR / ETR 
        If traffic reverses itself, the roles would be swapped. 
    Map Server 
        Stores EID <--> RLOC mappings 
        ETR will tell it of new devices 
    Map Resolver 
        ITR will check in with the map resolver. 
        Map resolver will check in with the map server and then send response back to the ITR. 
        MS and MR can usually be combined on a single device for smaller environments. 
    Proxy 
        PITR / PETR  
            PXTR 
        Sits between a LISP and a non-LISP area 
        PETR does not send Map-register messages to the map server 
        Use it like a default route 
Sum: 
    Map server stores the EID-RLOC mappings 
    Map Resolver handles ITR requests 
    Proxy ITR/ETR sits at LISP boundary  
    Proxy ETRs do not send map registers 

LISP Operation 
    Map Register 
        Maps the EID-RLOC 
        Sent to the Map Server 
            Once received, MS will send a map-notify back 
    Map Request  
        Sent to the Map Resolver 
            Map Resolver queries the Map Server 
            Map Resolver forwards that request to the ETR 
                ETR sends a map-reply to the ITR 
                    This is how it works by default. You can set a flag to tell the Map server to directly respond, similar to in an SDA CP node 
                ITR will store that EID-RLOC mapping in its cache. Will store for 24 hours. 
    While all this is happening, the ITR will drop packets until it knows where to send them.(usually just the first packet) 
    Will encapsulate the traffic in a LISP tunnel and send it to the ETR RLOC. 
        ETR will de-encapsulate and forward on to destination 
    NMR will be sent when a map-request for a subnet that MS doesn't have hits it. 
        If you don’t have a PXTR, traffic will be dropped. 
        If you do, traffic will be sent to the PXTR. 
    PXTR will redistribute the LISP routes into the non-LISP domain (usually via BGP) 
        When non-LISP traffic hits the PXTR, it acts like an ITR. 


# VXLAN (non-SDA specific) [Virtual Extensible LAN] 
    Focused on solving problems when building out L2 networks 
    2 places in the network you build out L2 networks: 
        Campus environment 
            Usually have dist. Layer + access layer 
            Scanners, WAPs, etc. that need to be in the same subnet 
            Issues:  
                Creates a large broadcast domain 
                Spanning-tree 
                Having to use etherchannel / VSS pairs 
                Converting to L3 solves these problems, but can no longer share subnets between different switches 
        Data Center 
            Dealing with similar problems 
                VPC (virtual port channel) to try to deal with this 
                Keeping diff. customers / orgs separate  
                    Private VLANs difficult to scale out 
                    In a traditional network, a VLAN is the best way to segment the network 
                        Only have 4094 to work with 
    Solution? L2 tunneling 
        Convert everything to L3 links and tunnel your L2 traffic. 
 
Sum: 
    VXLAN allows for sharing subnets across L3 boundaries  
    4094 VLANs allowed per VXLAN segment  

VXLAN Operation 
    Encapsulates the ethernet header as part of the payload 
    VXLAN is data plane only 
        If you only deploy VXLAN in your network, it will use a flood and learn concept 
            Will send to every single VTEP. Once it recieves a response it will be more efficient moving forward. 
            Not ideal. 
            Want to deploy a control plane protocol to share info necessary to forward packet to the correct place 
                Many options: 
                    Most popular is Multi-Protocol BGP (MP-BGP) 
                    In SDA, we use LISP. 
                        Will share the  
    You take the entire L2 frame and add the VXLAN header 
        Called a MAC-in-IP/UDP encapsulation 
            UDP destination port is 4789 
    Failing to increase MTU will cause fragmentation  
        In an SDA solution, only need to increase MTU by 50B because the VLAN information can be stored in the VXLAN header 
    VXLAN header carries: 
        VNI (also called VNID) 
        VXLAN-GPO carries SGT 
    VTEP 
        VXLAN Tunneling End Point 
        Have a L3 interface used for establishing the tunnel 
        Have a LAN interface 
            The LAN interfaces face the traditional L2 network 

Sum:
    VXLAN preserves Ethernet header, uses MAC-in-IP/UDP encapsulation (UDP dest. 4789) 
    MTU should be increased by 50B or 54B 
    VXLAN needs a control plane (MP-BGP or LISP) 
VXLAN Use-cases 
    Service Providers can segment customers 
    Data centers can massively scale at L2 
    SDA in campus = no STP, improved scaling 


# Identify SDA Components  
    Will generally see Catalyst 9k line these days 
    ASIC 
        Need to have UADP (programmable ASIC) for SDA 
        Catalyst 9Ks have the latest version of UADP 
            Cat 9s built on x86 technology 
                Because they want to be able to support local applications on the switches 
            IOS-XE operating system 
                A form of Linux. IOS is running as an application on top of Linux. 
                Catalyst 9ks can also support wireless with some models. 
    Catalyst 9200 
        UADP-Mini 
        Replacement for 2k line of switches 
    Catalyst 9300 
        Replacement for the 3k line of switches 
        Stackable/ modular  
    Catalyst 9400 
        Replacement for the 4k line of switches 
        Chassis switches 
    Catalyst 9500 
        Sort of replacement for the 4500X, 6880-X switches 
        Fixed, core switches 
            High performance, lower port count, fixed configuration 
        Will see a lot in the core/distribution layer in network designs 
    Catalyst 9600 
        Replacement for the 6k line of switches 
        Chassis switches 
    Catalyst 9100 
        Replacement for the 1k, 2k, 3k (the Access point series) 
        Access points 
    Catalyst 9800 
        Replacement for the 3k, 5k, 8k (WLC) 
        WLC
Sum: 
    Cat 9Ks are THE product family for campus  
    Cat 9Ks leverage the programmable UADP, as well as support applications 
    Cat 9k products align with classic Cisco families  

ISRs and ASRs (for border / CP nodes. Neither can be Edge node) 
    Two diff. families of routers 
        ISR (FEATURES) 
            Built for branch deployment 
            Several diff. technologies in one platform 
                Unified Communications supported 
                    For phones. POTS/PRI lines 
                SW / wireless/ server modules 
            Focused more on features than performance 
        ASR (PERFORMANCE) 
            Build for service provider environments  
            No UC integration 
            No wireless/server modules on ASRs 
            Built for performance, not features. 
    Virtual router 
        CSR 
            A virtual version of an ASR 
            Can be in a cloud space or in our own virtual environment 
            1000v 
    Supported in SD Fabric. 
        ISR 
            4300 
            4400 
        ASR 
            1000 
        CSR 
            1000v 
                CP node only 
Sum: 
    Features = ISR, Performance = ASR 
    Routers can be a Fabric Border Node or a CP Node 
    CSR can only be a CP node 


## Other platforms supported in SDA 
    Fabric Edge Nodes 
        3650
        3850 
        4500E Chassis  
            Need SUP 8E or 9E 
    Fabric Border / CP Nodes 
        3850 
        6500 
            Need SUP 2T or 6T 
            None can support IPv6 
        6807 
            Need SUP 2T or 6T 
        6840-X 
        6880-X 
        Nexus 7700 
            Can only be an external fabric border node 
    Wireless 
        802.11ac 
            Wave 1 
                1700, 2700, 3700 
            Wave 2 
                1800, 2800, 1540, 1560, 4800 
        WLC 
            3504, 5520, 8540 
    Extended Nodes 
        To expend reachability into 'non-carpeted work spaces' 
            Warehouses, etc. 
        No Cat 9ks in these spaces 
        Will have an extended node with an upstream fabric edge node. 
        Info passed along via a trunk before being encapsulated into VXLAN 
        IE 3300, 3400, 4000, 5000 
        CDB - all supported 
        Catalyst 3560-CX 
            Quiet. Better suited for classrooms. 
Sum: 
    Certain legacy platforms are supported in SDA 
    Extended nodes fit where Cat9Ks do not 


## DNAC and ISE 
    DNAC responsible for automation / assurance 
    ISE responsible for policy place 
        Specifically SGTs 
    ISE hardware or software 
        If software: 
            ESXi, Redhat KVM, Hyper-V 
        If Hardware 
            SNS (Secure Network Server) 
                Size based on scale of your network 
        Typically deployed in a clustered infrastructure 
            Will deploy 4 nodes: 
                Policy Services Node (PSN) 
                    RADIUS / TACACS communication 
                Policy Administration Node (PAN) 
                    How we're going to log in and manage ISE 
                Managing and Troubleshooting Node (MAT) 
                PXG (PX Grid Controller) 
                    An open framework that will allow ISE to integrate with 3rd party applications and DNAC 
                    Requirement in order to have a fully functional SDA Fabric 
    DNAC hardware only (now has VM version)
        3 sizes: 
            Dependent on number of nodes on the network 
        Can be standalone appliance or cluster of 3 
Sum:
    Both can be physical or virtual, but DNAC is usually physical.


# Design a SDA Fabric 


Hardware and Software verification 
    Google cisco SDA compatibility matrix 
        Look at correct version of software ex. 2.2.2.6 
SDA Licensing 
    Licensing is attached to the network nodes themselves (Cat 9ks, etc.) 
        Hardware (base) license 
            Perpetual license 
                Network Essentials  
                Network Advantage 
                    Ex. VXLAN / LISP 
            These determine the feature set available to us 
        Software (DNA) license 
            Subscription 
                3,5,7 year options 
                How we interact with DNAC 
                    DNA Essentials  
                        Basic levels of automation/monitoring, etc. 
                    DNA Advantage 
                        Full automation package 
                       Gives access to encrypted traffic analytics 
                    DNA Premier  
                        The advantage package + security features (stealthwatch + ISE licenses) 
        Have to pair essentials with essentials, and advantage with advantage (for licenses) 
            Ex. Cant have network essentials with DNA advantage 
    What do you need for network nodes so they can participate in an SDA fabric? 
        You need network advantage for the hardware, and at least DNA Advantage for the software. 

Traditional 
    Lan base 
        Not enough for SDA Fabric 
    IP Base 
        Need to apply the DNA advantage licensing  
ISR/ASR 
    SD-WAN licensing also tied to DNA licensing  
Wireless 
    DNA licensing applied to a WAP directly instead of getting from a WLC.  
Sum: 
    SDA requires Network Advantage 
    SDA requires DNA Advantage 
    SDA requires ISE licensing  
    Work closely with Cisco when procuring licenses  

    
# Sizing the DNA Center and ISE Appliances 
    Google Cisco DNA center data sheet 
        Select appliance and hardware specifications 
            Looking at total number of devices / number of ports 
            Looking at physical dimensions of the appliance  
    Google cisco ISE ordering guide 
        Look at appliances section 
        For physical, google cisco secure network server data sheet 
Sum: 
    Size the DNA Center according to the size of the network - nodes, endpoints, ports, etc. 
    ISE can be virtual or physical, and sizing is based on the number of endpoints 

 
 #  Underlay Design 
    Usually L3 switches forming a L3 environment to facilitate overlay traffic 
        L3 highly recommended  
            One of the purposes of going with SDA is to get away from L2 communication 
                Ex. STP / Spreading VLANs everywhere 
            ISIS recommended 
                LAN automation will deploy ISIS 
                IF doing manually you can use other routing protocols 
            Need to make sure we're advertising the loopbacks of the edge devices into the underlay 
                We will have L3 interfaces on the edge devices that connect to L3 interfaces on the underlay devices 
        WLC is part of the underlay 
            Will hang off a Fabric Border Node (usually Internal Border Node) 
            Need to make sure our WAP are able to reach out to it and form that CAPWAP tunnel 
            Ensure that whatever network the WLC is on is advertised to the fabric so that the Fabric Edge nodes can route to it 
    Cisco recommends setting MTU to 9100 just to be safe. 
    When you connect Fabric edge nodes to the underlay, recommended to make Point to point links. 
        Recommended minimum of 10GB throughput  
    IGP Timers 
        Can tune for rapid convergence 
        Not recommended by Cisco in an SDA fabric  
            They want you to use Bi-direction Forwarding Detection (BFD) 
            Can pair with protocols to improve failure detection. Lower overhead than IGP hello messages. 
    SSO/ NSF 
        For failover. Mainly used with chassis switches 
Sum: 
    Recommended: L3 Underlay with IS-IS 
    L3 Links = Point-to-Point & MTU 9100 
    Leverage BFD for fast convergence, plus turn on NSF/SSO for any devices that need it.              


# Fabric Sites and Domains 

    Fabric Site 
        Edge devices, Border Nodes, CP Nodes (WLC controllers can be used between multiple fabric sites) 
        Independent of a physical location 
            Can have 1 fabric site that’s spread between multiple physical buildings on a campus 
        Can have multiple fabric sites managed by a single DNAC instance 
        For devices with a single switch: 
            Use FIAB (Fabric In A Box) 
                Has CP, Border and Edge node functionality  
    Fabric Domain 
        Multiple fabric sites managed by a single DNAC instance 
        Need to connect the sites together 
            Will use a transit network 
        Management Layer 
            DNA Center 
                2 primary ways to interface with it: 
                    Web GUI 
                        Anything the web GUI does, it leverages REST APIs to do. 
                        REST API on the backend. 
                    REST APIs 
                        Using applications like POSTMAN to accomplish the same thing the GUI uses 
            4 step workflows for managing our fabric: 
                Design 
                    Setting up our network hierarchy  
                        Will set up sites, buildings, floors. 
                        Will create network settings like IP pools, DNS, DHCP 
                        Image management 
                        Network profiles 
                            Allows you to standardize network configurations for diff. locations 
                Policy 
                    Macro segmentation 
                        Keeping separate networks separate from each other 
                            Ex. Two different organizations in a shared space being kept separate 
                                Or two diff. departments, etc. 
                        Rough equivalent to a VRF (called VNs in DNAC) 
                    Micro segmentation 
                Management Layer 
                    DNA Center 
                        2 primary ways to interface with it: 
                            Web GUI 
                                Anything the web GUI does, it leverages REST APIs to do. 
                                REST API on the backend. 
                            REST APIs 
                                Using applications like POSTMAN to accomplish the same thing the GUI uses 
                    4 step workflows for managing our fabric: 
                        Design 
                            Setting up our network hierarchy  
                                Will set up sites, buildings, floors. 
                                Will create network settings like IP pools, DNS, DHCP 
                                Image management 
                                Network profiles 
                                    Allows you to standardize network configurations for diff. locations 
                        Policy 
                            Macro segmentation 
                                Keeping separate networks separate from each other 
                                    Ex. Two different organizations in a shared space being kept separate 
                                        Or two diff. departments, etc. 
                                Rough equivalent to a VRF (called VNs in DNAC) 
                            Micro segmentation  
                                SGTs 
                                    Determining whether devices in the same network can speak to one another 
                        Provision
                            Define what your devices are and then discover them on the network 
                            Connect the policy you defined in the policy workflow and connect them to the sites you designed in the design workflow 
                                Connect policies -> sites 
                            Deploy services here 
                        Assurance 
                            Pulling data from network, and NDP is crunching that data and trying to figure out whats going on in the network 
                            Health Dashboard 
                                Overview of whats happening in your network 
                            Trends/insights 
                                Deeper look into whats happening in your network 
                            360 Views 
                                Gives an in-depth view about a particular client/network device 
                                    Useful for solving problems 
                    SGTs 
                        Determining whether devices in the same network can speak to one another 
                Provision 
                    Define what your devices are and then discover them on the network 
                    Connect the policy you defined in the policy workflow and connect them to the sites you designed in the design workflow 
                        Connect policies -> sites 
                    Deploy services here 
                Assurance
                    Pulling data from network, and NDP is crunching that data and trying to figure out whats going on in the network 
                    Health Dashboard 
                        Overview of whats happening in your network 
                    Trends/insights 
                        Deeper look into whats happening in your network 
                    360 Views 
                        Gives an in-depth view about a particular client/network device 
                            Useful for solving problems 
            Will connect to all fabric sites via External Border Nodes 
            2 ways to configure: 
                SDA Transit network 
                    Will maintain VXLAN config between sites 
                        Will continue to carry VNIDs/SGTs  
                    Will need Transit CP nodes 
                    Best option, but need some level of control over the transit network 
                        Recommended Dark Fiber 
                        Can use Metro Ethernet / VPLS / other L2 ISP technologies 
                IP-Based transit network (traditional) 
                    Wont be able to carry VXLAN headers between sites 
                        Lose the VNIDs / SGTs 
                        Will need to re-identify traffic as it re-enters a fabric site 

Sum: 
    Fabric Site = Unique set of Control Plane, Border, and Edge Nodes. 
    Fabric Domain = Many fabric sites managed by a DNA-C instance 
    Fabric Site can span multiple physical sites, and a physical site can contain multiple fabric sites. 
    Transit network: SDA vs IP  
