+++
author = "Emre Erkoca"
title = "Dotnet Core Localization"
date = "2021-02-26"
description = "Dotnet Core Localization Sample"
tags = [
    "Dotnet",
    "Dotnet Core",
    ".NET",
    ".NET Core",
    "Localization",
    "Dotnet Core Localization",
    ".NET Core Localization"
]
categories = ["Dotnet Core"]
+++
I had to make a multi language application. Because there are a lot of languages and you might want to be global for more customers.
<!--more-->
I had to make a multi language application. Because there are a lot of languages and you might want to be global for more customers.  I wanted to write a simple example before localization. I read documents and I felt a little bit confused. Then, I wanted to share my sample code.

You can store your text messages in a resources file. A resource record is composed of two parts. Name and value. Actually there is one more part (comment) but probably you won’t use this part.

An example resource file looking like this:

![Resources](/images/Resources.png)

You can create resource files for different languages. 
Resource.resx, Resource.de.resx, Resource.es.resx, etc.

If you want to apply localization properly, names must be same in your resource files. When the localizer doesn't find resource content, it returns null. I’ll explain this project step by step. https://github.com/emreerkoca/LocalizationExample It’s a simple Dotnet Core Web Api project. 

1. Add Resources folder to your project. 
2. Add your resource files.

![ResourcesDirectory](/images/ResourcesDirectory.png)

3. Add a new folder  to your project as “Localize” and add a public class to Localize folder.

{{< highlight csharp >}}
namespace LocalizationExample.Localize
{
    public class Resource
    {
    }
}
{{< /highlight >}}

If you want you can  use different names but your resource names and your resources must be suitable. You will reach resource content through this empty class. 

Resource name: Localize.Resource.de.resx
Namespace is your class is LocalizationExample.Localize and class name is Resource.cs


4. Startup.cs configuration

Add this code block to into ConfigureServices(IServiceCollection services) method

{{< highlight csharp >}}
services.AddLocalization(options =>
{
    options.ResourcesPath = "Resources";
});

services.Configure<RequestLocalizationOptions>(options =>
{
    var supportedCultures = new[]
    {
        new CultureInfo("en-US"),
        new CultureInfo("de")
    };

    options.DefaultRequestCulture = new RequestCulture(culture: "en-US", uiCulture: "en-US");
    options.SupportedCultures = supportedCultures;
    options.SupportedUICultures = supportedCultures;
});
{{< /highlight >}}

It’s about adding supported languages for localizers. Showing your resource files. Selecting default language.

Add this code block to **bold**Configure(IApplicationBuilder app, IWebHostEnvironment env)**bold** method.

{{< highlight csharp >}}
var locOptions = app.ApplicationServices.GetService<IOptions<RequestLocalizationOptions>>();
app.UseRequestLocalization(locOptions.Value);
{{< /highlight >}}

It’s for automatically setting culture info for your localizer. You can set culture info by any request header value. 

5. Showing localized messages

I used IStringLocalizer interface for reaching resource files. You can see an example usage here. Add dependencies and call the resource field name. 

{{< highlight csharp >}}
private readonly IStringLocalizer<Resource> _stringLocalizer;

public HomeController(IStringLocalizer<Resource> stringLocalizer)
{
    _stringLocalizer = stringLocalizer;
}

[HttpGet]
public string Get()
{
    var value = _stringLocalizer["Hello"];

    return value;
}
{{< /highlight >}}


Get() method will return “Hello!” because English is default culture and English resource value is “Hello!” When you add “culture=de” to your request url, you’ll change culture as “de” and you’ll see “Hallo!” message. 

If you want you can detect current language from “Accept-Language” header. You can change your configuration. 

Change your ConfigureServices configuration part like this: 

{{< highlight csharp >}}
public void ConfigureServices(IServiceCollection services)
{
    services.AddControllers();

    services.AddLocalization(options =>
    {
        options.ResourcesPath = "Resources";
    });

    services.Configure<RequestLocalizationOptions>(options =>
    {
        var supportedCultures = new[]
        {
            new CultureInfo("en-US"),
            new CultureInfo("de")
        };

        options.SupportedCultures = supportedCultures;
        options.SupportedUICultures = supportedCultures;

        options.RequestCultureProviders.Insert(0, new CustomRequestCultureProvider(context =>
        {
            var languages = context.Request.Headers["Accept-Language"].ToString();
            var currentLanguage = languages.Split(',').FirstOrDefault();
            var defaultLanguage = string.IsNullOrEmpty(currentLanguage) ? "en-US" : currentLanguage;

            if (defaultLanguage != "de" && defaultLanguage != "en-US")
            {
                defaultLanguage = "en-US";
            }

            return Task.FromResult(new ProviderCultureResult(defaultLanguage, defaultLanguage));
        }));
    });
}
{{< /highlight >}}

It’s reading the “Accept-Language” header value and getting the default language. Put a breakpoint after the language variable and see values. It’s changeable for different browser languages. If you feel confused when you read CustomRequestCultureProvider, ProviderCultureResult, etc. you can read them from official Microsoft documents. 

Now, you don’t have to add a culture parameter to request url. You can try to get values from  English and German browsers. 

You can read this document for detailed information.
https://docs.microsoft.com/en-us/aspnet/core/fundamentals/localization?view=aspnetcore-5.0

I have nothing to say after this.  I hope it might be helpful. 





