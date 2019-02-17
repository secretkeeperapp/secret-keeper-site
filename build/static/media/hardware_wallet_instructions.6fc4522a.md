## Hardware Wallet Instructions

**Currently as of 1/1/2018, this functionality is only supported on OSX and Linux, using a Ledger or Trezor hardware device.**

It’s recommended to use an air gapped machine when executing the GPG commands, since it’s possible that the PGP private key could be exposed in memory. No other keys on the Ledger or Trezor will be revealed when using this method.

When running the commands, you will need to confirm the sign and encrypt operations on your Ledger or Trezor.

General notes: 

* It is advised to use an ETH address that you don’t use for anything else when interacting with SecretKeeper. Be aware that your interactions with the contract are public, for everyone to see, for a very long time.

### Setup:

First, make sure you have the tools you’ll need.

1. Install the [GPG agent](https://github.com/romanz/trezor-agent) for your computer. This may require installing additional libraries on Linux, the instructions there are fairly clear.

2. If you’re using a Ledger, install the Ledger SSH/GPG Agent app using the [Ledger Manager](https://chrome.google.com/webstore/detail/ledger-manager/beimhnaefocolcplfimocfiaiefpkgbf?hl=en). Check the link carefully, since there have been fake Ledger apps on the Chrome web store before.

3. Download the utils scripts and copy them to use locally in whatever directory you are running the commands from. You should have installed python already for the first step.

    1. [join_pgp_lines.py](https://pastebin.com/zXmExGJR) 

    2. [break_pgp_lines.py](https://pastebin.com/a4rJGLfP) 

Then, connect your hardware wallet to a trusted computer. If there instructions for the GPG agent that apply to your wallet model, follow those.

* If you’re using the Trezor, make sure to enter your PIN.

* If you’re using the Ledger, make sure that you have entered your PIN, and the SSH/GPG Agent app is selected.

> **Note:** On Linux, you won’t be able to pipe single line PGP messages into the gpg tool, since they need to be on separate lines. The `break_pgp_blocks.py` script will do this for you. So for messages like this:
> 
> ```
> -----BEGIN PGP PUBLIC KEY BLOCK-----\n\nmFIEAAAAABMIKoZIzj0DAQcCAwScqdQPCY5q1z9ElD0oS5/GK8sHDDYbeP5okqzH\n1qMVsjTBHh8LZnTL/aNhc4iuTiZnsYw6C9YUN61jq45mDFbLtAxUZXN0YXRvciBU\naW2IgAQTEwgAHAUCAAAAAAILCQIbAwQVCAkKBBYCAwECF4ACHgEAFgkQh4S7AAAj\n2IILGlRSRVpPUi1HUEcMOgD/Yv7YQeHshIX2aNnNMbqK5h+ZnNnP3BHHRspp/Ezw\nmtMA/0NHZwY768Sz6kxuM4FtcCO/5OjqOt+q9JY25e5R1zBGuFYEAAAAABIIKoZI\nzj0DAQcCAwRwGg2zY9ptKBBDEwG/WAHPmv7Gv3yVs58XQkztv03D18t/2T9OXF9q\nQ/GGC0sOSauFFa8NnXNxNsCM4HmWOnOlAwEIB4hsBBgTCAAJBQIAAAAAAhsMABYJ\nEIeEuwAAI9iCCxpUUkVaT1ItR1BHDD8BAODRj5xRjEq5uhvs09FN46yjnKFaJ8uP\nGH/m2GIH74MmAPjRDZe8GnlkO6sGdDxwgYiOscJSE73SwZ01kwswuU7g\n=+R4g\n-----END PGP PUBLIC KEY BLOCK-----\n
> ```
> 
> 
> You can pipe them like this:
> 
> ```
> echo '-----BEGIN PGP PUBLIC KEY BLOCK-----\n\nmFIEAAAAABMIKoZIzj0DAQcCAwScqdQPCY5q1z9ElD0oS5/GK8sHDDYbeP5okqzH\n1qMVsjTBHh8LZnTL/aNhc4iuTiZnsYw6C9YUN61jq45mDFbLtAxUZXN0YXRvciBU\naW2IgAQTEwgAHAUCAAAAAAILCQIbAwQVCAkKBBYCAwECF4ACHgEAFgkQh4S7AAAj\n2IILGlRSRVpPUi1HUEcMOgD/Yv7YQeHshIX2aNnNMbqK5h+ZnNnP3BHHRspp/Ezw\nmtMA/0NHZwY768Sz6kxuM4FtcCO/5OjqOt+q9JY25e5R1zBGuFYEAAAAABIIKoZI\nzj0DAQcCAwRwGg2zY9ptKBBDEwG/WAHPmv7Gv3yVs58XQkztv03D18t/2T9OXF9q\nQ/GGC0sOSauFFa8NnXNxNsCM4HmWOnOlAwEIB4hsBBgTCAAJBQIAAAAAAhsMABYJ\nEIeEuwAAI9iCCxpUUkVaT1ItR1BHDD8BAODRj5xRjEq5uhvs09FN46yjnKFaJ8uP\nGH/m2GIH74MmAPjRDZe8GnlkO6sGdDxwgYiOscJSE73SwZ01kwswuU7g\n=+R4g\n-----END PGP PUBLIC KEY BLOCK-----\n' \
> | python break_pgp_blocks.py | gpg --import -
> ```
> 
> 
> If you don’t do this, you may get the following output:
> 
> ```
> gpg: no valid OpenPGP data found
> gpg: Total number processed: 0
> ```

Next, to use the script with your wallet, use the following commands to set the `GPG_WALLET` and `GNUPGHOME` environment variables. These will only last for the current session:

```
export GPG_WALLET=ledger
export GNUPGHOME=~/.gnupg/ledger
```


Or

```
export GPG_WALLET=trezor
export GNUPGHOME=~/.gnupg/trezor
```


You can also just replace the `$GPG_WALLET` in the following command with `ledger` or `trezor` instead of exporting that environment variable, since you should only need to interact with the ledger-agent or trezor-agent to initialize the keyring.

To initialize the keyring, you can recover the key using the default timestamp (which is 0). **NOTE: This will destroy any existing keys in `~/.gnupg/[ledger,trezor]/`, where the last name in that path corresponds to which hardware wallet you’re using**.
Use the following commands:

```
sudo rm -rf ~/.gnupg/$GPG_WALLET
$GPG_WALLET-gpg init --time=0 [user info] -v
```

The `[user info]` does not have to be your real name and email, but it should be unique if possible. Here are some examples:

```
ledger-gpg init --time=0 "Beneficiary Bob <bbob@notmail.com>" -v
trezor-gpg init --time=0 "Testator Tim" -v
```
Make sure to confirm on the device each time it asks you to provide the public key or sign. 

### Beneficiary Instructions (Exporting a secret):

1. Export your public GPG key. You can export your key into a file: 

```
gpg --armor --output [public key output path] --export [user info]
```


If you want to print the key to the command line to copy/paste it, use a `-` (dash) symbol:

```
gpg --armor --output - --export [user info]
```


2. Verify the above GPG key with your ETH account. You can use the "Sign" functionality at  [MyCrypto](https://mycrypto.com/sign-and-verify-message/sign) for this. Note that Metamask supports both Trezor and Ledger.  

3. Deliver the signed message from the step above to the testator. If you did the above step while offline, it’s worth keeping the signed message offline if you are able to. The final result should look like this:

```
{
  "address": "0xF021525F814AE849ADfBf954caefafE0E38A701b",
  "msg": "-----BEGIN PGP PUBLIC KEY BLOCK-----\n\nmFIEAAAAABMIKoZIzj0DAQcCAwQOekoNXoNKljSo+0L0/sy0CkQw6pjkA7Pxi8nO\nGswIsNCncW5JKANyQKasvs5KbnGWiW5t4zCSBKlvQp+tLhrftB4nU2VjcmV0IEFn\nZW50IDxudWxsQG51bGwuY29tPieIgAQTEwgAHAUCAAAAAAILCQIbAwQVCAkKBBYC\nAwECF4ACHgEAFgkQ/ZyAB9VJ1pMLGlRSRVpPUi1HUEemZwD7BZ+bY4J/L36YihBr\nyqD8tLvJmh4zjHQSrx3QGU01fpcA/1xFBlxq2rk9L/u0Q/B+JlNCd64/QAkhGXLr\nFYOtdSv8uFYEAAAAABIIKoZIzj0DAQcCAwRoitm0zlet3j/Wp+K5U54nT+s6/9VE\niOT0BnuSSZma/Up4zX2TQ8e1eCIBEYUXq6+6MpLoif3F43WaWFeJ4VDDAwEIB4ht\nBBgTCAAJBQIAAAAAAhsMABYJEP2cgAfVSdaTCxpUUkVaT1ItR1BH/SYBANuEdiTH\n12TXw+SAWDzQGpBSroLT/oa4tJWZn9diaS8XAP0btI52dZebWuZnnN4pPAr/JtrU\n7BUt/f446xODl6MyBQ==\n=4TgG\n-----END PGP PUBLIC KEY BLOCK-----",
  "sig": "0xa739225b6b04f7959d3643753cfd1c935cd1edfe45975f04cd535f25c82be750495f7dce5e8b5fae21d60c08be26c84710e007e3311a87acbc87c9b6660417ff1b",
  "version": "2"
}
```

### Testator Instructions (Importing + encrypting a secret):

1. Get the signed GPG key from the beneficiary (see above for an example of what this looks like). Use the "Verify" functionality at [MyCrypto](https://mycrypto.com/sign-and-verify-message/sign) and double check that the ETH address is the one that you want to mark as the beneficiary. 

2. Import this key by either

Saving the key to a file and passing the path:

```
gpg --import [public key input path]
```


Or, copying just the PGP key block from the message, and piping it into the script (example included below). Note that only the public key from the signed message is passed in, the other parts of the message are not needed.

```
echo [pgp public key block] | gpg --import -
```

Example:

```
echo "-----BEGIN PGP PUBLIC KEY BLOCK-----\n\nmFIEAAAAABMIKoZIzj0DAQcCAwQOekoNXoNKljSo+0L0/sy0CkQw6pjkA7Pxi8nO\nGswIsNCncW5JKANyQKasvs5KbnGWiW5t4zCSBKlvQp+tLhrftB4nU2VjcmV0IEFn\nZW50IDxudWxsQG51bGwuY29tPieIgAQTEwgAHAUCAAAAAAILCQIbAwQVCAkKBBYC\nAwECF4ACHgEAFgkQ/ZyAB9VJ1pMLGlRSRVpPUi1HUEemZwD7BZ+bY4J/L36YihBr\nyqD8tLvJmh4zjHQSrx3QGU01fpcA/1xFBlxq2rk9L/u0Q/B+JlNCd64/QAkhGXLr\nFYOtdSv8uFYEAAAAABIIKoZIzj0DAQcCAwRoitm0zlet3j/Wp+K5U54nT+s6/9VE\niOT0BnuSSZma/Up4zX2TQ8e1eCIBEYUXq6+6MpLoif3F43WaWFeJ4VDDAwEIB4ht\nBBgTCAAJBQIAAAAAAhsMABYJEP2cgAfVSdaTCxpUUkVaT1ItR1BH/SYBANuEdiTH\n12TXw+SAWDzQGpBSroLT/oa4tJWZn9diaS8XAP0btI52dZebWuZnnN4pPAr/JtrU\n7BUt/f446xODl6MyBQ==\n=4TgG\n-----END PGP PUBLIC KEY BLOCK-----" \
| gpg --import -
```

3 . Encrypt a message with the PGP public key you imported.

To see the keys you’ve imported, use 

```
gpg -k
```
    
Then use the following command to encrypt a secret, using either a recipient’s name or their public key ID from the output above. Examples are below.

```
gpg --armor --output [encrypted secret output path] --encrypt --recipient [recipient identifier] [raw secret input path]
```

Documentation for recipient identifiers: [https://www.gnupg.org/documentation/manuals/gnupg/Specify-a-User-ID.html](https://www.gnupg.org/documentation/manuals/gnupg/Specify-a-User-ID.html)

Example: Typing the secret out on the command line, encrypting for a specific recipient’s key ID, and getting the encrypted secret back on the command line (note the two dashes that refer to stdout and stdin):

```
echo "my secret" | gpg --armor --output - --encrypt --recipient 795969B46BDC09B92B69A213FD9C8007D549D693 -
```


You will get a warning when encrypting saying `"It is NOT certain that the key belongs to the person named in the user ID"`. You can ignore this.

**Note:** If you forgot which key you imported when referring to a recipient, you can view information about a PGP public key block by using it with

```
gpg -v
```

Example:

```
echo "-----BEGIN PGP PUBLIC KEY BLOCK-----\n\nmFIEAAAAABMIKoZIzj0DAQcCAwQOekoNXoNKljSo+0L0/sy0CkQw6pjkA7Pxi8nO\nGswIsNCncW5JKANyQKasvs5KbnGWiW5t4zCSBKlvQp+tLhrftB4nU2VjcmV0IEFn\nZW50IDxudWxsQG51bGwuY29tPieIgAQTEwgAHAUCAAAAAAILCQIbAwQVCAkKBBYC\nAwECF4ACHgEAFgkQ/ZyAB9VJ1pMLGlRSRVpPUi1HUEemZwD7BZ+bY4J/L36YihBr\nyqD8tLvJmh4zjHQSrx3QGU01fpcA/1xFBlxq2rk9L/u0Q/B+JlNCd64/QAkhGXLr\nFYOtdSv8uFYEAAAAABIIKoZIzj0DAQcCAwRoitm0zlet3j/Wp+K5U54nT+s6/9VE\niOT0BnuSSZma/Up4zX2TQ8e1eCIBEYUXq6+6MpLoif3F43WaWFeJ4VDDAwEIB4ht\nBBgTCAAJBQIAAAAAAhsMABYJEP2cgAfVSdaTCxpUUkVaT1ItR1BH/SYBANuEdiTH\n12TXw+SAWDzQGpBSroLT/oa4tJWZn9diaS8XAP0btI52dZebWuZnnN4pPAr/JtrU\n7BUt/f446xODl6MyBQ==\n=4TgG\n-----END PGP PUBLIC KEY BLOCK-----" \
| gpg -v
```

### Decrypting a secret:

To decrypt a secret, you can do one of the following:

1. The easiest method is to pipe the secret into the utils script. If the secret looks like this:

```
-----BEGIN PGP MESSAGE-----

hH4Dj94JO0/EwSUSAgMEtABhDoJVXxqOUO/cuCInDX3Xkz2tfAHP4ANtGX/FGzPm
xoqvFoTAXqrkw6H/gd5xWxkO2htW8wjzHVbGMGBBeDCO1yWCRIGuorOnDux57jST
V0lXeekSJAa6pfEOab3txVyjQ9QbPZX1HmI/TDr5ejbSRQHjJiRwpYW9hnWbnXtO
3aj5OHaMsuBNq3k1orMET5hk6G9bY0bV4fmxI1bmFIxOfwH5NRalMkwbn+gYfl2g
d6gAdeQSVw==
=g2i4
-----END PGP MESSAGE-----
```

And not like this:

```
'-----BEGIN PGP MESSAGE-----\n\nhH4Dj94JO0/EwSUSAgMEtABhDoJVXxqOUO/cuCInDX3Xkz2tfAHP4ANtGX/FGzPm\nxoqvFoTAXqrkw6H/gd5xWxkO2htW8wjzHVbGMGBBeDCO1yWCRIGuorOnDux57jST\nV0lXeekSJAa6pfEOab3txVyjQ9QbPZX1HmI/TDr5ejbSRQHjJiRwpYW9hnWbnXtO\n3aj5OHaMsuBNq3k1orMET5hk6G9bY0bV4fmxI1bmFIxOfwH5NRalMkwbn+gYfl2g\nd6gAdeQSVw==\n=g2i4\n-----END PGP MESSAGE-----\n'
```

Then you’ll need to format it using the provided `join_pgp_blocks.py` script. Do this by first copying the secret, and then run

```
python join_pgp_blocks.py
```

This program will wait for input from you. Paste the secret, and hit Ctrl+D to obtain the formatted result. Once you copy that, to decrypt the secret you can run:

```
gpg --output [encrypted secret input path] --decrypt [decrypted secret output path]
```

To input the formatted secret and output it to the command line, you can use:

```
echo [pgp message] | gpg --output - --decrypt -
```

Example:

```
echo '-----BEGIN PGP MESSAGE-----\n\nhH4Dj94JO0/EwSUSAgMEtABhDoJVXxqOUO/cuCInDX3Xkz2tfAHP4ANtGX/FGzPm\nxoqvFoTAXqrkw6H/gd5xWxkO2htW8wjzHVbGMGBBeDCO1yWCRIGuorOnDux57jST\nV0lXeekSJAa6pfEOab3txVyjQ9QbPZX1HmI/TDr5ejbSRQHjJiRwpYW9hnWbnXtO\n3aj5OHaMsuBNq3k1orMET5hk6G9bY0bV4fmxI1bmFIxOfwH5NRalMkwbn+gYfl2g\nd6gAdeQSVw==\n=g2i4\n-----END PGP MESSAGE-----\n' \
| gpg --output - --decrypt -
```
