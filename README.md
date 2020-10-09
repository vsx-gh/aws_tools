# aws_tools
Tools for interacting with AWS - my best attempts, anyway. Some/all of these may be done
better and/or obsoleted by other folks or AWS engineers themselves.

## `awsenv`

Decrypts your GPG-decrypted AWS credentials and exports values as environment variables.

**Usage**:
```
. <path_to>/awsenv (initenv <credentials_file> | setdef <profile> | destroy)
```
We do this without a sub-shell so we can get the environment variables where we need them.

Currently this program works with files formatted like what you'd find in `~/.aws/credentials`,
that is, compatible with `boto3` and others. The `awscli` `~/.aws/config`, at least last I checked,
delineates non-default profiles with a `[profile <profile_name>]` tag, which I'm too lazy to
parse.

Since this code uses GPG, you'll obviously need some way to decrypt your file. You can set up 
`gpg-agent` to do what you need to do.

I wrote this program on macOS, which means older Bash (yeah, I know about Homebrew) and hence some
sexy features are missing. Feel free to fork and bring this into the new century, whydontcha.

**Initialize**:
```
. awsenv initenv <credentials_file>
```
The program defaults to `~/.aws/credentials`, but you can specify other files as you wish. Variables
are exported for each one set in the credentials file, prefixed with the profile name. So if you have
an `ilikeaws` profile, you'll end up with `ILIKEAWS_AWS_ACCESS_KEY_ID`, etc.

A variable containing all of your profile names is also exported, which helps you when switching profiles.
You can do a `printenv AWS_PROFILES` to see the list of your profiles.

**Set Default Variables**:
```
. awsenv setdef <profile>
```
Specify the profile whose variables you want to "link" to `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`,
and others. `AWS_DEFAULT_REGION` also gets set here if there is a value for the profile in question.
Run this after initializing, else you won't get anything.

**Unset Variables**:
```
. awsenv destroy
```
A cumulative variable, `AWS_VARS`, is set at initialization and contains all of the variables you set (except
for `AWS_ACCESS_KEY_ID` and the basic `AWS_` variables that `awscli` and others are looking for). Don't worry,
these variables still get unset; they just are not part of `AWS_VARS` because their "links" aren't known at
initialization. No decryption/encryption necessary for this item, so you can run it whenever and how often
you want.

**Programming Note**:

I attempted to make this script work with `getopts`, which is much classier for handling arguments, but I ran into scoping issues. If you crack that nut, I'm happy to see your pull request.

**Environment vs. Flat Files**  
Yeah, yeah, I know. I don't like environment variables any more than anyone else. But I hate the idea of leaving plain-text files on the filesystem and relying on permissions to protect them. Worse, I have to trust that there's no one on a shared system with the ability to elevate and subvert these permissions because they assumed root. This seems like a better path forward.
