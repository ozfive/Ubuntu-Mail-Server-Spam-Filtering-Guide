# Install and Configure SpamAssassin with Sendmail on Ubuntu Server

Follow these steps to set up an automated spam filter on your Linux Ubuntu web server using SpamAssassin and Sendmail:
Step 1: Install Sendmail, SpamAssassin, and related dependencies

Update the package lists and install Sendmail, SpamAssassin, and spamc:

```shell
sudo apt update
sudo apt install sendmail sendmail-bin spamassassin spamc
```

## Step 2: Enable and start the SpamAssassin service

Enable the SpamAssassin service to start automatically at boot, and then start the service:

```shell
sudo systemctl enable spamassassin
sudo systemctl start spamassassin
```

## Step 3: Configure SpamAssassin settings

Edit the SpamAssassin configuration file /etc/spamassassin/local.cf using a text editor (e.g., nano or vim):

```shell
sudo nano /etc/spamassassin/local.cf
```

Add or modify the following settings in the file:

```shell
rewrite_header Subject [SPAM]
required_score 5.0
use_bayes 1
bayes_auto_learn 1
```

These settings will mark emails with a spam score of 5.0 or higher by adding "[SPAM]" to the subject. You can adjust the required_score value as needed. Save the changes and exit the editor.
Step 4: Configure Sendmail to use SpamAssassin

Create a new Sendmail configuration file, /etc/mail/spamassassin/spamassassin.m4:

```shell
sudo mkdir -p /etc/mail/spamassassin
sudo nano /etc/mail/spamassassin/spamassassin.m4
```

Add the following content to the file:

```shell
dnl
dnl Process mail through spamassassin
dnl
define(`confMILTER_MACROS_ENVFROM', `i, {auth_type}, {auth_authen}, {auth_ssf}, {auth_author}, {mail_mailer}, {mail_host}, {mail_addr}, r, b, v, Z')dnl
INPUT_MAIL_FILTER(`spamassassin', `S=local:/var/spool/spamassassin/spamass.sock, F=, T=C:15m;S:4m;R:4m;E:10m')dnl
```

Save the changes and exit the editor.

Edit the Sendmail configuration file /etc/mail/sendmail.mc:

```shell
sudo nano /etc/mail/sendmail.mc
```

Add the following line before the MAILER_DEFINITIONS section:

```shell
include(`/etc/mail/spamassassin/spamassassin.m4')dnl
```

Save the changes and exit the editor.

## Step 5: Build the Sendmail configuration

Generate the Sendmail configuration file /etc/mail/sendmail.cf:

```shell
sudo sendmailconfig
```

Press Y and Enter to accept the default answers when prompted.

## Step 6: Configure and start the spamd daemon

Edit the SpamAssassin daemon configuration file /etc/default/spamassassin:

```shell
sudo nano /etc/default/spamassassin
```

Change the following settings:

```shell
ENABLED=1
OPTIONS="--create-prefs --max-children 5 --helper-home-dir --socketpath=/var/spool/spamassassin/spamass.sock --socketowner=debian-spamd --socketgroup=debian-spamd --socketmode=0660"
```

Save the changes: Ctrl-x then Enter
