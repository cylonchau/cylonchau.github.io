# 

```conf
//
// named.conf
//
// Provided by Red Hat bind package to configure the ISC BIND named(8) DNS
// server as a caching only nameserver (as a localhost DNS resolver only).
//
// See /usr/share/doc/bind*/sample/ for example named configuration files.
//
// See the BIND Administrator&#39;s Reference Manual (ARM) for details about the
// configuration located in /usr/share/doc/bind-{version}/Bv9ARM.html

options {
	listen-on port 53 { any; };
//	listen-on-v6 port 53 { ::1; };
	directory 	&#34;/data/named&#34;;
	dump-file 	&#34;/data/named/data/cache_dump.db&#34;;
	statistics-file &#34;/data/named/data/named_stats.txt&#34;;
	memstatistics-file &#34;/data/named/data/named_mem_stats.txt&#34;;
	recursing-file  &#34;/data/named/data/named.recursing&#34;;
	secroots-file   &#34;/data/named/data/named.secroots&#34;;
	allow-query     { any; };
	allow-query-cache { any; };
	/* 
	 - If you are building an AUTHORITATIVE DNS server, do NOT enable recursion.
	 - If you are building a RECURSIVE (caching) DNS server, you need to enable 
	   recursion. 
	 - If your recursive DNS server has a public IP address, you MUST enable access 
	   control to limit queries to your legitimate users. Failing to do so will
	   cause your server to become part of large scale DNS amplification 
	   attacks. Implementing BCP38 within your network would greatly
	   reduce such attack surface 
	*/
	recursion yes;

	dnssec-enable no;
	dnssec-validation no;

	/* Path to ISC DLV key */
	bindkeys-file &#34;/etc/named.root.key&#34;;

	managed-keys-directory &#34;/data/named/dynamic&#34;;

	pid-file &#34;/run/named/named.pid&#34;;
	session-keyfile &#34;/run/named/session.key&#34;;
};
statistics-channels {

       inet 127.0.0.1 port 53 allow { 127.0.0.1; };

};

logging {
        channel default_debug {
                file &#34;/data/logs/named/named.run&#34;;
                severity dynamic;
        };
	channel warning {
          file &#34;/data/logs/named/named.log&#34; versions 100 size10m;
          severity warning;
          print-category yes;
          print-severity yes;
          print-time yes;
         };
         channel query {
           file &#34;/data/logs/named/query.log&#34; versions 100 size 10m;
           severity info;
           print-category yes;
           print-severity yes;
           print-time yes;
         };
         category default { warning; };
         category queries { query; };
};

zone &#34;.&#34; IN {
	type hint;
	file &#34;named.ca&#34;;
};

key &#34;rndc-key&#34; {
    algorithm hmac-md5;
    secret &#34;R&#43;pzomztOItyduEqVF2gjA==&#34;;
};

include &#34;/etc/named.rfc1912.zones&#34;;
include &#34;/etc/named.root.key&#34;;
```



```conf
zone &#34;5share.com&#34; IN {
	type master;
	file &#34;5share.com.zone&#34;;
	allow-transfer { 10.11.17.90; 10.11.17.89; };
        allow-update { 10.11.17.89; };
};

zone &#34;r_5share_prod.service.ybshare&#34; IN {
	type forward;
        forwarders { 10.11.11.5;  10.11.11.6; };
         forward only;
};

zone &#34;w_5share_prod.service.ybshare&#34; IN {
        type forward;
        forwarders { 10.11.11.4; };
        forward only;
};

```





slave

```conf
zone &#34;5share.com&#34; IN {
	type slave;
	masters { 10.11.17.89; };
	masterfile-format text;
	file &#34;slaves/5share.com.zone&#34;;
};

zone &#34;r_5share_prod.service.ybshare&#34; IN {
        type forward;
        forwarders { 10.11.11.5;  10.11.11.6; };
         forward only;
};

zone &#34;w_5share_prod.service.ybshare&#34; IN {
        type forward;
        forwarders { 10.11.11.4; };
        forward only;
};

```

http://www.361way.com/bind-master-slave/4811.html

https://www.cnblogs.com/fuhai0815/p/8459670.html


