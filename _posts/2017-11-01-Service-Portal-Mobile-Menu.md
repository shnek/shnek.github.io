---
layout: post
cover: 'assets/images/cover3.jpg'
title: Fix Mobile Menu on Service Portal
date:   2017-10-04 10:18:00
tags: platform
subclass: 'post tag-test tag-content'
categories: jakubsynowiec
navigation: True
author: 'Jakub Synowiec'
nickname: jakubsynowiec
bio: "Certified ServiceNow Application Developer"
image: 'assets/images/jsynowiec.jpg'
logo: 'assets/images/jg_logo_white.png'
---

While developing an application for Service Portal, you are provided with  many features “out of the box”. You get an responsive environment with widgets that can be used on several different pages. But what if you need to change some of those  “out of the box” parts? After one of my applications was tested by QA,  I was given a task to change the behavior  of the menu on mobile view.

## How it works now
![Incorrect](/assets/images/MenuMobile/mobile-menu-incorrect.gif)
Throughout testing, the mobile menu performance was largely inconsistent. When pressing  on one of the options it occasionally  collapsed however, other times it did not.

## How it should be
![Correct](/assets/images/MenuMobile/mobile-menu-correct.gif)
I was asked to change the menu to collapse every time a link is clicked, thus improving the user experience by freeing up valuable screen real estate. Overall, you usually want to see the intended page after clicking the menu link, and not have to collapse the menu manually every time.Ideally, this would be something that could be toggled on and off  in Service  Portal, but sadly that option doesn’t exist.l   The current behavior performs intermittently, which makes for an unacceptable user experience. 

##  Changing it

Fortunately, there is an easy fix. All you’ll need to do is go to Service Portal Menu application:

![Menu](/assets/images/MenuMobile/service-portal-menu.jpeg)
Select the Header Menu that your Service Portal uses.

There are options for changing Menu as a widget itself. You can modify it to your needs, but the change that we need to make is not in this widget itself, but in an angular template. Therefore you’ll need to scroll down, select *ng-template* tab and click on *menuTemplate*:

![menuTemplate](/assets/images/MenuMobile/menu-bottom.jpeg)

At this point you may need to change the current application scope to global or if you are in a different scope than global, just use the link on the popup dialog showing. . After that you can then change the code:

![Code change](/assets/images/MenuMobile/menu-template.jpeg)

All the changes we need to make  are  going to happen in the first line. You should add *data-toggle* and *data-target* to the anchor tag:
``` html
<a data-toggle="collapse" data-target=".navbar-collapse" ng-if="item.items.length == 0 && !item.scriptedItems" ng-href="{{item.href}}"  title="{{item.hint}}">
```
Keep in mind that if you have  any *toggle* or *target* parameters inside an element, you need to get rid of them.

Now, you’re done! Now you need to save your  changes by clicking *Update* in the top right corner and then test away.

##  Summary

There are few things you need to keep in mind when making this change. First of all, this is applicable  to ***all*** the applications using this Header Menu widget on the instance. Then, if you want the change to apply to your application, you’ll need to redo all steps on every instance (because you’re modifying original header). If you need to have the change bind with your application, it should work if you clone this header and use it as your own custom widget. Lastly, this is a change to existing code that was created by ServiceNow itself, tested and deployed to be used by customers. My change may break something, so keep in mind that your own personal mileage may vary. After making  this change, test your application thoroughly and check if it meets your requirements. 

