oclHashcat walkthrough
=====================

This document is meant to highlight why [short passwords can be a problem][1] and provide a walkthrough to anyone who wants to evaluate this problem themselves.

----------
#### Background and Motivation
Our environment's minimum password requirement (8 characters using a mixture of uppercase, lowercase and digits) seemed reasonable. We wanted to test that assumption and also get a feel for what percentage of our users only set this minimum requirement.

We suspect many other people have similar environments and wanted to document the results we found and the methods used when testing our passwords.

Ideally this document will help when setting password policies or auditing your own system security.

#### Caveat and Caution
While we are IT professionals, we are not security experts.  Setting these basic tests took only a few hours to accomplish.  Take all of our methods and findings with a grain of salt, but also be aware that professionals with nefarious intentions would likely be much more successful than we were.

#### Saltfree Hashes
We use a locally maintained [LDAP][2] server to replicate passwords and accounts in [Google Apps][3].

The [Google Apps Directory Sync][4] tool requires [unsalted][5] [SHA1][6] passwords to keep an enterprise LDAP or Active Directory in sync with its Google Apps accounts.

Unsalted SHA1s are a known to be a poor choice for maintaining password hashes. The lack of an individual salt makes them susceptible to [rainbow attacks][7] and the SHA1 hash can be computed very efficiently using GPU hardware.

You SHOULD **lock down** the LDAP attribute which holds the unsalted hashes to limit access.  Only supporting this type of hash for synchronization is an unfortunate security choice by Google Apps.

----------
### System Setup
#### Hardware Setup
 -  **Desktop Machine:** Lenovo M91p
     -  Older era hardware, circa 2011
     -  Any basic desktop with a recent PCI Express 3.0 slot will suffice.
 -  **Graphics Card:** Radeon R9 270X 4GB PCI-Express Graphics Card
     -  Currently available for around $200
     -  Late 2013 hardware

#### Software Setup
 - **Operating System:** Ubuntu 14.04 Desktop
 - **GPU Driver:** AMD Catalyst Graphics Drivers and OpenCL - Version 14.4-rev2
 - **Cracking Software:** [oclHashcat][8] v1.21
 - **Password Dictionary:** [CrackStation Dictionary][9] (they also include all words found in Wikipedia!)

#### Security Considerations
Since user hashes and potentially passwords will be stored in plaintext on the test machine,
we disconnected it from the local network after the initial configuration and [sneakernetted][10] the sensitive data.

----------
### Performance

#### Raw Hashes Per Second
The GPU used for this exercise is able to test about 1.6 **Billion** hashes every second in brute force mode.

For a comparison of an alternate method, an [Amazon EC2 Compute Node][11] (g2.2xlarge) which can be rented for less than a dollar an hour, can test about 400 **Million** hashes every second in brute force mode. (and would also require you to upload your hashes to their cloud)

#### Performance Tweaks

You can modify the 'workload profile' setting in hashcat to be slightly more aggressive. It typically gave about 10% speedup, but wasn't used for most of the testing.

----------
### Results

#### Dictionary Attack
The dictionary attack is less efficient to run, but very effective in getting passwords.  It was able to process about 13 million hashes per second.  However it took only **7 minutes** to check over a billion known passwords against our hashes.  It was able to quickly find about **5%** of our passwords. About half **2% in total** of the dictionary derived passwords were 8 characters in length.

#### Brute Force Attacks
Below are the approximate time windows it took or would take to brute force these password configurations.
(Simple = Uppercase, Lowercase and Digits, Complex = all characters)

    7 Digit Simple   -   37 Minutes (yielded a handful of legacy user passwords and passwords set by admins)
    7 Digit Complex  -   12 Hours (not run)
    8 Digit Simple   -   40 Hours (yielded about 10% of user passwords)
    8 Digit Complex  -   ~50 Days (not run)
    9 Digit Simple   -   ~100 Days (not run)
    More complex     -   >10 years (not run -- software estimates here)

----------
### Discussion

#### Meeting The Minimum
At least 15% of our users set their passwords to **JUST** meet our minimum specification.  This is better than I had originally expected, but still included a large number of users including many 'engineer-types' who I had hoped would know better.

#### Speed Ups
Brute force attacks can be parallelized, it would be trivial to add GPUs or additional machines to speed up performance.
We considered doing a parallelized test using Amazon's cloud, but it would have cost about $120 to perform a simple 8 character test and I wasn't comfortable uploading our hashes to the cloud.

Newer GPUs appear to continue to increase the available the hash rate.

#### Problematic Passwords
A common problem seemed to be things like a person's name followed by digits. (eg. Joel1985)  Passwords like this are particularly susceptible to dictionary attacks. It's quite likely that users use passwords like this when they are forced to change them frequently.  (eg. Joel1985 becomes Joel1986, etc.)

----
### Conclusion

#### Raising The Bar (10 Character Minimum)
Considering that our minimum password specification was used by about 15% of users and a $200 investment was able to break all of those passwords over a weekend, we feel it is reasonable to increase our minimium password standard to at least 10 characters including uppercase, lowercase, digits and special characters. We will need to improve our password change page to help users easily manage this change.

#### Password Managers
A realistic option for managing so many long (12 character+) passwords seems to be using software password managers.  We need to be able to help users set these tools up and make sure they secure them with good passwords as well.

#### Checking Against The Dictionary
We should check new passwords against the dictionary lists we have.  This will prevent dictionary words or commonly used passwords from getting set as active passwords.

----------
### Methods Used

#### Explanation of hashcat command line options
    --hash-type=100 means unsalted SHA1

    .out files contain cracked passwords

    hashlist is text file with one unsalted SHA1 hash per line (40 characters each)

    crackstation.dict is the dictionary file referenced above \
    (contains a billion commonly used passwords)

    --attack-mode=3  means brute force

    --custom-charset1=?l?u?d means check lowercase, uppercase and digits

    ?1 matches charset 1

    ?a matches ANY charcater



#### Dictionary Attack
    ./oclHashcat64.bin --hash-type=100 --outfile=dictionary.out hashlist crackstation.dict

#### Brute Force Attacks
    # 7 Character Simple (upper, lower, digit)
    ./oclHashcat64.bin --hash-type=100 --outfile=brute7simple.out --attack-mode=3 --custom-charset1=?l?u?d  hashlist ?1?1?1?1?1?1?1

    # 7 Character Complex (all character)
    ./oclHashcat64.bin --hash-type=100 --outfile=brute7complex.out --attack-mode=3 hashlist ?a?a?a?a?a?a?a


    # 8 Character Simple (upper, lower, digit)
    ./oclHashcat64.bin --hash-type=100 --outfile=brute8simple.out --attack-mode=3 --custom-charset1=?l?u?d  hashlist ?1?1?1?1?1?1?1?1

    # 8 Character Complex (all character)
    ./oclHashcat64.bin --hash-type=100 --outfile=brute8complex.out --attack-mode=3 hashlist ?a?a?a?a?a?a?a?a

    ...







> Written using [StackEdit](https://stackedit.io/).


  [1]: http://en.wikipedia.org/wiki/Password_strength
  [2]: https://en.wikipedia.org/wiki/Ldap
  [3]: https://en.wikipedia.org/wiki/Google_apps
  [4]: http://static.googleusercontent.com/external_content/untrusted_dlcp/www.google.com/en/us/support/enterprise/static/gapps/docs/admin/en/gads/admin/gads_admin.pdf
  [5]: https://en.wikipedia.org/wiki/Salt_%28cryptography%29
  [6]: https://en.wikipedia.org/wiki/Sha1
  [7]: https://en.wikipedia.org/wiki/Rainbow_table
  [8]: http://hashcat.net/oclhashcat/
  [9]: https://crackstation.net/buy-crackstation-wordlist-password-cracking-dictionary.htm
  [10]: https://en.wikipedia.org/wiki/Sneaker_net
  [11]: http://aws.amazon.com/ec2/
