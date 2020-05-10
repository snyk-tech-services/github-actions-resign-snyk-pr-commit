# Github action to resign Snyk PR commits

Snyk opens PRs for remediation or dependencies. Commits are not signed.
Protected branches can sometime require signed commits (that's what we do at Snyk!)

This action triggers on PR branches matching a pattern of your choice 
(snyk-fix- for Snyk Fix PRs) and amend the commit to recommit it signed using
the signature configured.

This is a workaround. Snyk capability to push signed commit will be added shortly.
To be honest, it might need less configuration than needed here but just hacked it together.

## Prerequisites
#### A bot account (one time setup)
- A bot (if you don't want to use your own/existing profile). Here a "bot" account is really just 
a github account, nothing more. Open incognito window, create a new account with an email
not already in use by an existing account, verify the email, and you just made a bot !

#### GPG setup for the bot account (one time setup)
- Create a GPG key as instructed [here](https://help.github.com/en/github/authenticating-to-github/generating-a-new-gpg-key)
Keep your GPG Key Id (step 11) and Passphrase handy
- Add your GPG to your bot github account as show [here](https://help.github.com/en/articles/adding-a-new-gpg-key-to-your-github-account)

- Export your keys using the commands\
`gpg --export KEYID > public.key` \
`gpg --export-secret-key KEYID > private.key`
- Get their Base64 encoding\
`base64 -w 0 public.key`\
`base64 -w 0 private.key`

#### Repo secrets - to add to each repo where you want that workflow to work
Add the base64 encoded keys, KeyID and passphrase to your repo secrets\
`PUBLICKEY` (the output of `base64 -w 0 public.key`)\
`PRIVATEKEY` (the ouput of `base64 -w 0 private.key`)\
`GPG_PASSPHRASE`\
`KEYID`

It is now ready to use.

## Usage
In your github repo, add .github/workflows/config.yml after tweaking what you need:
- line 10: PR pattern (snyk-fix for Fix PRs)
- line 12: User name of your bot
- line 13: Email of your bot

Commit and merge, and you'll be good to go.


### How does that work?
- This workflow uses actions/checkout@v2 to checkout the repository PR branch
- It then configures git client the same way you would on your machine
- Setups a bunch of things to allow to use gpg-preset-passphrase 
to preset the passphrase programmatically
- Encrypts a dummy text to force the GPG cache to cache the preset passphrase
- Then runs the git amend commend to recommit things signed

#### Where it sucks
This will work on a repo basis. Meaning you need to setup the secrets for each repo.
Might be a way to distribute secrets across numerous repos, but I don't know about it.
Admittedly I haven't looked for it :D 