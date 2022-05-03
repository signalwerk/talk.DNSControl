---
tags: TeamTalk
slideOptions:
  theme: white
  controls: false
---

<style>
    @import url('https://fonts.googleapis.com/css2?family=Open+Sans:wght@400;700&display=swap');
    
    :root {
        --r-heading-text-transform: none;
        --r-main-font: 'Open Sans';
        --r-heading-font: 'Open Sans';
        --r-heading-font-weight: 700;
        --r-heading-color: #0054a2;
        --r-link-color: #006984;
        --r-heading-margin: 0 0 .6em 0;
    }
    
    :root .reveal {
        font-family: var(--r-main-font);
    }
    
    
    :root .reveal img {
        max-height: 50vh;
    }
    
    
.scroll {
  height: 0;
  overflow: hidden;
  padding-top: calc(9 / 16 * 100%);
  background: white;
  position: relative;
}
.scroll__inner {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  overflow-y: auto;
}
    
    :root .reveal .scroll img {
        max-height: none !important;
    }    
</style>

#  DNSControl
**Administrate and document DNS-Entries**

<br />
<br />
<br />

<small>2022 · Stefan Huber</small>

---

## DNSControl

* Developed by **Stack Overflow** network
* Manages DNS related **administration**

---

## What can I do?

* **create/edit/delete** DNS-Entries
* manage **as code** (infrastructure as code)

---

## What do you need?

* Domain (registrar)
* DNS Provider with API-Support


---


## Example

---

## sfgz.ch

* sfgz.ch is having ~50 DNS entries
* many people just update DNS-Entries...

---

## Switch DNS-Provider

* DNS-Provider with an **API**
* **Global** DNS-Provider (fast)
* Transfer DNS with **«zone file»**


---


## DNS-Provider · Cloudflare 

<div class="scroll">
<div class="scroll__inner">

![](https://i.imgur.com/vPqoCJe.png)

</div>
</div>

---

## cloud.sfgz.ch

![](https://i.imgur.com/lCiJE0I.png)


---

## Questions?


* What machine is `77.109.129.176`?
* Who's responsible for that entry?
* When was this entry introduced?
* Is it still in use?


---

## Change request

Hey Stefan! <br>`cloud.sfgz.ch` <br>is now on the IP <br>`77.109.129.170`<br>can you change the DNS?


---

## Now what?

I have informations other than just the IP

* Requester
* Why?
* When?
* What about the old IP?


---

## DNSControl to the rescue

---

## Let's reproduce it in code



---

## Config


File: `infrastructure.js`


```js
// this is my infrastructure
var SERVERS = {
  nextcloud: "77.109.129.176", // SfGZ Limmatplatz UG (Michi Gmuer AnyKeyIT)
};

// cloudflare is the DNS Provider
var CLOUDFLARE = NewDnsProvider("cloudflare", "CLOUDFLAREAPI");

// no registrar handling (manually)
var REGISTRAR = NewRegistrar("none", "NONE");
```



---

## Config

File: `dnsconfig.js` (Credentials: `creds.json`)

```js
require("infrastructure.js");


D( // definition for a domain
  "sfgz.ch",
  REGISTRAR,
  DnsProvider(CLOUDFLARE),

  A("cloud", SERVERS.nextcloud, TTL(86400)) // A-Record
  // … many many more 
)
```


---

## Changes

File: `infrastructure.js`


```js
// this is my infrastructure
var SERVERS = {
  nextcloud: "77.109.129.170", // SfGZ Limmatplatz UG (Michi Gmuer AnyKeyIT)
};

// cloudflare is the DNS Provider
var CLOUDFLARE = NewDnsProvider("cloudflare", "CLOUDFLAREAPI");

// no registrar handling (manually)
var REGISTRAR = NewRegistrar("none", "NONE");
```

---

## Version-Control

Configuration is in a GIT

```sh
$ git commit -m "Nextcloud-Server switched to new Hardware (confirmed Urs Bernet)"
```

---

## Publish!

Sync code to DNS-Entries

---

## Oh wait!

Let's preview first!

```sh
$ dnscontrol preview
```

---

## Preview output


```sh
******************** Domain: sfgz.ch
----- Getting nameservers from: cloudflare
----- DNS Provider: cloudflare...
1 corrections
#1: MODIFY A cloud.sfgz.ch: (77.109.129.176 ttl=86400 proxy=false) -> (77.109.129.170 ttl=86400 proxy=false)
----- Registrar: none...
0 corrections
Done. 1 corrections.
```



---

## Let's do it!


```sh
$ dnscontrol push
```

---


## cloud.sfgz.ch


### OLD
![](https://i.imgur.com/lCiJE0I.png)


### NEW
![](https://i.imgur.com/XQ4FgKJ.png)

---

## But why?

Version & Blame: [see Configuration](https://github.com/sfgz/sfgz.dns/blame/master/sfgz.ch.js)



---


## Random edits by users…


---

## Check 


```sh
$ dnscontrol preview
```


---

## Ideal output

```sh
******************** Domain: sfgz.ch
----- Getting nameservers from: cloudflare
----- DNS Provider: cloudflare...
0 corrections
```


---

## Reality output


```sh
******************** Domain: sfgz.ch
----- Getting nameservers from: cloudflare
----- DNS Provider: cloudflare...
8 corrections
#1: DELETE record: sfgz.ch TXT 86400 apple-domain-verification=M24oX5lMSaM3DTbf (id=414322ac9329c8f238bbf0b183767a3f)
#2: DELETE record: xserver4.sfgz.ch A 600 77.109.129.150 (id=807df5944a31d43c199cea53d5858683)
#3: CREATE record: alt A 3600 80.74.145.50
#4: MODIFY A sfgz.ch: (34.65.45.216 ttl=86400 proxy=false) -> (34.65.45.216 ttl=3600 proxy=false)
#5: MODIFY TXT _dmarc.sfgz.ch: ("v=DMARC1; p=none; rua=mailto:reports@edu.zh.ch,mailto:mailauth-reports@sfgz.ch; ruf=mailto:reports@edu.zh.ch; fo=1; aspf=r;" ttl=1800) -> ("v=DMARC1; p=none; rua=mailto:mailauth-reports@sfgz.ch" ttl=1800)
#6: MODIFY TXT sfgz.ch: ("adobe-idp-site-verification=7f639379ede377da60ff7316a0a4e09c2e78d0fea95fc0101013f0cc0f2d9c96" ttl=86400) -> ("adobe-idp-site-verification=7f639379ede377da60ff7316a0a4e09c2e78d0fea95fc0101013f0cc0f2d9c96" ttl=3600)
#7: MODIFY A cloud.sfgz.ch: (77.109.129.176 ttl=86400 proxy=false) -> (77.109.129.176 ttl=3600 proxy=false)
#8: MODIFY CNAME www.sfgz.ch: (sfgz.ch. ttl=86400 proxy=false) -> (sfgz.ch. ttl=3600 proxy=false)
----- Registrar: none...
0 corrections
Done. 8 corrections.
```

---

## Thats about it.

---

## exit 0; thx

Questions?
