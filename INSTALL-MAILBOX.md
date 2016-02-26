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
### Turn on qmail
