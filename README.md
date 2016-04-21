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
