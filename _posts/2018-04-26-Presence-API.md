---
layout: post
cover: 'assets/images/presence.jpg'
title: ServiceNow Presence API
date:   2018-04-26 10:18:00
tags: platform ServiceNow Now API Presence
subclass: 'post tag-test tag-content'
categories: jakubsynowiec
navigation: True
author: 'Jakub Synowiec'
nickname: jakubsynowiec
bio: "Certified ServiceNow Application Developer"
image: 'assets/images/jsynowiec.jpg'
logo: 'assets/images/jg_logo_white.png'
---

With the introduction of new user interface called UI16 ServiceNow implemented a new feature, called User Presence. It allows you to put green dots near avatars of users that are online in multiple different places like: Activity Streams, Visual Task Boards, Live Feeds and Connect Conversations. If the user the activity on the instance, the green dot will turn orange for several seconds, and after no user interaction for the next few seconds the dot will disappear. This is a great way of showing users currently on the system, so I wanted to enable this functionality in one of my Service Portal applications.

##Using it in Service Portal

It appeared that there is no documentation available to developers describing the usage of the Presence API. I was also searching through community forums and found no clear path for a developer to follow here either. Some people suggested using the *sys_user_presence* table, which has information of the user activity, but this solution in my opinion, was not following best practice.My desire to implement the Presence functionality in Service Portal pushed me to reverse engineer existing presence implementation in the regular ServiceNow UI16 view.

##I want it too!

For all of you that want to implement this functionality, the fastest way is downloading the Angular Directive that I've created and uploading to ServiceNow (to the *sp_angular_providers* table), adding it to the widget and using it by passing to the method an array of sys_ids of users you want to get the presence of.

```
var user = {sys_id: "sys_id"};
var users = [user];
spUserPresence.getUserPresence(users)
    .then(fuction(response){
        // use the response
    });
```
This method will return an array of objects containing the sys_id, just like the object passed to the method, and also the *status* property, which will be one of *online*, *away* or *offline*. The download link is at the bottom of this post.

At this point I'd like to point out that ServiceNow could change how the Presence API works in the future, and this method **may not work** for future releases.

##How does it work?

When looking for existing implementation of Presence functionality I noticed that the website every few seconds sends a "connect" REST message, and on the pages that leverage presence functionality there is a record calling the presence API. The URL for the rest call is:

```
/api/now/ui/presence
```
The result of this call contains two important things, first is server time, under property called `serverTimeMilis`, and the second is an array of users, the `presenceArray`. This array consists of objects having user sys_ids and the last time they were present on the system. By subtracting server time from that number, you can get the information how long ago the user was active in the ServiceNow world.

##Things to keep in mind

The way this Angular Directive is implemented, it gives you information about the presence of users, but only once. If you want to have continuous updates, you need to periodically call it. The best way to do it is to enclose the execution in `setInterval` function:
```
setInterval(function(){
    spUserPresence.getUserPresence(users)
        .then(function(response){
            // #TODO: Implement handing
        });
}, 5000);
```

The second argument of this method is time in milliseconds of how often should the function declared in first parameter be called. You can change it, depending how often do you want to refresh it but keep in mind that refreshing it too often may cause performance problems and heavy network usage. Having a long interval on the other hand may result in the green circle showing for a long time after user logs out, so it may be not giving accurate enough information.

The other thing related to time is the decision how long does the user stay online in the system, and after what time the status shall change from away to offline. I have set predefined values using my best estimate on how the ServiceNow determines that in the regular view, but you can change these values in the Angular Directive:

```
var deltaTime = serverTime - present.last_on;
if(deltaTime < 30000) person.status = "online";
else if(deltaTime < 60000) person.status = "away";
else person.status = "offline";
```

##Summary

The Presence API gives us a nice way of showing active users on Service Portal. From my testing, it's efficient and because it follows the same architecture that UI16 has built in, follows the best practices for ServiceNow. However, it's not all good news. This API is not documented at ServiceNow Developer site, and that may be for the reason. It could be changed or completely revoked in the future, leaving you with the code that doesn't work. Having said that, if you develop your widget in a way that it will still work even when the API is not available, you should not have any problems other that the lack of the green dot by the avatar that wasn't even there in the first place.


Download link:
[spUserPresence.xml]({{file name='assets/download/spUserPresence.xml'}})