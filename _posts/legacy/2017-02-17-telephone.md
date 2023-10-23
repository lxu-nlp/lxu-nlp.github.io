---
layout: post
title: "History of Telephone: Analog to SONET to VoIP"
date: 2017-02-17
categories: [Others]
tags: [telecom, legacy]
---

I am not majoring in Telecom and I don’t really know much about it,  but I just learned some interesting stuff about telephone and voice product, so I did a review here. Pretty basic knowledge and hope it would help a little to understand voice transmission.

## Analog Signals
Before 1960s, all telephones used analog signals. The signals are continuous, making every call has to have a dedicated copper wire stretching from one end of the call to the other for the duration of the call. When a call is made between two parties, the connection of the analog signals between two sites, or the circuit, is continuously opened. Note that this is very inefficient and causing telephone calls expensive, since one wire or one pair of wires can only serve one call at a time. Let’s show this by one example:

If you were in New York and you wanted to call Los Angeles, the switches between New York and Los Angeles would connect pieces of copper wire all the way across the United States. You would use all those pieces of wire just for your call for the full 10 minutes. You paid a lot for the call, because you actually owned a 3,000-mile-long copper wire for 10 minutes. (example from <https://computer.howstuffworks.com/ip-telephony2.htm>)

Not only it is inefficient, but a more realistic problem started to show: how much copper we would need, in order to enable calls between any two people? If every two people needs a wire between them, that would be an exponential number for the total copper wires needed. The total copper on Earth is not even enough to build all these wires. So something had to be done.

## Digitization
Well, nowadays most telephone network uses digital signals. Some analog telephones still exist in some houses, but that is usually the “last mile” wire that uses analog signals. Once the signals go out from the consumer’s house and get to some switches, all analog signals will be digitized and transmitted for long distance.

So how does the digital signals work in a telephone? The telephone will sample the voice 8,000 times per second, and each sample generate 8 bits data. So each second we have 8000*8 = 64Kb voice data. That is 64/8 = 8KB voice data per second (1Byte = 8 bits) for each direction. The voice now can be sent in the form of digital signals.

Note that digital signals still need the circuit continuous open during the phone call, since the voice is sampled at a fixed rate (8,000 times per second), and transmitted at a fixed rate of data (8KB every second) without interruption.

## Internet Access
Both DSL (Digital Subscriber Line) and DS1 (or called T1 in US and Japan) are two kinds of traditional Internet services we can get in our houses that utilize the twisted pair of copper telephone lines. However, though they use the same transmission media (telephone lines), they have some major differences as they use different protocols and technologies:

**DSL**: shared Internet access, speed not guaranteed; typically asymmetrical, meaning uploading and downloading speeds are different; not available if too far from the telephone company’s central office; cheaper than DS1(T1), main market is towards residents; only used for Internet access.

**DS1(T1)**: synchronous, speed always guaranteed and reliable; symmetrical, same uploading and downloading speeds; available everywhere as long as there is telephone access; expensive, main market is towards business; can be used for carrying both phone calls and Internet access.

A good article I found that explains the differences:

<https://www.t1rex.com/dslt1.html>

Let’s only focus on DS1(T1) in the following section, as it is used in both the telephone world and Internet world.

## TDM and SONET
So how are the digital signals superior to the analog signals, as they all use circuit-switching, which means the circuit is established and always open during the phone call?

Well, with digital signals, we can use technique called “Time-Devision Multiplexing (TDM)” over DS1(T1) lines. Wikipedia has a pretty good explanation:

In circuit-switched networks, such as the public switched telephone network (PSTN), it is desirable to transmit multiple subscriber calls over the same transmission medium to effectively utilize the bandwidth of the medium. TDM allows transmitting and receiving telephone switches to create channels (tributaries) within a transmission stream. A standard DS0 voice signal has a data bit rate of 64 kbit/s. A TDM circuit runs at a much higher signal bandwidth, permitting the bandwidth to be divided into time frames (time slots) for each voice signal which is multiplexed onto the line by the transmitter. If the TDM frame consists of n voice frames, the line bandwidth is n*64 kbit/s. (<https://en.wikipedia.org/wiki/Time-division_multiplexing>)

Simply said, we call the minimum measurement of voice line as DS0. As we calculate before, DS0 can transmit at the data rate 8KB/s, just as one phone call needs. We say that DS0 has a bandwidth of 8KBps. By using TDM, we can make a DS1 line (or called T1 line in US and Japan), that multiplexes or aggregates 24 of these DS0 channels into a DS1 with a total bandwidth of 1.536 Mbps. Inside the DS1 line frame, it has 24 time slots for 24 DS0 signals.

So back to the question, digital signals are superior to analog signals, because digital signals can be multiplexed, so that we can transmit multiple phone calls over one line. By using TDM, we can increase the bandwidth, transmitting more data during the same time span. There are lines with more bandwidth being created and used. DS3 is the equivalent of 28 DS1s. An **OC3** (or called **SONET** in US) fiber optic carrier can deliver 3 DS3 services or 84 DS1 services or 2,016 DS0 channels, with a bandwidth of 155.520 Mbps.

Hence, digital telephone network with high bandwidth started to grow all across the country. If you think about an OC3 (or SONET) can carry over 2,016 phone calls at the same time, you may understand how much more efficient the telephone is by applying digitization and TDM technique. With the SONET ring built in most cities, telephone services were finally available to almost every family. In addition, the DS1(T1) and SONET are also used for highly reliable Internet. Till now, many buildings still use DS1 (T1) as Internet access.

Modern fiber optic network lines also use techniques like “Wavelength-Division Multiplexing (WDM)” to increase more bandwidth. Almost all long hauls for long distance transmission use DWDM (Dense Wavelength-Division Multiplexing) for increasing bandwidth significantly. The concept is we can put multiple light (channels of signals) of different wavelength into one fiber.

## SONET vs Ethernet
SONET and Ethernet are two types of network technologies with different protocols that are used in metro areas. In the Internet world (not the traditional telephone world), Ethernet as a newer product are replacing the old technologies of SONET, even in the Enterprise market that was dominated by SONET before. Whether SONET and Ethernet, they all have its unique advantages over the other. Here is an article I found that has pretty clear details about why Ethernet is getting more popular:

<https://www.tccomm.com/Literature/Literature/Ethernet-Network-White-Papers/SONET-to-Ethernet-comparison>

## VoIP: Circuit-Switching to Packet-Switching
After digitization, what could be the next improvement? Well, let’s say we have a phone call for 10 minutes. During the 10 minutes which is 600 seconds, how long do we actually speak totally? Maybe each person only uses 200 seconds for speaking, and the rest of the time is just for waiting, listening or thinking. Plus, while we’re talking, there are short intervals between each word and each sentence, making only 100 seconds out of the 200 seconds are for the actual speaking.

However, since the phone is using circuit-switching, all the data of 600 seconds are being transmitted continuously, even though only 1/6 of the data are actually “valid”, and 5/6 of the transmitted data with no actual information are just wasted. So there could still be a big improvement if we can just transmit the “useful” data in a phone call.

Here comes the packet-switching. “While circuit-switching keeps the connection open and constant, packet-switching opens a brief connection — just long enough to send a small chunk of data, called a packet, from one system to another.” “Instead of routing the data over a dedicated line, the data packets flow through a chaotic network along thousands of possible paths.” (<https://computer.howstuffworks.com/ip-telephony3.htm>) We make the voice data as a series of IP packets transmitted over the whole internet network, just like other normal internet traffic (hence, voice over IP). The data being transmitted only contains the “useful” information, and we try to send the noisy data such as silent intervals as little as possible.

The concept is pretty clear and the advantages of VoIP are huge. First, the voice transmission is much more efficient now, since we only transmit the valid data and abandon the noisy data. Second, while there isn’t a dedicated circuit that opens continuously between the two parties, each party can accept data from others during the phone call, which benefits phone calls over computers. Third, since the transmission makes use of Internet Protocol, the transmission path can be dynamically selected during the phone call. When a path is getting crowded suddenly, the voice can choose another alternative path that has less traffic to guarantee the data will arrive on time. Fourth, theoretically we can apply VoIP anywhere as long as there is Internet access, weather using copper lines or optical fiber.

Nowadays most phone service companies like AT&T and others already applied VoIP in their back-end. Routing phone calls over the Internet makes the operation cost decreased significantly. One obvious benefit for end users is that we don’t need to pay long-distance calling fee anymore.

Of course the voice packets are not treated exactly the same as other data packets. One big difference is that voice packets have much higher routing priority than other data packets. SIP (Session Initiation Protocol) is typically the most widely used protocol to support VoIP, which is usually over UDP in the OSI Transport Layer, as voice needs real-time delivery. There are lots of technical details behind the VoIP, such as how to control the latency, and especially the jitter of the voice packets. But I guess they are out of my areas and I will just end it here…

Hope this is helpful!
