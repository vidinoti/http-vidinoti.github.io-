---
layout: documentation
title: Enable multilingual support
category: Sdk
order: 5
---

# Introduction

This page describes how to enable multilingual support in your application.

# In your application

When syncing content with your application you call `syncRemoteContextsAsynchronously` on iOS or `syncRemoteContextsAsynchronouslyWithPriors` on Android, passing `nil/null` instead of a list of priors loads all contents available. Consider "priors" as filters for synchronizing only certain contents. To enable multilingual you need to add the desired language tag to the list of priors.

## Tags

Tags are basically filters for the contents. Loading content with a tag means that only the contents marked with the specified tag will be loaded. Since this is just what we want, we can use tag to filter contents by language. A little bit further in this page you will see how to add language tag in V-Director. In the meantime, you can find a language using a string representation containing the prefix "lang_" and the suffix corresponding to the 
[ISO 639-1 code](https://en.wikipedia.org/wiki/List_of_ISO_639-1_codes) of the language.

Example: French -> lang_fr, English -> lang_en

## Detect the phone language

The first thing to do is to find the language used by the phone in your application.

**Android**

```
String tagName = "lang_en" //Default language
String langCode = Locale.getDefault().getLanguage();
if(langCode == "fr"){
    tagName = "lang_fr"
}
```

**iOS**

```
var tagName: String;
if let langCode = Locale.current.languageCode, langCode == "fr" {
    tagName = "lang_fr"
} else {
    tagName = "lang_en" //Default Language
}
```

## Load contents with the language tag

Now that you have the tag corresponding to language used by the phone, you can call `syncRemoteContextsAsynchronously` on iOS or `syncRemoteContextsAsynchronouslyWithPriors` on Android with a list of priors.

**Android**

```
List<VDARPrior> priors = new LinkedList<>();
priors.add(new VDARTagPrior(tagName));
VDARRemoteController.getInstance().syncRemoteContextsAsynchronouslyWithPriors(priors, new Observer() {
    public void update(Observable observable, Object data) {
        // ...
    }
});
```

**iOS**

```
let priors: [Any] = [VDARTagPrior(tagName: tagName)]
VDARSDKController.sharedInstance()?.afterLoadingQueue.addOperation {
    DispatchQueue.main.async {
        VDARRemoteController.sharedInstance()?.syncRemoteModelsAsynchronously(withPriors: priors, withCompletionBlock: {
        	// ...
        })
    }
}
```

## More information

Adding a `VDARIntersectionPrior` will act as a logical AND, so the content must have all tags to be loaded.
For example if you want to load all the content tagged as "best_ar_content" AND "lang_en", you need to add both tags to the list of priors within a `VDARIntersectionPrior`.

# V-Director

You are almost done ! You just need to enable 2 more features in V-Director and assign tags to your content.

## Enable multilingual support

Go in SDK>SDK Settings and check "Enable multilingual support". Here you can add any language you want to support, the tag generated by the lang you select is described in the [Tags](#tags) section.
![Enable multilingual support]({{ site.url }}/img/multilingual/enable.png)

## Create content

First you need to go in My Contents>Content list, then to assign a tag to content you have two ways : edit the content or drag and drop the image of the content on the tag.

**Edit the content**

![Edit content]({{ site.url }}/img/multilingual/lang.png)

**Drag and drop**

Drag the image and drop it on the desired tag.  

![Drag and drop]({{ site.url }}/img/multilingual/drag.png)

## Use tag management

Go to SDK>My Applications, edit your application to check "Use tag management system".


![Management]({{ site.url }}/img/multilingual/management.png)


***WARNING**: if your application is already in production you should update your application before checking this !*  
When you check "Use tag management system" and only when it's checked, your application will apply the priors list when synchronizing (i.e. contents will be filtered). When it is unchecked your system does not care about the priors list and synchronizes all contents (independently of their tags). This means that if you update your code and add multilingual support after releasing your application everyone that has the version without multilingual won't have access to any content. What you should do instead is : update your application with multilingual support, leave time for people to update your application and then check "Use tag management system".  
