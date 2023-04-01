# Install and Configure SpamAssassin with Exim on Ubuntu Server

Follow these steps to set up an automated spam filter on your Linux Ubuntu web server using SpamAssassin and Exim:

## Step 1: Install Exim and SpamAssassin

Update the package lists and install Exim, SpamAssassin, and spamc:

```shell
sudo apt update
sudo apt install exim4 spamassassin spamc
```

## Step 2: Configure Exim to use SpamAssassin

Edit the Exim configuration file /etc/exim4/exim4.conf.template:

```shell
sudo nano /etc/exim4/exim4.conf.template
```

Find the `acl_check_data` section and add the following lines right after it:

```shell
  # SpamAssassin
  warn    spam = Debian-exim:true
          add_header = X-Spam_score: $spam_score\n\
                       X-Spam_score_int: $spam_score_int\n\
                       X-Spam_bar: $spam_bar\n\
                       X-Spam_report: $spam_report
  warn    message = This message scored $spam_score spam points.
          condition = ${if >{$spam_score_int}{50}{1}{0}}
          add_header = X-Spam_flag: YES\n\
                       Subject: [SPAM] $h_Subject:
```

This configuration will run SpamAssassin on incoming emails and add headers with spam information. Emails with a spam score above 5.0 will have "[SPAM]" added to the subject. Save the changes and exit the editor.

## Step 3: Enable and start the SpamAssassin service

Enable the SpamAssassin service to start automatically at boot, and then start the service:

```shell
sudo systemctl enable spamassassin
sudo systemctl start spamassassin
```

## Step 4: Configure SpamAssassin settings

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

## Step 5: Restart Exim

Restart the Exim service to apply the changes:

```shell
sudo systemctl restart exim4
```

SpamAssassin will now automatically process incoming emails on your server and mark them as spam based on your configuration. You can further refine the configuration by editing /etc/spamassassin/local.cf and adjusting the settings as needed.

Optionally, you can set up a user-based spam filter using the Sieve mail filtering language and the Dovecot IMAP server. This allows creating custom filtering rules for each user on your server. To set up Sieve, follow the Dovecot documentation: https://doc.dovecot.org/configuration_manual/sieve/
