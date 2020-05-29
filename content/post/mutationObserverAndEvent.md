+++
author = "Emre Erkoca"
title = "MutationObserver and Event Usage"
date = "2020-05-29"
description = "A simple example of MutationObserver and Event"
tags = [
    "JavaScript",
    "Web APIs",
    "MutationObserver",
    "Event"
]
categories = [
    "JavaScript",
    "Web APIs",
]
series = ["JavaScript"]
+++
I wanted to share a simple example of MutationObserver and Event. 
<!--more-->

I want to explain it through an example. Say you have your own company and you're working with e-commerce companies. Your customer says to you "Can you provide user's last added product for another consultant company. We need this information" and you said "Yes, we can do this"

Of course, you can find a professional solution to this problem. I'll make this with MutationObserver and Event. MutationObserver is using detect DOM changes. For example, you have an element. You can detect any changes in this element. Adding a new child element, changing element content, adding new attribute, etc. I used it for my card page element. It's imitating a basic cart page. I'll add a child div to here as a product.


{{< highlight html >}}
<!DOCTYPE html>
<html>
    <head>
        <title>MutationObserve and Event Usage</title>
    </head>
    <body>
        <div class="cart-modal">
            <div class="product" data-product-id="424242">Product 1</div>
            <div class="product" data-product-id="213113">Product 2</div>
        </div>
    </body>
</html>
{{< /highlight >}}

I added a new child element in this code.

{{< highlight js >}}
setTimeout(function() {
    var product = document.createElement("div");
    var node = document.createTextNode("Product 3");

    product.appendChild(node);
    product.classList.add('product');
    product.setAttribute('data-product-id', '232323');

    var element = document.querySelector(".cart-modal");

    element.appendChild(product);
}, 4000);
{{< /highlight >}}

I used setTimeout for wait 4000 milliseconds. Because we want to detect DOM changes. It'll trigger after 4000 milliseconds. You can review MutationObserver sample codes from here:

{{< highlight js >}}
//Your target element
const targetNode = document.querySelector('.cart-modal');

//config defined for change types. I'm just listening child element additions
const config = { childList: true };

//You'll understand this from name. MutationList so list of detected mutations
const callback = function(mutationsList, observer) {
    for(let mutation of mutationsList) { 
        if (mutation.type === 'childList') {
            //if mutation type is childList, any child element has been added or any child element has been modified
            //I created a new event and I triggered it. I explained it below
            createAndTriggerEvent({
                id: mutation.addedNodes[0].attributes['data-product-id'].nodeValue,
                name: mutation.addedNodes[0].innerText
            });
        }
    }
};

const observer = new MutationObserver(callback);

// Start observing the target node for configured mutations
observer.observe(targetNode, config);

// Later, you can stop observing.
//observer.disconnect();
{{< /highlight >}}

I wrote createAndTriggerEvent function for creating a new Event and triggering it. When our data is ready we're triggering it. 

{{< highlight js >}}
createAndTriggerEvent = (productInfo) => {
    var event = new Event('listenLastAddedProduct');

    window.lastAddedProduct = productInfo;

    document.dispatchEvent(event);
}
{{< /highlight >}}

Finally, I'm listening to our custom event. When it's triggering we're catching data.

{{< highlight js >}}
document.addEventListener('listenLastAddedProduct', function (e) {
    console.log(window.lastAddedProduct);
}, false);
{{< /highlight >}}

You can get all JavaScript codes from here:

{{< highlight js >}}
 createAndTriggerEvent = (productInfo) => {
    var event = new Event('listenLastAddedProduct');

    window.lastAddedProduct = productInfo;

    document.dispatchEvent(event);
}

setTimeout(function() {
    var product = document.createElement("div");
    var node = document.createTextNode("Product 3");

    product.appendChild(node);
    product.classList.add('product');
    product.setAttribute('data-product-id', '232323');

    var element = document.querySelector(".cart-modal");

    element.appendChild(product);
}, 4000);


const targetNode = document.querySelector('.cart-modal');
const config = { childList: true };

const callback = function(mutationsList, observer) {
    for(let mutation of mutationsList) { 
        console.log(mutationsList);

        if (mutation.type === 'childList') {
            createAndTriggerEvent({
                id: mutation.addedNodes[0].attributes['data-product-id'].nodeValue,
                name: mutation.addedNodes[0].innerText
            });
        }
    }
};

const observer = new MutationObserver(callback);

observer.observe(targetNode, config);

document.addEventListener('listenLastAddedProduct', function (e) {
    console.log(window.lastAddedProduct);
}, false);

// Later, you can stop observing
//observer.disconnect();
{{< /highlight >}}
