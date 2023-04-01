Install and Configure SpamAssassin with Dovecot and Postfix on Ubuntu Server

Follow these steps to set up an automated spam filter on your Linux Ubuntu web server using SpamAssassin, Dovecot, and Postfix:

## Step 1: Install Postfix, Dovecot, and SpamAssassin

Update the package lists and install Postfix, Dovecot, SpamAssassin, and spamc:

```shell
sudo apt update
sudo apt install postfix dovecot-imapd dovecot-pop3d spamassassin spamc
```

## Step 2: Enable and start the SpamAssassin service

Enable the SpamAssassin service to start automatically at boot, and then start the service:

```shell
sudo systemctl enable spamassassin
sudo systemctl start spamassassin
```
## Step 3: Configure SpamAssassin settings

Edit the SpamAssassin configuration file `/etc/spamassassin/local.cf` using a text editor (e.g., nano or vim):

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

These settings will mark emails with a spam score of 5.0 or higher by adding "[SPAM]" to the subject. You can adjust the `required_score` value as needed. Save the changes and exit the editor.

## Step 4: Configure Postfix to use SpamAssassin

Edit the Postfix configuration file `/etc/postfix/master.cf`:

```shell
sudo nano /etc/postfix/master.cf
```

Add the following lines to the end of the file:

```shell
spamassassin unix -     n       n       -       -       pipe
  user=spamd argv=/usr/bin/spamc -f -e
  /usr/sbin/sendmail -oi -f ${sender} ${recipient}
```

Save the changes and exit the editor.

## Step 5: Restart Postfix

Restart the Postfix service to apply the changes:

```shell
sudo systemctl restart postfix
```

## Step 6: Configure Dovecot

Edit the Dovecot configuration file `/etc/dovecot/conf.d/90-sieve.conf`:

```shell
sudo nano /etc/dovecot/conf.d/90-sieve.conf
```

Uncomment or add the following lines:

```shell
plugin {
  sieve = ~/.dovecot.sieve
  sieve_default = /etc/dovecot/sieve/default.sieve
  sieve_dir = ~/sieve
  sieve_global_dir = /etc/dovecot/sieve/global/
}
```

Save the changes and exit the editor.

Create a default Sieve script `/etc/dovecot/sieve/default.sieve`:

```shell
sudo nano /etc/dovecot/sieve/default.sieve
```

Add the following content to the file:

```shell
require ["fileinto"];
if header :contains "X-Spam-Flag" "YES" {
  fileinto "Junk";
}
```

This script will automatically move messages marked as spam to the "Junk" folder. Save the changes and exit the editor.

Compile the Sieve script:

```shell
sudo sievec /etc/dovecot/sieve/default.sieve
```

Step 7: Restart Dovecot

Restart the Dovecot service to apply the changes:

```shell
sudo systemctl restart dovecot
```

SpamAssassin will now automatically process incoming emails on your server and mark them as spam based on your configuration. The Dovecot Sieve script will move the messages marked as spam to the "Junk" folder for each user. You can further refine the configuration by editing `/etc/spamassassin/local.cf` and adjusting the settings as needed.

Additionally, users can create their own Sieve scripts to manage their spam filters and email organization based on their preferences. To do this, they should place their scripts in their `~/sieve` directory and compile them using the `sievec` command.
