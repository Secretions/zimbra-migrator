# zimbra-migrator
Zimbra to Zimbra Migrator

This is a tool for migrating data from one Zimbra instance to another. It can migrate data as well as settings, using imapsync for mail, REST import/export for contacts/calendard/briefcase data, and various SOAP calls for other provisioning items (signatures, settings, etc.). When running imapsync, it specifies only mail folders. By not using import/export functionality for mail, it avoids size limit issues induced by the Zimbra proxy.

It is designed so it can be run repeatedly. It will not create duplicate data.

It can be configured to migrate to an alternate or temporary domain.

It requires an administrator account with adminLoginAs permissions for the domain.

It's currently very immature. It supports the following items, and they should be ran in this order:

1. imapsync
2. contacts
3. calendar
4. tasks
5. briefcase
6. signatures
7. account attributes
8. distribution lists
9. aliases
10. shares

This order is reflected in the --help output.

IMPORTANT: Contact migration is implemented DESTRUCTIVELY. Contacts are currently migrated via the export/import REST API, and the "resolve" options that let you choose to skip or replace duplicates do not function for contacts. This results in duplicates. To avoid duplicates, this tool will empty the contact folders on the destination account. If the destination account is already in use, and new contacts were added, those new contacts WILL BE LOST. Ideally, this tool's contact support will be rewritten to use SOAP requests to handle contacts directly, but this is a lower priority at present.

If you run signatures prior to moving contact and briefcase data, you could potentially lose image links and contact attachments in your signatures.

If you run shares before either imapsync/contact/calendar/briefcase data, folders that need sharing permissions may not exist yet on the destination server.

An ideal full migration process would be:

1. Create temporary domain on destination server (ie temp.domain.com)
2. Get a full account list
3. Provision all accounts
4. Get crypts from LDAP
   (or 'zmprov -l gaa -v domain.com | egrep "^# name |^userPassword"')
5. Manually apply crypts (this will be rolled into the tool)
6. Use migrator --imapsync --contacts --tasks --briefcase --calendar
8. migrator --signatures --attrs --distlists
9. migrator --shares --aliases
10. Move mx to new server
11. Rerun migrator --imapsync --contacts --tasks --briefcase --calendar
12. (optional) Rerun extra stuff if needed (attrs, etc.)
13. Have a burger, you're done! That's what you do when you're done!

```
usage: migrator [-h] [-c CONFIG] [-l LIST] [-a ACCOUNT] [-d] [-v] [-n]
                [--get_all_accounts] [--ssl_level SSL_LEVEL]
                [--temp_dir TEMP_DIR] [--imapsync] [--contacts] [--calendar]
                [--tasks] [--briefcase] [--signatures] [--attrs] [--aliases]
                [--distlists] [--resources] [--sharing]

Zimbra to Zimbra Migration Helper

optional arguments:
  -h, --help            show this help message and exit
  -c CONFIG, --config CONFIG
                        Config file (default: zmigrator.cfg)
  -l LIST, --list LIST  Account list (defaults to this)
  -a ACCOUNT, --account ACCOUNT
                        Account to work on, or enter 'all' to use all accounts
                        as working set
  -d, --debug           Debug mode (SOAP Trace)
  -v, --verbose         Verbose mode
  -n, --no_changes      No changes (dry run)
  --get_all_accounts    Get full source account list (via SOAP, sketchy on
                        large installs)
  --ssl_level SSL_LEVEL
                        ssl/tls level (ssl3/tls1/tls1.1/tls1.2)
  --temp_dir TEMP_DIR   Directory for temporary files (default is /tmp)
  --imapsync            Perform imapsyncs
  --contacts            Migrate contacts (DESTRUCTIVE, see readme)
  --calendar            Migrate calendar data
  --tasks               Migrate tasks
  --briefcase           Migrate briefcase data
  --signatures          Signatures
  --attrs               Account Attributes
  --distlists           Distribution Lists
  --resources           Calendar Resources
  --aliases             Aliases (should be the last thing)
  --sharing             Sharing (should be the last thing)

```
