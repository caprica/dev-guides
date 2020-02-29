# SSH Configuration for AWS CodeCommit

Source code repository access is via SSH.

This requires, if not already done, SSH configuration and an SSH key-pair on the developer PC.

Using SSH means that username/password credentials need not be provided each time a change is made to the repository - as would be the case if using HTTPS/SSL.

See also the [reference documentation](https://docs.aws.amazon.com/codecommit/latest/userguide/setting-up-ssh-windows.html#setting-up-ssh-windows-keys-windows).

## Generate an SSH key-pair

In a command shell:

```
ssh-keygen
```

Follow the prompts.

Since it is possible to have many SSH identities, do not use the default filename for the key file and instead use `codecommit_rsa`.

The defaults can be used for everything else, entering a passphrase for the key-pair is entirely optional.

Here is an example terminal session:

```
mark@serenity:~$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/mark/.ssh/id_rsa): /home/mark/.ssh/codecommit_rsa
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/mark/.ssh/codecommit_rsa.
Your public key has been saved in /home/mark/.ssh/codecommit_rsa.pub.
The key fingerprint is:
SHA256:zBsKeiw3krlWS34/eQEnCNDsaERZFewWoLsMEXAMPLE mark@Serenity
The key's randomart image is:
+--[ RSA 2048]----+
|        E.+.o*.++|
|        .o .=.=o.|
|       . ..  *. +|
|        ..o . +..|
|        So . . . |
|          .      |
|                 |
|                 |
|                 |
+-----------------+
mark@serenity:~$
```

This will create two files in the `~/.ssh` directory:

```
codecommit_rsa     (private key)
codecommit_rsa.pub (public key)
```

## Associate the SSH key-pair with the AWS user identity

The public key needs to be associated with the AWS user identity.

Display the `codecommit_rsa.pub` file in the terminal or load it into a text editor, and copy the entire contents to the clipboard:

```
mark@serenity:~$ cat ~/.ssh/codecommit_rsa.pub
ssh-rsa EXAMPLE-AfICCQD6m7oRw0uXOjANBgkqhkiG9w0BAQUFADCBiDELMAkGA1UEBhMCVVMxCzAJB
gNVBAgTAldBMRAwDgYDVQQHEwdTZWF0dGxlMQ8wDQYDVQQKEwZBbWF6b24xFDASBgNVBAsTC0lBTSBDb2
5zb2xlMRIwEAYDVQQDEwlUZXN0Q2lsYWMxHzAdBgkqhkiG9w0BCQEWEG5vb25lQGFtYXpvbi5jb20wHhc
NMTEwNDI1MjA0NTIxWhcNMTIwNDI0MjA0NTIxWjCBiDELMAkGA1UEBhMCVVMxCzAJBgNVBAgTAldBMRAw
DgYDVQQHEwdTZWF0dGxlMQ8wDQYDVQQKEwZBbWF6b24xFDAS=EXAMPLE user-name@computer-name
mark@serenity:~$
```

_Make sure it is the public key file `codecommit_rsa.pub` and **not** the private key file `codecommit_rsa`._

The public key is everything inclusive from `ssh-rsa` up to and including  `user-name@computer-name`.

Use the [AWS console](https://console.aws.amazon.com/iam/home#/security_credentials), and select the `AWS CodeCommit credentials` tab.

From this tab, choose `Upload SSH public key`.

In the dialog that opens, paste the public key previously copied to the clipboard and confirm the dialog via the `Upload SSH public key` button.

When the dialog closes, an `SSH public key successfully updated` should be shown, and the new key included in the table below.

Copy the value in the `SSH key ID` column for the newly uploaded key to the clipboard (if there is more than one key look for the one with the most recent date).

The key ID will look something like this:

```
APKAZ3HSCS3JREXAMPLE
```

## SSH Client Configuration

Create a new `~/.ssh/config` file, with contents similar to the following:

```
Host git-codecommit.*.amazonaws.com
User APKAZ3HSCS3JREXAMPLE
IdentityFile ~/.ssh/codecommit_rsa
```

Replace the `User` value with that of the SSH key ID previously copied to the clipboard.

_Make sure this time it is the private key file `codecommit_rsa` and **not** the public key file `codecommit_rsa.pub`._

## Testing the SSH Client Configuration

The SSH configuration can be tested by executing the `ssh` command:

```
ssh git-codecommit.eu-west-1.amazonaws.com
```

An example terminal session:

```
mark@serenity:~$ ssh git-codecommit.eu-west-1.amazonaws.com
The authenticity of host 'git-codecommit.eu-west-1.amazonaws.com (52.95.116.110)' can't be established.
RSA key fingerprint is SHA256:zBsKeiw3krlWS34/eQEnCNDsaERZFewWoLsMEXAMPLE.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'git-codecommit.eu-west-1.amazonaws.com,52.95.116.110' (RSA) to the list of known hosts.
You have successfully authenticated over SSH. You can use Git to interact with AWS CodeCommit. Interactive shells are not supported.Connection to git-codecommit.eu-west-1.amazonaws.com closed by remote host.
Connection to git-codecommit.eu-west-1.amazonaws.com closed.
mark@serenity:~$
```

If this fails, some additional information may be available be retrying the `ssh` command with `-v`.

```
ssh -v git-codecommit.eu-west-1.amazonaws.com
```

If this command times out, most likely SSH access is blocked by a corporate proxy or firewall. If the required firewall change can not be made, then the fallback is to use the HTTPS/SSL configuration instead of SSH.

If this command fails for some other reason, check carefully all aspects of the SSH configuration.

## Conclusion

Generating the SSH key-pair is a necessary prerequisite to cloning the project source code from the repository.

The foregoing process only needs to be performed once, it is possible to use the same SSH credentials with multiple repositories.
