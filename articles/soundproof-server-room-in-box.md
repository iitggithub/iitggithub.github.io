## Soundproof Server Room In A Box

[](../images/16303466257_795e5f53ea_o.jpg)

This project formed part of a secondary office fitout. The customer leased an unused unit on the current floor and wanted to expand into the new space. After performing a cost-benefit analysis we agreed that it didn't make sense to to run the network cabling for each network port, all of the way to the server room on the other side of the building. This would involve an incredible amount of copper, labour and money. All of this was not acceptable so we needed to come up with another solution.

Instead, we agreed on using a couple of network switches stacked together and run a multicore fibre cable encased in corrugated PVC conduit connected to two, 1RU patch panels at each end. We also ran a couple of cat 5e cables back to the server room. The cat 5e cables are used to connect the console port on each switch and UPS to an Out Of Band (OOB) management device allowing network maintenance to be performed remotely because let's face it.. no one likes being dragged to the other side of the office or in from home to do something which could have been done remotely.

While this sounded good in theory, we didn't have anywhere to put a rack full of switches, patch panels, UPS, PDU and cooling that didn't look ugly. There was also the noise consideration as well since most managed switches are super loud!

The fit out consisted of a total of 140 odd network ports. This involved working quite closely with the customer and electricians to determine how many ports were needed, where the cable runs should be placed, how big of a service loop to provide, and making sure the bend radius for cables isn't too step going into the enclosure.

The SFP modules are connected to a stack of three, Cisco Catalyst 2960S 48 port switches (non POE) providing a small measure of redundancy.

[](../images/16463390716_f21312288f_o.jpg)

Different coloured power cabling is used to make it quick and easy to identify whether the device is powered by mains (Blue) or UPS (Red). The image above was taken before the project was finished and as such doesn't show the labeling that was added to each power cable. When your in the field and you've got an issue to deal with, it's nice having the information as fast as possible centred around where the problem usually is.

The switches are connected to a Zero U basic PDU which came with the rack although i tried really hard to find a switched IP PDU but couldn't find a single one which would fit. If someone out there makes switched IP PDUs, please make me a half height zero U PDU please.

There are three Cisco SPA 514G IP Phones around the office, a 2n Helios IP Uni Single Button intercom and two Cisco Aironet 2600i Access Points. Being POE devices, we installed POE injectors which are hidden inside the rack. The cabling for POE devices is standard cat 6 cabling but with yellow cabling it sticks out. Obviously, i would have preferred to use POE switches but the limited number of POE devices at the time of switch selection coupled with the additional cost put POE switches out of reach.

The PDU is powered by a an APC 750VA UPS giving the entire system a good few hours of battery backup. An APC UPS Network Management Card with Environmental Monitoring ensures UPS status as well as temperature and humidity can be monitored within the server rack.

[](../images/15866851244_35936287e1_o.jpg)

All of the data across the floor is connected to a number of Krone 24 port 2RU patch panels. These patch panels are pretty standard fare but they don't work well with the Neatpatch NP2K648 2RU Cable Management Kit. The reason is because they provide 2 rows of 12 ports per patch panel. Most of the installations i've seen of the Neatpatch cable management kit used a single row of 24 ports rather than two rows of 12 ports per patch panel. This ultimately made it a really tight squeeze and disconnecting cables can be a bit of a pain. If i ever do this again, I might try to find a krone compatible face frame that provides 1 row of 24 ports. If you're wondering why i went with Krone i'ts because it's an industry standard here and thus is going to be cheaper on labour.

The Neatpatch NP2K648 2RU Cable Management Kit came with 48, 2 foot cat 6 patch leads which I have to say is really good quality. Each of these patch leads were labelled at both ends using cable labels oriented so the label was easy to read. Have a look at the before and after pics to see what it can do.

| [](../images/16301948600_a513970738_o.jpg) | [](../images/16303466257_795e5f53ea_o.jpg) |

All of this goodness is contained within an APC NetShelter CX 18U Secure Soundproofed Server Room in a Box Enclosure. It's padded really well to cut down on noise and virations and build like a brick outdoor toilet. Airflow is routed so that it's very quiet from the outside as well. The overall build quality is definitely worth the cost as the wood look definiately makes it look like a regular office supply cabinet.

It remains locked at all times and when closed, no one really knows there's IT equipment whirring around inside the thing. Check out the video from APC for more information on the enclosure.

<iframe width="320" height="266" src="https://www.youtube.com/embed/lkSMQ4k55sM" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

Thanks for reading everyone!
