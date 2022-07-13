# Parameters

**Writeup by:** nitrowv   
**Category:** Log Analysis  
**Points:** 100

This challenge gives us an Apache log with HTTP requests:

```
**.***.***.** - lemke1570 [18/May/2022:20:05:45 -0400] "PUT /viral/markets HTTP/2.0" 416 12504
**.**.***.*** - - [18/May/2022:20:05:45 -0400] "PUT /partnerships/niches HTTP/2.0" 304 1348
***.***.**.** - - [18/May/2022:20:05:45 -0400] "POST /rich/bsftw{9X95ySYz0OYP9fbbrXiZkKEnUGawOGmV1Rop1L7mjhubY yJE6ZTapWDUCk5hLqhu8np7qOkb2Odq4 bR2S7kVJrHzA} HTTP/1.1" 205 6820
```

This challenge is similar to what we saw in Obscure Memo, but since this log has many more than three instances of the string `bsftw`, using regular expressions would be a bit more convienent. All flags consist of bsftw{, 32 characters, and a }. While not the regex provided, we can use `/bsftw{.{32}}/g` to get us closer to what we're looking for.

Using regexr.com to test, we see that there are two strings that match our regular expression:

```
bsftw{lSklv6GfDmjeDCWCm Hn1ju2fr3s mQ1}
bsftw{Regex_Is_The_Key_To_Text_Parsing}
```

That regular expression worked! We can see the correct flag of `bsftw{Regex_Is_The_Key_To_Text_Parsing}`.