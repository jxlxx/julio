---
title: RADIUS Server Basics
date: "2022-05-01"
description: A simple explaination of AAA servers, specifically freeRADIUS, and how to get starting using them.
tldr: 
draft: true
---

***free***Radius??? Maybe for you.... It has cost me everything.

# Quick Summary

### Step 1 → ****Picking an Auth-Type (authorization)****

The radius server receives a request and decides what to do with it. It decides based on:

- what authentication types you have enabled in the server
- what the server can look up in a DB
- what’s in the request

The server starts by querying the modules in the authorize section:

- `unix` module, can you handle this?
- `pap` module, can you do this?
- `mschap` module can you do this?
- etc.

Until there is some kind of match by one of the modules.

A module will find a match by searching the request for key attributes, such as `MS-CHAP-Challenge` (for `mschap`) or `EAP-Message` (for `eap`), etc.

If the module does find a match, it sets the `Auth-Type` to itself.

### Step 2 → ****Authenticating a user (authentication)****

At the end of the authorize step, the server will check if anything set the `Auth-Type` if not, it will immediately reject the request.

For example, suppose that the user sent a request with a `User-Password` attribute and `pap` is enabled. The pap module will have set `Auth-Type=pap`.

So it then compares the local "known good" password to the password as entered by the user. This is how authentication works.

If anything is missing or in the wrong form?  → reject

# What is RADIUS?

RADIUS is a network protocol.

Servers that accept and send the RADIUS protocol are called radius servers.

Everyone who has used the internet has probably interacted with a radius server. They are commonly used by ISPs, cellular network providers, and corporate/educational networks.

The primary functions of RADIUS is usually referred to as AAA.

1. Authentication
    - are the users/devices who they say they are?
    - validate some credentials
2. Authorization
    - are the users/devices allowed to use the network?
3. Accounting
    - track the usage of the network by users/devices
    

For example, Eduroam uses RADIUS servers.

If you’re trying to wrap your mind around Open Roaming, just think about how Eduroam works. We actually even use some of Eduroam’s code in our system! (Eduroam is open source).

Open Roaming just has different users/goals then Eduroam, for example:

- Students/etc  → Anybody
- Schools/research institutions → bars/restaurants/Adentro merchants

## How Does Authentication Work?

The user sends a request, that reaches the radius server.

The radius server looks at the content of the request and decides on some sort of authentication method. Radius is really flexible in the sense that you can use a lot of different methods/tools to authenticate a user.

For Adentro, Open Roaming, HS2 etc, in general, the user sends a request with credentials from a config file on their device. Sometimes these config files were created by Adentro and users installed them on their phones, other times devices come pre-installed with them.

This is an excerpt from sample config:

```xml
<key>PayloadContent</key>
<array>
	<dict>
		<key>AutoJoin</key>
		<true/>
		<key>CaptiveBypass</key>
		<false/>
		<key>DisplayedOperatorName</key>
		<string>Adentro DEV English</string>
		<key>DomainName</key>
		<string>zd.cntr.io</string>
		<key>EAPClientConfiguration</key>
		<dict>
			<key>AcceptEAPTypes</key>
			<array>
				<integer>21</integer>
			</array>
			<key>OuterIdentity</key>
			<string>anonymous</string>
			<key>TLSMaximumVersion</key>
			<string>1.2</string>
			<key>TLSMinimumVersion</key>
			<string>1.2</string>
			<key>TLSTrustedServerNames</key>
			<array>
				<string>hs2aaa.zenreach-develop.com</string>
			</array>
			<key>TTLSInnerAuthentication</key>
			<string>MSCHAPv2</string>
			<key>UserName</key>
			<string>e18b1d11-f4b3-411f-9849-791325f1496b</string>
			<key>UserPassword</key>
			<string>OTZhODAwZjMtOTk3Mi00OTc3LWE5NDQtZjIwMzljNWE3MGI0</string>
		</dict>
		<key>EncryptionType</key>
		<string>WPA2</string>
		<key>HIDDEN_NETWORK</key>
		<false/>
		<key>IsHotspot</key>
		<true/>
		<key>NAIRealmNames</key>
		<array>
			<string>osu.zenreach-develop.com</string>
		</array>
		<key>PayloadDescription</key>
		<string>Configures Wi-Fi settings</string>
		<key>PayloadDisplayName</key>
		<string>Wi-Fi</string>
		<key>PayloadIdentifier</key>
		<string>com.apple.wifi.managed.e3beb25b-3a51-4d5b-a3ed-6650c79b6172</string>
		<key>PayloadType</key>
		<string>com.apple.wifi.managed</string>
		<key>PayloadUUID</key>
		<string>e3beb25b-3a51-4d5b-a3ed-6650c79b6172</string>
		<key>PayloadVersion</key>
		<integer>1</integer>
		<key>ProxyType</key>
		<string>None</string>
		<key>ServiceProviderRoamingEnabled</key>
		<false/>
	</dict>
</array>

<key>PayloadDescription</key>
<string>Fast, secure Wi-Fi for local businesses &amp; their customers</string>
.
... blah, blah, blah
.
<key>PayloadVersion</key>
<integer>1</integer>
```

You can see we have:

```xml
<key>TTLSInnerAuthentication</key>
<string>MSCHAPv2</string>

<key>UserName</key>
<string>e18b1d11-f4b3-411f-9849-791325f1496b</string>

<key>UserPassword</key>
<string>OTZhODAwZjMtOTk3Mi00OTc3LWE5NDQtZjIwMzljNWE3MGI0</string>
```

So a user tries to connect to an Adentro SSID. The AP creates/sends a request to `or-anp-freeradius`. `or-anp-freeradius` proxies everything to `or-anp-radsecproxy`. Then radsec proxy will authenticate with a 3rd party IdP, via the WBA networks (some cool federated authentication magic happens here?) with MSCHAP as the `Auth-Type`.

The result gets back to `or-anp-freeradius`, at which point it will begin the process of authorization (`Autz`) of the user/device.

Literally, what is happening? What are the ‘things’ involved? 

## Network Access Server

The NAS acts as the gateway between the user and the wider network.

In our system, Access Points (sometimes misnamed as routers) act as our NASs. 

The NAS will send the users request to access the network to the Radius server. The server will try to authenticate user give the information in the request. After that, the radius server will instruct the NAS whether to allow access to the user and how much.

The NAS acts as the gateway router and firewall for that user.

This means that the server simply returns a decision to the NAS, but it’s up to the NAS to enforce it.

There is nothing the Radius server can do to make the NAS behave as intended.

The radius server receives a request and decides what to do with it. Its decision is based on:

- what authentication types you have enabled in the server
- what the server can look up in a DB
- what’s in the request

The server cannot request additional information from the NAS, so it has to decided what to do with the information it has. It figures out what to with via policies. 

A policy can be something like “accept anyone with a correct username/password combo”

Or more complicated like: “allow basic users to request premium services in non-premium hours, except for Sundays and holidays, so long as their payment status is up to date”

## Radius Dictionaries

Radius is a binary-based protocol, not a text-based one. so when we are talking about attributes in the request, like `User-Name` they are actually encoded in the message as binary data.

Dictionary files are used to map between the names used by people and the binary data in the RADIUS packets.

Packets sent be a NAS contain: a number, length, and binary data

A dictionary file contains a list of entries with a name, a number and a data type.

The server uses the dictionary like this: it searches the dictionaries to match the number in the packet, the corresponding data type is used by the server to interprest the data type and the nam is used in all the debug/logging messages.

Note that the NAS and server may use different dictionaries, which may cause problems.

Also note that *Servers require access to a vendor dictionary to understand vendor attributes.*

## The `raddb/` directory

The `raddb` directory contains all the configuration for FreeRadius.

The main configuration file for the server is `radiusd.conf`  This file loads the other configuration files via "include" statements.

`raddb` contains several subdirectories, such as:

`mods-available` directory contains sample configurations for all of the modules.

`mods-enabled` enabled modules; sometimes these are just softlinks to the equivalent file in `mods-avaliable`

`mods-config` extra configuration for modules

`sites-available` directory contains sample "virtual servers". Most of these will not be used. They exist as documentation and as examples of "best practices".

`sites-enabled` directory contains the configuration for "virtual servers" that are being used by the server.

## `radiusd.conf`

The `radiusd.conf` file contains the server configuration.

The "unlang" policy language can be used to create complex if / else policies. 

The client configuration is defined in `clients.conf`.

### Configuration File Syntax

Configuration files are UTF-8 text.

Line-orientated, everything has to be on a separate line.

A configuration item is an internal variable that has a name and holds a value.

```xml
variable = value
```

Variables have data types. The can be IP addresses, strings, number, etc.

Portions of the configuration can be grouped in sections:

```xml
texas {
  dallas = yes
  houston = no
  san_antonio = 70
}
```

A configuration file can load another configuration file via the `$INCLUDE` statement:

```xml
$INCLUDE other.conf
```

Or a directories. Note that dotfiles `.imadotfile.conf` are ignored but editor “backup” files with tildes `~backup.conf` are not. Also this is how you reuse variable definitions:

```xml
somedir = "hello"
$INCLUDE ${somedir}/foo/bar/baz
```

Variable references `${var_name}` act as macros, and expand when the server loads.

This is different from run-time expansion, which is done like this: `%{...}`

You can also use environment variables:

```xml
$ENV{variable}
```

And here’s convoluted one for fun: 

```xml
${reference1[name].reference2}
```

Sections can have instance names:

```xml
section-type instance-name {
    [ statements ]
}
```

For example, the `client` section is used to define information about a client. When multiple clients are defined, they are distinguished by their `instance-name`.

Booleans are weird in radius.

The boolean data type contains a true or false value. The values `yes`, `on`, and `1` evaluate to *true.* The values `no`, `off`, and `0` evaluate to *false*.

```xml
var = yes 
```

Delay is a data type. It contains fractional numbers, like `1.4`. These numbers are base 10. Usually they are used for timers. The resolution of delay is no more that microseconds, but usually in milliseconds.

A *word* string is composed of one word, without any surrounding quotes, such as `testing123`

Strings can have single or double quotes, or back-ticks.

The main difference between the single and double quoted strings is that the double quoted strings can be dynamically expanded.

The syntax `${…}`  is used for parse-time expansion and `%{…}` is used for run-time expansion.

### `checkrad`

The program to execute to go concurrency checks.

### `cleanup_delay`

The time to wait (in seconds) before “cleaning up” a reply that was sent to the NAS.

The request and replay are usually cached intenally for a short period of time after the reply is sent to the NAS. If the packets gets lost or something, the NAS might send a re-send the request.

If too low, then it’s useless.

If it’s too high, then the server will cache too many requests and some new requests may get block (see `max_requests`)

### `hostname_lookups`

Default is no, enabled it mean DNS requests which may take super long and block other requests.

### `libdir`

The libdir is where to find the `rlm_*` modules.

### `max_request_time`

Maximum seconds to handle a request.

### `max_requests`

The maximum number of requests of which the server keeps track

### `panic_action`

What to do when panic. Its a string.

### `pidfile`

Where to store the PID of the server. To kill it,

```xml
kill -HUP `cat /var/run/radiusd/radiusd.pid`
```

## Security Configuration

This directives go into a security section, like so:

```xml
security {
	...
}
```

I left out a lot but here are some interesting ones:

### `reject_delay`

Seconds to wait before sending an `Access-Reject`

Slows down DDoS attack and slows down brute force password cracking.

### `status_server`

Boolean. Whether to respond to Status-Server requests.

See also: `raddb/sites-available/status`

## ****Thread pool****

The thread pool is a long-lived group of threads that take turns (round-robin) handling any incoming requests.

### **`max_requests_per_server`**

Default is 0.

It says: There may be memory leaks or resource allocation problems with the server. If so, set this value to approximately 300 so that the resources will be cleaned up periodically. 

huh?

Also: '0' is a special value meaning 'infinity' or 'the servers never exit'.

## Virtual Servers

A virtual server is a (nearly complete) RADIUS server.

FreeRADIUS can run multiple virtual servers at the same time. 

Virtual servers can even proxy requests to each other.

The simplest way to define a virtual server would be to take all of the request processing sections from radius.conf ("authorize" , "authenticate", etc.) and wrap them in a "server {}" block.

```json
server foo {
		listen {
			ipaddr = 127.0.0.1
			port = 2000
			type = auth
		}

		authorize {
			update control {
				Cleartext-Password := "bob"
			}
			pap
		}

		authenticate {
			pap
		}
	}
```

Only certain sub-sections can appear in a virtual server section.

- `listen`
- `client`
- `authorize`
- `authenticate`
- `port-auth`
- `pre-proxy`
- `post-proxy`
- `preacct`
- `accounting`
- `session`

When a `listen` section is inside of a virtual server definition, it means that all requests sent to that IP/port will be processed through the virtual server. There can not be two `listen` sections with the same IP address and port number.

### Authorization Section

The name of this section is authorize for historical reasons, as earlier versions of the server did not have a `post-auth section`. A more accurate description of this section would be `pre-authentication`.

The authorize section processes `Access-Request` packets by normalizing the request, determining
which authentication method to use, and either setting the “known good” password (the valid password found in the database) for the user or informing the server that the request should be proxied.

Once the authorize section has finished processing the packet, the return code for the section is examined by the server. If the return code is `noop`, `notfound`, `ok`, or `updated`, then request processing continues.

If the return code is `handled`, then it is presumed that one of the modules set the contents of the reply, and the server sends the reply message. 

Otherwise, the server treats the authentication as being rejected and runs the `post-auth` section.

If the authentication has not been rejected, then the server continues processing the request by
searching for the `Auth-Type` attribute. Then the named sub-section of `authenticate` is executed. 

The authorization section is starts when the server receives an `Access-Request` packet.

First preprocesses (`hints` and `huntgroups`), then does `realms`, then finally the `users` file.

The order of the realm modules will determine the order in which a matching realm is found. 

```python
authorize {
	filter-username
  preprocess
  # operator-name
  # cui
  # authlog
  chap
  mschap
	digest
	# mimax
	# IPASS
	suffix 
  ntdomain
  eap {
		ok = return
  }
  # unix
  files
  -sql
  # smbpasswd
  - ldap
  # daily
	expiration
	logintime
	pap
	Autz-Type Status-Server {
  
	}
}
```

`filter-username` this is a policy that sanitizing usernames in requests by removing garbage like spaces and invalid characters. If it appears invalid, the request is rejected. 

See `policy.d/filter` for the definition of the `filter-username` policy.

`preprocess` sanitizes the non-standard format attributes in the request and makes them standard.

(I think that this mean like how a mac address can be written in like 6953 different ways)

If `CUI` is used then `Operator-Name` is required to be set for CUI generation.

Not sure what this means or why it is true.

`auth_log` generates a log of authentication requests.

`chap` sets `Auth-Type := CHAP` for requests which contain a `CHAP-Password` attribute.

When users log in with an `MS-CHAP-Challenge` attribute for authentication, the `mschap` module finds the `MS-CHAP-Challenge attribute` and adds `Auth-Type := MS-CHAP` to the request, which causes the server to then use the `mschap` module for authentication.

`digest`is for SIP (session initiation protocol) servers. (VoIP). who cares.

WiMAX refers to implementations of the [IEEE 802.16](https://en.wikipedia.org/wiki/IEEE_802.16) family of wireless-networks standards created by the WiMAX Forum. The WiMAX specification states that the `Calling-Station-Id` is 6 octets of the MAC.

This definition conflicts with with all common RADIUS practices. Uncommenting the `wimax` module here means that it will fix the `Calling-Station-Id` attribute to the normal format. whatever.

<aside>
💡 ***Did you know***: WiMAX was sometimes referred to as "Wi-Fi on steroids"[[5]](https://en.wikipedia.org/wiki/WiMAX#cite_note-5) lol

</aside>

Looks for `IPASS`-style 'realm/' and, if not found, looks for '@realm' and decides whether or not to proxy based on those results.

When using multiple kinds of realms, set `ignore_null = yes` for all of them. Otherwise, if the first style of realm doesn’t match, then the other styles won’t be checked.

The `eap` module takes care of `EAP-MD5`, `EAP-TLS`, and `EAP-LEAP` authentication.

`unix` pulls crypt’d passwords from `/etc/passwd` or `/etc/shadow` using the system APIs to get the password

`files` reads the `radbb/users` file.

The `-sql`  module looks in an SQL database. I don’t know why it gets a dash, but `-ldap` gets one too, so maybe its a theme.

... skipping a couple idc

`daily` enforces daily limits on time spent logged in. I think it requires `expiration` and `logintime`

If no other module has claimed responsibility for authentication, then try `pap`. This process allows the other modules listed above to only add a "known good" password to the request and nothing else. The PAP module will then see that password and use it to do PAP authentication. This module should be listed last so that the other modules get a chance to set `Auth-Type` for themselves.

If `status_server = yes`, then `Status-Server` messages are passed through the following section and **only** the following section.

```python
Autz-Type Status-Server {

}
```

### Authentication Section

This section lists those modules that are available for authentication. 

Note that the order of modules listed below does **not** mean “try each module in order”. 

Instead, a module from the `authorize` section adds a configuration attribute `Auth-Type := FOO`. That authentication type is then used to pick the appropriate module from the list below.

The authenticate section is only used when the server is authenticating requests locally and is
bypassed completely when proxying

This section is different from each of the other sections: it is composed of a series of subsections, only one of which is executed.

The `Auth-Type` attribute can also refer to a module (e.g. `eap`) instead of a subsection, in which case that module, and only that module, is processed.

A simple example:

```python
authenticate {
	Auth-Type PAP {
		pap
	}
	Auth-Type MS-CHAP {
		mschap
	}
	Auth-Type CHAP {
		chap
	}
	eap
}
```

In general, the `Auth-Type` attribute **should not** be set. The server will figure it out on its own and will do the right thing. The above is just defining `Auth-Types` but not actually setting the attribute.

Do not put `unlang` configurations into the `authenticate` section. Put them in the `post-auth` section instead. That’s what the `post-auth` section is for.

### Pre-accounting Section

This section decides which accounting type to use.

Session start times are **implied** in RADIUS. The NAS never sends a "start time". Instead, it sends a start packet, **possibly** with an Acct-Delay-Time.

You can create a start time with the following code:

```python
update request {
	FreeRADIUS-Acct-Session-Start-Time = "%{expr: %l - %{%{Acct-Session-Time}:-0} - %{%{Acct-Delay-Time}:-0}}"
}
```

`acct_unique` ensures that a semi-unique identifier is available for every request because many NAS boxes are broken.

The `files`  module reads the `acct_user` file.

### Accounting Section

The `accounting` section Iogs the accounting data.

`cui` updates accounting packets by adding the `CUI` attribute from the corresponding `Access-Accept`

Used for when the NAS doesn’t support CUI themselves.

`detail` creates a ``detail``'ed log of the packets.

`-sql` logs traffic into a SQL db.

`exec` is for `Exec-Program` and `Exec-Program-Wait`.

### Post-authentication Section

Once it is **verified** that the user has been authenticated, there are additional steps that can be taken.

### Pre-proxy Section

### Post-proxy Section