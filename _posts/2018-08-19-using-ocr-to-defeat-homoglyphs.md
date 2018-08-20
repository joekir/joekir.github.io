---
layout: post
title: Using optical character recognition (OCR) to defeat Homoglyph attacks 
comments: true
type: post
published: true
status: publish
---

_this isn't a super-baked idea, just one that I thought of at Defcon while playing linecon_

I've written about [Homoglyph attacks](https://en.wikipedia.org/wiki/Homoglyph) before, mostly as it's a simple trick that really seems like it should have been eradicated by now, related writings:
- [https://www.josephkirwin.com/2FA-Demand-Bidirectional](https://www.josephkirwin.com/2FA-Demand-Bidirectional)
- [https://www.josephkirwin.com/2016/12/03/introducing-elfin](https://www.josephkirwin.com/2016/12/03/introducing-elfin)

Ok so what I was thinking, the whole issue is humans mistaking one character or collection of characters for another and thereby being phished, pwned etc. Therefore why not tune an OCR engine to get "fooled" like that and check the mismatch against the true input?

The flow:
1. string comes in e.g. grnail.com
2. render that to a HTML canvas element in a font that can often lead to misinterpretation, e.g Arial and make it pretty small
3. use [tesseract](https://opensource.google.com/projects/tesseract) to OCR ([Optical Character Recognition](https://en.wikipedia.org/wiki/Optical_character_recognition)) that string back 
4. compare the input string to the OCR'd string
5. if there's a mismatch, tell the user

I think this could be cool as a chrome extension, or something in email flow checks on a backend.
 
============== Below is an example you can play with =================

**This seems to crash mobile browsers, at least firefox, chrome and brave on my android, so I'd recommend running on a laptop, presumably tesseract is memory hungry?**

_try out "grnail.com" and "josephkirwin.com" as examples of it in action, click view-source to see what it do_

<script src="https://cdnjs.cloudflare.com/ajax/libs/tesseract.js/1.0.10/tesseract.min.js" integrity="sha256-Yg4eYEUSthKzFM/fE68KncKWRSzIRHjTYxGIYdbW90s=" crossorigin="anonymous"></script>

Hostname: <input value="grnail.com" id="hostname" type="text" name="hostname"/>
<button onclick="checkDomain()">Click to test</button><br/>
OCR Result:<input readonly id="ocrResult"/><br/>
Did they match?: <input readonly id="isMatch"/><br/><br/>
<p id="working" style="display:none;">Processing...</p>
<b>canvas</b>
<canvas id="myCanvas" style="border:1px solid #000000;"></canvas>
<script type="text/javascript">
    function checkDomain(){
        var name = document.getElementById("hostname").value;
        var canvas = document.getElementById("myCanvas");
        var ctx = canvas.getContext("2d");
        ctx.clearRect(0, 0, canvas.width, canvas.height);

        document.getElementById("ocrResult").value = "";
        var isMatch = document.getElementById("isMatch");
        isMatch.value = "";
        
        ctx.font = "18px Arial";
        ctx.letterSpacing = "-1px";
        ctx.fillText(name,50,50);

        var working = document.getElementById('working');
        working.style.display="block";

        Tesseract.recognize(canvas,{
            lang: 'eng'
        }).then(function(result){
            working.style.display="none";
            var parsed = result.text.replace(/[\r\n]/g, '').replace(/[\s]/g,'.');
            document.getElementById("ocrResult").value = parsed;

            if (parsed === name){
                isMatch.value = "✅";
            } else {
                isMatch.value = "❌";
            }
        }); 
    };
</script>
