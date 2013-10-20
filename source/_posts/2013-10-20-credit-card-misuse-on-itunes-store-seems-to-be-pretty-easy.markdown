---
layout: post
title: "Credit Card misuse on iTunes Store seems to be pretty easy"
date: 2013-10-20 09:37
comments: true
categories: iTunes, Security
---

It is already almost two years ago since I wrote a blog post about the
misusage of my Credit Card over iTunes. At that time, I was pretty sure that
someone hacked into my account and bought apps/music/whatever with my data
stored in there.

So I filed (pretty angry that time) this blog post under the title
[iTunes Store hacked?](http://ramonski.de/blog/2012/04/26/itunes-store-hacked)
(german language only).

But this apparently wasn't the case. I seemed to be much less sophisticated
than I thought and is sadly still beeing pratised ...

<!--more-->

## The beginning (short story)

It was the 24th of April 2012 when I checked my online banking account and
couldn't trust my eyes. There were several bookings from **APPLE ITUNES STORE
EUR/LUXEMBOURG** with a total of already 160 € and it seemed that this flood
of write-offs from my credit card just started and that there are more in the
queue to get booked off my bank account.

{% img /downloads/images/visa.jpg %}

In panic I checked my iTunes Store account and had already horror scenarios in
my head about what else these guys will do with my theft personal data which
is stored in there. But surprisingly I saw nothing suspicious in there.

So I decided to block my credit card and write to the iTunes Store Support
about this incident, which can be read in the [original blog post](http://ramonski.de/blog/2012/04/26/itunes-store-hacked)

The end of the story was, that I got the money back, received a new credit
card and changed my passwords almost everywhere...


## The days after

A short time after I posted my story, other people wrote me via email or in
the comments section that this happened to them as well. Some people told me
that they got ripped off over 800 € and more.

But it was the very first comment which openened my eyes what is really going
on there. It wasn't the account which got hacked, it was simply my credit card
data which got stolen and afterwards beeing used to create a new iTunes
account to buy stuff from the iTunes Store. Probably contents like iPhone/iPad
apps or books which they had released by themselves to get "real" money for
the sold apps from Apple... What a clever idea ...

But is it really that easy?

Probably yes, this is how I would do that:


## Steps to reproduce (theoretically/untested)

**Disclaimer**:
  This is only how I *thought* this could be reproduced and is not meant to be a
  guide to do this in real. Actually, I don't know if it actually works that
  way...


1. **You need some valid credit card data.**

   *My credit card data got probably stolen in a Hotel in Bangkok/Thailand.
   So I assume that places where people give their cards away to someone for
   payment are good places to grab the desired data with almost no effort
   (especially in Thailand and/or if you know someone who is working in such a
   position)*

2. **You need to create an iTunes account where you use this credit card.**

   https://appleid.apple.com/cgi-bin/WebObjects/MyAppleId.woa/wa/createAppleId

   *There is no check if the credit card is already in use or if it actually
   belongs to you. So just add some of the credit card data gathered
   from step 1. Hint: repeat step 2 for each dataset from step 1*

3. **Download [iBooks Author](http://www.apple.com/de/ibooks-author/) and
   publish some crappy content (use another account for that).**

   *The advantage with this approach is that you don't need to participate
   on the Apple developer program. This saves 80 € and is much more simpler.*

4. **Buy this stuff with all of your credit card owned fake accounts.**

5. **Get the money from Apple**

   *probably only if the people don't recognize it early enough or your
   account gets locked*


## Ways to avoid this

After the first responses to my [first blog post](http://ramonski.de/blog/2012/04/26/itunes-store-hacked)
I thought about ways how to avoid the misuse of stolen credit card numbers within
the iTunes store. I also decided to write an email to the Apple support
describing my insights.

1. **Check if the credit card is alredy in use by another account**

``` python

def is_credit_card_in_use(number):
    if number in get_all_credit_card_numbers():
        # credit card used by another account!
        # get the accont and send a notification mail
        # to this email to confirm.
        ...
```

2. **Check if the credit card holder is the owner and has access to the account.**

   This could be done by a 1 cent debit from this credit card with a generated
   **UID in the reference field** which has to be entered afterwards to *activate*
   the account. Probably this 1 Cent could be credited after account
   activation to this new created account or simply be donated to charities:)

Neither one of the two insights have been replied/recognized by the Apple support.
Probably this email has just made it directly to the trashcan, or printed out to
be immediately shreddered.


## Why is this still possible?

The last comment from a guy who faced the same problem was now 1 month ago. So
it seems to be still possible to simply misuse payment data in the iTunes
store.

So I ask myself why this is still possible and came to the following
assumptions:

- It is a calculated risk which is covered by the credit card insurances.

- Any kind of security is too complicated to the consumer - or - the time from
  account creation to the first possible electronic purchase is too long.


## Conclusion

It seems to be pretty easy to abuse credit card payment to quickly get your
low quality and overpriced crap-app on the road.

**And ironically:** You can give your app 5 star ratings with your fake
accounts so that others buy it as well.

If I would have more criminal energies, I would give that definitely a shot..

So for the time being, check your credit card account in a regular manner and
don't be too afraid if you have write offs from APPLE ITUNES on your account.

Just lock your credit card, you'll get your money back, thats the reason why
we pay for having a credit card, to be insured when somtehing like this
happens:)

Cheers ramonski
