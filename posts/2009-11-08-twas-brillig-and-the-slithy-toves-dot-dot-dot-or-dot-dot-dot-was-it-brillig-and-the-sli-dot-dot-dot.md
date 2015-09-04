
So I managed to "forget" the passphrase to unlock my PGP private key. I kind of knew what the passphrase was, just wasn't quite
sure what it looked like. For instance, I knew it flowed as "alice smacked the jabberwock", but I couldn't remember if it was
"Alice smacked the Jabberwock", "alice SMACKED the jabberwock" or "Alice Smacked The Jabberwock" and so on...

So instead of revoking my key, I decided to find a way to guess the passphrase. Since I knew the words that made up the
passphrase and the order they were in, I decided to try a wordlist attack on my key. 

<!--more-->

If anyone else has "forgotten" their passphrase, this is one possible solution to getting it back. This won't let you crack
anyone's private key passphrase since you have to know the words that make up the passphrase, and the order they should be in. 

Knowing the order of the words isn't technically required. You could write a script that shuffles it around but it would make
it more complicated. In which case you might as well just do a full dictionary attack on the passhprase and hope for the best. 

Anyway the idea is to create a script that will generate a wordlist of all possible variants of the passphrase. Depending on
the length of your passphrase and the complexity, your script could become quite long and complicated.

Once you've got this list all you have to do is loop through the wordlist and pass each possible passphrase to gpg using the
--passphrase parameter to sign some dummy file. gpg returns 0 on success so once it does, you can quit the script and you've got your passphrase.

Using this method I was able to retrieve my passphrase. Note to self: don't forget it again. *Ever*.
