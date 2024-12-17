# Depixelating information in the document (finding what was not supposed to be found)

### In short

* While playing the GreenHorn box on Hack The Box, I encountered a situation where a PDF contained a pixelated password
  * https://www.hackthebox.com/achievement/machine/409699/617

<figure><img src="../.gitbook/assets/image (78).png" alt=""><figcaption></figcaption></figure>

* Upon deeper investigation, I found ways to depixelate the image
  * with
    * https://github.com/spipm/Depix
* the result&#x20;

<figure><img src="../.gitbook/assets/image (80).png" alt=""><figcaption></figcaption></figure>



* potential password

```
sidefromsidetheothersidesidefromsidetheotherside
```



### Diving deeper

* I will dive deeper into how it works another day
* Today, I'll focus on how to depixelate the image
* I extracted the image from the PDF. _(Note: Do not take a screenshot! Always extract the image directly from the PDF to preserve quality.)_
  * I initially made the mistake of thinking it _would be fine_ to use a screenshot.
  * As a result, Depix was unable to depixelate the image
* After I passed it to Depix

```
python3 depix.py -p pix.png -s ./images/searchimages/debruinseq_notepad_Windows10_closeAndSpaced.png -o output.png
```

* the result from



<figure><img src="../.gitbook/assets/image (82).png" alt=""><figcaption></figcaption></figure>



* to

<figure><img src="../.gitbook/assets/image (81).png" alt=""><figcaption></figcaption></figure>

## References

* https://github.com/spipm/Depix
