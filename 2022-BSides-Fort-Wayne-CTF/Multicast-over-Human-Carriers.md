# Multicast over Human Carriers (MoHC)

**Writeup by:** nitrowv   
**Category:** Miscellaneous  
**Points:** 200

This challenge was a nice change of pace, since in order to get the flag, competitors had to speak with the vendors that had a table at the event. Each vendor had a fragment of the flag and we had to combine all ten of them into the correct flag. The fragments are as follows:

```
rkus
psbro
esno
asoc
twor
sesn
cial
lnet
otips
name
```

It took quite a while to fit the pieces together in the right order, but there were some parts that were easily decipherable. I was able to gather `Asocialnetworkuses` relatively quickly, but I was initially stumped on the order of the last bit, specifically over the placement of names and notips. Eventually, I got it in the right order and the correct flag is: `bsftw{ASocialNetworkUsesNamesNotIpsBro}`.
