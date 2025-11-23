# User Management

This document explains how to create and manage email accounts in Mail-in-a-Box.

## Overview

Mail-in-a-Box provides a web-based interface for creating and managing email accounts. Each user gets:
- Email address (e.g., `roque@obusan.me`)
- Mailbox with IMAP/SMTP access
- Webmail access (Roundcube)
- Configurable storage quota

## Prerequisites

- Mail-in-a-Box installed and running
- Access to admin panel at `https://mail.obusan.me/admin`
- Admin credentials

## Accessing User Management

1. **Log into admin panel**: `https://mail.obusan.me/admin`
2. **Navigate to**: **Mail** → **Users** (or **System** → **Users**)

## Creating Email Accounts

### Step 1: Add New User

1. In the **Users** section, click **Add Mail User** (or **+ Add** button)
2. Fill in the form:

   - **Email Address**: Enter the local part (e.g., `roque` for `roque@obusan.me`)
     - The domain (`@obusan.me`) is automatically appended
   - **Password**: Enter a strong password
     - Or click **Generate Password** for a secure random password
   - **Privileges**: 
     - **Admin**: Check if this user should have admin access
     - Leave unchecked for regular email users
   - **Mailbox Quota**: 
     - Leave empty for unlimited (or default quota)
     - Or specify in MB/GB (e.g., `5000` for 5GB)

3. Click **Add User** (or **Save**)

### Step 2: Verify User Creation

- The new user should appear in the users list
- Status should show as active
- Mailbox quota should be displayed

## Example: Creating Multiple Users

To create the users mentioned in your requirements:

### User 1: roque@obusan.me

1. Click **Add Mail User**
2. **Email Address**: `roque`
3. **Password**: (set secure password)
4. **Privileges**: Unchecked (regular user)
5. **Mailbox Quota**: (leave empty or set limit)
6. Click **Add User**

### User 2: matteo@obusan.me

1. Click **Add Mail User**
2. **Email Address**: `matteo`
3. **Password**: (set secure password)
4. **Privileges**: Unchecked (regular user)
5. **Mailbox Quota**: (leave empty or set limit)
6. Click **Add User**

Repeat for any additional users.

## Managing Existing Users

### View User Details

1. Go to **Mail** → **Users**
2. Click on a user in the list to view details
3. View:
   - Email address
   - Mailbox size/quota
   - Last login
   - Account status

### Change User Password

1. Go to **Mail** → **Users**
2. Click on the user
3. Click **Change Password** (or edit user)
4. Enter new password
5. Click **Save**

### Modify Mailbox Quota

1. Go to **Mail** → **Users**
2. Click on the user
3. Edit **Mailbox Quota** field
4. Enter new quota (in MB, e.g., `10000` for 10GB)
5. Click **Save**

### Disable User Account

1. Go to **Mail** → **Users**
2. Click on the user
3. Look for **Disable** or **Remove** option
4. Confirm action

**Note**: Disabling a user prevents login but preserves the mailbox. Removing a user deletes the mailbox permanently.

### Delete User Account

**Warning**: This permanently deletes the user's mailbox and all emails!

1. Go to **Mail** → **Users**
2. Click on the user
3. Click **Remove User** or **Delete**
4. Confirm deletion

## User Access Information

After creating a user, they can access their email via:

### Webmail (Roundcube)

- **URL**: `https://mail.obusan.me/mail`
- **Username**: Full email address (e.g., `roque@obusan.me`)
- **Password**: The password you set

### IMAP/SMTP (Email Clients)

- **IMAP Server**: `mail.obusan.me`
- **IMAP Port**: `993`
- **IMAP Security**: SSL/TLS
- **SMTP Server**: `mail.obusan.me`
- **SMTP Port**: `587`
- **SMTP Security**: STARTTLS
- **Username**: Full email address (e.g., `roque@obusan.me`)
- **Password**: The password you set

See [gmail-integration.md](gmail-integration.md) for detailed Gmail client setup.

## Mailbox Quotas

### Setting Quotas

Mailbox quotas limit the amount of storage a user can use. To set a quota:

1. Edit the user
2. Enter quota in **Mailbox Quota** field
3. Format: Number in MB (e.g., `5000` = 5GB, `10000` = 10GB)
4. Leave empty for unlimited (subject to system limits)

### Recommended Quotas

- **Personal use**: 5-10GB per user
- **Light usage**: 2-5GB per user
- **Heavy usage**: 10-20GB per user
- **Unlimited**: Leave empty (monitor disk usage)

### Monitoring Quota Usage

1. Go to **Mail** → **Users**
2. View **Mailbox Size** column
3. Compare to **Quota** column
4. Users approaching quota will receive warnings

## Aliases and Forwarding

### Email Aliases

Aliases allow multiple email addresses to deliver to the same mailbox:

1. Go to **Mail** → **Aliases** (or similar section)
2. Click **Add Alias**
3. Configure:
   - **Alias**: The alias email (e.g., `info@obusan.me`)
   - **Forwards to**: The actual user email (e.g., `roque@obusan.me`)
4. Click **Add**

**Example**: Create `info@obusan.me` that forwards to `roque@obusan.me`

### Email Forwarding

Forward all emails from one address to another:

1. Go to **Mail** → **Aliases**
2. Create alias that forwards to external email (e.g., Gmail)
3. Or configure in user settings if available

## Admin Users

### Creating Admin Users

Admin users have access to the Mail-in-a-Box admin panel:

1. When creating/editing a user, check **Admin** privilege
2. Admin users can:
   - Access admin panel
   - Manage other users
   - Configure system settings
   - View system status

### Best Practices for Admins

- Limit number of admin users
- Use strong passwords
- Consider 2FA if available
- Regular users should not have admin privileges

## Password Policies

Mail-in-a-Box enforces password requirements:
- Minimum length (typically 8+ characters)
- Complexity requirements may apply
- Use strong, unique passwords

### Password Best Practices

- Use long, random passwords
- Consider password manager
- Don't reuse passwords
- Change passwords periodically

## User Statistics

View user statistics in the admin panel:

1. Go to **Mail** → **Users**
2. View columns:
   - **Mailbox Size**: Current storage used
   - **Quota**: Storage limit
   - **Last Login**: Last webmail access
   - **Status**: Active/Inactive

## Troubleshooting

### User Can't Login

1. Verify email address is correct (full address: `user@obusan.me`)
2. Check password is correct
3. Verify account is not disabled
4. Check mailbox quota hasn't been exceeded
5. Review system logs for errors

### Mailbox Full

**Symptom**: User can't receive new emails

**Solution**:
1. Increase mailbox quota
2. Or user should delete old emails
3. Or configure email forwarding/archiving

### Password Reset

If a user forgets their password:

1. Log into admin panel
2. Go to **Mail** → **Users**
3. Click on the user
4. Click **Change Password**
5. Set new password
6. Communicate new password securely to user

### User Not Receiving Emails

1. Verify user email address is correct
2. Check DNS MX record is properly configured
3. Verify port forwarding is working
4. Check mail server logs
5. Test sending email to the address

## Security Considerations

1. **Strong Passwords**: Enforce strong passwords for all users
2. **Regular Audits**: Review user list periodically
3. **Disable Unused Accounts**: Disable accounts that are no longer needed
4. **Monitor Quotas**: Prevent disk space issues
5. **Admin Access**: Limit admin privileges to necessary users only

## Next Steps

After creating users:
1. Test email sending/receiving for each user
2. Configure email clients (see [gmail-integration.md](gmail-integration.md))
3. Provide users with access information
4. Set up email forwarding/aliases if needed

## Notes

- User email addresses are case-insensitive (`Roque@obusan.me` = `roque@obusan.me`)
- Mailbox storage counts against container disk space
- Users can change their own passwords via webmail (if enabled)
- Consider setting up email aliases for common addresses (info@, contact@, etc.)

