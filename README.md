# P12ToGPG
Steps to convert a PKCS#12 (.p12 or .pfx) file containing certificates and a private key to GPG format for RPM signing or other digital signature purposes. This was done with CentOS 7, but the same basic steps should work on other flavors of Linux.

**Warning:** This process creates two unencrypted private key files on disk (my_prv.pem and my_prv.gpg) because pem2openpgp does not support encrypted PEM. These steps should be performed one time only on a system (virtual machine) that is completely encrypted (in VMWare Workstation this means selecting "Full VM Encryption") and without a network connection. The resulting GPG key file will be encrypted with a password.
1. Convert the PKCS#12 file into a PEM private key file with no password.
   - `openssl pkcs12 -in ~/mycertificate.pfx -nodes -nocerts -out my_prv.pem`
2. Convert the PEM private key into a GPG privatekey using pem2openpgp from monkeysphere
   - `sudo yum install monkeysphere`
   - `export PEM2OPENPGP_USAGE_FLAGS=sign,authenticate`
   - `cat my_prv.pem | pem2openpgp "Me <me@myemail.com>" > my_prv.gpg`
3. Import the GPG private key into the GPG keyring and change its password
   - `gpg --import my_prv.gpg`
   - `gpg --list-secret-keys`
   - `gpg --edit-key My-Key-ID`
   - `passwd`
   - `save`
4. Delete the unencrypted private keys files from disk
   - `rm my_prv.prm my_prv.gpg`
5. Export the public and encrypted private key from GPG
   - `gpg -a --output my_pub.gpg --export me@myemail.com`
   - `gpg -a --output my_encrypted_prv.gpg --export-secret-key me@myemail.com`

You now have a GPG public and password-protected private key matching those in the original PKCS#12 file which can be imported into RPM for signing packages with a public key that can be chained to a certificate trust anchor:
- Add `%_gpg_name Me <me@myemail.com>` to ~/.rpmmacros
- `sudo rpm --import my_pub.gpg`
- `rpmsign --addsign myrpm.rpm`

   
