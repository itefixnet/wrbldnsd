# rbldnsd
---

## NAME

rbldnsd − DNS daemon suitable for running DNS−based blocklists

## SYNOPSIS

**rbldnsd** options *zone*:*dataset*...

## DESCRIPTION

**rbldnsd** is a small DNS−protocol daemon which is designed to handle queries to DNS−based IP−listing or NAME−listing services. Such services are a simple way to share/publish a list of IP addresses or (domain) names which are "listed" for for some reason, for example in order to be able to refuse a service to a client which is "listed" in some blocklist.

**rbldnsd** is not a general−purpose nameserver. It will answer to A and TXT (and SOA and NS if such RRs are specified) queries, and has limited ability to answer to some other types of queries.

**rbldnsd** tries to handle data from two different perspectives: given a set (or several) of "listed entries" (e.g. IP address ranges or domain names), it builds and serves a DNS zone. Note the two are not the same: list of spammer’s IPs is NOT a DNS zone, but may be represented and used as such, provided that some additional information necessary to build complete DNS zone (e.g. NS and SOA records, maybe A records necessary for http to work) is available. In this context, **rbldnsd** is very different from other general−purpose nameservers such as BIND or NSD: **rbldnsd** operates with *datasets* (sets of entries − IP addresses or domain names, logically grouped together), while other general−purpose nameservers operates with zones. The way how **rbldnsd** operates may be somewhat confusing to BIND experts.

For **rbldnsd**, a building block is a dataset: e.g., set of insecure/abuseable hosts (IP addresses), set of network ranges that belongs to various spam operations (IP ranges), domain names that belong to spammers (RHSBL) and so on. Usually, different kind of information is placed into separate file, for easy maintenance. From a number of such datasets, **rbldnsd** constructs a number of DNS zones as specified on command line. A single dataset may be used for several zones, and a single zone may be constructed from several datasets.

**rbldnsd** will answer queries to DNS zones specified on the command line as a set of zone specifications. Each zone specification consists of zone basename, dataset type and a comma−separated list of files that forms a given dataset: *zone*:*type*:*file*,*file*,...

Several zones may be specified in command line, so that **rbldnsd** will answer queries to any of them. Also, a single zone may be specified several times with different datasets, so it is possible to form a zone from a combination of several different dataset. The same dataset may be reused for several zones too (and in this case, it will be read into memory only once).

There are several dataset formats available, each is suitable and optimized (in terms of memory, speed and ease of use) for a specific task. Available dataset types may be grouped into the following categories:

lists of IP addresses. When a query is done to a zone with such data, query is interpreted as an IP address in a reverse form (similar to in−addr.arpa zone). If the address is found in dataset data, **rbldnsd** will return A and TXT records specified in data for that IP. This is a classical IP−based blocklist.

lists of domain names. Similar to list of IP addresses, but with generic domain names instead of IPs (wildcards allowed). This type of data may be used to form a blocklist of e.g. sender domain names.

generic list of various types of records, as an auxilary data to form a complete nameserver. This format is similar to bind−style datafiles, but very simplified. One may specify A, TXT, NS and MX records here.

combined set, different datasets from the list above combined in the single (set of) source files, for easy maintenance.

acl, or Access Control List. This is a pseudo dataset, that works by overweriting query results based on the requestor (peer) IP address.

## OPTIONS

The following options may be specified: **
−u** *user*[:*group*]

**rbldnsd** will change its userid to the specified *user*, which defaults to **rbldns**, and *group*, which by default is the primary group of a *user*. **rbldnsd** will refuse to run as the root user, since this is insecure.

**−r** *rootdir*

**rbldnsd** will chroot to *rootdir* if specified. Data files should be available inside *rootdir*.

**−w** *workdir*

**rbldnsd** will change its working directory to *workdir* (after chrooting to *rootdir* if **−r** option is also specified). May be used to shorten filename paths.

**−b** *address*/*port*

This option is *required*. **rbldnsd** will bind to specified *address* and *port* (port defaults to port 53, domain). Either numeric IP address or a hostname may be specified, and either port number or service name is accepted. It is possible to specify several addresses to listen on this way, by repeating **−b** option. Additionally, if there are several addresses listed for a hostname, **rbldnsd** will listen on all of them. Note that **rbldnsd** will work slightly faster if only one listening address is specified. Note the delimiter between host and port is a slash (/), not a colon, to be able to correctly handle IPv6 addresses.

**−t** *defttl*:*minttl*:*maxttl*

Set default reply time−to−live (TTL) value to be *defttl*, and set constraints for TTL to *minttl* and *maxttl*. Default applies when there’s no TTL defined in a given scope (in data file), and constraints are applied when such value provided in data. Any of the values may be omitted, including trailing colon (:) characters, e.g. "**−t **30" set default TTL to be 30 secound, and "**−t **::120" or "**−t **::2m" sets maximum allowed TTL to 2 minutes. All 3 values are in time units, with optional suffix: **s**(secounds, default), **m** (minutes), **h** (hours), **d** (days) or **w** (weeks). Zero *minttl* or maxttl means no corresponding constraint will be enforced. Default *defttl* is 35m.

**−c** *check*

Set interval between checking for zone file changes to be *check*, default is 1m (one minute). **rbldnsd** will check zone file’s last modification time every so often, and if it detects a change, zone will be automatically reloaded. Setting this value to 0 disables automatic zone change detection. This procedure may also be triggered by sending a SIGHUP signal to **rbldnsd** (see SIGNALS section below).

**−p** *pidfile*

Write rbldnsd’s pid to the specified *pidfile*, so it will be easily findable. This file gets written before entering a chroot jail (if specified) and before changing userid, so it’s ok to specify e.g. /var/run/rbldnsd.pid here.

**−l** *logfile*

Specifies a file to which log all requests made. This file is created after entering a chroot jail and becoming a user. Logfiles may be quite large, esp. on busy sites (**rbldnsd** will log *every* recognized request if this option is specified). This option is mainly intended for debugging purposes. Upon receiption of SIGHUP signal, **rbldnsd** reopens its logfile. If *logfile* prefixed with a plus sign (+), logging will not be buffered (i.e. each line will be flushed to disk); by default, logging is buffered to reduce system load. Specify a single hyphen (−) as a filename to log to standard output (filedescriptor 1), either buffered by default, or line-buffered if specified as ‘+−’ (standard output will not be "reopened" upon receiving SIGHUP signal, but will be flushed in case logging is buffered).

**−s** *statsfile*

Specifies a file where **rbldnsd** will write a line with short statistic summary of queries made per zone, every check (**−c**) interval. Format of each line is: *
timestamp zone*:*qtot*:*qok*:*qnxd*:*bin*:*bout zone*:... 
where *timestamp* is unix time (secounds since epoch), *zone* is the name of the base zone, *qtot* is the total number of queries received, *qok* − number of positive replies, *qnxd* − number of NXDOMAIN replies, *bin* is the total number of bytes read from network (excluding IP/UDP overhead and dropped packets), *bout* is the total number of bytes written to network. Ther are as many such tuples as there are zones, and one extra, total typle at the end, with *zone* being "*", like: 
1234 bl1.ex:10:5:4:311:432 bl2.ex:24:13:7:248:375 *:98:35:12:820:987 
Note the total values may be larger than the sum of per-zone values, due to queries made against unlisted zones, or bad/broken packets.

**Rbldnsd** will write bare timestamp to *statsfile* when it is starting up, shutting down or when statistic counters are being reset after receiving SIGUSR2 signal (see below), to indicate the points where the counters are starting back from zero.

By default, **rbldnsd** writes absolute counter values into *statsfile* (number of packets (bytes) since startup or last reset). *statsfile* may be prefixed with plus sign (+), in which case **rbldnsd** will write delta values, that is, number of packets or bytes since last write, or number of packets (bytes) per unit of time ("incremental" mode, hence the "+" sign).

**−x** *extension*

Load the given *extension* file (a dynamically-linked library, usually with ".so" suffix). This allows to gather custom statistics or perform other custom tasks. See separate document for details about building and using extensions. This feature is not available on all platforms, and can be disabled at compile time.

**−X** *extarg*

Pass the given argument, *extarg*, to the extension loaded with **−x**.

## DATASET TYPES AND FORMATS

Dataset files are text files which are interpreted depending on type specified in command line. Empty lines and lines starting with hash character (#) or semicolon (;) are ignored, except for a special case outlined below in section titled "Special Entries".

A (comma−separated) list of files in dataset specification (in *type*:*file*,*file*,...) is interpreted as if all files where logically combined into one single file.

When compiled with zlib support, **rbldnsd** is able to read gzip−compressed data files. So, every *file* in dataset specification can be compressed with **gzip**(1), and **rbldnsd** will read such a file decompressing it on−the−fly. This feature may be turned off by specifying **−C** option.

**rbldnsd** is designed to service a DNSBL, where each entry have single A record and optional TXT record assotiated with it. **rbldnsd** allows to specify A value and TXT *template* either for each entry individually, or to use default A value and TXT template pair for a group of entries. See section "Resulting A values and TXT templates" below for a way to specify them.

**Special Entries** 
If a line starts with a dollar sign ($), hash character and a dollar sign (#$), semicolon and dollar sign (;#) or colon and a dollar sign (:$), it is interpreted in a special way, regardless of dataset type (this is one exception where a line starting with hash character is not ignored − to be able to use zone files for both **rbldnsd** and for DJB’s rbldns). The following keywords, following a dollar sign, are recognized: **
$SOA** *ttl origindn persondn serial refresh retry expire minttl*

Specifies SOA (Start Of Authority) record for all zones using this dataset. Only first SOA record is interpreted. This is the only way to specify SOA − by default, **rbldnsd** will not add any SOA record into answers, and will REFUSE to answer to certain queries (notably, SOA query to zone’s base domain name). It is recommended, but not mandatory to specify SOA record for every zone. If no SOA is given, negative replies will not be cacheable by caching nameservers. Only one, first $SOA line is recognized in every dataset (all subsequent $SOA lines encountered in the same dataset are silently ignored). When constructing a zone, SOA will be taken from *first* dataset where $SOA line is found, in an order as specified in command line, subsequent $SOA lines, if any, are ignored. This way, one may overwrite $SOA found in 3rd party data by *prepending* small local file to the dataset in question, listing it before any other files.

If *serial* value specified is zero, timestamp of most recent modified file will be substituted as *serial*.

If *ttl* field is zero, default ttl (**−t** option or last **$TTL** value, see below) will be used.

All time fields (ttl, refresh, retry, expire, minttl) may be specified in time units. See **−t** option for details.

**$NS** *ttl nameserverdn nameserverdn...*

Specifies NS (Name Server) records for all zones using this dataset. Only first $NS line in a dataset is recognized, all subsequent lines are silently ignored. When constructing a zone from several datasets, rbldnsd uses nameservers from $NS line in only first dataset where $NS line is given, in command-line order, just like for $SOA record. Only first 32 namservers are recognized. Individual nameserver(s) may be prefixed with a minus sign (−), which means this single nameserver will be ignored by **rbldnsd**. This is useful to temporary comment out one nameserver entry without removing it from the list. If *ttl* is zero, default ttl will be used. The list of NS records, just like $SOA value, are taken from the *first* data file in a dataset where the $NS line is found, subsequent $NS lines, if any, are ignored.

**$TTL** *time-to-live*

Specifies TTL (time-to-live) value for all records in current dataset. See also **−t** option. **$TTL** special overrides **−t** value on a per-dataset basis.

**$TIMESTAMP** *dstamp* [*expires*]

(experimental) Specifies the data timestamp *dstamp* when the data has been generated, and optionally when it will expire. The timestamps are in form *yyyy*:*mm*:*dd*[:*hh*[:*mi*[:*ss*]]], where *yyyy* is the year like 2005, *mm* is the month number (01..12), *dd* is the month day number (01..31), *hh* is hour (00..23), *mi* and *ss* are minutes and secounds (00.59); hours, minutes and secounds are optional and defaults to 0; the delimiters (either colon or dash may be used) are optional too, but are allowed for readability. Also, single zero (0) or dash (−) may be used as *dstamp* and/or *expires*, indicating the value is not given. *expires* may also be specified as **+***rel*, where *rel* is a time specification (probably with suffix like s, m, h, d) as an offset to *dstamp*. **rbldnsd** compares *dstamp* with current timestamp and refuses to load the file if *dstamp* specifies time in the future. And if *expires* is specified, **rbldnsd** will refuse to service requests for that data if current time is greather than the value specified in *expires* field.

Note that **rbldnsd** will check the data expiry time every time it checks for data file updates (when receiving SIGHUP signal or every **−c** interval). If automatic data reload timer (**−c** option) is disabled, zones will not be exipired automatically.

**$MAXRANGE4** *range-size*

Specifies maximum size of IPv4 range allowed for IPv4−based datasets. If an entry covers more IP addresses than *range-size*, it will be ignored (and a warning will be logged). *range-size* may be specified as a number of hosts, like 256, or as network prefix lenght, like /24 (the two are the same): 
$MAXRANGE4 /24 
$MAXRANGE4 256 
This constraint is active for a dataset it is specified in, and can be owerwritten (by subsequent $MAXRANGE statement) by a smaller value, but can not be increased.

**$***n text*

(*n* is a single digit). Specifies a *substitution variable* for use as $*n* placeholders (the **$***n* entries are ignored in generic daaset). See section "Resulting A values and TXT templates" below for description and usage examples.

**$=** *text*

Set the base template for all individual TXT records. See section "Resulting A values and TXT templates" below for more information.

**ip4set Dataset** 
A set of IP addresses or CIDR address ranges, together with A and TXT resulting values. IP addresses are specified one per line, by an IP address prefix (initial octets), complete IP address, CIDR range, or IP prefix range (two IP prefixes or complete addresses delimited by a dash, inclusive). Examples, to specify 127.0.0.0/24: 
127.0.0.0/24 
127.0.0 
127/24 
127−127.0.0 
127.0.0.0−127.0.0.255 
127.0.0.1−255 
to specify 127.16.0.0−127.31.255.255: 
127.16.0.0−127.31.255.255 
127.16.0−127.31.255 
127.16−127.31 
127.16−31 
127.16.0.0/12 
127.16.0/12 
127.16/12 
Note that in prefix range, last boundary is completed with all−ones (255), not all−zeros line with first boundary and a prefix alone. In prefix ranges, if last boundary is only one octet (127.16−31), it is treated as "suffix", as value of last *specified* octet of the first boundary prefix (127.16.0−31 is treated as 127.16.0.0−127.16.31.255, i.e. 127.16.0.0/19).

After an IP address range, A and TXT values for a given entry may be specified. If none given, default values in current scope (see below) applies. If a value starts with a colon, it is interpreted as a pair of A record and TXT template, delimited by colon (:127.0.0.2:This entry is listed). If a value does not start with a colon, it is interpreted as TXT template only, with A record defaulting to the default A value in current scope.

IP address range may be followed by a comment char (either hash character (#) or semicolon (;)), e.g.: 
127/8 ; loopback network 
In this case all characters up to the end of line are ignored, and default A and TXT values will be used for this IP range.

Every IP address that fits within any of specified ranges is "listed", and **rbldnsd** will respond to reverse queries against it within specified zone with positive results. In contrast, if an entry starts with an exclamation sign (!), this is an *exclusion* entry, i.e. corresponding address range is excluded from being listed (and any value for this record is ignored). This may be used to specify large range except some individual addresses, in a compact form.

If a line starts with a colon (:), this line specifies the default A value and TXT template to return (see below) for all subsequent entries up to end of current file. If no default entry specified, and no value specified for a given record, **rbldnsd** will return 127.0.0.2 for matching A queries and no record for matching TXT queries. If TXT record template is specified and contains occurences of of dollar sign ($), every such occurence is replaced with an IP address in question, so singe TXT template may be used to e.g. refer to a webpage for an additional information for a specific IP address.

**ip4trie Dataset** 
Set of IP4 CIDR ranges with corresponding (A, TXT) values. This dataset is similar to ip4set, but uses a different internal representation. It accepts CIDR ranges only (not a.b.c.d−e.f.g.h), and allows for the specification of A/TXT values on a per CIDR range basis. (If multiple CIDR ranges match a query, the value for longest matching prefix is returned.) Exclusions are supported too.

This dataset is not particularly memory-efficient for storing many single IP addresses — it uses about 50% more memory than the ip4set dataset in that case. The ip4trie dataset is better adapted, however, for listing CIDR ranges (whose lengths are not a multiple of 8 bits.)

**ip4tset Dataset** 
"trivial" ip4set: a set of single IP addresses (one per line), with the same A+TXT template. This dataset type is more efficient than ip4set (in both memory usage and access times), but have obvious limitation. It is intended for DNSBLs like DSBL.org, ORDB.org and similar, where each entry uses the same default A+TXT template. This dataset uses only half a memory for the same list of IP addresses compared to **ip4set**.

**ip6trie Dataset** 
Set of IP6 CIDR ranges. This is the IP6 equivalent of the ip4trie dataset. It allows the sepecification of individual A/TXT values for each CIDR range and supports exclusions. Compressed ("::") ip6 notation is supported.

Example zone data: 
# Default A and TXT template valuse 
:127.0.1.2: Listed, see http://example.com/lookup?$

# A listing, note that trailing :0s can be omitted 
2001:21ab:c000/36

# /64 range with non-default A and TXT values 
2001:21ab:def7:4242 :127.0.1.3: This one smells funny

# compressed notation 
2605:6001:42::/52 
::1 # localhost 
!2605:6001:42::bead # exclusion

**ip6tset Dataset** 
"Trivial" ip6 dataset: a set of /64 IP6 CIDR ranges (one per line), all sharing a single A+TXT template. Exclusions of single IP6 (/128) addresses are also supported. This dataset type is quite memory-efficient — it uses about 40% of the memory that the ip6trie dataset would use — but has obvious limitations.

This dataset wants the /64s listed as four ip6 words, for example: 
2001:20fe:23:41ed 
abac:adab:ad00:42f 
Exclusions are denoted with a leading exclamation mark. You may also use compressed "::" notation for excluded addresses. E.g.: 
!abac:adab:ad00:42f:face:0f:a:beef 
!abac:adab:ad00:42f::2

**dnset Dataset** 
Set of (possible wildcarded) domain names with associated A and TXT values. Similar to **ip4set**, but instead of IP addresses, data consists of domain names (*not* in reverse form). One domain name per line, possible starting with wildcard (either with star−dot (*.) or just a dot). Entry starting with exclamation sign is exclusion. Default value for all subsequent lines may be specified by a line starting with a colon.

Wildcards are interpreted as follows: 
example.com

only example.com domain is listed, not subdomains thereof. Not a wildcard entry.

*.example.com

all subdomains of example.com are listed, but not example.com itself.

.example.com

all subdomains of example.com *and* example.com itself are listed. This is a shortcut: to list a domain name itself and all it’s subdomains, one may either specify two lines (example.com and *.example.com), or one line (.example.com).

This dataset type may be used instead of **ip4set**, provided all CIDR ranges are expanded and reversed (but in this case, TXT template will be expanded differently).

**generic Dataset** 
Generic type, simplified bind−style format. Every record should be on one line (line continuations are not supported), and should be specified completely (i.e. all domain names in values should be fully−qualified, entry name may not be omitted). No wildcards are accepted. Only A, TXT, and MX records are recognized. TTL value may be specified before record type. Examples:

# bl.ex.com 
# specify some values for current zone 
$NS 0 ns1.ex.com ns2.ex.com 
# record with TTL 
www 3000 A 127.0.0.1 
about TXT "ex.com combined blocklist"

**combined Dataset** 
This is a special dataset that stores no data by itself but acts like a container for several other datasets of any type except of combined type itself. The data file contains an optional common section, where various specials are recognized like $NS, $SOA, $TTL (see above), and a series of sections, each of which defines one (nested) dataset and several subzones of the base zone, for which this dataset should be consulted. New (nested) dataset starts with a line 
$DATASET *type*[:*name*] *subzone subzone*... 
and all subsequent lines up to the end of current file or to next $DATASET line are interpreted as a part of dataset of type *type*, with optional *name* (name is used for logging purposes only, and the whole ":*name*" (without quotes or square brackets) part is optional). Note that combined datasets cannot be nested. Every *subzone* will always be relative to the base zone name specified on command line. If *subzone* specified as single character "@", dataset will be connected to the base zone itself.

This dataset type aims to simplify subzone maintenance, in order to be able to include several subzones in one file for easy data transfer, atomic operations and to be able to modify list of subzones on remote secondary nameservers.

Example of a complete dataset that contains subzone ‘proxies’ with a list of open proxies, subzone ‘relays’ with a list of open relays, subzone ‘multihop’ with output IPs of multihop open relays, and the base zone itself includes proxies and relays but not multihops: 
# common section 
$NS 1w ns1.ex.com ns2.ex.com 
$SOA 1w ns1.ex.com admin.ex.com 0 2h 2h 1w 1h 
# list of open proxies, 
# in ‘proxies’ subzone and in base zone 
$DATASET ip4set:proxy proxies @ 
:2:Open proxy, see http://bl.ex.com/proxy/$ 
127.0.0.2 
127.0.0.10 
# list of open relays, 
# in ‘relays’ subzone and in base zone 
$DATASET ip4set:relay relays @ 
:3:Open relay, see http://bl.ex.com/relay/$ 
127.0.0.2 
127.0.2.10 
# list of optputs of multistage relays, 
# in ‘multihop’ subzone only 
$DATASET ip4set:multihop-relay multihop 
:4:Multihop open relay, see http://bl.ex.com/relay/$ 
127.0.0.2 
127.0.9.12 
# for the base zone and all subzones, 
# include several additional records 
$DATASET generic:common proxies relays multihop @ 
@ A 127.0.0.8 
www A 127.0.0.8 
@ MX 10 mx.ex.com 
# the above results in having the following records 
# (provided that the base zone specified is bl.ex.com): 
# proxies.bl.ex.com A 127.0.0.8 
# www.proxies.bl.ex.com 127.0.0.8 
# relays.bl.ex.com A 127.0.0.8 
# www.relays.bl.ex.com 127.0.0.8 
# multihop.bl.ex.com A 127.0.0.8 
# www.multihop.bl.ex.com 127.0.0.8 
# bl.ex.com A 127.0.0.8 
# www.bl.ex.com 127.0.0.8

Note that $NS and $SOA values applies to the base zone *only*, regardless of the placement in the file. Unlike the $TTL values and $*n* substitutions, which may be both global and local for a given (sub−)dataset.

**Resulting A values and TXT templates** 
In all zone file types except generic, A values and TXT templates are specified as following: 
:127.0.0.2:Blacklisted: http://example.com/bl?$ 
If a line starts with a colon, it specifies default A and TXT for all subsequent entries in this dataset. Similar format is used to specify values for individual records, with the A value (enclosed by colons) being optional: 
127.0.0.2 :127.0.0.2:Blacklisted: http://example.com/bl?$ 
or, without specific A value: 
127.0.0.2 Blacklisted: http://example.com/bl?$

Two parts of a line, delimited by second colon, specifies A and TXT record values. Both are optional. By default (either if no default line specified, or no IP address within that line), **rbldnsd** will return 127.0.0.2 as A record. 127.0.0 prefix for A value may be omitted, so the above example may be simplified to: 
:2:Blacklisted: http://example.com/bl?$ 
There is no default TXT value, so **rbldnsd** will not return anything for TXT queries it TXT isn’t specified.

When A value is specified for a given entry, but TXT template is omitted, there may be two cases interpreted differently, namely, whenever there’s a second semicolon (:) after the A value. If there’s no second semicolon, default TXT value for this scope will be used. In contrast, when second semicolon is present, no TXT template will be generated at all. All possible cases are outlined in the following example:

# default A value and TXT template 
:127.0.0.2:IP address $ is listed 
# 127.0.0.4 will use default A and TXT 
127.0.0.4 
# 127.0.0.5 will use specific A and default TXT 
127.0.0.5 :5 
# 127.0.0.6 will use specific a and *no* TXT 
127.0.0.6 :6: 
# 127.0.0.7 will use default A and specific TXT 
127.0.0.7 IP address $ running an open relay

In a TXT template, references to substitution variables are replaced with values of that variables. In particular, single dollar sign ($) is replaced by a listed entry (an IP address in question for IP−based datasets and the domain name for domain−based datasets). **$***n*−style constructs, where *n* is a single digit, are replaced by a substitution variable $*n* defined for this dataset in current scope (see section "Special Entries" above). To specify a dollar sign as−is, use **$$**.

For example, the following lines: 
$1 See http://www.example.com/bl 
$2 for details 
127.0.0.2 $1/spammer/$ $2 
127.0.0.3 $1/relay/$ $2 
127.0.0.4 This spammer wants some $$$$. $1/$ 
will result in the following text to be generated: 
See http://www.example.com/bl/spammer/127.0.0.2 for details 
See http://www.example.com/bl/relay/127.0.0.3 for details 
This spammer wants some $$. See http://www.example.com/bl/127.0.0.4

If the "base template" (**$=** variable) is defined, this template is used for expansion, instead of the one specified for an entry being queried. Inside the base template, **$=** construct is substituted with the text given for individual entries. In order to stop usage of base template **$=** for a single record, start it with **=** (which will be omitted from the resulting TXT value). For example, 
$0 See http://www.example.com/bl?$= ($) for details

127.0.0.2

127.0.0.3

127.0.0.4

produces the following TXT records: 
See http://www.example.com/bl?r123 (127.0.0.2) for details 
See http://www.example.com/bl?127.0.0.3 (127.0.0.3) for details 
See other blocklists for details about 127.0.0.4

**acl Dataset** 
This is not a real dataset, while the syntax and usage is the same as with other datasets. Instead of defining which records exists in a given zone and which do not, the **acl** dataset specifies which client hosts (peers) are allowed to query the given zone. The dataset specifies a set of IPv4 and/or IPv6 CIDR ranges (with the syntax exactly the same as understood by the **ip4trie** and **ip6trie** datasets), together with action specifiers. When a query is made from an IP address listed (not *for* the IP address), the specified action changes rules used to construct the reply. Possible actions and their meanings are: 
:**ignore**

ignore all queries from this IP address altogether. **rbldnsd** acts like there was no query received at all. This is the default action.

:**refuse**

refuse all queries from the IP in question. **rbldnsd** will always return REFUSED DNS response code.

*a_txt_template*

usual A+TXT template as used by other datasets. This means that **rbldnsd** will reply to any valid DNSBL query with "it is listed" answer, so that the client in question will see every IP address or domain name is listed in a given DNSBL. TXT record used in the reply is taken from the acl dataset instead of real datasets. Again, just like with **empty** case, **rbldnsd** will continue replying to metadata queries (including generic datasets if any) as usual.

Only one ACL dataset can be specified for a given zone, and each zone must have at least one non−acl dataset. It is also possible to specify one global ACL dataset, by specifying empty zone name (which is not allowed for other dataset types), like **
rbldnsd** ... :**acl**:*filename*... 
In this case the ACL defined in *filename* applies to all zones. If there are both global ACL and local zone-specific ACL specified, both will be consulted and actions taken in the order specified above, ie, if either ACL returns **ignore** for this IP, the request will be ignored, else if either ACL returns **refuse**, the query will be refused, and so on. If both ACLs specifies "always listed" A+TXT template, the reply will contain A+TXT from global ACL.

For this dataset type, only a few $-style specials are recognized. In particular, $SOA and $NS keywords are not allowed. When **rbldnsd** performs **$** substitution in the TXT template returned from ACL dataset, it will use client IP address to substitute for a single $ character, instead of the IP address or domain name found in the original query.

## SIGNALS

**Rbldnsd** handles the following signals:

**SIGTERM**, **SIGINT**

Terminate process.

**SIGUSR1**

Log current statistic counters into syslog. **Rbldnsd** collects how many packets it handled, how many bytes was received, sent, how many OK requests/replies (and how many answer records) was received/sent, how many NXDOMAIN answers was sent, and how many errors/refusals/etc was sent, in a period of time.

**SIGUSR2**

The same as SIGUSR1, but reset all counters and start new sample period.

## NOTES

Some unsorted usage notes follows.

**Generating and transferring data files** 
When creating a data file for **rbldnsd** (and for anything else, it is a general advise), it is a good idea to create the data in temporary file and rename the temp file when all is done. *Never* try to write to the main file directly, it is possible that at the same time, **rbldnsd** will try to read it and will get incomplete data as the result. The same applies to copying data using **cp**(1) utility and similar (including **scp**(1)), that performs copying over existing data. Even if you’re sure noone is reading the data while you’re copying or generating it, imagine what will happen if you will not be able to complete the process for whatever reason (interrupt, filesystem full, endless number of other reasons...). In most cases is better to keep older but correct data instead of leaving incomplete/corrupt data in place.

Right: 
scp remote:data target.tmp && mv target.tmp target *
Wrong*: 
scp remote:data target 
Right: 
./generate.pl > target.tmp && mv target.tmp target *
Wrong*: 
./generate.pl > target

From this point of view, **rsync**(1) command seems to be safe, as it *always* creates temporary file and renames it to the destination only when all is ok (but note the −−partial option, which is good for downloading something but may be wrong to transfer data files −− usually you don’t want partial files to be loaded). In contrast, **scp**(1) command is *not* safe, as it performs direct copying. You may still use **scp**(1) in a safe manner, as shown in the example above.

Also try to eliminate a case when two (or more) processes performs data copying/generation at the same time to the same destination. When your data is generated by a cron job, use file locking (create separate lock file (which should never be removed) and flock/fcntl it in exclusive mode without waiting, exiting if lock fails) before attempting to do other file manipulation.

**Absolute vs relative domain names** 
All *keys* specified in dataset files are always relative to the zone base DN. In contrast, all the *values* (NS and SOA records, MX records in generic dataset) are absolute. This is different from BIND behaviour, where trailing dot indicates whenever this is an absolute or relative DN. Trailing dots in domain names are ignored by **rbldnsd**.

**Aggregating datasets** 
Several zones may be served by **rbldnsd**, every zone may consist of several datasets. There are numerous ways to combine several data files into several zones. For example, suppose you have a list of dialup ranges in file named ‘dialups’, and a list of spammer’s ip addresses in file named ‘spammers’, and want to serve 3 zones with **rbldnsd**: dialups.bl.ex.com, spam.bl.ex.com and bl.ex.com which is a combination of the two. There are two ways to do this:

rbldnsd *options...* \ 
dialups.bl.ex.com:ip4set:dialups \ 
spam.bl.ex.com:ip4set:spammers \ 
bl.ex.com:ip4set:dialups,spammers

or:

rbldnsd *options...* \ 
dialups.bl.ex.com:ip4set:dialups \ 
spam.bl.ex.com:ip4set:spammers \ 
bl.ex.com:ip4set:dialups \ 
bl.ex.com:ip4set:spammers

(note you should specify combined bl.ex.com zone *after* all its subzones in a command line, or else subzones will not be consulted at all).

In the first form, there will be 3 independent data sets, and every record will be stored 2 times in memory, but only one search in internal data structures will be needed to resolve queries for aggregate bl.ex.com. In second form, there will be only 2 data sets, every record will be stored only once (both datasets will be reused), but 2 searches will be performed by **rbldnsd** to answer queries against aggregate zone (but difference in speed is almost unnoticeable). Note that when aggregating several data files into one dataset, an exclusion entry in one file becomes exclusion entry in the whole dataset (which may be a problem when aggregating dialups, where exclusions are common, with open relays/proxies, where exclusions are rare if at all used).

Similar effect may be achieved by using **combined** dataset type, sometimes more easily. **combined** dataset results in every nested dataset to be used independantly, like in second form above.

**combined** dataset requires **rbldnsd** to be the authoritative nameserver for the whole base zone. Most important, one may specify SOA and NS records for the base zone *only*. So, some DNSBLs which does not use a common subzone for the data, cannot use this dataset. An example being DSBL.org DNSBL, where each of list.dsbl.org, multihop.dsbl.org and unconfirmed.dsbl.org zones are separate, independant zones with different set of nameservers. But for DSBL.org, where each dataset is really independant and used only once (there’s no (sub)zone that is as a combinations of other zones), **combined** dataset isn’t necessary. In contrast, SORBS.net zones, where several subzones used and main zone is a combination of several subzones, **combined** dataset is a way to go.

**All authoritative nameservers should be set up similarily** 
When you have several nameservers for your zone, set them all in a similar way. Namely, if one is set up using **combined** dataset, all the rest should be too, or else DNS meta−data will be broken. This is because metadata (SOA and NS) records returned by nameservers using**combined** and other datasets will have different origin. With combined dataset, **rbldnsd** return NS and SOA records for the base zone, not for any subzone defined inside the dataset. Given the above example with dialups.bl.ex.com, spammers.bl.ex.com and aggregate bl.ex.com zones, and two nameservers, first is set up in any ways described above (using individual datasets for every of the 3 zones), and second is set up for the whole bl.ex.com zone using **combined** dataset. In this case, for queries against dialups.bl.ex.com, first nameserver will return NS records like 
dialups.bl.ex.com. IN NS a.ns.ex.com. 
while second will always use base zone, and NS records will look like 
bl.ex.com. IN NS a.ns.ex.com. 
All authoritative nameservers for a zone must have consistent metadata records. The only way to achieve this is to use similar configuration (combined or not) on all nameservers. Have this in mind when using other software for a nameserver.

**Generic dataset usage 
generic** dataset type is very rudimentary. It’s purpose is to complement all the other type to form complete nameserver that may answer to A, TXT and MX queries. This is useful mostly to define A records for HTTP access (relays.bl.example.com A, www.bl.example.com A just in case), and maybe descriptive texts as a TXT record.

Since **rbldnsd** only searches *one*, most closely matching (sub)zone for every request, one cannot specify a single e.g. **generic** dataset in form 
proxies TXT list of open proxies 
www.proxies A 127.0.0.8 
relays TXT list of open relays 
www.relays A 127.0.0.9 
for several (sub)zones, each of which are represented as a zone too (either in command line or as **combined** dataset). Instead, several **generic** datasets should be specified, separate one for every (sub)zone. If the data for every subzone is the same, the same, single dataset may be used, but it should be specified for every zone it should apply to (see **combined** dataset usage example above).

## BUGS

Most of the bugs outlined in this section aren’t really bugs, but present due to non-standartized and thus unknown expected behaviour of a nameserver that serves a DNSBL zone. **rbldnsd** matches BIND runtime behaviour where appropriate, but not always.

**rbldnsd** lowercases some domain names (the ones that are lookup keys, e.g. in ‘generic’ and ‘dnset’ datasets) when loading, to speed up lookup operations. This isn’t a problem in most cases.

There is no TCP mode. If a resource record does not fit in UDP packet (512 bytes), it will be silently ignored. For most usages, this isn’t a problem, because there should be only a few RRs in an answer, and because one record is usually sufficient to decide whenever a given entry is "listed" or not. **rbldnsd** isn’t a full−featured nameserver, after all.

**rbldnsd** will not always return a list of nameserver records in the AUTHORITY section of every positive answer: NS records will be provided (if given) only if there’s a room for them in single UDP packet. If records does not fit, AUTHORITY section will be empty.

**rbldnsd** does not allow AXFR operations. For DNSBLs, AXFR is the stupidiest yet common thing to do − use rsync for zone transfers instead. This isn’t a bug in **rbldnsd** itself, but in common practice of using AXFR and the like to transfer huge zones in a format which isn’t suitable for such a task. Perhaps in the future, if there will be some real demand, I’ll implement AXFR "server" support (so that **rbldnsd** will be able to act as master for BIND nameservers, but not as secondary), but the note remains: use rsync.

**rbldnsd** truncates all TXT records to be at most 255 bytes. DNS specs allows longer TXTs, but long TXTs is something that should be avoided as much as possible − TXT record is used as SMTP rejection string. Note that DNS UDP packet is limited to 512 bytes. **rbldnsd** will log a warning when such truncation occurs.

## VERSION

This manpage corresponds to **rbldnsd** version **0.998**.

## AUTHOR

The **rbldnsd** daemon written by Michael Tokarev <mjt+rbldnsd@corpit.ru>, based on ideas by Dan Bernstein and his djbdns package, with excellent contributions by Geoffrey T. Dairiki <dairiki@dairiki.org>.

## LICENCE

Mostly GPL, with some code licensed under 3-clause BSD license.

---
