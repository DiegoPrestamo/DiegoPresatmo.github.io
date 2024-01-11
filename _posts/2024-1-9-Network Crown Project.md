---
layout: post
title: Industrial Factory Mock Network 
tags: [Mathjax, Mathematic]
mathjax: true
---
## Consider a Highly Automated Roof Tile Plant
While I'm not fanatical about watches, I do enjoy them on a basic level. The way hundreds of different parts synchronize to achieve a single, straightforward goal of telling time reliably fascinates me. Similarly, a modern roof tile plant represents the same concept, albeit in a completely different context
<div align="center">
<iframe title="vimeo-player" src="https://player.vimeo.com/video/901713643?h=233a11118d" width="640" height="360" frameborder="0"    allowfullscreen></iframe>>
</div>

Creating a concrete roof tile is simple. Combine sand, cement, pigments, oxides, and water to form a moist sandy mixture. Spread it onto a mold, shape the top with the appropriate extrusion profile, and punch in nail holes. Then, bake it in an oven at 50 degrees Celsius with controlled humidity for a few hours. The result is a sturdy concrete roof tile like this:
<div align="center">
 <p>
<img src="https://ubp.mu/sites/default/files/tuile_vortex_canadienne.jpg" width="600">
  </p>
	A basic description of a modern roof tile plant 
</div>

## Scaling: When doing the 'thing' is no longer about the 'thing' 
As discussed, creating a rigid roof tile that can withstand high impact and adverse weather conditions is a solved problem. The complexity does not lie in making one quality roof tile, but making one million. Technology has allowed complex machinery to automate the roof tile making process. A long line of machinery works in congruency to yield the final product, but a lot can go wrong in the process.

## A simple network topology for the machinery 
We aim to monitor specific data and metrics from each of the four primary machines: the mixer, kiln, robots, and packer. To achieve this, we need to establish communication pathways from each machine to a router, which in turn connects to the CEO, COO,and plant managers' computers through an access point.
 <div align="center">
 <p>
<img src="https://github.com/DiegoPrestamo/DiegoPrestamo.github.io/blob/master/images/Screenshot%202024-01-09%20at%204.55.34%20AM.png?raw=true" width="1000">
  </p>
</div>
The red nodes represent the point to point connections that will communicate with the green and red router at the bottom of the plant. The green and red router will communicate with the green and blue router that is placed in the offices at the right side of the plant.That router is now our access point which will wirelessly communicate to the previous machines previously mentioned.

We will now create the network topology in ns-3. ns-3 is an open-source, education-oriented network simulation tool. You can use python or c++ to create and simulate network topology. I will break apart my script to make it more digestible.
### The following are standard header files that you must include for your script to run with the protocols that we wish to implement (point2point, and wifi are the ones that are more specific for this script). It is crucial that you include the final header file which is the NetAnim simulator. We will use NetAnim as our visual simulator. This will help us make way more sense out of our outputs.You must download NetAnim as it is not included in ns-3.
```bash 
#include "ns3/core-module.h"
#include "ns3/point-to-point-module.h"
#include "ns3/network-module.h"
#include "ns3/applications-module.h"
#include "ns3/mobility-module.h"
#include "ns3/internet-module.h"
#include "ns3/yans-wifi-helper.h"
#include "ns3/ssid.h"
#include "ns3/netanim-module.h"

using namespace ns3;

NS_LOG_COMPONENT_DEFINE ("ThirdScriptExample");
```
## The main function 
inside our main function will be the entirety of our topology and our initializing of NetAnim for visual simulation
``` bash
int
main (int argc, char *argv[])
{
  bool verbose = true;
  uint32_t nWifi = 3; // this will dictate how many wireless nodes you create 
  bool tracing = false;

  CommandLine cmd (__FILE__); // set up for command line and terminal interacting 
  cmd.AddValue ("nWifi", "Number of wifi STA devices", nWifi);
  cmd.AddValue ("verbose", "Tell echo applications to log if true", verbose);
  cmd.AddValue ("tracing", "Enable pcap tracing", tracing);

  cmd.Parse (argc,argv);
```
### These are some basic configurations for where our wifi nodes will be allowed to travel
``` bash
  // The underlying restriction of 18 is due to the grid position
  // allocator's configuration; the grid layout will exceed the
  // bounding box if more than 18 nodes are provided.

  if (nWifi > 18)
    {
      std::cout << "nWifi should be 18 or less; otherwise grid layout exceeds the bounding box" << std::endl;
      return 1;
    }

  if (verbose)
    {
      LogComponentEnable ("UdpEchoClientApplication", LOG_LEVEL_INFO);
      LogComponentEnable ("UdpEchoServerApplication", LOG_LEVEL_INFO);
    }

```
### Creating nodes for the machinery
As mentioned, we will have four nodes on the factory floor (mixer, kiln, stacking robots, and packer) connecting point-to-point to a router. The router will connect to the router at the offices. This means we have six total point-to-point nodes (four machines and two routers). An important question is how come we don't feed the point-to-point connections of each machine directly to the office router. The answer is that in reality, there are more than these four machines that are all in the PLC room. There is already a connection to the PLC room so connecting straight to the office router would actually double the amount of connections necessary. Node 1 will be our factory floor router and node 0 will be our office router. 

Not only will we create the nodes but we will also dictate the specific point-to-point connections that we wish to model.
``` bash
  NodeContainer p2pNodes;
  p2pNodes.Create (6);

  PointToPointHelper pointToPoint;
  pointToPoint.SetDeviceAttribute ("DataRate", StringValue ("5Mbps"));
  pointToPoint.SetChannelAttribute ("Delay", StringValue ("2ms"));

  NetDeviceContainer p2pDevices;
  p2pDevices = pointToPoint.Install (p2pNodes.Get(0), p2pNodes.Get(1));

  NetDeviceContainer p2pDevices1;
  p2pDevices1 = pointToPoint.Install (p2pNodes.Get(2), p2pNodes.Get(1));

  NetDeviceContainer p2pDevices2;
  p2pDevices2 = pointToPoint.Install (p2pNodes.Get(3), p2pNodes.Get(1));

  NetDeviceContainer p2pDevices3;
  p2pDevices3 = pointToPoint.Install (p2pNodes.Get(4), p2pNodes.Get(1));

  NetDeviceContainer p2pDevices4;
  p2pDevices4 = pointToPoint.Install (p2pNodes.Get(5), p2pNodes.Get(1));

```
### Creating and assigning the wifi nodes 
We are virutally doing the same process as before but now to the wifi nodes we will also assign the p2p node 0 to a new object called 'wifiApNode'. This will set our p2p node as a wifi access point, thereby linking our p2p network to our wifi network.
```bash
NodeContainer wifiStaNodes;
  wifiStaNodes.Create (nWifi);
  NodeContainer wifiApNode = p2pNodes.Get (0);

  YansWifiChannelHelper channel = YansWifiChannelHelper::Default ();
  YansWifiPhyHelper phy;
  phy.SetChannel (channel.Create ());

  WifiHelper wifi;
  wifi.SetRemoteStationManager ("ns3::AarfWifiManager");

  WifiMacHelper mac;
  Ssid ssid = Ssid ("ns-3-ssid");
  mac.SetType ("ns3::StaWifiMac",
               "Ssid", SsidValue (ssid),
               "ActiveProbing", BooleanValue (false));

  NetDeviceContainer staDevices;
  staDevices = wifi.Install (phy, mac, wifiStaNodes);

  mac.SetType ("ns3::ApWifiMac",
               "Ssid", SsidValue (ssid));

  NetDeviceContainer apDevices;
  apDevices = wifi.Install (phy, mac, wifiApNode);
```

### Keeping our wifi nodes fixed
We will now install our mobility model to our wifi nodes. There are a couple of different methods we can choose from but we want our wifi nodes to remain fixed in place since they will simulate stationary computers in the office.
```bash
 MobilityHelper mobility;

  mobility.SetPositionAllocator ("ns3::GridPositionAllocator",
                                 "MinX", DoubleValue (0.0),
                                 "MinY", DoubleValue (0.0),
                                 "DeltaX", DoubleValue (5.0),
                                 "DeltaY", DoubleValue (10.0),
                                 "GridWidth", UintegerValue (3),
                                 "LayoutType", StringValue ("RowFirst"));

  mobility.SetMobilityModel ("ns3::ConstantPositionMobilityModel");
  mobility.Install (wifiStaNodes);

  mobility.SetMobilityModel ("ns3::ConstantPositionMobilityModel");
  mobility.Install (wifiApNode);
```
### Installing internet protocol rules
We will install the internet rules onto all our devices and we will set the IP addresses with our subnet mask
```bash
 InternetStackHelper stack;
  stack.Install (p2pNodes.Get(1));
  stack.Install (p2pNodes.Get(2));
  stack.Install (p2pNodes.Get(3));
  stack.Install (p2pNodes.Get(4));
  stack.Install (p2pNodes.Get(5));
  stack.Install (wifiApNode);
  stack.Install (wifiStaNodes);

  Ipv4AddressHelper address;

  address.SetBase ("10.1.1.0", "255.255.255.0");
  Ipv4InterfaceContainer p2pInterfaces;
  p2pInterfaces = address.Assign (p2pDevices);

  address.SetBase ("10.1.2.0", "255.255.255.0");
  Ipv4InterfaceContainer p2pInterfaces1;
  p2pInterfaces1 = address.Assign (p2pDevices1);

  address.SetBase ("10.1.4.0", "255.255.255.0");
  Ipv4InterfaceContainer p2pInterfaces2;
  p2pInterfaces2 = address.Assign (p2pDevices2);

  address.SetBase ("10.1.5.0", "255.255.255.0");
  Ipv4InterfaceContainer p2pInterfaces3;
  p2pInterfaces3 = address.Assign (p2pDevices3);

  address.SetBase ("10.1.6.0", "255.255.255.0");
  Ipv4InterfaceContainer p2pInterfaces4;
  p2pInterfaces4 = address.Assign (p2pDevices4);

  address.SetBase ("10.1.3.0", "255.255.255.0");
  address.Assign (staDevices);
  address.Assign (apDevices);
```
### Creating ports for our server node
We need to add ports to our p2p node 1, which is our server. I assigned 9-13 arbitrarily. 
```bash
  UdpEchoServerHelper echoServer (9);
  UdpEchoServerHelper echoServer1 (10);
  UdpEchoServerHelper echoServer2 (11);
  UdpEchoServerHelper echoServer3 (12);
  UdpEchoServerHelper echoServer4 (13);
```
### Setting our simulation length and initiating echoes
We will set the length of our simulation to 14 seconds and we will echo specific clients to specific server ports
```bash
  ApplicationContainer serverApps = echoServer.Install (p2pNodes.Get (1));
  serverApps.Start (Seconds (1.0));
  serverApps.Stop (Seconds (14.0));

  UdpEchoClientHelper echoClient (p2pInterfaces.GetAddress (1), 9);
  echoClient.SetAttribute ("MaxPackets", UintegerValue (1));
  echoClient.SetAttribute ("Interval", TimeValue (Seconds (1.0)));
  echoClient.SetAttribute ("PacketSize", UintegerValue (1024));

  UdpEchoClientHelper echoClient1 (p2pInterfaces1.GetAddress (1), 10);
  echoClient1.SetAttribute ("MaxPackets", UintegerValue (1));
  echoClient1.SetAttribute ("Interval", TimeValue (Seconds (1.0)));
  echoClient1.SetAttribute ("PacketSize", UintegerValue (1024));

  UdpEchoClientHelper echoClient2 (p2pInterfaces2.GetAddress (1), 11);
  echoClient2.SetAttribute ("MaxPackets", UintegerValue (1));
  echoClient2.SetAttribute ("Interval", TimeValue (Seconds (1.0)));
  echoClient2.SetAttribute ("PacketSize", UintegerValue (1024));

  UdpEchoClientHelper echoClient3 (p2pInterfaces3.GetAddress (1), 12);
  echoClient3.SetAttribute ("MaxPackets", UintegerValue (1));
  echoClient3.SetAttribute ("Interval", TimeValue (Seconds (1.0)));
  echoClient3.SetAttribute ("PacketSize", UintegerValue (1024));

  UdpEchoClientHelper echoClient4 (p2pInterfaces4.GetAddress (1), 13);
  echoClient4.SetAttribute ("MaxPackets", UintegerValue (1));
  echoClient4.SetAttribute ("Interval", TimeValue (Seconds (1.0)));
  echoClient4.SetAttribute ("PacketSize", UintegerValue (1024));
```
### Setting our echoes to specific times
We will be very deliberate about our echoes and send them one after another all the way through our 14 seconds of simulation.
```bash
  ApplicationContainer clientApps9 =
  echoClient.Install (wifiStaNodes.Get (nWifi - 1));
  clientApps9.Start (Seconds (1.0));
  clientApps9.Stop (Seconds (14.0));

  ApplicationContainer clientApps = echoClient.Install (p2pNodes.Get (0));
  clientApps.Start (Seconds (2.0)); 
  clientApps.Stop (Seconds (6.0));

  ApplicationContainer clientApps1 = echoClient1.Install (p2pNodes.Get (2));
  clientApps1.Start (Seconds (6.0));
  clientApps1.Stop (Seconds (8.0));

  ApplicationContainer clientApps2 = echoClient2.Install (p2pNodes.Get (3));
  clientApps2.Start (Seconds (8.0));
  clientApps2.Stop (Seconds (10.0));

  ApplicationContainer clientApps3 = echoClient3.Install (p2pNodes.Get (4));
  clientApps3.Start (Seconds (10.0));
  clientApps3.Stop (Seconds (12.0));

  ApplicationContainer clientApps4 = echoClient4.Install (p2pNodes.Get (5));
  clientApps4.Start (Seconds (12.0));
  clientApps4.Stop (Seconds (14.0));

  Ipv4GlobalRoutingHelper::PopulateRoutingTables ();

  Simulator::Stop (Seconds (14.0));
  if (tracing == true)
    {
      pointToPoint.EnablePcapAll ("third");
      phy.EnablePcap ("third", apDevices.Get (0));
    }
```
### Positioning our nodes in space
We will position our nodes exactly where we put them in our cad model from earlier. I made them an order of magnitude smaller (300 -->30.0) because NetAnim was not happy with the larger magnitude. However, the proportion is the same and we can visually confirm that they are correctly positioned when we compare them to where we have them in our AutoCad model.
```bash
AnimationInterface anim ("wireless-Anim-file.xml");

  anim.SetConstantPosition (p2pNodes.Get (1), 15.95, 19.15);
  anim.SetConstantPosition (p2pNodes.Get (2), 25.827, 18.024);
  anim.SetConstantPosition (p2pNodes.Get (3), 31.1, 16.65);
  anim.SetConstantPosition (p2pNodes.Get (4), 11.35, 13.9);
  anim.SetConstantPosition (p2pNodes.Get (5), 21.95, 5);

  anim.SetConstantPosition (wifiApNode.Get (0), 35.95, 11.65);
  anim.SetConstantPosition (wifiStaNodes.Get (0), 36.95, 16.15);
  anim.SetConstantPosition (wifiStaNodes.Get (1), 36.95, 17.4);
  anim.SetConstantPosition (wifiStaNodes.Get (2), 36.95, 11.95);

  anim.UpdateNodeColor (wifiApNode.Get (0), 0, 255, 120); // RGB format
  anim.UpdateNodeColor (p2pNodes.Get (1), 0, 255, 120);
  anim.UpdateNodeColor (wifiStaNodes.Get (0), 0, 0, 255);
  anim.UpdateNodeColor (wifiStaNodes.Get (1), 0, 0, 255);
  anim.UpdateNodeColor (wifiStaNodes.Get (2), 0, 0, 255);

  Simulator::Run ();
  Simulator::Destroy ();
  return 0;
}
```
### Run your model
You shall now be able to run your network model using the ./waf file. This takes a bit of practice to get everything in the right directories but ns-3 has documentation to help.
```bash
At time +1s client sent 1024 bytes to 10.1.1.2 port 9
At time +1.00586s server received 1024 bytes from 10.1.3.3 port 49153
At time +1.00586s server sent 1024 bytes to 10.1.3.3 port 49153
At time +1.02055s client received 1024 bytes from 10.1.1.2 port 9
At time +2s client sent 1024 bytes to 10.1.1.2 port 9
At time +2.00369s server received 1024 bytes from 10.1.1.1 port 49153
At time +2.00369s server sent 1024 bytes to 10.1.1.1 port 49153
At time +2.00737s client received 1024 bytes from 10.1.1.2 port 9
At time +6s client sent 1024 bytes to 10.1.2.2 port 10
At time +8s client sent 1024 bytes to 10.1.4.2 port 11
At time +10s client sent 1024 bytes to 10.1.5.2 port 12
At time +12s client sent 1024 bytes to 10.1.6.2 port 13
```



