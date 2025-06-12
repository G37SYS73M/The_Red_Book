# Cobalt Strike Site Clone

The techniques described above typically require the red team to build a legitimate looking website on a standalone web server (e.g. running Apache or Nginx) and manually backdoor one or more pages with the smuggling code.  Cobalt Strike has it's own drive-by capabilities, although it doesn't use smuggling.  It can clone a legitimate website and then embed a URL that the target's browser will automatically download without needing a user to click anything on the page.

In this example, I'm going to clone [https://www.bleepingcomputer.com/download/gpu-z/](https://www.bleepingcomputer.com/download/gpu-z/) which looks something like this:

<figure><img src="https://lwfiles.mycourse.app/66e95234fe489daea7060790-public/13ebecb18479bd3c817f223a1dbf810d.png" alt="" width="100%"><figcaption></figcaption></figure>

We want a user to visit a page that looks like this, but also have it automatically download our initial access package to their computer.
