
**Botflow**  is programming framework  built for createing fast data pipeline. 
This tutorial is a sample for analyse  Python book price distrubution with amazon book search result . And the tutorial dose not require extra tools and make you leave this page.

more information can be found in project home page:

    https://github.com/kkyon/botflow


The package you need to install 

```pip3 instll -U botflow```

update tonado to 4.5.3 because  jupyter asyncio support issue.


```pip3 install tornado==4.5.3```

and you should have baic knowlege with html and beautiful soup.


## All Steps  
1. Donwloads the serach result page (html) and get the book item html
2. parse the price ,book title,images src
1. filter out spam item
3. create pandas dataframe from the prices .And draw the hist 


### Step 1. Create download pipe.

Make amazon book search with keyword *Python* in your chrome. You can get the search result page url :

```https://www.amazon.com/s/ref=sr_pg_2?fst=p90x%3A1&rh=n%3A283155%2Ck%3Apython&page=2&d=1&keywords=python&ie=UTF8&qid=1536905092```

We can fingure the page no and keyword word in the url . In this tutorial we only replace page no  with variable . You can try change the search keywords later.
```python 

lambda p:f"https://www.amazon.com/s/ref=sr_pg_{p}?fst=p90x%3A1&page={p}&rh=n%3A283155%2Ck%3Apython&keywords=python&ie=UTF8&qid=1536500367",

```

And with inspect the html ,we can find book item is under *li* tage . get all tag with soup api .*find_all*. 
It also return many unnecceary items .Don't worry about this .will clean it later.

Let's get start with only one page. We can easy fetch more page just change run param.


```python
from botflow import *
p_download=Pipe(
    
    lambda p:f"https://www.amazon.com/s/ref=sr_pg_{p}?fst=p90x%3A1&page={p}&rh=n%3A283155%2Ck%3Apython&keywords=python&ie=UTF8&qid=1536500367",
    HttpLoader(), #download the page

    lambda r:r.soup.find_all("li"),
    Flat()  #we don't need keep the hierarchy of page no. 
)

lis=p_download.run(1) #get first page
#lis=p.run(range(1,10)) get top 10 pages items
print(len(lis))


```

    2018-09-14 14:59:08,459 root         INFO:pipe_4470375312 ready to exit


    175


We got many items now. let's  pick one and see inside. It is actually a Html Tag . html code seem messy .How can first price infrom from it?


```python
lis[1]
```




    <li class="s-result-item celwidget AdHolder" data-asin="1788295269" id="result_0"><div class="s-item-container"><div class="a-fixed-left-grid"><div class="a-fixed-left-grid-inner" style="padding-left:218px"><div class="a-fixed-left-grid-col a-col-left" style="width:218px;margin-left:-218px;float:left;"><div class="a-row"><div aria-hidden="true" class="a-column a-span12 a-text-center"><a class="a-link-normal a-text-normal" href="/gp/slredirect/picassoRedirect.html/ref=pa_sp_atf_stripbooks_sr_pg1_1?ie=UTF8&amp;adId=A04794103EZM52VSV07ZR&amp;url=https%3A%2F%2Fwww.amazon.com%2FMatplotlib-2-x-Example-Multi-dimensional-charts%2Fdp%2F1788295269%2Fref%3Dsr_1_1_sspa%2F136-9982183-6544141%3Fs%3Dbooks%26ie%3DUTF8%26qid%3D1536908344%26sr%3D1-1-spons%26keywords%3Dpython%26psc%3D1&amp;qualifier=1536908344&amp;id=88207326666658&amp;widgetName=sp_atf"><img alt="Matplotlib 2.x By Example: Multi-dimensional charts, graphs, and plots in Python" class="s-access-image cfMarker" data-search-image-load="" height="218" src="https://images-na.ssl-images-amazon.com/images/I/410gNSSY-+L._AC_US218_.jpg" srcset="https://images-na.ssl-images-amazon.com/images/I/410gNSSY-+L._AC_US218_.jpg 1x, https://images-na.ssl-images-amazon.com/images/I/410gNSSY-+L._AC_US327_FMwebp_QL65_.jpg 1.5x, https://images-na.ssl-images-amazon.com/images/I/410gNSSY-+L._AC_US436_FMwebp_QL65_.jpg 2x, https://images-na.ssl-images-amazon.com/images/I/410gNSSY-+L._AC_US500_FMwebp_QL65_.jpg 2.2935x" width="218"/></a><div class="a-section a-spacing-none a-text-center"></div></div></div></div><div class="a-fixed-left-grid-col a-col-right" style="padding-left:2%;float:left;"><h5 class="a-spacing-none a-color-tertiary s-sponsored-list-header s-sponsored-header sp-pixel-data a-text-normal" data-alt-pixel-url="/gp/sponsored-products/logging/log-action.html?qualifier=1536908344&amp;id=88207326666658&amp;widgetName=sp_atf&amp;adId=200004567537671&amp;eventType=2&amp;adIndex=0" data-page-layout="List" data-page-num-columns="1" data-page-number="1" data-pixel-url="https://fls-na.amazon.com/1/amazon-clicks/1/OP/?requestId=CQ95D0VAKKAKEPRYMC9R&amp;childRequestId=CQ95D0VAKKAKEPRYMC9R&amp;widgetName=sp_atf&amp;searchResultNumber=1&amp;impressionRankOnAsin=1&amp;qualifier=1536908344&amp;id=88207326666658" data-view-pixel-url="/gp/sponsored-products/logging/log-action.html?qualifier=1536908344&amp;id=88207326666658&amp;widgetName=sp_atf&amp;adId=200004567537671&amp;adIndex=0" data-widget="sp_atf">Sponsored<span class="a-declarative" data-a-popover='{"name":"sponsored-header-1788295269","position":"triggerRight"}' data-action="a-popover"><span class="s-sponsored-info-icon a-inline-block s-sponsored-info-icon-base64"></span>
    </span><script type="text/javascript">
             P.when('A','SponsoredProductsRendering').execute('force-impression-logging-clicktime', function sponsoredRendering (A, SL) {
               var $ = A.$;
               var data_asin = 'asin';
               var data_view_pixel_url = 'view-pixel-url';
               var selector_header = '[data-view-pixel-url="'+ "/gp/sponsored-products/logging/log-action.html?qualifier=1536908344&id=88207326666658&widgetName=sp_atf&adId=200004567537671&adIndex=0"+'"]';
               var selectors_asinElement = '[data-asin]';
               var picassoHrefSelector = '[href*=\'picassoRedirect.html\']';
    
               var header = $(selector_header);
               var asinElement = header.closest(selectors_asinElement);
    
               if (!asinElement || asinElement.data(data_asin) === undefined) {
                   // Fallback logic if closest function is not supported
                   A.$(selectors_asinElement).each(function(){
                    var asin = $(this);
                    var header = asin.find(selector_header);
                    
                    if (header.data(data_view_pixel_url) !== undefined) {
                       asinElement = asin;
                    }
                   });
               }
    
               var $spAsin = asinElement.first();
    
               if (asinElement.data(data_asin) !== undefined) {
                   SL.logMessage("Checking impression pixel for asin : " + asinElement.data(data_asin) + ", with pixel url : " + "/gp/sponsored-products/logging/log-action.html?qualifier=1536908344&id=88207326666658&widgetName=sp_atf&adId=200004567537671&eventType=2&adIndex=0");
                   // Find all the jQuery objects which contains "picassoredirct.html" in the href and bind them with onclick.
                   $spAsin.find(picassoHrefSelector)
                     .click(function() {
                       SL.forceFireRenderPixel("/gp/sponsored-products/logging/log-action.html?qualifier=1536908344&id=88207326666658&widgetName=sp_atf&adId=200004567537671&eventType=2&adIndex=0");
                     })
                     .mousedown(function(e) {
                       switch (e.which) {
                         // 2 is middle click, 1 is left click which is covered in the above click
                         case 2:
                           SL.forceFireRenderPixel("/gp/sponsored-products/logging/log-action.html?qualifier=1536908344&id=88207326666658&widgetName=sp_atf&adId=200004567537671&eventType=2&adIndex=0");
                           break;
                         // 3 is right click which we do not handle the case here,
                         default:
                           break;
                       }
                   });
                }
             });
          </script>
    <script type="text/javascript">
           P.when('A','SponsoredProductsViewability').execute('sponsored-products-track-views-t6', function sponsoredViewability (A, SV) {
             var $ = A.$;
             var data_asin = 'asin';
             var data_view_pixel_url = 'view-pixel-url';
             var selector_header = '[data-view-pixel-url="'+ "/gp/sponsored-products/logging/log-action.html?qualifier=1536908344&id=88207326666658&widgetName=sp_atf&adId=200004567537671&adIndex=0"+'"]';
             var selectors_asinElement = '[data-asin]';
    
             var header = $(selector_header);
             var asinElement = header.closest(selectors_asinElement);
    
             if (!asinElement || asinElement.data(data_asin) === undefined) {
                 // Fallback logic if closest function is not supported
                 A.$(selectors_asinElement).each(function(){
                  var asin = $(this);
                  var header = asin.find(selector_header);
                  
                  if (header.data(data_view_pixel_url) !== undefined) {
                     asinElement = asin;
                  }
                 });
             } 
    
             if (asinElement.data(data_asin) !== undefined) {
                 SV.logMessage("Registering asin: " + asinElement.data(data_asin));
                 var pageNumber = ("1" === "1") ? "P1":"P2+";
                 var subPlacement = "sp_atf" + '-' + pageNumber + 
                                 '-' +"List" + '-' +"1";
    
                 SV.registerViewTrackingElement(asinElement, "search-pc",
                                            "/gp/sponsored-products/logging/log-action.html?qualifier=1536908344&id=88207326666658&widgetName=sp_atf&adId=200004567537671&adIndex=0", subPlacement);
             }
           });
         </script>
    <noscript>
    <img alt="" src="https://fls-na.amazon.com/1/amazon-clicks/1/OP/?requestId=CQ95D0VAKKAKEPRYMC9R&amp;childRequestId=CQ95D0VAKKAKEPRYMC9R&amp;widgetName=sp_atf&amp;searchResultNumber=1&amp;impressionRankOnAsin=1&amp;qualifier=1536908344&amp;id=88207326666658"/></noscript>
    <noscript>
    <img alt="" src="/gp/sponsored-products/logging/log-action.html?qualifier=1536908344&amp;id=88207326666658&amp;widgetName=sp_atf&amp;adId=200004567537671&amp;eventType=2&amp;adIndex=0"/></noscript>
    <noscript>
    <img alt="" src="/gp/sponsored-products/logging/log-action.html?qualifier=1536908344&amp;id=88207326666658&amp;widgetName=sp_atf&amp;adId=200004567537671&amp;adIndex=0&amp;eventType=101"/></noscript>
    </h5><div class="a-popover-preload" id="a-popover-sponsored-header-1788295269"><span>These are ads for products you'll find on Amazon.com. <br/>Clicking an ad will take you to the product's page.</span><span class="a-letter-space"></span><a class="a-link-normal" href="https://advertising.amazon.com/products-self-serve?ref_=ext_amzn_wtsp" rel="noopener" target="_blank">Learn more about Sponsored Products.</a><div class="a-row a-spacing-top-small s-sponsored-feedback">See a problem with these advertisements?<span class="a-letter-space"></span><span class="a-declarative" data-a-modal="{&quot;name&quot;:&quot;sponsored-products-feedback-modal-sp_atf&quot;,&quot;width&quot;:&quot;350&quot;,&quot;header&quot;:&quot;Share your feedback&quot;,&quot;footer&quot;:&quot;&lt;div id='sponsored-products-feedback-footer'&gt;&lt;\/div&gt;&quot;,&quot;url&quot;:&quot;/gp/sponsored-products/lazyLoad/handler/sp-feedback-handler.html?pl=D48z4zAieFWWQuIpntKE6KqCE38Hx7YE92S2N44PABQbYU5RtcoDrrR0teHx6ew0r1hQHIcQXG6j%0AIMdW2WcGUivcBNrWaEKvU6mlVCdRhS%2Bo629Dcqw4FYSrxBFyGYmvm%2FetTUMfLdjxViEuzXrOzl2X%0App%2F5mW066mI0RT%2Fp%2FJ00OJTt1DbI4PSOJbwJD8KzLwiIL624Q7KziREa1YPKmmTgWC4cPkydKERk%0AcW5e64o5AkT0aQHgkJD1rKVAYVd7ZQLZg7xMyUAe%2FiJqQPuQwfWqR6FKyY5wN5koBYpllXgYdn3g%0A39f6IM1SmmrWmtcv5yQmO3YZ3Xu0cZL%2BIOWMEGVKBSDOKPKPzjFM23KUGS50pfgLirdhMnV01GiS%0A6LbU8VBhoo4FC5ZM9dYSAJaPKYCxArvx73q2qHtsb9Gnv63XGHJnJOfIgr2r2CMo7wkQXphndzHT%0AF%2BZzWyA3BzgXubkjMzSrHNGgRLEaNJ1DCjO0nPumbJTICQ%3D%3D&quot;}" data-action="a-modal"><a class="a-link-normal" href="#">Leave ad feedback</a></span></div></div><div class="a-row a-spacing-small"><div class="a-row a-spacing-none"><a class="a-link-normal s-access-detail-page s-color-twister-title-link a-text-normal" href="/gp/slredirect/picassoRedirect.html/ref=pa_sp_atf_stripbooks_sr_pg1_1?ie=UTF8&amp;adId=A04794103EZM52VSV07ZR&amp;url=https%3A%2F%2Fwww.amazon.com%2FMatplotlib-2-x-Example-Multi-dimensional-charts%2Fdp%2F1788295269%2Fref%3Dsr_1_1_sspa%2F136-9982183-6544141%3Fs%3Dbooks%26ie%3DUTF8%26qid%3D1536908344%26sr%3D1-1-spons%26keywords%3Dpython%26psc%3D1&amp;qualifier=1536908344&amp;id=88207326666658&amp;widgetName=sp_atf" title="Matplotlib 2.x By Example: Multi-dimensional charts, graphs, and plots in Python"><h2 class="a-size-medium s-inline s-access-title a-text-normal" data-attribute="Matplotlib 2.x By Example: Multi-dimensional charts, graphs, and plots in Python" data-max-rows="0"><span class="a-offscreen">[Sponsored]</span>Matplotlib 2.x By Example: Multi-dimensional charts, graphs, and plots in Python</h2></a><span class="a-letter-space"></span><span class="a-letter-space"></span><span class="a-size-small a-color-secondary">Aug 28, 2017</span></div><div class="a-row a-spacing-none"><span class="a-size-small a-color-secondary">by </span><span class="a-size-small a-color-secondary">Allen Yu and </span><span class="a-size-small a-color-secondary">Claire Chung</span></div></div><div class="a-row a-spacing-none"><span class="a-size-small">Eligible for Shipping to China</span></div><div class="a-row"><div class="a-column a-span7"><div class="a-row a-spacing-none"><a class="a-link-normal a-text-normal" href="/gp/slredirect/picassoRedirect.html/ref=pa_sp_atf_stripbooks_sr_pg1_1?ie=UTF8&amp;adId=A04794103EZM52VSV07ZR&amp;url=https%3A%2F%2Fwww.amazon.com%2FMatplotlib-2-x-Example-Multi-dimensional-charts%2Fdp%2F1788295269%2Fref%3Dsr_1_1_twi_pap_1%2F136-9982183-6544141%3Fs%3Dbooks%26ie%3DUTF8%26qid%3D1536908344%26sr%3D1-1-spons%26keywords%3Dpython%26psc%3D1&amp;qualifier=1536908344&amp;id=88207326666658&amp;widgetName=sp_atf" title="Paperback"><h3 class="a-size-small s-inline a-text-normal" data-attribute="Paperback" data-max-rows="0" data-truncate-by-character="false">Paperback</h3></a></div><div class="a-row a-spacing-none"><a class="a-link-normal a-text-normal" href="/gp/slredirect/picassoRedirect.html/ref=pa_sp_atf_stripbooks_sr_pg1_1?ie=UTF8&amp;adId=A04794103EZM52VSV07ZR&amp;url=https%3A%2F%2Fwww.amazon.com%2FMatplotlib-2-x-Example-Multi-dimensional-charts%2Fdp%2F1788295269%2Fref%3Dsr_1_1_sspa%2F136-9982183-6544141%3Fs%3Dbooks%26ie%3DUTF8%26qid%3D1536908344%26sr%3D1-1-spons%26keywords%3Dpython%26psc%3D1&amp;qualifier=1536908344&amp;id=88207326666658&amp;widgetName=sp_atf"><span class="a-offscreen">$44.99</span><span aria-hidden="true" class="a-color-base sx-zero-spacing"><span class="sx-price sx-price-large">
    <sup class="sx-price-currency">$</sup>
    <span class="sx-price-whole">44</span>
    <sup class="sx-price-fractional">99</sup>
    </span>
    </span></a></div><div class="a-row a-spacing-none"><span class="a-size-small a-color-secondary">Available to ship in 1-2 days</span></div></div><div class="a-column a-span5 a-span-last"></div></div></div></div></div></div></li>



## Step2: Parse book item (price,title,image)

Ok .Let's us  play magic with jupyter book ,print out the html visually .and will inspect the code whitout leave this page. 


```python
from IPython.core.display import display, HTML
display(HTML(str(lis[1])))
```


<li class="s-result-item celwidget AdHolder" data-asin="1788295269" id="result_0"><div class="s-item-container"><div class="a-fixed-left-grid"><div class="a-fixed-left-grid-inner" style="padding-left:218px"><div class="a-fixed-left-grid-col a-col-left" style="width:218px;margin-left:-218px;float:left;"><div class="a-row"><div aria-hidden="true" class="a-column a-span12 a-text-center"><a class="a-link-normal a-text-normal" href="/gp/slredirect/picassoRedirect.html/ref=pa_sp_atf_stripbooks_sr_pg1_1?ie=UTF8&amp;adId=A04794103EZM52VSV07ZR&amp;url=https%3A%2F%2Fwww.amazon.com%2FMatplotlib-2-x-Example-Multi-dimensional-charts%2Fdp%2F1788295269%2Fref%3Dsr_1_1_sspa%2F136-9982183-6544141%3Fs%3Dbooks%26ie%3DUTF8%26qid%3D1536908344%26sr%3D1-1-spons%26keywords%3Dpython%26psc%3D1&amp;qualifier=1536908344&amp;id=88207326666658&amp;widgetName=sp_atf"><img alt="Matplotlib 2.x By Example: Multi-dimensional charts, graphs, and plots in Python" class="s-access-image cfMarker" data-search-image-load="" height="218" src="https://images-na.ssl-images-amazon.com/images/I/410gNSSY-+L._AC_US218_.jpg" srcset="https://images-na.ssl-images-amazon.com/images/I/410gNSSY-+L._AC_US218_.jpg 1x, https://images-na.ssl-images-amazon.com/images/I/410gNSSY-+L._AC_US327_FMwebp_QL65_.jpg 1.5x, https://images-na.ssl-images-amazon.com/images/I/410gNSSY-+L._AC_US436_FMwebp_QL65_.jpg 2x, https://images-na.ssl-images-amazon.com/images/I/410gNSSY-+L._AC_US500_FMwebp_QL65_.jpg 2.2935x" width="218"/></a><div class="a-section a-spacing-none a-text-center"></div></div></div></div><div class="a-fixed-left-grid-col a-col-right" style="padding-left:2%;float:left;"><h5 class="a-spacing-none a-color-tertiary s-sponsored-list-header s-sponsored-header sp-pixel-data a-text-normal" data-alt-pixel-url="/gp/sponsored-products/logging/log-action.html?qualifier=1536908344&amp;id=88207326666658&amp;widgetName=sp_atf&amp;adId=200004567537671&amp;eventType=2&amp;adIndex=0" data-page-layout="List" data-page-num-columns="1" data-page-number="1" data-pixel-url="https://fls-na.amazon.com/1/amazon-clicks/1/OP/?requestId=CQ95D0VAKKAKEPRYMC9R&amp;childRequestId=CQ95D0VAKKAKEPRYMC9R&amp;widgetName=sp_atf&amp;searchResultNumber=1&amp;impressionRankOnAsin=1&amp;qualifier=1536908344&amp;id=88207326666658" data-view-pixel-url="/gp/sponsored-products/logging/log-action.html?qualifier=1536908344&amp;id=88207326666658&amp;widgetName=sp_atf&amp;adId=200004567537671&amp;adIndex=0" data-widget="sp_atf">Sponsored<span class="a-declarative" data-a-popover='{"name":"sponsored-header-1788295269","position":"triggerRight"}' data-action="a-popover"><span class="s-sponsored-info-icon a-inline-block s-sponsored-info-icon-base64"></span>
</span><script type="text/javascript">
         P.when('A','SponsoredProductsRendering').execute('force-impression-logging-clicktime', function sponsoredRendering (A, SL) {
           var $ = A.$;
           var data_asin = 'asin';
           var data_view_pixel_url = 'view-pixel-url';
           var selector_header = '[data-view-pixel-url="'+ "/gp/sponsored-products/logging/log-action.html?qualifier=1536908344&id=88207326666658&widgetName=sp_atf&adId=200004567537671&adIndex=0"+'"]';
           var selectors_asinElement = '[data-asin]';
           var picassoHrefSelector = '[href*=\'picassoRedirect.html\']';

           var header = $(selector_header);
           var asinElement = header.closest(selectors_asinElement);

           if (!asinElement || asinElement.data(data_asin) === undefined) {
               // Fallback logic if closest function is not supported
               A.$(selectors_asinElement).each(function(){
                var asin = $(this);
                var header = asin.find(selector_header);
                
                if (header.data(data_view_pixel_url) !== undefined) {
                   asinElement = asin;
                }
               });
           }

           var $spAsin = asinElement.first();

           if (asinElement.data(data_asin) !== undefined) {
               SL.logMessage("Checking impression pixel for asin : " + asinElement.data(data_asin) + ", with pixel url : " + "/gp/sponsored-products/logging/log-action.html?qualifier=1536908344&id=88207326666658&widgetName=sp_atf&adId=200004567537671&eventType=2&adIndex=0");
               // Find all the jQuery objects which contains "picassoredirct.html" in the href and bind them with onclick.
               $spAsin.find(picassoHrefSelector)
                 .click(function() {
                   SL.forceFireRenderPixel("/gp/sponsored-products/logging/log-action.html?qualifier=1536908344&id=88207326666658&widgetName=sp_atf&adId=200004567537671&eventType=2&adIndex=0");
                 })
                 .mousedown(function(e) {
                   switch (e.which) {
                     // 2 is middle click, 1 is left click which is covered in the above click
                     case 2:
                       SL.forceFireRenderPixel("/gp/sponsored-products/logging/log-action.html?qualifier=1536908344&id=88207326666658&widgetName=sp_atf&adId=200004567537671&eventType=2&adIndex=0");
                       break;
                     // 3 is right click which we do not handle the case here,
                     default:
                       break;
                   }
               });
            }
         });
      </script>
<script type="text/javascript">
       P.when('A','SponsoredProductsViewability').execute('sponsored-products-track-views-t6', function sponsoredViewability (A, SV) {
         var $ = A.$;
         var data_asin = 'asin';
         var data_view_pixel_url = 'view-pixel-url';
         var selector_header = '[data-view-pixel-url="'+ "/gp/sponsored-products/logging/log-action.html?qualifier=1536908344&id=88207326666658&widgetName=sp_atf&adId=200004567537671&adIndex=0"+'"]';
         var selectors_asinElement = '[data-asin]';

         var header = $(selector_header);
         var asinElement = header.closest(selectors_asinElement);

         if (!asinElement || asinElement.data(data_asin) === undefined) {
             // Fallback logic if closest function is not supported
             A.$(selectors_asinElement).each(function(){
              var asin = $(this);
              var header = asin.find(selector_header);
              
              if (header.data(data_view_pixel_url) !== undefined) {
                 asinElement = asin;
              }
             });
         } 

         if (asinElement.data(data_asin) !== undefined) {
             SV.logMessage("Registering asin: " + asinElement.data(data_asin));
             var pageNumber = ("1" === "1") ? "P1":"P2+";
             var subPlacement = "sp_atf" + '-' + pageNumber + 
                             '-' +"List" + '-' +"1";

             SV.registerViewTrackingElement(asinElement, "search-pc",
                                        "/gp/sponsored-products/logging/log-action.html?qualifier=1536908344&id=88207326666658&widgetName=sp_atf&adId=200004567537671&adIndex=0", subPlacement);
         }
       });
     </script>
<noscript>
<img alt="" src="https://fls-na.amazon.com/1/amazon-clicks/1/OP/?requestId=CQ95D0VAKKAKEPRYMC9R&amp;childRequestId=CQ95D0VAKKAKEPRYMC9R&amp;widgetName=sp_atf&amp;searchResultNumber=1&amp;impressionRankOnAsin=1&amp;qualifier=1536908344&amp;id=88207326666658"/></noscript>
<noscript>
<img alt="" src="/gp/sponsored-products/logging/log-action.html?qualifier=1536908344&amp;id=88207326666658&amp;widgetName=sp_atf&amp;adId=200004567537671&amp;eventType=2&amp;adIndex=0"/></noscript>
<noscript>
<img alt="" src="/gp/sponsored-products/logging/log-action.html?qualifier=1536908344&amp;id=88207326666658&amp;widgetName=sp_atf&amp;adId=200004567537671&amp;adIndex=0&amp;eventType=101"/></noscript>
</h5><div class="a-popover-preload" id="a-popover-sponsored-header-1788295269"><span>These are ads for products you'll find on Amazon.com. <br/>Clicking an ad will take you to the product's page.</span><span class="a-letter-space"></span><a class="a-link-normal" href="https://advertising.amazon.com/products-self-serve?ref_=ext_amzn_wtsp" rel="noopener" target="_blank">Learn more about Sponsored Products.</a><div class="a-row a-spacing-top-small s-sponsored-feedback">See a problem with these advertisements?<span class="a-letter-space"></span><span class="a-declarative" data-a-modal="{&quot;name&quot;:&quot;sponsored-products-feedback-modal-sp_atf&quot;,&quot;width&quot;:&quot;350&quot;,&quot;header&quot;:&quot;Share your feedback&quot;,&quot;footer&quot;:&quot;&lt;div id='sponsored-products-feedback-footer'&gt;&lt;\/div&gt;&quot;,&quot;url&quot;:&quot;/gp/sponsored-products/lazyLoad/handler/sp-feedback-handler.html?pl=D48z4zAieFWWQuIpntKE6KqCE38Hx7YE92S2N44PABQbYU5RtcoDrrR0teHx6ew0r1hQHIcQXG6j%0AIMdW2WcGUivcBNrWaEKvU6mlVCdRhS%2Bo629Dcqw4FYSrxBFyGYmvm%2FetTUMfLdjxViEuzXrOzl2X%0App%2F5mW066mI0RT%2Fp%2FJ00OJTt1DbI4PSOJbwJD8KzLwiIL624Q7KziREa1YPKmmTgWC4cPkydKERk%0AcW5e64o5AkT0aQHgkJD1rKVAYVd7ZQLZg7xMyUAe%2FiJqQPuQwfWqR6FKyY5wN5koBYpllXgYdn3g%0A39f6IM1SmmrWmtcv5yQmO3YZ3Xu0cZL%2BIOWMEGVKBSDOKPKPzjFM23KUGS50pfgLirdhMnV01GiS%0A6LbU8VBhoo4FC5ZM9dYSAJaPKYCxArvx73q2qHtsb9Gnv63XGHJnJOfIgr2r2CMo7wkQXphndzHT%0AF%2BZzWyA3BzgXubkjMzSrHNGgRLEaNJ1DCjO0nPumbJTICQ%3D%3D&quot;}" data-action="a-modal"><a class="a-link-normal" href="#">Leave ad feedback</a></span></div></div><div class="a-row a-spacing-small"><div class="a-row a-spacing-none"><a class="a-link-normal s-access-detail-page s-color-twister-title-link a-text-normal" href="/gp/slredirect/picassoRedirect.html/ref=pa_sp_atf_stripbooks_sr_pg1_1?ie=UTF8&amp;adId=A04794103EZM52VSV07ZR&amp;url=https%3A%2F%2Fwww.amazon.com%2FMatplotlib-2-x-Example-Multi-dimensional-charts%2Fdp%2F1788295269%2Fref%3Dsr_1_1_sspa%2F136-9982183-6544141%3Fs%3Dbooks%26ie%3DUTF8%26qid%3D1536908344%26sr%3D1-1-spons%26keywords%3Dpython%26psc%3D1&amp;qualifier=1536908344&amp;id=88207326666658&amp;widgetName=sp_atf" title="Matplotlib 2.x By Example: Multi-dimensional charts, graphs, and plots in Python"><h2 class="a-size-medium s-inline s-access-title a-text-normal" data-attribute="Matplotlib 2.x By Example: Multi-dimensional charts, graphs, and plots in Python" data-max-rows="0"><span class="a-offscreen">[Sponsored]</span>Matplotlib 2.x By Example: Multi-dimensional charts, graphs, and plots in Python</h2></a><span class="a-letter-space"></span><span class="a-letter-space"></span><span class="a-size-small a-color-secondary">Aug 28, 2017</span></div><div class="a-row a-spacing-none"><span class="a-size-small a-color-secondary">by </span><span class="a-size-small a-color-secondary">Allen Yu and </span><span class="a-size-small a-color-secondary">Claire Chung</span></div></div><div class="a-row a-spacing-none"><span class="a-size-small">Eligible for Shipping to China</span></div><div class="a-row"><div class="a-column a-span7"><div class="a-row a-spacing-none"><a class="a-link-normal a-text-normal" href="/gp/slredirect/picassoRedirect.html/ref=pa_sp_atf_stripbooks_sr_pg1_1?ie=UTF8&amp;adId=A04794103EZM52VSV07ZR&amp;url=https%3A%2F%2Fwww.amazon.com%2FMatplotlib-2-x-Example-Multi-dimensional-charts%2Fdp%2F1788295269%2Fref%3Dsr_1_1_twi_pap_1%2F136-9982183-6544141%3Fs%3Dbooks%26ie%3DUTF8%26qid%3D1536908344%26sr%3D1-1-spons%26keywords%3Dpython%26psc%3D1&amp;qualifier=1536908344&amp;id=88207326666658&amp;widgetName=sp_atf" title="Paperback"><h3 class="a-size-small s-inline a-text-normal" data-attribute="Paperback" data-max-rows="0" data-truncate-by-character="false">Paperback</h3></a></div><div class="a-row a-spacing-none"><a class="a-link-normal a-text-normal" href="/gp/slredirect/picassoRedirect.html/ref=pa_sp_atf_stripbooks_sr_pg1_1?ie=UTF8&amp;adId=A04794103EZM52VSV07ZR&amp;url=https%3A%2F%2Fwww.amazon.com%2FMatplotlib-2-x-Example-Multi-dimensional-charts%2Fdp%2F1788295269%2Fref%3Dsr_1_1_sspa%2F136-9982183-6544141%3Fs%3Dbooks%26ie%3DUTF8%26qid%3D1536908344%26sr%3D1-1-spons%26keywords%3Dpython%26psc%3D1&amp;qualifier=1536908344&amp;id=88207326666658&amp;widgetName=sp_atf"><span class="a-offscreen">$44.99</span><span aria-hidden="true" class="a-color-base sx-zero-spacing"><span class="sx-price sx-price-large">
<sup class="sx-price-currency">$</sup>
<span class="sx-price-whole">44</span>
<sup class="sx-price-fractional">99</sup>
</span>
</span></a></div><div class="a-row a-spacing-none"><span class="a-size-small a-color-secondary">Available to ship in 1-2 days</span></div></div><div class="a-column a-span5 a-span-last"></div></div></div></div></div></div></li>


With chrome inspect (right click menu > inspect ) the intresting element of the above Html block.
    it is easy to find out the tage anem with developer windows right side:
 
```
    images : with tag imge ,inside A link
    price : with css class span.a-offscreen and it is child of A link
    title: with tag h2 ,inside  A link
```

We can build pipes for parsing price, image,title separatly . 
Remenber ,test with only item firstly .If you feel it is ok we can run it will full list.




```python
p_get_img_src=Pipe(lambda t:t.select("a img"),lambda t:t['src'])
p_get_img_src.run(lis[1])
```

    2018-09-14 14:59:10,521 root         INFO:pipe_4514098648 ready to exit





    'https://images-na.ssl-images-amazon.com/images/I/410gNSSY-+L._AC_US218_.jpg'




```python
p_get_title=Pipe(lambda t:t.select("a h2"),lambda t:t.get_text())
p_get_title.run(lis[1])
```

    2018-09-14 14:59:12,536 root         INFO:pipe_4489890280 ready to exit





    '[Sponsored]Matplotlib 2.x By Example: Multi-dimensional charts, graphs, and plots in Python'




```python

p_get_price=Pipe(lambda t:t.select("a > span.a-offscreen"),lambda t:t.get_text())
p_get_price.run(lis[1])
```

    2018-09-14 14:59:14,550 root         INFO:pipe_4514098200 ready to exit





    '$44.99'




```python
p_get_price.Map(lambda x:float(x.replace("$",'')))
p_get_price.run(lis[1])
```

    2018-09-14 14:59:16,563 root         INFO:pipe_4514098200 ready to exit





    44.99



 We created the imge,title,price parser pipes. now need to combine the results together. Zip operator is design for this purpose . it will combine muliti pipe resul to a list. 





```python
item_parse=Pipe(Zip(p_get_price,p_get_title,p_get_img_src))
item_parse.run(lis[1])
```

    2018-09-14 14:59:18,577 root         INFO:pipe_4514224448 ready to exit





    [44.99,
     '[Sponsored]Matplotlib 2.x By Example: Multi-dimensional charts, graphs, and plots in Python',
     'https://images-na.ssl-images-amazon.com/images/I/410gNSSY-+L._AC_US218_.jpg']



Now we run the p_item_parse with full list. Actully there is now differenc from runnning one item. 



```python
item_parse=Pipe(Zip(p_get_price,p_get_title,p_get_img_src))
item_parse.run(lis)
```

    2018-09-14 14:59:20,596 root         INFO:pipe_4514222768 ready to exit





    [[None, None, None],
     [44.99,
      '[Sponsored]Matplotlib 2.x By Example: Multi-dimensional charts, graphs, and plots in Python',
      'https://images-na.ssl-images-amazon.com/images/I/410gNSSY-+L._AC_US218_.jpg'],
     [None,
      "Amazon's Timothy C. Needham Page",
      'https://m.media-amazon.com/images/G/01/nav2/images/tc/authors/tc_author-pages._AA115_.png'],
     [34.99,
      '[Sponsored]SciPy Recipes: A cookbook with over 110 proven recipes for performing mathematical and scientific computations',
      'https://images-na.ssl-images-amazon.com/images/I/517etmpLAwL._AC_US218_.jpg'],
     [27.16,
      'Python Crash Course: A Hands-On, Project-Based Introduction to Programming',
      'https://images-na.ssl-images-amazon.com/images/I/51F48HFHq6L._AC_US218_.jpg'],
     [None, None, None],
     [31.24,
      'Learning Python, 5th Edition',
      'https://images-na.ssl-images-amazon.com/images/I/515iBchIIzL._AC_US218_.jpg'],
     [41.45,
      'Python for Data Analysis: Data Wrangling with Pandas, NumPy, and IPython',
      'https://images-na.ssl-images-amazon.com/images/I/51lod1FJujL._AC_US218_.jpg'],
     [None,
      None,
      ['https://images-na.ssl-images-amazon.com/images/I/51cRqX8DTgL._SCLZZZZZZZ__BG255,255,255_AA218_.jpg',
       'https://images-na.ssl-images-amazon.com/images/I/51FD3C3kLiL._SCLZZZZZZZ__BG255,255,255_AA218_.jpg',
       'https://images-na.ssl-images-amazon.com/images/I/41T830Bh4jL._SCLZZZZZZZ__BG255,255,255_AA218_.jpg',
       'https://images-na.ssl-images-amazon.com/images/I/51hFtDvqgfL._SCLZZZZZZZ__BG255,255,255_AA218_.jpg',
       'https://images-na.ssl-images-amazon.com/images/I/518KlDL92eL._SCLZZZZZZZ__BG255,255,255_AA218_.jpg',
       'https://images-na.ssl-images-amazon.com/images/I/51dgaTOYqmL._SCLZZZZZZZ__BG255,255,255_AA218_.jpg',
       'https://images-na.ssl-images-amazon.com/images/I/51qvS8cyLCL._SCLZZZZZZZ__BG255,255,255_AA218_.jpg',
       'https://images-na.ssl-images-amazon.com/images/I/51ntMgZgihL._SCLZZZZZZZ__BG255,255,255_AA218_.jpg',
       'https://images-na.ssl-images-amazon.com/images/I/517XL4pO6jL._SCLZZZZZZZ__BG255,255,255_AA218_.jpg',
       'https://images-na.ssl-images-amazon.com/images/I/41bIwiXUNPL._SCLZZZZZZZ__BG255,255,255_AA218_.jpg',
       'https://images-na.ssl-images-amazon.com/images/I/51tjBN6kkEL._SCLZZZZZZZ__BG255,255,255_AA218_.jpg',
       'https://images-na.ssl-images-amazon.com/images/I/51LH36eCyaL._SCLZZZZZZZ__BG255,255,255_AA218_.jpg',
       'https://images-na.ssl-images-amazon.com/images/I/51zDEWm5kcL._SCLZZZZZZZ__BG255,255,255_AA218_.jpg',
       'https://images-na.ssl-images-amazon.com/images/I/51n9T4nLCeL._SCLZZZZZZZ__BG255,255,255_AA218_.jpg',
       'https://images-na.ssl-images-amazon.com/images/I/51+YNv4mAfL._SCLZZZZZZZ__BG255,255,255_AA218_.jpg']],
     [None,
      None,
      'https://images-na.ssl-images-amazon.com/images/I/51cRqX8DTgL._SCLZZZZZZZ__BG255,255,255_AA218_.jpg'],
     [None,
      None,
      'https://images-na.ssl-images-amazon.com/images/I/51FD3C3kLiL._SCLZZZZZZZ__BG255,255,255_AA218_.jpg'],
     [None,
      None,
      'https://images-na.ssl-images-amazon.com/images/I/41T830Bh4jL._SCLZZZZZZZ__BG255,255,255_AA218_.jpg'],
     [None,
      None,
      'https://images-na.ssl-images-amazon.com/images/I/51hFtDvqgfL._SCLZZZZZZZ__BG255,255,255_AA218_.jpg'],
     [None,
      None,
      'https://images-na.ssl-images-amazon.com/images/I/518KlDL92eL._SCLZZZZZZZ__BG255,255,255_AA218_.jpg'],
     [None,
      None,
      'https://images-na.ssl-images-amazon.com/images/I/51dgaTOYqmL._SCLZZZZZZZ__BG255,255,255_AA218_.jpg'],
     [None,
      None,
      'https://images-na.ssl-images-amazon.com/images/I/51qvS8cyLCL._SCLZZZZZZZ__BG255,255,255_AA218_.jpg'],
     [None,
      None,
      'https://images-na.ssl-images-amazon.com/images/I/51ntMgZgihL._SCLZZZZZZZ__BG255,255,255_AA218_.jpg'],
     [None,
      None,
      'https://images-na.ssl-images-amazon.com/images/I/517XL4pO6jL._SCLZZZZZZZ__BG255,255,255_AA218_.jpg'],
     [None,
      None,
      'https://images-na.ssl-images-amazon.com/images/I/41bIwiXUNPL._SCLZZZZZZZ__BG255,255,255_AA218_.jpg'],
     [None,
      None,
      'https://images-na.ssl-images-amazon.com/images/I/51tjBN6kkEL._SCLZZZZZZZ__BG255,255,255_AA218_.jpg'],
     [None,
      None,
      'https://images-na.ssl-images-amazon.com/images/I/51LH36eCyaL._SCLZZZZZZZ__BG255,255,255_AA218_.jpg'],
     [None,
      None,
      'https://images-na.ssl-images-amazon.com/images/I/51zDEWm5kcL._SCLZZZZZZZ__BG255,255,255_AA218_.jpg'],
     [None,
      None,
      'https://images-na.ssl-images-amazon.com/images/I/51n9T4nLCeL._SCLZZZZZZZ__BG255,255,255_AA218_.jpg'],
     [None,
      None,
      'https://images-na.ssl-images-amazon.com/images/I/51+YNv4mAfL._SCLZZZZZZZ__BG255,255,255_AA218_.jpg'],
     [28.45,
      'Automate the Boring Stuff with Python: Practical Programming for Total Beginners',
      'https://images-na.ssl-images-amazon.com/images/I/517XL4pO6jL._AC_US218_.jpg'],
     [34.99,
      '[Sponsored]Extending SaltStack',
      'https://images-na.ssl-images-amazon.com/images/I/51XFUXE0K5L._AC_US218_.jpg'],
     [46.18,
      '[Sponsored]Tkinter GUI Programming by Example: Learn to create modern GUIs using Tkinter by building real-world projects in Python',
      'https://images-na.ssl-images-amazon.com/images/I/51ot3HNrhUL._AC_US218_.jpg'],
     [17.96,
      'A Smarter Way to Learn Python: Learn it faster. Remember it longer.',
      'https://images-na.ssl-images-amazon.com/images/I/41i1z4hAJAL._AC_US218_.jpg'],
     [19.72,
      'Python: For Beginners: A Crash Course Guide To Learn Python in 1 Week',
      'https://images-na.ssl-images-amazon.com/images/I/41XYO7E9vcL._AC_US218_.jpg'],
     [25.09,
      'Introducing Python: Modern Computing in Simple Packages',
      'https://images-na.ssl-images-amazon.com/images/I/51RI0fguiLL._AC_US218_.jpg'],
     [6.53,
      "Python Pocket Reference: Python In Your Pocket (Pocket Reference (O'Reilly))",
      'https://images-na.ssl-images-amazon.com/images/I/517k0XB9ogL._AC_US218_.jpg'],
     [42.89,
      'Fluent Python: Clear, Concise, and Effective Programming',
      'https://images-na.ssl-images-amazon.com/images/I/51cRqX8DTgL._AC_US218_.jpg'],
     [14.2,
      'Effective Python: 59 Specific Ways to Write Better Python (Effective Software Development Series)',
      'https://images-na.ssl-images-amazon.com/images/I/518KlDL92eL._AC_US218_.jpg'],
     [[53.16, 106.32],
      'Starting Out with Python (4th Edition)',
      'https://images-na.ssl-images-amazon.com/images/I/41QomZI0DSL._AC_US218_.jpg'],
     [[34.7, 0.0],
      'Deep Learning with Python',
      'https://images-na.ssl-images-amazon.com/images/I/41kxrQD7R+L._AC_US218_.jpg'],
     [44.99,
      '[Sponsored]Artificial Intelligence with Python: A Comprehensive Guide to Building Intelligent Apps for Python Beginners and Developers',
      'https://images-na.ssl-images-amazon.com/images/I/511dKcnBYfL._AC_US218_.jpg'],
     [29.99,
      '[Sponsored]Python for Offensive PenTest: A practical guide to ethical hacking and penetration testing using Python',
      'https://images-na.ssl-images-amazon.com/images/I/51V-4WJ7+YL._AC_US218_.jpg'],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None],
     [None, None, None]]



## Step3: Filter the None item

We will use Filter operator to filter out the item with None price.



```python
item_parse=Pipe(Zip(p_get_price,p_get_title,p_get_img_src)).Filter(lambda i : i[0])
r=item_parse.run(lis)
```

    2018-09-14 14:59:22,622 root         INFO:pipe_4514472960 ready to exit



```python
r
```




    [[44.99,
      '[Sponsored]Matplotlib 2.x By Example: Multi-dimensional charts, graphs, and plots in Python',
      'https://images-na.ssl-images-amazon.com/images/I/410gNSSY-+L._AC_US218_.jpg'],
     [34.99,
      '[Sponsored]SciPy Recipes: A cookbook with over 110 proven recipes for performing mathematical and scientific computations',
      'https://images-na.ssl-images-amazon.com/images/I/517etmpLAwL._AC_US218_.jpg'],
     [27.16,
      'Python Crash Course: A Hands-On, Project-Based Introduction to Programming',
      'https://images-na.ssl-images-amazon.com/images/I/51F48HFHq6L._AC_US218_.jpg'],
     [31.24,
      'Learning Python, 5th Edition',
      'https://images-na.ssl-images-amazon.com/images/I/515iBchIIzL._AC_US218_.jpg'],
     [41.45,
      'Python for Data Analysis: Data Wrangling with Pandas, NumPy, and IPython',
      'https://images-na.ssl-images-amazon.com/images/I/51lod1FJujL._AC_US218_.jpg'],
     [28.45,
      'Automate the Boring Stuff with Python: Practical Programming for Total Beginners',
      'https://images-na.ssl-images-amazon.com/images/I/517XL4pO6jL._AC_US218_.jpg'],
     [34.99,
      '[Sponsored]Extending SaltStack',
      'https://images-na.ssl-images-amazon.com/images/I/51XFUXE0K5L._AC_US218_.jpg'],
     [46.18,
      '[Sponsored]Tkinter GUI Programming by Example: Learn to create modern GUIs using Tkinter by building real-world projects in Python',
      'https://images-na.ssl-images-amazon.com/images/I/51ot3HNrhUL._AC_US218_.jpg'],
     [17.96,
      'A Smarter Way to Learn Python: Learn it faster. Remember it longer.',
      'https://images-na.ssl-images-amazon.com/images/I/41i1z4hAJAL._AC_US218_.jpg'],
     [19.72,
      'Python: For Beginners: A Crash Course Guide To Learn Python in 1 Week',
      'https://images-na.ssl-images-amazon.com/images/I/41XYO7E9vcL._AC_US218_.jpg'],
     [25.09,
      'Introducing Python: Modern Computing in Simple Packages',
      'https://images-na.ssl-images-amazon.com/images/I/51RI0fguiLL._AC_US218_.jpg'],
     [6.53,
      "Python Pocket Reference: Python In Your Pocket (Pocket Reference (O'Reilly))",
      'https://images-na.ssl-images-amazon.com/images/I/517k0XB9ogL._AC_US218_.jpg'],
     [42.89,
      'Fluent Python: Clear, Concise, and Effective Programming',
      'https://images-na.ssl-images-amazon.com/images/I/51cRqX8DTgL._AC_US218_.jpg'],
     [14.2,
      'Effective Python: 59 Specific Ways to Write Better Python (Effective Software Development Series)',
      'https://images-na.ssl-images-amazon.com/images/I/518KlDL92eL._AC_US218_.jpg'],
     [[53.16, 106.32],
      'Starting Out with Python (4th Edition)',
      'https://images-na.ssl-images-amazon.com/images/I/41QomZI0DSL._AC_US218_.jpg'],
     [[34.7, 0.0],
      'Deep Learning with Python',
      'https://images-na.ssl-images-amazon.com/images/I/41kxrQD7R+L._AC_US218_.jpg'],
     [44.99,
      '[Sponsored]Artificial Intelligence with Python: A Comprehensive Guide to Building Intelligent Apps for Python Beginners and Developers',
      'https://images-na.ssl-images-amazon.com/images/I/511dKcnBYfL._AC_US218_.jpg'],
     [29.99,
      '[Sponsored]Python for Offensive PenTest: A practical guide to ethical hacking and penetration testing using Python',
      'https://images-na.ssl-images-amazon.com/images/I/51V-4WJ7+YL._AC_US218_.jpg']]



## Step4:  Draw hist with Pandas


```python
import pandas as pd
headers=["price","title","image_src"]
df = pd.DataFrame(r, columns=headers)

```


```python
df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>price</th>
      <th>title</th>
      <th>image_src</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>44.99</td>
      <td>[Sponsored]Matplotlib 2.x By Example: Multi-di...</td>
      <td>https://images-na.ssl-images-amazon.com/images...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>34.99</td>
      <td>[Sponsored]SciPy Recipes: A cookbook with over...</td>
      <td>https://images-na.ssl-images-amazon.com/images...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>27.16</td>
      <td>Python Crash Course: A Hands-On, Project-Based...</td>
      <td>https://images-na.ssl-images-amazon.com/images...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>31.24</td>
      <td>Learning Python, 5th Edition</td>
      <td>https://images-na.ssl-images-amazon.com/images...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>41.45</td>
      <td>Python for Data Analysis: Data Wrangling with ...</td>
      <td>https://images-na.ssl-images-amazon.com/images...</td>
    </tr>
    <tr>
      <th>5</th>
      <td>28.45</td>
      <td>Automate the Boring Stuff with Python: Practic...</td>
      <td>https://images-na.ssl-images-amazon.com/images...</td>
    </tr>
    <tr>
      <th>6</th>
      <td>34.99</td>
      <td>[Sponsored]Extending SaltStack</td>
      <td>https://images-na.ssl-images-amazon.com/images...</td>
    </tr>
    <tr>
      <th>7</th>
      <td>46.18</td>
      <td>[Sponsored]Tkinter GUI Programming by Example:...</td>
      <td>https://images-na.ssl-images-amazon.com/images...</td>
    </tr>
    <tr>
      <th>8</th>
      <td>17.96</td>
      <td>A Smarter Way to Learn Python: Learn it faster...</td>
      <td>https://images-na.ssl-images-amazon.com/images...</td>
    </tr>
    <tr>
      <th>9</th>
      <td>19.72</td>
      <td>Python: For Beginners: A Crash Course Guide To...</td>
      <td>https://images-na.ssl-images-amazon.com/images...</td>
    </tr>
    <tr>
      <th>10</th>
      <td>25.09</td>
      <td>Introducing Python: Modern Computing in Simple...</td>
      <td>https://images-na.ssl-images-amazon.com/images...</td>
    </tr>
    <tr>
      <th>11</th>
      <td>6.53</td>
      <td>Python Pocket Reference: Python In Your Pocket...</td>
      <td>https://images-na.ssl-images-amazon.com/images...</td>
    </tr>
    <tr>
      <th>12</th>
      <td>42.89</td>
      <td>Fluent Python: Clear, Concise, and Effective P...</td>
      <td>https://images-na.ssl-images-amazon.com/images...</td>
    </tr>
    <tr>
      <th>13</th>
      <td>14.2</td>
      <td>Effective Python: 59 Specific Ways to Write Be...</td>
      <td>https://images-na.ssl-images-amazon.com/images...</td>
    </tr>
    <tr>
      <th>14</th>
      <td>[53.16, 106.32]</td>
      <td>Starting Out with Python (4th Edition)</td>
      <td>https://images-na.ssl-images-amazon.com/images...</td>
    </tr>
    <tr>
      <th>15</th>
      <td>[34.7, 0.0]</td>
      <td>Deep Learning with Python</td>
      <td>https://images-na.ssl-images-amazon.com/images...</td>
    </tr>
    <tr>
      <th>16</th>
      <td>44.99</td>
      <td>[Sponsored]Artificial Intelligence with Python...</td>
      <td>https://images-na.ssl-images-amazon.com/images...</td>
    </tr>
    <tr>
      <th>17</th>
      <td>29.99</td>
      <td>[Sponsored]Python for Offensive PenTest: A pra...</td>
      <td>https://images-na.ssl-images-amazon.com/images...</td>
    </tr>
  </tbody>
</table>
</div>



There is some bad item with mulit prices .I just drop it out .You can do more complex convert with pandas.


```python
price=pd.to_numeric(df['price'], errors='coerce')
```


```python
price.hist()
```




    <matplotlib.axes._subplots.AxesSubplot at 0x11a3bdb00>



## Run with more data:

### Get more item by set range of page no. 
``` python

lis=p_download.run(range(1,10)

```

### Change search keywords.
it is unable to create nest loop in pipes. we generate all  combination with prodcut function.


``` python 
from  itertools import product

lis=p_download.run(product(range(1,10),['Python','Java','C++'])

                 ```







```python

```
