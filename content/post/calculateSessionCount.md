+++
author = "Emre Erkoca"
title = "Calculating User's Session Count"
date = "2020-04-19"
description = "Basic function for calculating user's sesion count"
tags = [
    "session",
    "javascript"
]
categories = [
    "JavaScript",
    "Sample Functions",
]
series = ["JavaScript"]
+++
I wanted to calculate the user's session count through session storage and local storage. 
<!--more-->

1. Get the last session value from local storage.
2.  
    *  If there is no stored value create new storage items. Session storage prevents increase value in the same session.
    *  If the last session value is not null, the user was closed browser and opened it again. Increase the last storage value and save the last values. 
3.  Finally it returns session count.

{{< highlight js >}}
var updateStorages = (storageValue) => {
    localStorage.setItem('last-session-value', storageValue);
    sessionStorage.setItem('current-session', storageValue);
};

var getSessionCount = () => {
    var lastSessionValue = localStorage.getItem('last-session-value');

    if (lastSessionValue === null) {
        updateStorages(1);
    } else if (lastSessionValue && sessionStorage.getItem('current-session') === null) {
        lastSessionValue++;

        updateStorages(lastSessionValue);
    }

    return parseInt(lastSessionValue);
};


getSessionCount();
{{< /highlight >}}

It's my first technical post. It's just basic solution and I wanted to share it. I would like write more complicated things too. Cheers.  

