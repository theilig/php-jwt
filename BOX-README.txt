This document is a forensic reconstruction of the provenance of this module.
History created on 15 Oct 2019 by Schnepper

Well, I started doing forensics - now I've added documentation and
recommendations as well

JWT is an industry standard format to specify tokens. It is used in two different flows at Box (as of Oct 2019):

1. OAuth2 Server Login Flow : https://developer.box.com/docs/construct-jwt-claim-manually
    This allows a customer to get an access token to an account in an enterprise without the user having to login manually.
    This is maintained by #developer-ecosystem

2. General API Authentication
    The service Security Token Service converts all API tokens (i.e. OAuth2 access_tokens) into another token (called internal-token)
    which is written in the JWT format. Some of the API calls are handled in the monolith so this library is used to validate the token created by STS.
    This is maintained by #eng-isf-team

References:
Note: JWT is a standard internet data format, these references are about the format,
      not necessarily about this library
https://confluence.inside-box.net/display/ETO/Box-JWT+Tool+Support+Guide
https://en.wikipedia.org/wiki/JSON_Web_Token
https://auth0.com/docs/jwt
https://jwt.io/
https://github.com/firebase/php-jwt

Problems with the current vendor_legacy/JWT
- It's originally vendor code that Box has modified
- We haven't updated it in 4.5 years (at the time of this writing)
- One of our modifications *may* have reintroduced a security hole that was the reason for
  our most recent update. (The 'none' signature algorithm)
  https://auth0.com/blog/critical-vulnerabilities-in-json-web-token-libraries/#Meet-the--None--Algorithm
    Per @nchandler -- 'none' is actually required, and it's a problem that the library doesn't
    support it.   And we need to review any code which specifies the 'none' algorithm
- Our implementation doesn't support time-skew between client & server (which has
  been a live-site problem recently) (and the reason I started to look into this module)
- Our implementation has code paths that allow a token to be used by the library user without being verified
  (This happens when a token is "hacked" to specify the "none" algorithm).
- Our implementation has scaffolding added to allow conversion to using `kid` in the header
  (Key ID) -- but we don't require it (this looks like transition code that was never completed)
   (Per @nchandler - we do rely on kid, which is why we modified the library to support it)

Additional problems:
- Providence of our vendor code is well understood, but not of vendor_legacy
- Our version apparently came from https://github.com/firebase/php-jwt/releases/tag/v2.0.0
- That repo has not been active for close to 2 years
- That repo is not documented as supporting PHP 7.3 (but likely is just due to inactivity)

Near Future:
- Update to a proper 'vendor' implementation.

Later Future:
- Look to change vendors - they are listed on https://jwt.io/ (filter for PHP)
  - a highly rated, and active, repo is: https://github.com/lcobucci/jwt


Box annotated history:

====================
commit bcf53b34d47b174097fd6e291f6030f0780d82b1
Author: Nadeem Ahmad <nahmad@box.com>
Date:   Thu Oct 3 07:37:24 2019 +1000

Style difference

====================
commit 6462fa7de747eac2716abba351d342a692a1e3b7
Author: Aniket <apatil@box.com>
Date:   Tue Nov 8 17:07:32 2016 -0800

        BOX-159379: Token exchange does not validate certain actor assertion claims

        "exp", "iat" and "nbf" were not being validated because the assertion is in plaintext


Apparent style change - refactoring code into methods

====================
commit e3b3038d5972f919e193e2d0b94fc4091ab3a346
Author: Vasishta Jayanti <vasishta@box.com>
Date:   Wed Sep 14 13:55:46 2016 -0700

        BOX-155897: Refactoring the JWTAdapter to be reusable for TokenExchange and JWTFlow

Introduces 'none' algorithm (security hole?)

====================
commit f980a6271c7d5d899c2a96a68bd152dd10b5b841
Author: Vasishta Jayanti <vasishta@box.com>
Date:   Mon Sep 12 20:23:52 2016 -0700

        BOX-155896: Enhance the JWT flow with additional validation checks

a) Style change
b) adds convenience method: decodeHeaderFromJWT

====================
commit 05029c3cfb733202113472f4447386bc0e0bcaf7
Author: Wayne Cheng <wayne@box.com>
Date:   Sun Jun 14 17:47:16 2015 -0400

        BOX-125973: Add support for "kid" in header for JWT tokens

---

Preparation work for supporting 'kid' in the header block.
This commit supports 'kid' (Key ID), if it is present in the header, but does not require it.
There seems to be no corresponding change in the vendor library


====================
commit 7d3082445d2482909e7eae7d82bec0cc021cac4d
Author: Wayne Cheng <wayne@box.com>
Date:   Thu Mar 26 10:11:00 2015 -0700

        BOX-121450: Create class for Box OAuth2 JWT Token and modify library to use it

        This is for Box Dev

adds encryption SHA384, SHA512
      Same as: commit 7f72b48d1fc07525b277ee83533b5e2305e3de14
      Author: Joost Faassen <j.faassen@linkorb.com>
      Date:   Mon Jun 19 18:29:59 2017 +0200



====================
2015-Apr-2: Upgraded library from vendor
Due to https://auth0.com/blog/2015/03/31/critical-vulnerabilities-in-json-web-token-libraries/

Upgrading JWT library to 2.2.0 (sic: should have read 2.0.0)
https://github.com/firebase/php-jwt/releases/tag/v2.0.0

commit 10bca7cae8a542eb6aa057bde745b4398b713a06
Author: Wayne Cheng <wayne@box.com>
Date:   Thu Apr 2 10:40:05 2015 -0700

        BOX-121829: Upgrade JWT library to protect against vulnerabilities


====================
2015-Mar-19: Upgraded library from vendor

commit e1514fca489243c952742396db31b90ccd5e6b60
Author: Wayne Cheng <wayne@box.com>
Date:   Thu Mar 19 10:11:19 2015 -0400

        BOX-120976: Upgrade existing JWT library

This does not match any vendor version number,
but does match (except for blank newline differences) this
vendor commit:

commit 8b6d4f07753dcd8036dc54ecb1b42296bd2d82f7 (HEAD)
Author: Brendan Abbott <brendan@vuid.com>
Date:   Mon Nov 17 11:29:36 2014 +1000

    Minor documentation update for SignatureInvalidException



====================
2013-Sept-3: Original import of module

commit 8736a442b9d0d0a0783d9cbf5063afb1e364d066
Author: Adrien Loison <aloison@box.com>
Date:   Tue Sep 3 16:53:24 2013 -0700

        BOX-74622: Zendesk remote auth deprecated

apparently from
https://github.com/firebase/php-jwt.git
at revision: 0d3b9be5ea490c05108458d69803f38d7fb3eaa7
No version number supplied by vendor (previous to v1.0.0)

Note: JWT.php isn't identical, as two comment lines mismatch,
but the lines almost certainly were changed by automated lint
removing unneeded spaces at the end of lines

HEAD in original repo:
commit 0d3b9be5ea490c05108458d69803f38d7fb3eaa7 (HEAD)
Author: Anant Narayanan <anant@kix.in>
Date:   Wed Apr 3 16:17:10 2013 -0700

    Fix warnings and errors reported by PHP_CodeSniffer


