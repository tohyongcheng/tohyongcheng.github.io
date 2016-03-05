---
layout: post
comments: true
title:  "Using ZBar for faster scanning of QR codes or Barcodes on Ionic"
date:   2016-03-05 11:00:00
categories: "ionic"
---

## Zbar and ZXing

In the realm of barcode and QR code scanning in mobile applications, there are generally 2 types or libraries - ZBar and ZXing. 

[ZXing](https://github.com/zxing/zxing):
  
> ZXing ("zebra crossing") is an open-source, multi-format 1D/2D barcode image processing library implemented in Java, with ports to other languages.

[ZBar](http://zbar.sourceforge.net/):
  
> ZBar is an open source software suite for reading bar codes from various sources, such as video streams, image files and raw intensity sensors. It supports many popular symbologies(types of bar codes) including EAN-13/UPC-A, UPC-E, EAN-8, Code 128, Code 39, Interleaved 2 of 5 and QR Code.

While building an Ionic application, I had a problem with using the `ng-cordova` provided BarCodeScanner library which mainly uses ZXing as its base library to scan for both Barcode and QR Codes. Using ZXing can be very slow on Ionic, and can affect the user experience if your application relies heavily on scanning barcodes. It becomes worse when you are running an old Android phone or iPhone... You have to accomodate for these edge cases.

I have found ZBar to detect QR Codes faster with decent accuracy and suitable for most use cases. Also, it performs even when the scanned code is rotated in any direction, or when the image lighting is affected by shadow and glare. However, the downside is that ZXing currently supports more formats of barcodes and QR Codes compared to ZBar (which was last updated since 2012). For me, the speed outweights that disadvantage, and I have always chosen ZBar over ZXing after I tried it.


## Using ZBar in Ionic

To use it when building Ionic applications is very simple. First download a plugin that `tjwoon` has created for using the ZBar SDK with Cordova at your Ionic application directory:

```
$ ionic plugins add org.cloudsky.cordovaplugins.zbar
```

This helps to install the plugin into `/plugins` and you can find more information of the plugin [here](https://github.com/tjwoon/csZBar).

To make it more Angular-ish, let us create a service that uses this plugin. Create a service module with the following code:

{% highlight javascript %}
  angular.module('myApp').factory("$zBarService", [
    '$q', function($q) {
      return {
        scan: function(params) {
          var q;
          q = $q.defer();
          cloudSky.zBar.scan(params, function(result) {
            return q.resolve(result);
          }, function(error) {
            return q.reject(error);
          });
          return q.promise;
        }
      };
    }
  ]);
{% endhighlight %}

We are making use of `$q` here, to help us run function asynchronously and use the return values when the plugin is done processing. Make sure to include the service module in `index.html` before trying to run the `scan` method! 

Finally, to use `$zBarService`, you can use `$zBarService` in your controller, and call it in the following way:

{% highlight javascript %}
  angular.module('myApp').controller('BarCodeCtrl', function($scope, $zBarService) {
    return $scope.scanBarCode = function() {
      return $zBarService.scan({
        text_title: "Scan Barcode",
        text_instructions: "Aim at the barcode to scan",
        camera: "back",
        flash: "off",
        drawSight: false
      }).then(function(barcodeData) {
        // barcodeData is the result string
      });
    };
  });
{% endhighlight %}

Now, you are able to use the ZBar library in your Ionic application! You can test with the `$cordovaBarcodeScanner` for ionic [here](http://ngcordova.com/docs/plugins/barcodeScanner/), and compare the speed in detecting and processing the barcodes/QR codes. After using ZBar, I have never gone back to ZXing. It's definitely worth a try!