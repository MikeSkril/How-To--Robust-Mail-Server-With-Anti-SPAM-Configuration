## Introduction
This tutorial will teach you how to set up your own robust email server. We are focusing on a small personal server with up to a few email accounts. After following this guide, you will have a fully functional mail server and you can connect with your favourite client to access, read and send emails. The Anti-Spam configuration will drop unwanted messages.

This tutorial will use **yourdomain.com** as domain name and **mail.yourdomain.com** as hostname for our mail server. The desired email address will be **yourname@yourdomain.com**. We assume that our server has the IP address **1.2.3.4**.

### Software and technologies used
* Postfix v2.9.6 as SMTP server
* Dovecot v2.0.19 as IMAP server
* We will use Unix user accounts and tunnel the SASL authentication through TLS
* Postgrey v1.34 - to reject spam from the beginning
([more about postgrey]( http://postgrey.schweikert.ch/))
* SPF (Sender Policy Framework) validating to reduce spam
([more aboutSPF](https://www.digitalocean.com/community/articles/how-to-use-an-spf-record-to-prevent-spoofing-improve-e-mail-reliability))
* SPF and [DMARC](http://dmarc.org/) DNS entry to prevent spoofing
* DKIM (Domain Keys Identified Mail) to sign our email messages
([moreabout DKIM](http://www.dkim.org/))

## Prerequisites

### Personal
Every step will be explained in the tutorial and you will get it running even with minimal Unix knowledge. Nevertheless, you should be used to work on the command-line and know how to use a text editor. Furthermore it takes some Unix skills to administrate the working mail server.
You're invited to follow the links in this tutorial to learn more about the software and techniques used.

### System

* A VPS running Ubuntu 12.04 or 14.04 (setup will be similar on any Debian based distribution). ([Get a VPS here](https://www.digitalocean.com/?refcode=79aec8435127))
* Your own FQDN domain name.

We will be working on a `root` shell and the tutorial will use `vim` as text editor.

## Preparing our system
#### Setting up the host name
~~~~
echo mail.yourdomain.com > /etc/hostname
~~~~

#### Adding our domain to /etc/hosts
~~~~
vim /etc/hosts
~~~~

We add **yourdomain.com** and **mail.yourdomain.com** in the first line.
~~~~
127.0.0.1 localhost yourdomain.com mail.yourdomain.com
~~~~

#### Setting up the mailname
~~~~
echo yourdomain.com > /etc/mailname
~~~~

That's the name that will appear on the right side of the '@' in our email address. In this case, **yourdomain.com**.

### Installing required packages

#### Updating the system
~~~~
apt-get update && apt-get upgrade
~~~~

#### Installing
~~~~
apt-get install postfix postfix-policyd-spf-perl postgrey dovecot-core dovecot-imapd opendkim opendkim-tools && postfix stop
~~~~

Hit ‘No’ if asked to create a SSL certificate. Choose `Internet Site` and press 'ok' 2 times when asked by the postfix installer.

## Setting up DNS
We assume that our domain is already setup in the DNS control panel and we see the default DNS records.
![DNS default](https://skrilnetz.net/wp-content/uploads/2014/06/itfXnF01.png)

### Setting up the A record

<a href="https://skrilnetz.net/wp-content/uploads/2014/06/geFtQf91.png"><img class="alignnone size-medium wp-image-140" src="https://skrilnetz.net/wp-content/uploads/2014/06/geFtQf91-600x211.png" alt="geFtQf9" width="600" height="211" /></a>

(There is a 'dot' after the domain name)

### Setting up the MX record

<a href="https://skrilnetz.net/wp-content/uploads/2014/06/c7Apht11.png"><img class="alignnone size-medium wp-image-155" src="https://skrilnetz.net/wp-content/uploads/2014/06/c7Apht11-600x171.png" alt="c7Apht1[1]" width="600" height="171" /></a>

(There is a 'dot' after the domain name)

### Setting up the SPF record

We create a new TXT record
~~~~
"v=spf1 a mx ip4:1.2.3.4 -all"
~~~~

<a href="https://skrilnetz.net/wp-content/uploads/2014/06/cyFoSjh11.png"><img class="alignnone size-medium wp-image-157" src="https://skrilnetz.net/wp-content/uploads/2014/06/cyFoSjh11-600x133.png" alt="cyFoSjh[1]" width="600" height="133" /></a>

The SPF record protects from email spoofing. It will simply tell other mail servers that only our server is authorized to send emails for **yourdomain.com** ([more aboutSPF](https://www.digitalocean.com/community/articles/how-to-use-an-spf-record-to-prevent-spoofing-improve-e-mail-reliability)).

### Setting up the DMARC record

We create a new TXT record named **`_dmarc.yourdomain.com.`**
(There is a ‘dot’ after the domain name)
~~~~
"v=DMARC1; p=quarantine; rua=mailto:postmaster@yourdomain.com"
~~~~

### Now we will setup the hostname for the PTR record

<a href="https://skrilnetz.net/wp-content/uploads/2014/06/Gg6s1vv1.png"><img class="alignnone size-medium wp-image-142" src="https://skrilnetz.net/wp-content/uploads/2014/06/Gg6s1vv1-600x255.png" alt="Gg6s1vv" width="600" height="255" /></a>

### Our configuration should look similar to this

<a href="https://skrilnetz.net/wp-content/uploads/2014/06/o1BA9Fy1.png"><img class="alignnone size-medium wp-image-143" src="https://skrilnetz.net/wp-content/uploads/2014/06/o1BA9Fy1-600x282.png" alt="o1BA9Fy" width="600" height="282" /></a>

It will take a while to propagate the new configuration throughout the entire internet.

## Generating SSL certificates

There are different ways to generate a SSL certificate. The tutorial will use a self signed certificate but we could also use a [CAcert](http://en.wikipedia.org/wiki/Certificate_authority), which can be obtained for free from [startssl](https://www.startssl.com).
Either ways, the connection from our email client to the mail server will be encrypted.

### Creating the certificate
~~~~
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/mail.yourdomain.key -out /etc/ssl/certs/mail.yourdomain.pem
~~~~

We will be asked to answer a few questions. It is important that we enter **mail.yourdomain.com** as `Common name`.
~~~~
Generating a 2048 bit RSA private key
..........+++
..................................................................+++
writing new private key to '/etc/ssl/private/mail.yourdomain.key'
- -----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
- -----
Country Name (2 letter code) [AU]:AT
State or Province Name (full name) [Some-State]:Styria
Locality Name (eg, city) []:Vienna
Organization Name (eg, company) [Internet Widgits Pty Ltd]:
Organizational Unit Name (eg, section) []:
Common Name (e.g. server FQDN or YOUR name) []:mail.yourdomain.com
Email Address []:postmaster@yourdomain.com
~~~~

## Postfix Configuration
### Backing up the configuration files

~~~~
cp /etc/postfix/master.cf /etc/postfix/master.cf_orig &&
cp /etc/postfix/main.cf /etc/postfix/main.cf_orig
~~~~

### Configuration in main.cf
~~~~
vim /etc/postfix/main.cf
~~~~

We empty the file and add our configuration

~~~~
#Base config
myhostname = mail.yourdomain.com
myorigin = /etc/mailname
mydestination = $myhostname, $mydomain, localhost, localhost.$mydomain
relayhost =
mynetworks = 127.0.0.0/8 [::ffff:127.0.0.0]/104 [::1]/128
mailbox_size_limit = 0
recipient_delimiter = +
inet_interfaces = all
relay_domains = $mydestination
syslog_name=postfix/submission
#Aliases / Recipients
alias_maps = hash:/etc/aliases
alias_database = hash:/etc/aliases
local_recipient_maps = proxy:unix:passwd.byname $alias_maps
#SSL/TLS
smtpd_tls_cert_file=/etc/ssl/certs/mail.yourdomain.pem
smtpd_tls_key_file=/etc/ssl/private/mail.yourdomain.key
smtpd_use_tls=yes
smtpd_tls_auth_only = yes
smtpd_tls_session_cache_database = btree:${data_directory}/smtpd_scache
smtp_tls_session_cache_database = btree:${data_directory}/smtp_scache
smtpd_tls_security_level=may
smtp_tls_security_level=may
smtpd_tls_protocols = !SSLv2, !SSLv3
smtpd_tls_wrappermode=no
smtpd_sasl_type=dovecot
smtpd_sasl_path=private/auth
smtpd_sasl_auth_enable=yes
milter_macro_daemon_name=ORIGINATING
#Security and Anti-Spam cinfig
policy-spf_time_limit = 3600s
smtpd_helo_required = yes

smtpd_recipient_restrictions =
 reject_non_fqdn_recipient
 reject_unknown_recipient_domain
 permit_mynetworks
 permit_sasl_authenticated
 reject_unauth_destination
 check_policy_service unix:private/policy-spf
 check_policy_service inet:127.0.0.1:10023

smtpd_helo_restrictions =
 permit_mynetworks
 reject_non_fqdn_helo_hostname
 reject_invalid_helo_hostname

smtpd_client_restrictions=
 permit_mynetworks
 permit_sasl_authenticated
 reject_unknown_client_hostname

smtpd_data_restrictions =
 reject_unauth_pipelining

#DKIM
milter_default_action = accept
milter_protocol = 6
smtpd_milters = inet:localhost:8891
non_smtpd_milters = inet:localhost:8891
~~~~

### Configuration in master.cf
~~~~
vim /etc/postfix/master.cf
~~~~

This line must be active:

~~~~
smtp inet n - - - - smtpd
~~~~

We uncomment the following:

~~~~
submission inet n - - - - smtpd
~~~~

and add those 2 lines at the end of the file (there are 2 whitespaces
before the second line).

~~~~
policy-spf unix - n n - - spawn
  user=nobody argv=/usr/sbin/postfix-policyd-spf-perl
~~~~

More information about the postfix configuration parameters can be found [here](http://www.postfix.org/postconf.5.html).

## Dovecot Configuration

### Backing up the configuration file

~~~~
cp /etc/dovecot/dovecot.conf /etc/dovecot/dovecot.conf_orig
~~~~

### Configuration in dovecot.conf

~~~~
vim /etc/dovecot/dovecot.conf
~~~~

We empty the file and add our configuration.

~~~~
disable_plaintext_auth = no
mail_privileged_group = mail
mail_location = mbox:~/mail:INBOX=/var/mail/%u
userdb {
driver = passwd
}

passdb {
args = %s
driver = pam
}
protocols = " imap"

protocol imap {
mail_plugins = " autocreate"
}
plugin {
autocreate = Trash
autocreate2 = Sent
autosubscribe = Trash
autosubscribe2 = Sent
}

service auth {
unix_listener /var/spool/postfix/private/auth {
group = postfix
mode = 0660
user = postfix
}
}
ssl=required
ssl_cert = </etc/ssl/certs/mail.yourdomain.pem
ssl_key = </etc/ssl/private/mail.yourdomain.key
~~~~
More information about the dovecot configuration parameters can be found [here](http://wiki2.dovecot.org/).

### Restarting dovecot

~~~~
service dovecot restart
~~~~

## Adding our mail user

Now we add the user for our mail account. In this example our email
address will be **yourname@yourdomain.com**.
We can add as many users/email addresses as we want.

~~~~
adduser --gecos '' --shell /bin/false yourname
~~~~

## Setting Aliases

~~~~
vim /etc/aliases
~~~~

The configuration can be adapted to your needs.
Here we tell postfix to forward all messages addressed to **aliases** to the mailbox of user: **yourname**.

~~~~
mailer-daemon: postmaster
postmaster: root
nobody: root
hostmaster: root
usenet: root
news: root
webmaster: root
www: root
ftp: root
abuse: root
noc: root
security: root
admin: root
root: yourname
~~~~

### Compiling the alias file

~~~~
newaliases
~~~~

## Setting up DKIM (Domain Keys Identified Mail)

### Creating the directory and files

~~~~
mkdir /etc/opendkim
~~~~

~~~~
vim /etc/opendkim/KeyTable
~~~~

We enter the following line (all in the first line):
~~~~
default._domainkey.mail.yourdomain.com mail.yourdomain.com:default:/etc/opendkim/default.private
~~~~

~~~~
vim /etc/opendkim/SigningTable
~~~~

We enter the following line:
~~~~
*@yourdomain.com default._domainkey.mail.yourdomain.com
~~~~

### Generating the key pair
~~~~
opendkim-genkey -s default -d mail.yourdomain.com -D /etc/opendkim
~~~~

### Changing ownership of the private key file

~~~~
chown opendkim:opendkim /etc/opendkim/default.private
~~~~

### Configuration in opendkim

~~~~
vim /etc/default/opendkim
~~~~

### We add the line below to the configuration
~~~~
SOCKET="inet:8891@localhost"
~~~~

### Configuration in opendkim.conf

~~~~
vim /etc/opendkim.conf
~~~~

### We fill the file with our configuration
~~~~
Syslog yes
SyslogSuccess Yes
LogWhy yes

UMask 002

KeyTable refile:/etc/opendkim/KeyTable
SigningTable refile:/etc/opendkim/SigningTable
Selector default
X-Header no
SignatureAlgorithm rsa-sha256

Canonicalization relaxed/simple
Mode sv
AutoRestart Yes
AutoRestartRate 5/1h
InternalHosts 127.0.0.1, localhost, yourdomain.com, mail.yourdomain.com

OversignHeaders From
~~~~

### Setting the DKIM DNS record

~~~~
cat /etc/opendkim/default.txt
~~~~

This is our public key which will be used to verify the signature in our emails.

~~~~
default._domainkey IN TXT "v=DKIM1; k=rsa;
p=MIGfHL0GCSqGSIb3DQESYJFOA4GNADCBiQKBgQDS+vPyWRs7w32xomf2oZIexmS2TuQAXKPiQ3AXn4j25NOReXdgKxIqAwl3O7dQtgluWw+TH85Mrbmx5UgwaaLenj9cfe2IRvx7hvkj7+6i0XQqrWqZlMw+QAJxAGhfa/GVTYa+/7PFWfXLoqoBW5arE+wO20O2uw5Ik62HjkKZbQIDAQAB" ; ----- DKIM key default for mail.yourdomain.com
~~~~

#### Now we add a new TXT record:

As 'name' we put (there is a 'dot' at the end):

~~~~
default._domainkey.mail.yourdomain.com.
~~~~

As 'text' we add (use your public key from /etc/opendkim/default.txt):
~~~~
"v=DKIM1; k=rsa; p=MIGfHL0GCSqGSIb3DQESYJFOA4GNADCBiQKBgQDS+vPyWRs7w32xomf2oZIexmS2TuQAXKPiQ3AXn4j25NOReXdgKxIqAwl3O7dQtgluWw+TH85Mrbmx5UgwaaLenj9cfe2IRvx7hvkj7+6i0XQqrWqZlMw+QAJxAGhfa/GVTYa+/7PFWfXLoqoBW5arE+wO20O2uw5Ik62HjkKZbQIDAQAB"
~~~~

<a href="https://skrilnetz.net/wp-content/uploads/2014/06/AFJ9uPk1.png"><img class="alignnone size-medium wp-image-144" src="https://skrilnetz.net/wp-content/uploads/2014/06/AFJ9uPk1-600x38.png" alt="AFJ9uPk" width="600" height="38" /></a>

It will take a while to propagate the new configuration throughout the entire internet.

## Starting postfix and restarting opendkim

~~~~
postfix start && service opendkim restart
~~~~

## Connecting with a Client

Now we will connect with our email client. In the tutorial we use Thunderbird as client.
Just add a new email account in Thunderbird and it will auto-detect the servers configuration.

The configuration should look like this:

<a href="https://skrilnetz.net/wp-content/uploads/2014/06/5hjbgbf1.png"><img class="alignnone size-full wp-image-145" src="https://skrilnetz.net/wp-content/uploads/2014/06/5hjbgbf1.png" alt="5hjbgbf" width="589" height="425" /></a>

<a href="https://skrilnetz.net/wp-content/uploads/2014/06/N2spZXd1.png"><img class="alignnone size-medium wp-image-146" src="https://skrilnetz.net/wp-content/uploads/2014/06/N2spZXd1-600x144.png" alt="N2spZXd" width="600" height="144" /></a>

## Testing the mail server

It's time to test our server. Let's send and receive emails within Thunderbird.
We can observe the ongoing transactions and spot possible errors by monitoring our syslog.
~~~~
tail -f /var/log/syslog
~~~~

We can test our SPF and DKIM configuration [here](http://www.appmaildev.com/en/dkim/) and DMARC [here](http://dmarc.org/).

Some more testing can be done [here](https://www.wormly.com/test_smtp_server), or [here](http://www.emailsecuritygrader.com/).

### Postgrey in action

Pass:
~~~~
May 11 16:26:38 mail postgrey[21408]: action=pass, reason=triplet found,delay=1290, client_name=another.server.com, client_address=4.2.3.1,
sender=user@another.server.com, recipient=yourname@yourdomain.com
~~~~

Reject at first try:

~~~~
May 10 12:42:33 mail postgrey[21408]: action=greylist, reason=new,
client_name=ns1.qubic.net, client_address=208.69.177.116,
sender=daemon@dk.elandsys.com, recipient=yourname@yourdomain.com 10
12:42:33 mail postfix/submission/smtpd[24718]: NOQUEUE: reject: RCPT
from ns1.qubic.net[208.69.177.116]: 450 4.2.0 < yourname@yourdomain.com
: Recipient address rejected: Greylisted, see http://postgrey.schweikert.ch/help/yourdomain.com.html;
from=<daemon@dk.elandsys.com> to=< yourname@yourdomain.com > proto=ESMTP helo=
~~~~

### DKIM in action

Sending an email:
~~~~
May 12 06:25:12 mail opendkim[1508]: 423AA62DC2: DKIM-Signature header
added (s=default, d=mail.yourdomain.com)
~~~~

### SPF in action

Pass:
~~~~
May 10 14:28:47 mail postfix/policy-spf[1085]: Policy action=PREPEND
Received-SPF: pass (verifier.port25.com: 96.244.219.19 is authorized to
use 'auth-results@verifier.port25.com' in 'mfrom' identity (mechanism
'a' matched)) receiver=yourdomain.com; identity=mailfrom;
envelope-from="auth-results@verifier.port25.com";
helo=verifier.port25.com; client-ip=96.244.219.19
~~~~

Reject:
~~~~
May 12 16:15:25 mail postfix/policy-spf[20469]: Policy action=550 Please see http://www.openspf.net/Why?s=mfrom;id=someone 4yourdomain.com;ip=184.72.226.23;r=yourdomain.com
May 12 16:15:25 mail postfix/submission/smtpd[20463]: NOQUEUE: reject:
RCPT from node-mec2.wormly.com[184.72.226.23]: 550 5.7.1 <yourname@
yourdomain.com >: Recipient address rejected: Please see http://www.openspf.net/Why?s=mfrom;id=someone%
yourdomain.com;ip=184.72.226.23;r=yourdomain.com;
from=<someone@yourdomain.com> to=<yourname@yourdomain.com> proto=ESMTP
helo=
~~~~

## Congratulations

You did it! You setup our own mail server and do not depend anymore on
any email provider.

Our anti-spam setup should filter out 99% of all unwanted messages and
won't drop a legitimate email. On the other hand, messages from our
server should not be declared as spam as it has a valid SPF record and
we are signing our messages.

Still getting a few spam messages and not tired yet? Go on over [here](https://www.digitalocean.com/community/articles/how-to-install-and-setup-spamassassin-on-ubuntu-12-04).

## Useful for troubleshooting

### Verify DNS

Check reverse entry
~~~~
dig -x 1.2.3.4 +short
~~~~

Check MX record
~~~~
dig yourdomain.com MX +short
~~~~

### Verify if ports are listening

~~~~
netstat -tulpen|grep LISTEN
~~~~

### Enable verbose logging for postfix in /etc/postfix/master.cf
Just add the `v` at the end of the smtp line.

~~~~
smtp inet n - - - - smtpd -v
~~~~

### Check the syslog with
~~~~
tail -f /var/log/syslog
~~~~
and observe the logging.
