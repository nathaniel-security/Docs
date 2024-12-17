# Depixelating information in the document (finding what was not supposed to be found)

### In short

* while I was playing the box GreenHorn box on hack the box came across the problem where a PDF that contains a password was pixilated
  * https://www.hackthebox.com/achievement/machine/409699/617

<figure><img src="../.gitbook/assets/image (78).png" alt=""><figcaption></figcaption></figure>

* now when I looked deeper there were ways of Depixelating it
  * with
    * https://github.com/spipm/Depix
* the result&#x20;

<figure><img src="../.gitbook/assets/image (80).png" alt=""><figcaption></figcaption></figure>



* potential password

```
sidefromsidetheothersidesidefromsidetheotherside
```

### Diving deeper

* I will dive deeper into how it works some other day
* today will focus on how to Depixelate the image
* I extracted the image from the PDF (DO NOT TAKE A SCREENSHOT EXTRACT THE IMAGE FROM THE PDF)
* after which I passed it to Depix

```
python3 depix.py -p pix.png -s ./images/searchimages/debruinseq_notepad_Windows10_closeAndSpaced.png -o output.png
```

* the result from



<figure><img src="../.gitbook/assets/image (82).png" alt=""><figcaption></figcaption></figure>





* to

<figure><img src="../.gitbook/assets/image (81).png" alt=""><figcaption></figcaption></figure>

## References

* https://github.com/spipm/Depix
