**Wrbldnsd** is a Windows anti-spam solution by serving [DNSBL](https://en.wikipedia.org/wiki/DNSBL) zones locally. It is a packaging of Rbldnsd, Rsync and Cygwin. You can use Wrbldnsd to locally mirror DNSBL zone files, thus eliminating latency problems in larger environments and decreasing time used for email filtering significantly.

[Rbldnsd](https://rbldnsd.io "A small and fast DNS daemon especially made to serve DNSBL zones") is a small and fast DNS daemon which is especially engineered for serving DNSBL zones. It can serve both IP-based and name-based blocklists. All zones are kept in memory for faster performance with a decent memory footprint.  _[Rsync](http://rsync.samba.org/)_  uses the [Rsync algorithm](http://rsync.samba.org/tech_report/) which provides a very fast method for bringing remote files into sync. It does this by sending just the differences in the files across the link, without requiring that both sets of files are present at one of the ends of the link beforehand. [Cygwin](http://www.cygwin.com/) is a Linux-like environment for Windows. It consists of a DLL, which emulates substantial Linux API functionality, and a collection of tools.

Provided by [itefix.net](https://itefix.net)
