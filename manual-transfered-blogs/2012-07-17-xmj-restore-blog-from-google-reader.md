转换时间： 2021-01-31

# Restore wordpress blog from Google Reader
Posted on 2012/07/17 by yao	

It is extremely bad that the data of our blog got lost one month ago. Today, I decide to recover it from “Google Reader” with following instructions or steps,

1. Get the atom xml file from this link http://www.google.com/reader/atom/feed/http://www.hellogcc.org/feed?n=1000 ,which is the atom xml file, and save it in my local disk, say, google_reader_atom.xml.
2. Copy a google_reader_atom.xml to some public http server, and make sure this xml file can be accessed.
3. Open this url http://atom2rss.semiologic.com/ and enter the url of google_reader_atom.xml into the box, and click “convert”. Then, we can get a rss xml file, save it google_reader_rss.xml.
4. Login wordpress blog as an admin, and click “Import” -> “RSS”, select google_reader_rss.xml on local disk. Then, everything is done.
