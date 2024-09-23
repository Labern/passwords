# How do passwords work?
By Labern

## Passwords and plain text
The phrase to refer to a password as-is is plain text, meaning literally what is written. So the plain text version of "password" is "password". It turns out that while we refer to passwords via the things we write, the way they actually work is quite different. Here is a brief descrition that, while short, will explain how that process works. 

Don't be intimidated by the technical terms: I'll explain everything in clear English, and only then introduce the technical term — never the other way around.

## Why aren't passwords good the way I write 'em?
Just having a password makes no sense. If every website you visited literally saved what you wrote, you would be hacked within minutes. If you know that most passwords are of a certain length, you can brute force them by writing a program which runs through every possible variation, e.g. aaaaaaaa through zzzzzzzz. (This would catch a password like "password" just after "passworc" and "passworb", assuming you run through every letter and number.) 

Yet passwords are still a useful way for a user to let a website or tool know that they are who they say they are. What is needed is a way to store some version of the password without actually storing the password itself. Best of all would be a scrambled version that no one could guess. It turns out this is exactly what happens, and that this scrambled form of a password is called a "hash". To produce a hash, we need a hashing function, by which I mean "a thing that takes plain text and produces a hashed version of it". One actual example of this — a non-reversible hash function — is [Bcrypt](https://github.com/bcrypt-ruby/bcrypt-ruby), which allows for the password to be stored in a new, scrambled form. All you need to know is that a hash is a scramble, and a hash function is a thing which does the scrambling. 

You don't need to understand how a hashing function works: you only need to know that it works, and why it is needed. You put the toast in; let the toaster do the work. Then eat the toast. (If you want; I'm not telling you what to eat. You look great.)

For example, it is far better to let the user enter "password" and let the database have the non-reversible form of "password" -> hashing function -> $2a$12$hS9f57hYG28yn5AaTh/0fu8suk/MbUn7ZTNrAgUQRWqTD9w7HXkBi (for example). This way, the database on which the password is stored never knows the user's actual input, but can verify the user later by matching the password. (The use of salt, which adds an extra element of unpredictability, is stored inside the long list of letters, which allows the database to match the input password "password" with the digest).

## Technical terms

The input password is what the user types in. This is referred to as the key. BCrypt is one hashing function that turns input (the key) into a long string of text. The long string is called a digest. 

So: key -> hash function -> digest. Here is an example of a key ("password"), a hashing function (BCrypt), and a digest.

```ruby
    "password" -> BCrypt -> $2a$12$hS9f57hYG28yn5AaTh/0fu8suk/MbUn7ZTNrAgUQRWqTD9w7HXkBi
```

### Now you know cryptography
Knowing just these few words means I just taught you quite a lot about cryptography. Some other bonus words: a cipher is the method or technique used to encode or encrypt something. Speaking of which, to encrypt something is to convert it from its original form into a version that is scrambled or encoded. Now, to really teach you something, technically encryption is reversible by decryption (makes sense, right?) while hashing — which passwords use — is not reversible. This is why we **encrypt messages** (so that the sender and the person receiving them can both read them) but hash passwords.

Now, if you're really interested, you can see how a hashing function works.

## The details (or: how a hash function works)

Bcrypt, applied like so:

```Ruby
    password = BCrypt::Password.create("password")
    password == "password" # True
    password.cost          # 12
```

Allows for this to happen. The result of the above hash is:

```Ruby
    $2a$12$hS9f57hYG28yn5AaTh/0fu8suk/MbUn7ZTNrAgUQRWqTD9w7HXkBi
```

This output, called a digest, correponsds to multiple parts:

```
    $2a$12$R9h/cIPz0gi.URNNX3kh2OPST9/PgBkqquzi.Ss7KIUgO2t0jWMUW
    \__/\/ \____________________/\_____________________________/
    Alg Cost      Salt                        Hash
```

That is:

1. The first three letters — <code>$2a</code> — refer to the algorithm used.
2. The second part — in this case, <code>12</code> — refers to the cost. The numbers refers to <code>2^cost</code>, that is <code>2<sup>12</sup> = 4096</code>. The greater the cost, the more computational work required.
3. The third part refers to the salt, which is extra spice which makes rainbow tables irrelevant. (That is, a rainbow table is a pre-prepared set of hashed passwords which would take the form of knowing all possible passwords for a specific set of characters. By adding a salt, knowing a password and hashing it does not allow for the crack.)
4. The fourth part is the hash itself, or the end product of the password having been run through Bcrypt.

## What?
The point of this was not for you to understand all of the details. It was to show that passwords are not what they appear. When you type out a password when you sign in to Google, the moment you click submit, the actual English you wrote is thrown away, and the relevant output (the digest) is compared to what Google stored when you saved your password. This is why you are asked to enter your password, and also why your password isn't hacked every five minutes.

All of this falls under a branch of computer known as cryptography, and technically has nothing to do with computers. It is, however, used by them, and so becomes something a person interested in programming comes to study. Since we use computers daily, it is nevertheless interesting.

### Extra details
I omitted some details, such as the fact that passwords can also include some characters other than a-z and 0-9, but the same remains true: for every new character you add to your password, the more possible outcomes there are. This is why longer passwords are forced upon us by sites more and more often.

Though strange at first, this subject is quite simple, conceptually; the difficultly is really in understanding how it is implemented.
