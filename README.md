shattr-it
=========

A web tool to securely split your secret keys using Shamir Secret Sharing.

Introduction
------------

Shattr is a web tool that lets you split your encryption keys into "shards"
in such a way that possessing at least m of n shards will recover the key,
but having less than m shard reveals no information about the key (except
possibly its length). This is useful in a number of scenarios, such as setting
up a "digital will," where several people need to agree to combine their shards
to recover the key used to encrypt a file with your passwords, etc. The DNS
root is signed in a similar manner, where at least 3 of 7 trusted community
representatives must combine their key shards to re-sign the root zone.
However, the original motivation behind Shattr was to enable coercion-resistant
full disk encryption.

Coercion-Resistant Full Disk Encryption
---------------------------------------

Full-Disk Encryption is useful for keeping your data safe from prying eyes.
However, users can be coerced into giving up their full-disk encryption keys
(e.g. at an unfriendly border crossing). With a bit of preparation, Shattr
allows you to split your full disk encryption key among trusted third parties
so that you do not have the key yourself, and thus can't be coerced into
providing it.

Of course, it would be a pain if you needed to call up m of n friends to boot
your system. This is where the Trusted Platform Module (TPM) is useful. The
TPM can store the full-disk encryption key, releasing it to the OS bootloader
when the boot sequence is unmodified and trusted. To access your files, an
attacker would have to bypass the OS's authentication mechanism. Booting into
another OS or transferring the drive to another machine would prevent the TPM
from releasing the key.

But now we've made the OS authentication mechanism prone to coercion. Or have we?
If you anticipate a situation where you might be coerced to give up your
password, you can clear your TPM and power off your machine. Then, the only way
to regain access to your files is to convince m of n trusted friends that you
haven't been coerced. Of course, this should be done securely. While you could
do it in person, having your trusted friends in different countries makes it much
more difficult for an adversary to coerce several of *them*. Using a secure
emphemeral communications medium (such as RedPhone or any other software that
uses ZRTP) is recommended.

Using Shattr with Bitlocker
---------------------------

Microsoft's Bitlocker has a bunch of nice properties that make it easy to use with
Shattr. The actual encryption key is stored on the disk, encrypted with several
different "protectors." For example, the disk key could be protected by a
recovery key and TPM. The recovery key would decrypt one copy of the disk key,
while the TPM would decrypt another copy. Bitlocker allows you to add and revoke
protectors at any time without re-encrypting the entire drive.

To use Shattr with Bitlocker:

1. If you're using Windows 8+ with a Microsoft Account, make sure your Bitlocker
recovery key is NOT backed up to Microsoft's cloud! Check at 
https://onedrive.live.com/RecoveryKey

2. Obtain your Bitlocker recovery key with this command in an elevated command prompt:
   manage-bde-protectors -get c:

3. Enter your Bitlocker key, desired number of shards, and minimum threshold
   in Shattr.

4. Before crossing a border (or any other situation where you might be compelled
   to provide your password), remove the TPM protector:
   manage-bde-protectors -delete c: TPM

5. If you're extra paranoid, clear and reset your TPM as well.

6. SHUT DOWN YOUR COMPUTER. ALL THE WAY. If you're extra paranoid, run a memtest first.

7. Once safe, contact enough trusted third parties OVER A SECURE CHANNEL to reconstruct
   your recovery key. Video conference over ZRTP lets you have a high degree of
   confidence in the secrecy and integrity without access to any private keys
   (like with OTP).

8. If the drive left your possession, make sure any notes of the recovered key
   are securely destroyed, or re-key the disk.

9. Re-enable the TPM protector:
   manage-bde -protectors -add c: -tpm