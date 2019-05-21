# GPG + YubiKey + SSH

[subkeys, yubikey]( https://karlgrz.com/2fa-gpg-ssh-keys-with-pass-and-yubikey-neo/ )
[yubikey]( https://malcolmsparks.com/posts/yubikey-gpg.html )

`gpg.conf`
```
no-emit-version
no-comments
keyid-format 0xlong
with-fingerprint
use-agent
personal-cipher-preferences AES256 AES192 AES CAST5
personal-digest-preferences SHA512 SHA384 SHA256 SHA224
cert-digest-algo SHA512
default-preference-list SHA512 SHA384 SHA256 SHA224 AES256 AES192 AES CAST5 ZLIB BZIP2 ZIP Uncompressed
```

`gpg-agent.conf`
```
default-cache-ttl 600
max-cache-ttl 7200
enable-ssh-support
write-env-file
```
(env file must be sourced!)
```bash
if [ -f "${HOME}/.gnupg/gpg-agent-info-$(hostname)" ]; then
     . "${HOME}/.gnupg/gpg-agent-info-$(hostname)"
       export GPG_AGENT_INFO
       export SSH_AUTH_SOCK
       export SSH_AGENT_PID
fi
```


## Article Notes
1. [subkeys](http://railslide.io/create-gpg-key-with-subkeys.html)
  - create master key `gpg --gen-key`
    - RSA & DSA
    - 4096
    - no expiration
    - this will have created master key and encryption subkey
  - set key to prefer strong hashes `gpg --edit-key <email>`
    - `setpref SHA512 SHA384 SHA256 SHA224 AES256 AES192 AES CAST5 ZLIB BZIP2 ZIP Uncompressed`
    - `save`
  - create a signing subkey: `gpg --edit-key <email>`
    - `addkey`
    - `RSA (sign only)`
    - no expiration
    - `save`
  - create a revocation cert for master key `gpg --output revoke.asc --gen-revoke <email>`
    - `key has been compromised`
  - remove master key
    - backup `.gpg` folder
    - export subkeys     `gpg --export-secret-subkeys <email> /media/encrypted-media/subkeys`
    - delete master key  `gpg --delete-secret-key <master-key-id>`
    - import subkeys     `gpg --import /media/encrypted-media/subkeys`
if [ -f "${HOME}/.gpg-agent-info" ]; then
     . "${HOME}/.gpg-agent-info"
       export GPG_AGENT_INFO
       export SSH_AUTH_SOCK
       export SSH_AGENT_PID
fi
    - delete export      `shred -u /media/encrypted-media/subkeys`
    - verify by running `gpg -K` there should be a `#` next to `sec` denoting key unavailable
  - upload key ` gpg --keyserver pgp.mit.edu --send-key <key-id>`

2. [subkeys - debian](https://wiki.debian.org/Subkeys?action=show&redirect=subkeys)
  - create backup of gpg folder `umask 077; tar -cf $HOME/gnupg-backup.tar -C $HOME/.gnupg`
  - IF GPG 2.1 or later, do the following to delete master
    - `rm  $HOME/.gnupg/private-keys-v1.d/$(gpg --with-keygrip --list-key <master-key-id>).key`
    - `rm $HOME/.gnupg/secring.gpg`
  - ELSE do as above
  - create subkey password ` gpg --edit-key YOURMASTERKEYID passwd` after removing master

3. [gen keys]( https://spin.atomicobject.com/2013/11/24/secure-gpg-keys-guide/ )
  - create master key `gpg --gen-key --expert`
    - `(8) RSA (set your own capabilities)`
      - `s`, `e`, `q` (only enable `Certify`)
    - 4096
    - 3 years expiration (can always be pushed)
    - this will have created master key and encryption subkey
  - create a signing subkey: `gpg --expert --edit-key <email>` (expert is needed for auth key)
    - `addkey`
    - `(8) RSA (set your own capabilities)`
      - `e`, `q` (only enable `Sign`)
    - 6 months
    - `save`
    - do again for encrypt and auth keys
  - backup the following:
    - `gpg -a --export-secret-key john.doe@example.com > secret_key`
    - `gpg -a --gen-revoke john.doe@example.com > revocation_cert.gpg`
    - `gpg -a --export john.doe@example.com > public_key.gpg`
  - remove master key
    -  `gpg -a --export-secret-subkeys john.doe@example.com > secret_subkeys.gpg`
    -  `gpg --delete-secret-keys john.doe@example.com`
    -  `gpg --import secret_subkeys.gpg`

4. [subkeys, yubikey](https://www.preining.info/blog/2016/04/gnupg-subkeys-yubikey/)
  - mostly same as #3
  - moving keys to card `gpg --edit-key <master-key-id>`
    - `keytocard`
    - select key
5. [yubikey](https://iain.learmonth.me/blog/2015/2015w517/)
  - add the following udev rule for access
    - `UBSYSTEMS=="usb", ATTRS{idVendor}=="1050", ATTRS{idProduct}=="0111", TAG+="uaccess"`
    - the `uaccess` tag depends on `systemd-logind`

6. [yubikey, subkeys]( https://blog.josefsson.org/2014/06/23/offline-gnupg-master-key-and-subkeys-on-yubikey-neo-smartcard/ )
  - starting off as 3 did
  - add identities: `adduid`
    - mark identity as primary:
      - `uid n`
      - `primary`
    - `save`
  -  (create subkeys)
  -  export subkeys for backup
    - `gpg -a --export-secret-keys 1C5C4717 > $GNUPGHOME/../mastersubkeys.txt`
    - `gpg -a --export-secret-subkeys 1C5C4717 > $GNUPGHOME/../subkeys.txt`
  - (perform yubikey setup, key must be in OTP or CCID mode)
  - configure OpenPGP applet `gpg --card-edit`
    - `admin`
    - `3 - change Admin pin`
    - `1 - change pin`
    - `q`
    - `name`
    - `lang`
    - `url` to public key
    - `sex`
    - `login` (account name)
    - `quit`
  - move subkeys (after creating backup) ` gpg --edit-key 1C5C4717`)
    - `toggle`
    - `key 1`
    - `keytocard`
  - import public key on your laptop
    - ` gpg -a --export 1C5C4717 > $GNUPGHOME/../publickey.txt`
    - (on laptop) `gpg --import < publickey.txt`)
  - trust master key `gpg --edit-key 1c5c4717`
    - `trust`
    - `5`
    - `quit`
    -

7. [yubikey + ssh](https://jclement.ca/gpg-smartcard/)
  - enable `CCID` mode using `ykpsonalize`
  - openpgp applet should be `>= 1.0.10`
  - create keys (`gpg --gen-key`)
    - `4 RSA (Sign Only)`
    - 1 year expiration
  - create subkeys (`gpg --expert --edit-key <email>`)
    - `addkey`
    - `4 RSA (sign only)` - `6mo`
    - `6 RSA (encrypt only)` - `6mo`
    - `8 RSA (set your own capabilities)` (a) - `6mo`
  - create revocation cert ` gpg --gen-revoke <email> > file`
  - backup
    - `tar -czf /media/BACKUP/gnupg.tgz ~/.gnupg`
    - `gpg -a --export-secret-key  0x2896DB4A0E427716 >> /media/BACKUP/2896DB4A0E427716.master.key`
    - `gpg -a --export-secret-subkeys  0x2896DB4A0E427716 >> /media/BACKUP/2896DB4A0E427716.sub.key`
  - setup smartcard `gpg --card-edit`
    - User PIN - this is the PIN required to use your key to sign/decrypt
    - Admin PIN - this PIN is required to make changes to the smart card and is not used day-to-day
    - URL - location of your public key file and can be used GnuPG to download key in new installations. (can use keybase.io here)
    - `admin`
    - `passwd`
    - `1 - change pin`
    - `3 - change Admin pin`
    - `q`
    - `url` to public key
    - `name`
    - `login` (account name)
    - `sex`
    - `lang` (en)
    - `quit`
  - move subkeys `gpg --edit-key 0x2896DB4A0E427716`
    - `toggle`
    - `key 1`
    - `keytocard`
    - pick type of key
    - `key 1`
    - `key 2`
    - `keytocard`
    - pick type of key
    - `key 2`
    - ...
  - get ssh public key `gpg -a --export 0x2896DB4A0E427716 > my-public-key.asc`
  ---- switch machines -----
  - client deps: `sudo apt-get install gnupg2 gnupg-agent libpth20 pinentry-curses libccid pcscd scdaemon libksba8`
  - import public key
    - either `gpg --import << public.key`
    - or `gpg --card-edit` followed by `fetch`  (gets from configured url)

8. [pgp, yubikey, ssh](https://www.esev.com/blog/post/2015-01-pgp-ssh-key-on-yubikey-neo/)
  - `mv .gnupg .gnupg.orig`
  - `ln -s /media/USB .gnupg`
  - create master: `gpg --expert --gen-key`
    - `8 RSA (set your own capabilities)` (c) - `1y`
  - create revoke cert `gpg --gen-revoke B8EFD59D > /media/USB/B8EFD59D-revocation-certificate.asc`
  - create (shared) encryption key `gpg --edit-key B8EFD59D`
    - `addkey` - `6 RSA (encrypt only)` - `1y`
    - `save`
  - backup secret keys `gpg --export-secret-key B8EFD59D > /media/USB/B8EFD59D-2015-01-01-EE86E896-secret.pgp`
  - create signing and auth keys `gpg --edit-key B8EFD59D`
    - `addcardkey` - `1 Signature Key` - `1yr`
    - `addcardkey` - `3 Authentication Key` - `1yr`
  - import encryption key
    - `toggle`
    - `key 1`
    - `keytocard` - `2 Encryption key`
    - `save`
  - export public key `gpg --armor --export B8EFD59D > B8EFD59D.asc`
  - remove drive `rm .gnupg && mv .gnupg.orig .gnupg`
  - setup yubikey as above `gpg --card-edit`
    - `forcesig` # make sure pin is entered before signing

[gpg ubikey](https://codingnest.com/how-to-use-gpg-with-yubikey-wsl/#fn1)
`git config --global user.signingkey <signing-subkey-id>`
`git config commit.gpgsign true`
```
$ gpg2 --export-secret-keys --armor 48CCEEDF > 48CCEEDF-private.gpg # maseter private key
$ gpg2 --armor --export 48CCEEDF > 48CCEEDF-public.gpg #public key
$ gpg2 --armor --export-secret-subkeys A85EA103 > A85EA103-private-subkey.gpg
```

need to revoke
```
gpg --import /path/to/\<bilbo@shire.org\>.public.gpg-key /path/to/\<bilbo@shire.org\>.private.gpg-key
```

### Me
1. Create `udev` rules
`/etc/udev/rules.d/70-u2f.rules `
```
KERNEL=="hidraw*", SUBSYSTEM=="hidraw", ATTRS{idVendor}=="1050", ATTRS{idProduct}=="0113|0114|0115|0116|0120|0200|0402|0403|0406|0407|0410", TAG+="uaccess", GROUP="plugdev", MODE="0660"
```
2. `gpg --card-edit`
  - `admin`
  - `passwd`
  - `1 change PIN` (default is 123456)
  - `3 change Admin PIN` (default is 12345678)
  - `q`
  - `name`
  - `lang` (en)
  - `sex`
  - `login` (account name)
  - `quit`
3.
  - `ykman openpgp touch sig on`
  - `ykman openpgp touch enc on`
  - `ykman openpgp touch aut off`
4. `gpg --expert --full-generate-key`
  - `8 RSA (set your own capabilities)`
  - `s, e, q` (leaves "Certify")
  - `4096`
  - `2y`
5. `gpg --edit-key gatwal@live.com`
  - `adduid`
  - `uid 1`
  - `primary`
  - `save`
6. `gpg --export-secret-key --armor > ~/mnt/$KEY_ID-private-key.asc`
 ( mv rev cert)
7. `gpg --expert --edit-key gatwal@live.com`
  - `addkey` x3
  - `6 RSA (encrypt only)`  - `4 RSA (sign only)` -  `8 RSA (set your own capabilities) (a)`
  - `4096` - `4096` - `4096`
  - `6m` - `6m` - `1y`
  - `save` (after all 3 keys)
8. `gpg --export-secret-subkeys --armor >> ~/mnt/$KEY_ID-private-subkeys.asc`
9. `gpg --edit-key gatwal@live.com`
    - `toggle`
    - `key 1`
    - `keytocard`
    - pick type of key
    - `key 1`
    - `key 2`
    - `keytocard`
    - pick type of key
    - `key 2`
    - `key 3`
    - `keytocard`
    - pick type of key
    - `key 3`
    - `save`
10. `gpg-connect-agent "DELETE_KEY <keygrip>"`

- `adduid` for adding users
Symmetric crypto is useful for encrypting files for yourself
-----------

# Public Key Crypto Overview

### Terminology
- Private Key:
  - used to sign files
  - decrypt files encrypted using the public key
- Public Key:
  - used by others to verify signatures
  - encrypt files for you
- Master/Primary Key:
  - Public/Private key pair that solely represents an identity [2]
- Keyring: a collection of keys in a file or database (could literally be a sequential list of keys)
-  Certify / Certification:
  - "trust" another key, i.e. certify that gatwal@live.com is indeed the owner of x key
- Key Types (`gpg --list-secret-keys (-K)`)
  - `sec` -> secret key
  - `ssb` -> secret subkey
  - `pub` -> public key
  - `sub` -> secret subkey
## Intro

## Subkeys
 A subkey is a key pair that has been signed by a master key pair. [3] The default behavior of `gpg` is to create a master key that is used for signing, authentication (SSH), and certification and a subkey that is used for encryption.

The reason subkeys work is because the process of certifying a key is relative to the user id (email) and the master key's public keys. All subkeys can be verified by checking their signatures with the master key's public key.

### Why
The reasons why subkeys should be used are as follows:  [3] [4]
 - Primary keys should be stored secure locations to minimize the risk of compromise
   - if your primary key is compromised, all of the reputation that you've built around that key is gone
   - if your Primary key is compromised, someone can pretend to be you
- easy revocation for subkeys
- easier to create many subkeys with short expirations and rotate

### Caveats
- a subkey _cannot_ be used certifying someone else's key
- it does not make sense to have multiple encryption subkeys at any given time [5]
  - gpg only encrypts for the most recent encryption subkey

References:
[1]: https://davesteele.github.io/gpg/2014/09/20/anatomy-of-a-gpg-key/
[2]: https://serverfault.com/a/399366
[3]: https://security.stackexchange.com/a/76950/208397
[4]: https://wiki.debian.org/Subkeys#Why.3F
[5]: https://wiki.debian.org/Subkeys#Multiple_Subkeys_per_Machine_vs._One_Single_Subkey_for_All_Machines
