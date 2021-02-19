---
title: "Restore wordpress blog from Google Reader"
date: "2012-07-16"
categories: 
  - "其它技术"
tags: 
  - "blog"
  - "maintenance"
---

It is extremely bad that the data of our blog got lost one month ago. Today, I decide to recover it from "Google Reader" with following instructions or steps,

1\. Get the atom xml file from this link [http://www.google.com/reader/atom/feed/http://www.hellogcc.org/feed?n=1000](http://atom2rss.semiologic.com/ "http://atom2rss.semiologic.com/") ,which is the atom xml file, and save it in my local disk, say, _google\_reader\_atom.xml_. 2. Copy a _google\_reader\_atom.xml_ to some public http server, and make sure this xml file can be accessed. 3. Open this url [http://atom2rss.semiologic.com/](http://atom2rss.semiologic.com/ "http://atom2rss.semiologic.com/") and enter the url of _google\_reader\_atom.xml_ into the box, and click "convert". Then, we can get a rss xml file, save it _google\_reader\_rss.xml_. 4. Login wordpress blog as an admin, and click "Import" -> "RSS", select _google\_reader\_rss.xml_ on local disk. Then, everything is done.
