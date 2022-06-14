Lab 2: Browsers, Automation Tools, and the ATI Dashboard
=======================================

1. Browser Devtools
-----------------------------------------------------------

In your browser switch to the JuiceShop tab.  Open the browser's "Developer Tools".

 .. figure:: ../_static/ati-browser-inspect.png

    In most browsers the quickest way to do this is to right-click anywhere on the webpage and select "Inspect" (or "Inspect Element") from the pop-up menu.

 .. figure:: ../_static/ati-devtools-dock.png
     
     It is easiest to view the request log in the network tab if the developer tools window is at the bottom of the browser window rather than on the side.
 
        
In the developer tools window, switch to the "Network" tab.
 
 .. image:: ../_static/ati-devtools-network.png

|

2. Inspecting Requests
-----------------------------------------------------------

With the developer tools (devtools) Network tab open, clear the request log (if any requests are visible), and then refresh the JuiceShop page.

Scroll to the top of the request log and select the first request. It should be a request to your JuiceShop \*.access.udf.f5.com domain.

 .. image:: ../_static/ati-devtools-juiceshop.png
|
Select this request and click on the "Response" tab.

 .. figure:: ../_static/ati-juiceshop-jstag.png
   
   Notice the script tag injected by the iApp, and the subsequent request to get that JavaScript.
|
Select the request for the javascript and look at the response.  This is the javascript returned by the F5XC ATI servers.  This javascript is generated "on-the-fly" and is unique for every user/session.

  .. image:: ../_static/ati-juiceshop-jsrequest.png
|
Scroll down in the request log and look for a request to "dip".  Select this request and then select the "Payload" tab.
 
 .. figure:: ../_static/ati-juiceshop-dip.png
   
   This is the XHR request initiated by the ATI JavaScript that reports the device ID and other telemetry to the F5XC ATI servers.
   
   The payload of this request contains encoded and encrypted data only accessible by the F5XC ATI servers.
|

In order to provide additional security for the Juiceshop website, it is only accessible to users who are authenticated to UDF.  Any requests that are sent to your JuiceShop UDF domain name will be blocked before they get to your UDF deployment BIGIP if they do not include your UDF session cookie.
We will need this cookie in order for automated requests to reach your BIGIP and show up in the ATI dashboard.
 
1.  In the browser devtools, select the "Application" tab.
2.  In the "Storage" section of the left-hand menu expand the "Cookies" section and click on the domain name listed.
3.  Look for a cookie named **udf.sid** and select it.
4.  From the "Cookie Value" pane select the entire cookie value.
5.  Copy the cookie value and paste it into a text editor for later use.

  .. image:: ../_static/ati-copy-cookie.png
|

3. Generating Interesting Traffic
-----------------------------------------------------------

Because the ATI dashboard is a high-level overview of your traffic, individual requests are not reflacted in the graphs and reports.  WE need to generate enough traffic for the graphs to populate and show data.

There are several ways that we can do this.  For a production website that is publicly accessible this will hapen automatically in a matter of minutes.  We can try to mimic that by browsing around on the Juiceshop app in our browser.

Browse around in the JuiceShop app for a few minutes.

Open a terminal and use curl to access your JuiceShop app.  You will need to include the udf.sid cookie in your curl request.

Be sure to replace ``<<your juice shop domain>>`` and ``<<your udf.sid cookie value>>`` with the actual values.

 ``curl 'https://<<your juice shop domain>>/' -H 'Cookie: udf.sid=<<your udf.sid cookie value>>'``

This will only send one request but, we need lots of requests.  Also, this will send the request with the default curl User-Agent string.

If you are on a Linux or Mac computer you can use the following script from the command line to send 300 requests:

 ``for i in `seq 300`; do curl '<<your juice shop domain>>/' -H 'Cookie: udf.sid=<<paste cookie value here>>' -H 'User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/100.0.4896.75 Safari/537.36'; done``