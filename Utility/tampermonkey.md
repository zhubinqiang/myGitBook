# tampermonkey


```js
// ==UserScript==
// @name         myCSDN
// @namespace    http://tampermonkey.net/
// @version      0.1
// @description  try to take over the world!
// @author       You
// @match        http://blog.csdn.net/notify-Caution_Notification*
// @grant        none
// @require      https://cdn.bootcss.com/jquery/3.2.1/jquery.min.js
// ==/UserScript==

(function() {
    'use strict';
    $().ready(function(){
        /*
        var a = $("a")[1];
        a.click(function(event) {
            Accept();
        });
        */


        var cookie = 'notified-Caution_Notification=1';
        var expires = new Date();
        expires.setTime(expires.getTime() + 1000 * 60 * 60);
        var domain = ';Domain=.csdn.net';
        document.cookie = cookie+';expires='+expires.toUTCString()+domain;
        if (document.cookie.indexOf(cookie) == -1) {
            document.cookie = cookie+';expires='+expires.toUTCString();
        }
        window.location.href = "http://blog.csdn.net/accepted-Caution_Notification?aHR0cDovL2Jsb2cuY3Nkbi5uZXQva215aHkvYXJ0aWNsZS9kZXRhaWxzLzQyMDA1NjM=;kdH8iGq8Ch42Uw+ic+wJM1UGnuJER+Mnz+tErP22rOw=";
        /* */
    });
})();
```
