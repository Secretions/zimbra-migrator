# zimbra-migrator
Zimbra to Zimbra Migrator

This is a tool for migrating data from one Zimbra instance to another. It can migrate data as well as settings, using imapsync for mail, REST import/export for contacts/calendard/briefcase data, and various SOAP calls for other provisioning items (signatures, settings, etc.). When running imapsync, it specifies only mail folders. By not using import/export functionality for mail, it avoids size limit issues induced by the Zimbra proxy.

It is designed so it can be run repeatedly. It will not create duplicate data.

It can be configured to migrate to an alternate or temporary domain.

It requires an administrator account with adminLoginAs permissions for the domain.

It's currently very immature. It supports the following items, and they should be ran in this order:

1. imapsync
2. zmztozmig for contacts/calendars/briefcase
   - this will be rolled into the tool later
   - don't migrate mail (ie types=contact,appointment,task,wiki,document)
   - make sure you set "resolve=reset" in zmztozmig or you may get duplicate contacts
3. Signatures
4. Shares

If you run signatures prior to moving contact and briefcase data, you could potentially lose image links and contact attachments in your signatures.

If you run shares before either imapsync/contact/calendar/briefcase data, folders that need sharing permissions may not exist yet on the destination server.

An ideal full migration process would be:

1. Create temporary domain on destination server (ie temp.domain.com)
2. Get a full account list
3. Provision all accounts
4. Get crypts from LDAP
   (or 'zmprov -l gaa -v domain.com | egrep "^# name |^userPassword"')
5. Manually apply crypts (this will be rolled into the tool)
6. Use migrator --imapsync
7. zmztozmig
8. migrator --signatures
9. migrator --shares
10. Move mx to new server
11. Rerun migrator --imapsync
12. (optional) Rerun zmztozmig, signatures and shares
13. Have a burger, you're done! That's what you do when you're done!

```
usage: migrator [-h] [-c CONFIG] [-l LIST] [-d] [-v] [-n] [--imapsync]
                [--signatures] [--sharing]

Zimbra to Zimbra Migration Helper

optional arguments:
  -h, --help            show this help message and exit
  -c CONFIG, --config CONFIG
                        Config file (default: zmigrator.cfg)
  -l LIST, --list LIST  Account list
  -d, --debug           Debug mode (SOAP Trace)
  -v, --verbose         Verbose mode
  -n, --no_changes      No changes (dry run)
  --imapsync            Perform imapsyncs
  --signatures          Signatures
  --sharing             Sharing (should be the last thing)
```
