# Relay only setup

```

INTERNET --(smtp)> [ RELAY ] -->(qmqp) [INTERNAL SURELIA SERVERS]

```

All emails are received via SMTP by the relay. Then all emails will be forwarded immediately to the QMQP servers internally. The relay doesn't keep the emails, so when internal servers are down, all emails will be rejected temporarily.

## Information needed
1. Relays' hostname (`INFO1`). The hostname of the relay.
1. List of internal surelia servers (`INFO2`). All mails will be forwarded to these servers.
1. List of domain names accepted by internal servers (`INFO3`). Other than these domain names will be rejected at SMTP level.

## Examples
This is a list of example information needed as described above.
### INFO1
```
relay.domain.com
```
### INFO1
```
internal.domain.com
```
### INFO3
```
domain.info
domain.com
```

## Packages

1. ucspi-tcp
1. surelia-qmail-relay-only

## Steps

### Setup surelia-qmail
1. Install `surelia-qmail-relay-only`
1. Setup `/etc/qmail/rcpthosts` fill with `INFO03`, one domain per line
```
domain.info
domain.com
```
1. Setup `/etc/qmail/me` fill with `INFO01`
```
relay.domain.com
```
1. Setup `/etc/qmail/qmqpservers` fill with `INFO02`, one host per line
```
internal.domain.com
```

### Setup tcpserver and rblsmtpd
tcpserver elevates a program to become an internet server. Here we are using tcpserver to make qmail's qmail-smtpd program to be an SMTP server.
By using `surelia-qmail-relay-only`, you already have them setup. The systemd compatible service is installed at `/etc/systemd/system/surelia-smtpd.service` and internally 
it invokes `supervise` service located at `/etc/qmail/service/smtpd`.


