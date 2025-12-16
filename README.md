# Wide-Area-Network-CA-Submission-Ex1

Name: Nzeugang Ngoptat Joël Daniel 
Matricule : ICTU20241389

EXERCISE 1: Multi-Site WAN Extension with Redundant Paths

1. The goal is to extend the basic linear WAN into a triangular topology with redundancy:
	• HQ (n0) connects to Branch (n1) and DC (n2)
	• Branch (n1) connects to HQ (n0) and DC (n2)
	• DC (n2) connects to HQ (n0) and Branch (n1)
This provides two possible paths between any two sites.
SOLUTION :
Create Additional Links (Link 3: DC)
This snippet must be inserted after the existing link2Devices installation

Code:
// Link3 : n0-n2 (network 3)
NodeCintainer link3Node(n0,n2);
NetDeviceContainer link3Devices = p2p.Install(link3Nodes);

2. Static Routing Table Analysis
Each node must have two routes to the other network:
	• Primary (direct)
	• Backup (via third node)

HQ (n0) routes to DC
staticN0->AddNetworkRouteTo(”10.1.2.0”,”255.255.255.0”,”10.1.3.2”,1); // primary
staticN0->AddNetworkRouteTo(”10.1.2.0”,”255.255.255.0”,”10.1.1.2”,2); // backup via Branch

Explanation
	• If the direct link fails, traffic automatically follows the backup path.
•	 Symmetric routes must also be added on n1 and n2.

3. Path Failure Simulation
Interpretation
Simulate failure of the HQ–DC primary link at t = 4s and verify traffic continues via Branch.

Code:

Simulator::Schedule(Seconds(4.0),
  &Ipv4::SetDown,
  n0->GetObject<Ipv4>(),
  n0->GetObject<Ipv4>()->GetInterfaceForDevice(dev02.Get(0)));

Verification
	• Use NetAnim → traffic shifts path
	• Use FlowMonitor → packets still delivered

4. Scalability Analysis
Interpretation

Static routing does not scale.

For N nodes:
Static routes required = N × (N − 1)
Example:
	• 10 sites → 90 static routes
Scalable Solution
Use dynamic routing (OSPF).
NS-3 Helpers
Ipv4GlobalRoutingHelper::PopulateRoutingTables();

5. Business Continuity Justification
Static routing with redundancy provides:
	• Improved reliability (failover paths)
	• Load-balancing potential
	• Deterministic troubleshooting
	• Lower cost than dynamic protocols in small WANs
Name: Nzeugang Ngoptat Joël Daniel 
Matricule : ICTU20241389

EXERCISE 1: Multi-Site WAN Extension with Redundant Paths

1. The goal is to extend the basic linear WAN into a triangular topology with redundancy:
	• HQ (n0) connects to Branch (n1) and DC (n2)
	• Branch (n1) connects to HQ (n0) and DC (n2)
	• DC (n2) connects to HQ (n0) and Branch (n1)
This provides two possible paths between any two sites.
SOLUTION :
Create Additional Links (Link 3: DC)
This snippet must be inserted after the existing link2Devices installation

Code:
// Link3 : n0-n2 (network 3)
NodeCintainer link3Node(n0,n2);
NetDeviceContainer link3Devices = p2p.Install(link3Nodes);

2. Static Routing Table Analysis
Each node must have two routes to the other network:
	• Primary (direct)
	• Backup (via third node)

HQ (n0) routes to DC
staticN0->AddNetworkRouteTo(”10.1.2.0”,”255.255.255.0”,”10.1.3.2”,1); // primary
staticN0->AddNetworkRouteTo(”10.1.2.0”,”255.255.255.0”,”10.1.1.2”,2); // backup via Branch

Explanation
	• If the direct link fails, traffic automatically follows the backup path.
•	 Symmetric routes must also be added on n1 and n2.

3. Path Failure Simulation
Interpretation
Simulate failure of the HQ–DC primary link at t = 4s and verify traffic continues via Branch.

Code:

Simulator::Schedule(Seconds(4.0),
  &Ipv4::SetDown,
  n0->GetObject<Ipv4>(),
  n0->GetObject<Ipv4>()->GetInterfaceForDevice(dev02.Get(0)));

Verification
	• Use NetAnim → traffic shifts path
	• Use FlowMonitor → packets still delivered

4. Scalability Analysis
Interpretation

Static routing does not scale.

For N nodes:
Static routes required = N × (N − 1)
Example:
	• 10 sites → 90 static routes
Scalable Solution
Use dynamic routing (OSPF).
NS-3 Helpers
Ipv4GlobalRoutingHelper::PopulateRoutingTables();

5. Business Continuity Justification
Static routing with redundancy provides:
	• Improved reliability (failover paths)
	• Load-balancing potential
	• Deterministic troubleshooting
	• Lower cost than dynamic protocols in small WANs


#include "ns3/applications-module.h"
#include "ns3/core-module.h"
#include "ns3/internet-module.h"
#include "ns3/mobility-module.h"
#include "ns3/netanim-module.h"
#include "ns3/network-module.h"
#include "ns3/point-to-point-module.h"

using namespace ns3;

NS_LOG_COMPONENT_DEFINE("SimpleLinearWAN");

int
main(int argc, char* argv[])
{
    // Enable logging
    LogComponentEnable("UdpEchoClientApplication", LOG_LEVEL_INFO);
    LogComponentEnable("UdpEchoServerApplication", LOG_LEVEL_INFO);

    // Create three nodes: n0 (HQ/Client), n1 (Branch/Router), n2 (DC/Server)
    NodeContainer nodes;
    nodes.Create(3);

    Ptr<Node> n0 = nodes.Get(0); // HQ (Client)
    Ptr<Node> n1 = nodes.Get(1); // Branch (Router)
    Ptr<Node> n2 = nodes.Get(2); // DC (Server)

    // Create point-to-point links (5Mbps, 2ms)
    PointToPointHelper p2p;
    p2p.SetDeviceAttribute("DataRate", StringValue("5Mbps"));
    p2p.SetChannelAttribute("Delay", StringValue("2ms"));

    // Link 1: n0 <-> n1 (HQ-Branch, Network 1)
    NodeContainer link1Nodes(n0, n1);
    NetDeviceContainer link1Devices = p2p.Install(link1Nodes);

    // Link 2: n1 <-> n2 (Branch-DC, Network 2)
    NodeContainer link2Nodes(n1, n2);
    NetDeviceContainer link2Devices = p2p.Install(link2Nodes);

    // Install mobility model to keep nodes at fixed positions (Linear Layout)
    MobilityHelper mobility;
    mobility.SetMobilityModel("ns3::ConstantPositionMobilityModel");
    mobility.Install(nodes);

    // Set positions for clear visualization
    Ptr<MobilityModel> mob0 = n0->GetObject<MobilityModel>();
    Ptr<MobilityModel> mob1 = n1->GetObject<MobilityModel>();
    Ptr<MobilityModel> mob2 = n2->GetObject<MobilityModel>();

    mob0->SetPosition(Vector(0.0, 0.0, 0.0));   // HQ
    mob1->SetPosition(Vector(10.0, 0.0, 0.0));  // Branch/Router
    mob2->SetPosition(Vector(20.0, 0.0, 0.0));  // DC

    // Install Internet stack on all nodes
    InternetStackHelper stack;
    stack.Install(nodes);

    // Assign IP addresses to Network 1 (10.1.1.0/24) - n0 <-> n1
    Ipv4AddressHelper address1;
    address1.SetBase("10.1.1.0", "255.255.255.0");
    Ipv4InterfaceContainer interfaces1 = address1.Assign(link1Devices);
    // 10.1.1.1 (n0), 10.1.1.2 (n1)

    // Assign IP addresses to Network 2 (10.1.2.0/24) - n1 <-> n2
    Ipv4AddressHelper address2;
    address2.SetBase("10.1.2.0", "255.255.255.0");
    Ipv4InterfaceContainer interfaces2 = address2.Assign(link2Devices);
    // 10.1.2.1 (n1), 10.1.2.2 (n2)

    // *** Configure Static Routing ***

    // Enable IP forwarding on the router (n1)
    Ptr<Ipv4> ipv4Router = n1->GetObject<Ipv4>();
    ipv4Router->SetAttribute("IpForward", BooleanValue(true));

    Ipv4StaticRoutingHelper staticRoutingHelper;

    // Configure routing on n0 (HQ) to reach 10.1.2.0/24 via n1
    Ptr<Ipv4StaticRouting> staticRoutingN0 = staticRoutingHelper.GetStaticRouting(n0->GetObject<Ipv4>());
    staticRoutingN0->AddNetworkRouteTo(
        Ipv4Address("10.1.2.0"),   // Destination network (DC side)
        Ipv4Mask("255.255.255.0"), // Network mask
        interfaces1.GetAddress(1),   // Next hop: 10.1.1.2 (n1's interface on Network 1)
        1                          // Interface index (n0's interface on Network 1)
    );

    // Configure routing on n2 (DC) to reach 10.1.1.0/24 via n1
    Ptr<Ipv4StaticRouting> staticRoutingN2 = staticRoutingHelper.GetStaticRouting(n2->GetObject<Ipv4>());
    staticRoutingN2->AddNetworkRouteTo(
        Ipv4Address("10.1.1.0"),   // Destination network (HQ side)
        Ipv4Mask("255.255.255.0"), // Network mask
        interfaces2.GetAddress(0),   // Next hop: 10.1.2.1 (n1's interface on Network 2)
        1                          // Interface index (n2's interface on Network 2)
    );

    // Print routing tables for verification
    Ptr<OutputStreamWrapper> routingStream =
        Create<OutputStreamWrapper>("scratch/simple-linear-wan.routes", std::ios::out);
    staticRoutingHelper.PrintRoutingTableAllAt(Seconds(1.0), routingStream);

    std::cout << "\n=== Network Configuration ===\n";
    std::cout << "Node 0 (HQ): " << interfaces1.GetAddress(0) << "\n";
    std::cout << "Node 1 (Router) Interface 1: " << interfaces1.GetAddress(1) << "\n";
    std::cout << "Node 1 (Router) Interface 2: " << interfaces2.GetAddress(0) << "\n";
    std::cout << "Node 2 (DC): " << interfaces2.GetAddress(1) << "\n";
    std::cout << "=============================\n\n";

    // Create UDP Echo Server on n2 (DC)
    uint16_t port = 9;
    UdpEchoServerHelper echoServer(port);
    ApplicationContainer serverApps = echoServer.Install(n2);
    serverApps.Start(Seconds(1.0));
    serverApps.Stop(Seconds(10.0));

    // Create UDP Echo Client on n0 (HQ) targeting n2's IP address
    UdpEchoClientHelper echoClient(interfaces2.GetAddress(1), port); // Server's IP on Network 2
    echoClient.SetAttribute("MaxPackets", UintegerValue(3));
    echoClient.SetAttribute("Interval", TimeValue(Seconds(1.0)));
    echoClient.SetAttribute("PacketSize", UintegerValue(1024));

    ApplicationContainer clientApps = echoClient.Install(n0);
    clientApps.Start(Seconds(2.0));
    clientApps.Stop(Seconds(10.0));

    // *** NetAnim Configuration ***
    AnimationInterface anim("scratch/simple-linear-wan.xml");

    anim.UpdateNodeDescription(n0, "HQ\n10.1.1.1");
    anim.UpdateNodeDescription(n1, "Router\n10.1.1.2 | 10.1.2.1");
    anim.UpdateNodeDescription(n2, "DC\n10.1.2.2");

    anim.UpdateNodeColor(n0, 0, 255, 0);   // Green
    anim.UpdateNodeColor(n1, 255, 255, 0); // Yellow
    anim.UpdateNodeColor(n2, 0, 0, 255);   // Blue

    // Enable PCAP tracing on all devices
    p2p.EnablePcapAll("scratch/simple-linear-wan");

    // Run simulation
    Simulator::Stop(Seconds(11.0));
    Simulator::Run();
    Simulator::Destroy();

    std::cout << "\n=== Simulation Complete ===\n";
    std::cout << "Files saved to the 'scratch' directory.\n";

    return 0;
}
