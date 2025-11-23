# Gmail Client Integration

This document explains how to configure Gmail to access your self-hosted email accounts using IMAP and SMTP.

## Overview

Gmail can be configured as an email client to send and receive emails from your self-hosted mail server. This allows you to use Gmail's interface while managing emails from your domain (e.g., `roque@obusan.me`).

## Prerequisites

- Mail-in-a-Box email server running and accessible
- Email account created in Mail-in-a-Box (e.g., `roque@obusan.me`)
- Gmail account with access to Gmail settings
- IMAP/SMTP server details from Mail-in-a-Box

## Server Information

Before configuring Gmail, gather the following information:

- **IMAP Server**: `mail.obusan.me`
- **IMAP Port**: `993`
- **IMAP Security**: SSL/TLS (required)
- **SMTP Server**: `mail.obusan.me`
- **SMTP Port**: `587`
- **SMTP Security**: STARTTLS (required)
- **Username**: Full email address (e.g., `roque@obusan.me`)
- **Password**: The password set in Mail-in-a-Box for this user

## Step 1: Access Gmail Settings

1. **Open Gmail** in your web browser
2. Click your **profile picture** (top right)
3. Click **Manage your Google Account**
4. Or go directly to: [Google Account Settings](https://myaccount.google.com/)

## Step 2: Navigate to Email Settings

1. In Google Account, click **Security** (left sidebar)
2. Scroll down to **2-Step Verification** section
3. Look for **App passwords** (or go to **Signing in to Google** → **App passwords**)

**Note**: If you don't see App passwords, you may need to enable 2-Step Verification first, or Gmail may allow you to use your regular password.

Alternatively:
1. In Gmail, click the **Settings gear icon** (top right)
2. Click **See all settings**
3. Go to **Accounts and Import** tab

## Step 3: Add Mail Account

1. In **Accounts and Import** tab, find **Check mail from other accounts**
2. Click **Add a mail account**
3. Enter the email address you want to add (e.g., `roque@obusan.me`)
4. Click **Next**

## Step 4: Configure IMAP Access

Gmail will prompt you to configure the account:

1. **Username**: Enter full email address (e.g., `roque@obusan.me`)
2. **Password**: Enter the password for this email account
3. **POP Server**: Leave this section - we'll use IMAP instead
4. Click **Add Account**

**Note**: If Gmail defaults to POP3, you may need to look for an option to use IMAP instead, or configure it in the next step.

## Step 5: Configure as IMAP Account (Alternative Method)

If the above method uses POP3, use this method instead:

1. In Gmail settings, go to **Accounts and Import** tab
2. Look for **Add a mail account** or **Add another email address**
3. Choose **Link accounts with Gmailify** or **Import emails from my other account (POP3)**
4. For IMAP, you may need to use Gmail's "Send mail as" feature combined with IMAP

### Using Gmail's "Send mail as" Feature

1. In **Accounts and Import**, find **Send mail as**
2. Click **Add another email address**
3. Enter your email address (e.g., `roque@obusan.me`)
4. Click **Next Step**

### Configure SMTP Settings

1. **SMTP Server**: `mail.obusan.me`
2. **Port**: `587`
3. **Username**: Full email address (e.g., `roque@obusan.me`)
4. **Password**: Your email account password
5. **Secured connection**: Select **TLS** or **STARTTLS**
6. Click **Add Account**

Gmail will send a verification email. Check your Mail-in-a-Box webmail to get the verification code.

## Step 6: Access Emails via IMAP (Recommended Method)

For receiving emails, Gmail's built-in "Check mail from other accounts" typically uses POP3. For full IMAP access, consider these options:

### Option A: Use Gmail's POP3 Import (Simpler)

1. In **Accounts and Import**, click **Add a mail account**
2. Enter email address: `roque@obusan.me`
3. Select **Import emails from my other account (POP3)**
4. Configure:
   - **Username**: `roque@obusan.me`
   - **Password**: Your email password
   - **POP Server**: `mail.obusan.me`
   - **Port**: `995`
   - **Use a secure connection (SSL)**: Yes
5. Click **Add Account**

**Note**: POP3 downloads emails to Gmail but doesn't sync bidirectionally like IMAP.

### Option B: Use Third-Party Email Client (Full IMAP)

For true IMAP synchronization, use a dedicated email client:
- **Thunderbird** (desktop)
- **Outlook** (desktop)
- **Apple Mail** (Mac/iOS)
- **K-9 Mail** (Android)

Then forward important emails to Gmail if needed.

### Option C: Forward Emails to Gmail

1. In Mail-in-a-Box admin panel, configure email forwarding
2. Forward `roque@obusan.me` → `your-gmail@gmail.com`
3. Use Gmail's "Send mail as" to send from `roque@obusan.me`

## Step 7: Send Emails from Your Domain

### Configure "Send mail as"

1. In Gmail settings → **Accounts and Import**
2. Under **Send mail as**, click **Add another email address**
3. Enter: `roque@obusan.me`
4. Click **Next Step**
5. Configure SMTP:
   - **SMTP Server**: `mail.obusan.me`
   - **Port**: `587`
   - **Username**: `roque@obusan.me`
   - **Password**: Your email password
   - **Secured connection**: **TLS** or **STARTTLS**
6. Click **Add Account**
7. Verify the email address (check Mail-in-a-Box webmail for verification code)

### Set as Default (Optional)

1. In **Send mail as** section, find your added address
2. Click **make default** if you want it as default sender

## Step 8: Verify Configuration

### Test Receiving Emails

1. Send a test email to `roque@obusan.me` from an external address
2. Check if it appears in Gmail
3. If using POP3 import, emails may take a few minutes to appear

### Test Sending Emails

1. Compose a new email in Gmail
2. Click **From** dropdown (if multiple addresses)
3. Select `roque@obusan.me`
4. Send test email to yourself or another address
5. Verify it's sent from the correct address
6. Check recipient receives it properly

## Troubleshooting

### Can't Connect to IMAP/SMTP Server

**Symptoms**: Gmail shows connection errors

**Solutions**:
1. Verify server address: `mail.obusan.me` (not `mail.obusan.me:993`)
2. Check ports are correct: IMAP `993`, SMTP `587`
3. Verify SSL/TLS is enabled
4. Test server accessibility: `telnet mail.obusan.me 993`
5. Check firewall port forwarding is configured
6. Verify DNS resolves correctly: `nslookup mail.obusan.me`

### Authentication Failed

**Symptoms**: "Username or password incorrect" error

**Solutions**:
1. Verify full email address is used as username: `roque@obusan.me`
2. Check password is correct (try logging into webmail to verify)
3. If 2FA is enabled, you may need an app password
4. Check Mail-in-a-Box user account is active
5. Verify mailbox quota hasn't been exceeded

### Emails Not Appearing in Gmail

**Symptoms**: Emails sent to your domain don't show in Gmail

**Solutions**:
1. If using POP3 import, check import frequency settings
2. Verify "Check mail from other accounts" is enabled
3. Check Gmail's "All Mail" folder, not just Inbox
4. Verify emails are actually being received (check Mail-in-a-Box webmail)
5. Check spam folder in both Gmail and Mail-in-a-Box

### Can't Send Emails

**Symptoms**: Emails fail to send or bounce

**Solutions**:
1. Verify SMTP settings are correct (server, port, security)
2. Check "Send mail as" is properly configured
3. Verify email address is verified in Gmail
4. Check Mail-in-a-Box logs for errors
5. Test sending from Mail-in-a-Box webmail to verify server works

### SSL/TLS Certificate Errors

**Symptoms**: Certificate validation errors

**Solutions**:
1. Verify Mail-in-a-Box SSL certificate is valid
2. Check certificate hasn't expired
3. Ensure you're using the correct domain (`mail.obusan.me`)
4. Try accessing webmail to verify certificate works
5. Mail-in-a-Box should auto-renew certificates

## Security Considerations

### App Passwords (If Required)

If Gmail requires an app password:

1. Go to Google Account → **Security**
2. Enable **2-Step Verification** (if not already enabled)
3. Go to **App passwords**
4. Generate password for "Mail"
5. Use this password instead of your regular Gmail password

### Password Security

- Use strong, unique passwords for email accounts
- Don't share passwords between accounts
- Consider using a password manager
- Regularly update passwords

### Two-Factor Authentication

- Enable 2FA on Gmail account
- Enable 2FA on Mail-in-a-Box admin account if available
- Use app passwords when 2FA is enabled

## Alternative: Using Gmail as Primary with Forwarding

If Gmail integration is too complex, consider this simpler approach:

1. **Forward emails to Gmail**: Configure Mail-in-a-Box to forward all emails to your Gmail address
2. **Send from Gmail**: Use Gmail's "Send mail as" feature to send emails appearing to come from `roque@obusan.me`
3. **Access webmail**: Use Mail-in-a-Box webmail for full email management when needed

This approach:
- Simplifies access (all emails in Gmail)
- Maintains ability to send from your domain
- Reduces complexity
- Still allows webmail access when needed

## Next Steps

After configuring Gmail:
1. Test sending and receiving emails
2. Set up email filters/labels in Gmail if needed
3. Configure other email clients if desired
4. Test email deliverability (see [testing-troubleshooting.md](testing-troubleshooting.md))

## Notes

- Gmail's "Check mail from other accounts" typically uses POP3, not IMAP
- For true IMAP sync, consider using a dedicated email client
- "Send mail as" works well for sending from your domain
- Emails imported via POP3 are copied to Gmail, not synced
- Consider using Mail-in-a-Box webmail for full email management features

