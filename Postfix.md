# Install and Configure SpamAssassin with Postfix on Ubuntu Server

Follow these steps to set up an automated spam filter on your Linux Ubuntu web server using SpamAssassin and Postfix:

## Step 1: Install SpamAssassin and related dependencies

Update the package lists and install SpamAssassin and spamc:

```shell
sudo apt update
sudo apt install spamassassin spamc
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

These settings will mark emails with a spam score of 5.0 or higher by adding "[SPAM]" to the subject. You can adjust the required_score value as needed. Save the changes and exit the editor.

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
SpamAssassin will now automatically process incoming emails on your server and mark them as spam based on your configuration. You can further refine the configuration by editing `/etc/spamassassin/local.cf` and adjusting the settings as needed.

Optionally, you can set up a user-based spam filter using the Sieve mail filtering language and the Dovecot IMAP server. This allows creating custom filtering rules for each user on your server. To set up Sieve, follow the Dovecot documentation: https://doc.dovecot.org/configuration_manual/sieve/
