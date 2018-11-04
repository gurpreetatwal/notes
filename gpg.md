# GPG + YubiKey + SSH

[subkeys, yubikey](https://www.preining.info/blog/2016/04/gnupg-subkeys-yubikey/)
[subkeys, yubikey]( https://karlgrz.com/2fa-gpg-ssh-keys-with-pass-and-yubikey-neo/ )
[gen keys]( https://spin.atomicobject.com/2013/11/24/secure-gpg-keys-guide/ )
[yubikey]( https://malcolmsparks.com/posts/yubikey-gpg.html )
[yubikey, subkeys]( https://blog.josefsson.org/2014/06/23/offline-gnupg-master-key-and-subkeys-on-yubikey-neo-smartcard/ )


# Subkeys
- can't be used to sign other's keys
- can be revoked independantly of the msater key
-
## Article Notes
1. [subkeys](http://railslide.io/create-gpg-key-with-subkeys.html)
  - create master key `gpg --gen-key`
    - RSA & DSA
    - 4096
    - no expration
    - this will have created master key and encryption subkey
  - set key to prefer strong hashes `gpg --edit-key <email>`
    - `setpref SHA512 SHA384 SHA256 SHA224 AES256 AES192 AES CAST5 ZLIB BZIP2 ZIP Uncompressed`
    - `save`
  - create a signing subkey: `gpg --edit-key <email>`
    - `addkey`
    - `RSA (sign only`
    - no expration
    - `save`
  - create a revocation cert `gpg --output revoke.asc --gen-revoke <email>`
    - `key has been compromised`
  - remove master key
    - backup `.gpg` folder
    - export subkeys     `gpg --export-secret-subkeys <email> /media/encrypted-media/subkeys`
    - delete master key  `gpg --delete-secret-key <key-id>`
    - import subkeys     `gpg --import /media/encrypted-media/subkeys`
    - delete export      `shred -u /media/encrypted-media/subkeys`

