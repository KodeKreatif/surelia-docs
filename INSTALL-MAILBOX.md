# Mailbox

```
    [RELAY]--(smtp/qmqp)>[MAILBOX] 
```

This setup is for collecting emails and put them into mailboxes. This setup supports multi domains out of the box. We have a directory containing all domains and mailboxes. The structure looks like this:
```
~sureliabox/mails/<DOMAIN>/<USER>@<DOMAIN>/<Maildir>
```
`sureliabox` is the Unix user name used to keep the mailbox on the storage. The `<Maildir>` is the mailbox format (https://en.wikipedia.org/wiki/Maildir) used in Surelia.

## Information needed
1. Accepted domains (`INFO01`). This is a list of domains accepted in the system.
1. Unix user which stores the mailboxes (`INFO021). The user should disable the shell, so one should not be able to login with this user name.

## Examples
### INFO01
```
domain1.com
domain.net
```
### INFO02
```
sureliabox
```

## Packages
* ucspi-tcp
* surelia-qmail (https://github.com/mdamt/netqmail/tree/debian)
* qmail-wildcard-store (https://github.com/KodeKreatif/qmail-wildcard-store)

## Steps
### Setup Unix user
Create the Unix user with standard Unix commands. Disable the shell
```
sudo adduser --system --shell /bin/false --disabled-password --disabled-login sureliabox
```
### Setup mailboxes
Go to shell as the `sureliabox` by doing this as root:
```
su -l -s /bin/bash sureliabox
```

Then create the .qmail mailbox configuration. Do this in the home directory of the `sureliabox` and as `sureliabox`. Doing this as other users will make the emails undeliverable to the mailbox:
```
cd
echo "|/usr/local/bin/qmail-wildcard-store mails/domain1.com" > .qmail-domain1com-default
chmod 644 .qmail-domain1com
mkdir -p mails/domain1.com
```
Repeat for each domain in `INFO01`. Please note the patterns.

### Setup qmail configuration
Add the domain name into qmail's configuration. Do this as root:
```
echo "domain1.com:sureliabox-domain1com" >> /etc/qmail/virtualdomains
echo domain1.com >> /etc/qmail/rcpthosts
```
Make sure you don't truncate the file by typing `>` instead of `>>`.

### Setup incoming SMTP server
1. Create SMTP service file at `/etc/qmail/services/smtp/run` to be run by `supervise` program from `daemontools` package
```
#!/bin/sh

QMAILDUID=`id -u qmaild`
NOFILESGID=`id -g qmaild`
MAXSMTPD=`cat /var/lib/qmail/control/concurrencyincoming`
LOCAL=`head -1 /var/lib/qmail/control/me`


if [ ! -f /etc/qmail/rcpthosts ]; then
    echo "No /etc/qmail/rcpthosts!"
    echo "Refusing to start SMTP listener because it'll create an open relay"
    exit 1
fi

tcprules /etc/qmail/tcp.smtp.cdb /tmp/tcp.smtp < /etc/qmail/tcp.smtp

exec softlimit -m 7000000 \
    tcpserver -v -R -l "$LOCAL" -x /etc/qmail/tcp.smtp.cdb -c "$MAXSMTPD" \
        -u "$QMAILDUID" -g "$NOFILESGID" 0 smtp qmail-smtpd 2>&1

```
2. Make sure it is runnable
```
chmod +x /etc/qmail/services/smtp/run
```
3. Create the corresponding systemd service. So it can be run by standard systemd commands. The file is `/etc/systemd/service/surelia-smtpd.service`
```
[Unit]
Description=Surelia SMTP server
After=network.target auditd.service

[Service]
ExecStart=/usr/bin/supervise /etc/qmail/service/smtpd
ExecReload=svc -h /etc/qmail/service/smtpd/
KillMode=process
Restart=on-failure

[Install]
Alias=smtp.service


```
