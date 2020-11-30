---
layout: post
section-type: post
title: "Network Engineer Whiteboard questions (My experience)"
category: 'Networking'
tags: [ 'networking', 'interviews' ]
---

## How i got the interview
i went ahead and clicked on the `Try Linkedin Premium for 30 days` button and things started happening, more visibility, more messages from recruiters and more data about job postings showed up. and also their learning platform was accessible which had great courses.

the recruiter simply reached out through linkedin messages and said they wanted to talk about a possible network engineering position (the actual job title was `Cloud Security Engineer` but all of our technical discussions and questions were about networking and routing/switching).

## first interview
the first interview is a bit hazy in my memory, but it was mainly about introducing myself and sharing my journy and passion for technology.
after introductions were made the interviewer asked me some technical questions to confirm my claims (but they weren't crazy hard)

one question that i remember was what's tcp and how does it work? i explained that it a layer 4 protocol and that it's basically a multiplexer layer for layer 3 (IP address).

## whiteboard interview
the interviewer drew a home network on the whiteboard without any labels and started asking questions to fill out the details on the diagram.

some questions were to general or open ended but the interviewer always helped narrowing down the topic they are trying to ask about.

Q) what needs to happen in order for this PC to get a web page from the webserver on the internet.
A) the first thing we need is to get an IP address, which means we need a DHCP server somewhere on this network.

Q) yes, so let say the firewall is also configured to be a DHCP servre. How does getting an IP address from the DHCP server work.
A) DORA message exchange, i remebered the discover and offer messages but forgot the other two

Q) ok let's say we've successfully got an IP address, what IP most likely that'd be?
A) 192.168.1.2/24, GW:192.168.1.1, DNS: 8.8.8.8

Q) what's special about this IP ?
A) class C (/24) private.

Q) what does private mean?
A) not reachable or routable throught the internet

Q) how is the network mask used?
A) as basic as this question is it was quite confusing, i answered it's used to define the range of addresses that can be allocated to hosts. but for some reason that wasn;t quite satisfying for the interviewer.

Q) what need to happen next ?
A) we need to translate the hostname `google.com` to an IP address, and for that we use DNS

Q) how do we get that IP address from the DNS?
A) we send a DNS request (we went through the packet format sent)

Q) ok once we got that, how do we get the webpage from the webserver?
A) send an HTTP request, we talked about HTTP response codes

Q) before sending the HTTP request something needs to happen, what is it?
   i was completely lost at this point and didn't realize they meant to talk about the TCP handshake process.
   once i realized that's the topic to talk about, i started explaining the handshake process (my memory wasn't very on point) they sat down and said that famous sentence :) do you have any questions for me? and that's when i knew that interview was over (not the best interview i had)

but overall it was a good learning experience as this was my first whiteboard interview in a while, however i will need to polish my knowledge and refresh my memory on some subjects.

good luck with your interviews :)
