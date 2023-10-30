---
layout: post
title:  "Introducing CAKE - An Open Authorization Framework for E2EE web."
date:   2023-10-07 10:00:00 -0800
categories: e2e2 cake yaki
mermaid: true

---
This post is a bit different from all the other ones here. I'm introducing something that I believe to be novel to the world here. I think it has the potential to be transformative in bringing the end-to-end encrypted web into the mainstream. I also think I'm filled to the brim with delusions of grandeur. Anyway, here it goes...

## Historical Context
Computer networking and the use of TCP/IP packet switching within siloed corporations was prevalent in the 1950s and 60s. But only after the protocols established by ARPANet in the late 60s and 70s were opened up did companies and countries begin connecting and participating in a much larger world wide network leading to the birth of the modern internet.

On this computer network, documents and resources were available since the early ARPANet days, and different document linking and retrieval protocols, albeit closed and commercially licensed, were established by companies. But only after the concepts of Hyperlinks, HTML, and HTTP were published for free use by Tim Berners Lee at CERN in 1989, did the wide spread adoption of document linking take a hold leading to the creation of the World Wide Web in the 90s.

On this world wide web, companies and websites started offering APIs in addition to documents as programmatic means of accessing their services. This took the form of RPC, SOAP/WSDL, and REST endpoints, but the hurdle of individually authenticating into each of these services led to them being accessed in silos, similar to the early days of computer networking. Although, there were some popular services with a substantial base adopting their APIs viz-a-viz Facebook Apps, this led to walled gardens were being formed once again.

But after an egineer at Twitter (Blaine Cook) decided to open up his work on OAuth and the eventual release of it's 1.0 draft spec, interoperability on the web exploded and Web 2.0, or the web as we all know it today, was born. OAuth 2.0 spec was even better, and the OpenID initiative have provided a foundation for trust and allowed disparate services to easily interoperate on behalf of their users.

Today, there are competing visions for what the next evolution of the Web looks like. Meta/Facebook is pushing for a Web 3.0 that involves everybody being logged in and plugged into the Metaverse. The blockheads from the world of cryptocurrencies want Web3 to be a completely rebuilt network of distributed apps or dApps that run on top of the blockchain (ideally their company's blockchain powered by their coin), where every protocol that exists on Web2.0 is reinvented with immutability and zero trust being a core selling point. I am not interested in either of those visions of the future.

There is a third vision of what Web3 should be that is primarily driven by consumer demand for privacy and control over their content. One of the undesired outcomes of the same technologies that enabled the convenience of interoperability in Web2.0 was that users lost control over how their data was shared, and amassing user data became the primary fuel for consolidating power and control on the internet. But, the rise of end-to-end encrypted apps such as WhatsApp, Signal, ProtonMail, Skiff, etc. is signaling the dawn of a new era of a private and decentralized web, where the user's demands for privacy and control of their data is put front and center.

However, these apps by design lack the ability to interoperate on behalf of the user due to their end-to-end encrypted design, thus costing the users a bit of inconvenience in exchange for privacy.

## The Problem with E2EE

Before we discuss why End-to-End Encryption has problems, let's first look at how OAuth works to enable services to talk to each other. In OAuth, there is an Authentication (authn) provider that verifies the identity of a logged in user, and two services then agree on authorizations (authz) for accessing resources and taking actions on them as the user. Usually, the authz provider is one of the services involved, usually the one holding the resources, and authn can also be provided by the same service or more likely provided by one of the OpenID providers like Google, Facebook, Github, etc.

Let's take the simple login case for Oauth. Website example.com wants to allow users to login with their Google ID.

1. The user goes to example.com and clicks login.
2. Example.com redirects user to Google.com's oauth endpoint.
3. Google.com performs the login for the user, and mints an access token for the user, and redirects them back to example.com with the access token in its parameter.
4. Example.com can now verify that the access token is valid by verifying the signed contents using Google's public signing key, or by hitting a verification endpoint on Google's side.

The above covers the basic Authentication flow in OAuth.

Now if example.com needs access to the user's Google Calendar, then there is an additional Authrorization flow, where:
1. The user is redirected back to Google's authz endpoint along with the access token
2. User verifies with Google.com that they want to provide access to their calendar...
3. Google then mints a new token that both proves identity (authentication) and also enables access to Calendar (authz)
4. Then token is then used by Example.com for subsequent requests to the calendar API.


## CAKE - Counterparty Authorization via Key Exchange
