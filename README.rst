*******************
Where's the Party?
*******************

Where's the Party is a scalable, censorship-resistant mirror network for the web. It aims to make mirroring more accessible for both clients and hosts, allowing several orders of magnitude greater participation. WTP is different from previous mirroring efforts because it works entirely with software users already have - *no* additional packages need to be installed for either clients or hosts.

WTP consists of software tools for building mirror networks and a website for matching content with volunteer hosts. It operates by using JavaScript in the browser to reconstruct the namespace that is a website from content hosted on multiple servers.

:author: Peter Fein
:email: pete@wearpants.org
:source: http://github.com/pfein/wherestheparty

===============
Problems Solved
===============
Where's the Party enables the distribution of static HTML content to users on censored networks. It aims to be resistant against several forms of censorship: host/IP blocking, content-aware filtering (DPI) and legal takedowns (DMCA).

The extremely low rates of uptake for browser plugins and other installable software is well known. At the height of the Egypt crisis, usage of `TOR skyrocketed`_ - to just over 2000 users, out of a population of 83 million. Peer-to-peer solutions such as BitTorrent often fail the "grandmother test", restricting access to the technologically savvy. For clients, WTP leverages the familiar web browser that users already have installed. It will support all browsers back through IE6 - a particular benefit in China, where `IE6 usage rates`_ remain as high as 35%.

Working on the `streisand.me`_ mirror project, we observed similar problems for volunteers who wish to host a mirror. Apache configuration requires a fair degree of technological ability, and is simply not available for many popular hosting platforms (S3, Dropbox). The sheer size of content, often including videos, is an obstacle to hosting. For example, the `HBGary leaks site`_ was 9 GB in size. While not enormous, a tarball this large will take several hours to download and may exceed the capacity of many hosting plans. WTP makes mirroring easier by serving content from any directory on a dumb web host. An individual host may mirror only a portion of the total content.

=====================
Problems Unsolved
=====================
Some features of an ideal mirroring system need to be sacrificed for greater accessibility. There is no support for dynamic content such as a blog or CMS (though this may be mitigated by static snapshots). WTP does not provide anonymity or privacy, leaving clients who surf mirrors vulnerable to detection. Such concerns are best left to special-purpose tools like TOR & VPNs.

WTP does not specify how to securely find an initial entry node. Rather, users find the network the same way as any other website - from a link, a URI written on a sidewalk, or a message from $DEITY (navigation *between* nodes is cryptographically verified). I call this approach a "rumor net" - as a user, you find the party via out-of-band channels. Once there, you can safely stay at the party and tell others where it is.

Finally WTP may result in a slightly slower browsing experience, as resources may need to be loaded multiple times. This is faster than completely unavailable, and therefore acceptable.

========
Censors
========
The task of a censor is complicated by WTP. Takedowns and blocking are more difficult because of the large number of mirrors, in multiple jurisdictions. The colocation of censorable and inoffensive content on the same server may make coarse blocking or domain seizure less appealing. The use of cryptographic signing makes hijacking a WTP session impossible.

Since the list of mirrors is public, it is relatively easy to map the extent of the network (once the censor understands how the system operates). Hopefully, new mirrors can be added quickly enough to stay ahead. Censors may run mirrors themselves as honeypots, to collect IPs and other user data.

=============
How it Works
=============

Basic Operation
++++++++++++++++
The content of a website is copied across the mirror network. An individual node may host only a fraction of the total pages; some resources, such as CSS or JS may be present on all mirrors. Each mirror has a list of the root URIs for some (not all) of the other nodes, and the public half of a keypair (the "verification keys"). A cryptographic signature is stored next to each resource (index.html.sig).

A browser connects to the network via an out-of-band link. All pages include JavaScript which intercepts clicks and resources loads (images, etc.) for URIs with the current host. Resources from other hosts are not modified. On a click, the JavaScript checks the current host for the resource. If found, the associated signature is checked with the verification public key. If the check passes, the resource is loaded.

Several errors are possible:

 1. the resource may not exist on the current server (404)
 2. the current server timeouts. This can occur if the user leaves a browser window open and the node is taken down or blocked.
 3. the verification signature is invalid
 
For (1) or (2), the JavaScript uses a cross-domain request (XDM) and walks through the list of mirrors to find the target resource. If found, it verifies the signature of the resource *and* the signature of the WTP JavaScript on the remote mirror, as well as that the remote public key is the same. If these tests pass, the browser is redirected to the remote resource. If none of the nodes in the current mirror list has the resource, their mirror lists are consulted by the same process.

For (3), the user is alerted via popup, and given the option to load the resource from the current host or from a different node. XXX user choice here is lame

Obfuscation
+++++++++++
Obfuscation is introduced to thwart content-aware filtering at the network level. A second keypair (the "obfuscation keys") are added; the private key is stored on the mirror. Resources are non-semantically mutated in a unique way for each node (by introducing spaces between HTML tags, say). By encrypting a mutated resource, the raw bytes transmitted by a particular node will be totally different than any other. The client JavaScript loads the resource and replaces the page body with the decrypted version. Similar results may be achieved for binary formats (images, video) by flipping a single bit. Note that this encryption provides only obfuscation, not security (as the publicly-accessible mirror has the private key),

The JavaScript itself cannot be so encrypted, as it would need to decrypt itself. Instead, existing JS obfuscaters can be used, ideally ones which take a user-provided seed.

The filenames of common resources may also be varied, including those of the WTP JavaScript itself. Such files could be found by examining the index.html at a mirror root.

Proof of Authorship
++++++++++++++++++++++
Proof of authorship may be added by signing the verification key with a known, identified keypair (the "author keys"). JavaScript cold be used to fetch the author's public key from the PGP keyservers (using XDM) and then verify the signature of the verification key. While anonymity may be maintained by using a newly-created email & keypair, this step is entirely optional.

Health Checks
++++++++++++++
A standalone application could be used to spider a WTP mirror network and report on down nodes, signature errors, resource replication statistics, and so on. Similarly, client JavaScript could optionally report back to a web service specified by the party creator about down nodes and signature errors. 

Versioning
+++++++++++
As publishing updates to a distributed mirror network may take some time, WTP can include a version number for the party as a whole (a la Subversion's revision numbers). JavaScript can detect if a resource on a remote mirror is older than the current generation. It can then look for newer copies on other hosts, alerting the user that content may be out of date if necessary.

========================
wherestheparty.net
========================
wherestheparty.net (WTPnet) is a website to facilitate the matching of content with volunteer hosts. Volunteers sign up, specify how much and what kind of content they want to host, and provide login credentials (rsync, (s)ftp, S3, etc.) for a webserver. WTPnet will periodically scan `The Pirate Bay`_ and other BitTorrent search engines for specially tagged content (`partywithme`). Such torrents will be automatically downloaded, their content extracted and then transformed to add the necessary JavaScript, keys and signatures. The resultant party will be divided into appropriately-sized portions and  uploaded to volunteer hosts. Mirror lists on existing hosts will be updated periodically.

As the website is highly likely to be blocked, its use is entirely optional. However, as content creators need access to BitTorrent, not the site itself, this problem is somewhat mitigated.

Updates
+++++++
By signing the content tarball using author keys (described in `Proof of Authorship`_), the party creator gains the ability to update content in the future. To update a party, the author creates an update tarball with new/changed files and a manifest of deletions. This file is signed using the author private key, and the tarball and signature are served through BitTorrent as described above. WTPnet can download this new torrent, verify the signature and update the mirrors as necessary. Note that the public author key can be included in the torrent and need not be uploaded to an external keyserver.

Community
+++++++++++++++++
Several difficulties arise from a fully-automated mirroring system. There may be more content than hosting space available. Some content may expose mirror owners to local legal or political liability. The existence of free storage is an attractive target for spammers and trolls.

These problems can be mitigated with the use of collaborative decision making systems (a la `Reddit`_). A small subset of content from a potential party will be unpacked and served to browsers (either by direct hosting or on nodes willing to host unreviewed content).  Users can help provide a brief description and other metadata (political relevance, legal risks), as well as flag potential parties as spam or inappropriate. They will be able to vote on whether that content should be mirrored on WTPnet. Additional weight will be given to the votes of users who:

 * provide more mirror space (logarithmic, so that small mirrors are not overwhelmed)
 * have a longer history of mirroring (again logarithmic, so that new users are not automatically outvoted)
 * mirror content on under served countries, languages and topics  
 * mirror under-replicated content (see below)

The actual content mirrored on a particular node is left up to that node's owner. Volunteers may allocate space to parties selected by the community, subject to constraints they specify (i.e., "exclude content that is legally risky in my jurisdiction"). Alternately, they may prefer individual parties, authors, topics or countries. Extra voting weight will be given to volunteers who mirror scarce (i.e., under-replicated) content.

System administrators may set reasonable limits on the number of mirrors for popular parties. For example, the world probably doesn't need any more `WikiLeaks mirrors`_ at present. 

Other Content & Services
++++++++++++++++++++++++
WTPnet will provide a list of known parties, instructions on how to use the software and links to information about communications safety. It could run a spider as described in `Health Checks`_ and use the reports to improve the redundancy of the networks it manages. Note that WTPnet will *not* host parties itself, as this would significantly increase its exposure to legal and technological threats.

========================
Implementation
========================
Core JavaScript logic will be written using `Coffeescript`_, a friendlier dialect of JavaScript. Cross-domain requests will use `EasyXDM`_.  Cryptography will use the  `Stanford JavaScript Crypto Library`_. The use of jQuery will be avoided to allow its use by content without conflicts.

Python will be used to transform content, using `lxml`_. Key generation and signing will be done with `Pycrypto`_ or `M2Crypto`_. A health check spider could be written with `scrapy`_. Testing can use `selenium`_ and/or `Browsershots`_.

For WTPnet, the main site could be written in `Django`_ or another of the many Python web frameworks. Screen scrapers for The Pirate Bay would be written with standard library modules, lxml or scrapy. The orignal `BitTorrent`_ client could be used for downloads. For updloading to mirrors, there is `ftplib`_ for FTP, `paramiko`_ for ssh/sftp, `pysync`_ for rsync. (several alternatives available for all of these, including wrappers around commandline utilities). `scipy`_ and `NLTK`_ can be used for automated language and topic identification, and spam filtering. `Google Translate`_ links will be present on sample pages.

======================
Open Questions/Issues
======================

 * Which JS obfuscater to use? Don't know if any support the ability to obfuscate in a variable way.
 * Is there a better domain than wherestheparty.net? All the good ones are taken.
 * Are there other ways of getting content into WTPnet? Searching for tags/links/named files on Google, file hosting services or links on pastebins perhaps?
 * Elliptic curve DSA would be preferable to RSA, but SJCL doesn't currently support it.
 * WTPnet could generate tarballs on demand for users who do not want to supply login credentials. This makes updating their mirror lists more difficult, but maybe a small mirror-list-update script could be provided.
 * Should WTPnet have a keypair so that tarballs can be transmitted to it securely? Motivation is to prevent content filtering on upload to a file hosting site.
 
.. _`TOR skyrocketed`: https://blog.torproject.org/blog/recent-events-egypt
.. _`IE6 usage rates`: http://micgadget.com/11633/why-the-chinese-still-favour-internet-explorer-6/
.. _`streisand.me`: http://streisand.me/
.. _`HBGary leaks site`: http://hbgary.anonleaks.ch/
.. _`The Pirate Bay`: http://thepiratebay.org/
.. _`Reddit`: http://reddit.com/
.. _`WikiLeaks mirrors`: http://wikileaks.ch/Mirrors.html
.. _`Coffeescript`: http://jashkenas.github.com/coffee-script/
.. _`EasyXDM`: http://easyxdm.net
.. _`Stanford JavaScript Crypto Library`: http://bitwiseshiftleft.github.com/sjcl/
.. _`lxml`: http://lxml.de/
.. _`Pycrypto`: http://pycrypto.org
.. _`M2Crypto`: http://chandlerproject.org/bin/view/Projects/MeTooCrypto
.. _`scrapy`: http://scrapy.org
.. _`selenium`: http://seleniumhq.org/
.. _`Browsershots`: http://browsershots.org/
.. _`Django`: http://djangoproject.org
.. _`BitTorrent`: http://pypi.python.org/pypi/BitTorrent/
.. _`ftplib`: http://docs.python.org/library/ftplib.html
.. _`paramiko`: http://www.lag.net/paramiko/
.. _`pysync`: http://freshmeat.net/projects/pysync/
.. _`scipy`: http://www.scipy.org
.. _`NLTK`: http://www.nltk.org/
.. _`Google Translate`: http://translate.google.com/