+++
title =  "A-Z Tech to be thankful for in 2020"
date =  2020-10-14T20:46:12-04:00
menu = "main"
+++
![visitors](https://visitor-badge.glitch.me/badge?page_id=gucci-ninja-thanksgiving)

![](../../img/tech.jpg)

Hi everyone! This Thanksgiving my mom did an activity with her grade 2’s where they went around their virtual classroom listing things they were thankful for from A-Z. This inspired me to write my very first Medium article on technologies that I’m thankful for this Thanksgiving :) These range from tech I’ve used to tech I am interested in using in the future.

### AWS
I started using Amazon’s web services earlier this year to host a project. It was really easy to get started since there were suggestions on what service to use based on your needs. I opted for their virtual cloud computing service, EC2. I chose from the free tier Linux options and in minutes my team and I had a server running in Ohio ready for use.

### Blockchain
This has been one of those buzz words in the tech industry for a few years now but I have never come around to using blockchain for any of my projects. When I attended Hack Western in 2018, the first place hack leveraged the use of Ethereum to create a gun mod that writes a transaction with the user’s GPS location when the trigger is pulled. They used nerf guns as a prototype, which was pretty cool. Since then, I’ve been thinking of projects that I could incorporate blockchain with since there’s a lot of potential for problem solving.

### Cordova
Like React Native, Cordova is a mobile app framework that lets you build applications for multiple platforms. I thought React Native was a little hard to work with due to a steep learning curve and I found myself re-implementing a lot of my web dev code blocks to work on mobile. This year, I want to explore my options and give Cordova a chance.

### Docker
When I was first introduced to Docker, I thought it was a virtual machine to run your code on. I was like, WTH, that’s overkill — I don’t need a whole VM to run Hello World. But I guess I kind of do because one of the hardest parts about working on a project in a group is that everyone has a different environment — some of us have all the dependencies to run the code, some of us have 50% of them and some have none. Docker basically removed the concept of installing software and solved 90% of the issues I would normally run into when setting up a project. My applications just sit in their own little container now.

### Elasticsearch
I don’t know a lot about Elasticsearch except that it’s a search engine used by A LOT of big companies. So it’s probably really powerful for its use case. Together with LogStash and Kibana, it comprises the ELK stack which is widely used for ingesting and visualizing big data. I don’t have a lot of big data lying around but I really want to use Elasticsearch some time in the future.

### Firebase
The first time I found out about Firebase was when I joined the tech team for McMaster’s hackathon, Delta Hacks. I was going through the codebase and when I got to the databases, I couldn’t find a schema. I got really concerned for a bit since I wasn’t sure how we were planning on saving data to a database when we don’t know what the data is :( But eventually I found out that they were using Firestore, one of Firebase’s products, a scalable, NoSQL document database. So yeah no schemas it’s all JSON, which is super easy to work with. Since then, I’ve used almost all of Firebase’s services — databases, hosting, cloud functions and I have no complaints!

### Git
I mean, I don’t see why this one wouldn’t be on everyone’s list. To be honest, I don’t even know any other version control tool because this is the only one I need. Making new changes to code and getting updates to code has never been easier thanks to git. And while we are at it, [follow me on Github](https://github.com/gucci-ninja) :stuck_out_tongue:

### Haskell
So, if you’re looking to learn functional programming and don’t know where to start, I would highly recommend Haskell. I took a course on this in my final year of engineering and it did more than just give me another language to add to my toolbox. Functional programming really allowed me to look at problems a little differently. For example, it made the concept of trees really simple. In Haskell, you can just define a tree like an equation.

```haskell
data Tree a = Leaf a | Branch (Tree a) (Tree a)
```

Isn’t that kind of cool? A Tree is a recursive data type that can either be a Leaf with value ‘a’ (Base Case) or a Branch with 2 Trees (Recursive Case). Completely changed my life. If you’re looking for a resource, [Learn You A Haskell](http://learnyouahaskell.com/chapters) was pretty solid.

### IoT
Internet of Things has been a big deal and I’ve slowly become obsessed with curating my perfect Smart Home with voice controlled lighting, Google routines and that type of stuff. I am super thankful for IoT technology for letting me be my laziest self and also giving me so many opportunities to delve into hardware like the Raspberry Pi, which I stuffed into a shoe with a bunch of sensors, that could sense if the person wearing it tripped. My brother and I called them [Trip Shoes](https://gucci-ninja.github.io/images/projects/tripshoes.png).

### Jenkins
I thought of a lot of things that start with J that I’m thankful for — JavaScript, Java, even JQuery. But I don’t want to fill up my list with tech that everyone knows. I haven’t personally used Jenkins but I want to add it to my dev workflow and ultimately have a solid pipeline that is automated, thoroughly tested and easily monitored. Thankfully, Jenkins is free and open source so I can make it a part of my stack, like a lot of big tech companies have done.

### Kubernetes
Recently I’ve been taking on a more DevOps role at work, managing a K8s deployment of one of our projects and it’s been a lot easier than I thought it would be. Even though a lot of the heavy lifting has already been done by people before me, most of last month I was debugging an issue with connections being reset for our end-users. I had to ssh into our K8s pod and figure out why things were going wrong and honestly, as long as you have the following commands with you — you can do anything.

```bash
$ kubectl get pods --all-namespaces    # get all pods
$ kubectl logs -f <pod-id>             # view logs
$ kubectl exec -t -i <pod-id> bash     # execute command
```

### Linux
So, I actually have a love-hate relationship with Linux because as an operating system, it’s open source, secure and doesn’t come with Windows bloatware, which is really cool. But sadly, my Linux distro of choice is Arch, so the setup process was basically like raising a child. I had to install my own display server, wifi manager and even the little volume controls I used to take for granted. I also learned the hard way that you should always back up your dotfiles (configurations) in case something bad happens. By the way if you ever wanted to see my dotfiles [here they are](https://github.com/gucci-ninja/.dotfiles)! I’m super thankful that Linux exists so I can have a sick operating system and even more thankful to the Arch Wiki for being so informative. Not so thankful for the Arch community though since it can get kind of toxic for newcomers.

### MongoDB
Can’t really say much here, it’s a great database for big and small data. It’s a document based, schema-less database option but for my projects I use `mongoose`, a Node library that lets you model your application data and connects to MongoDB. So I have to write a schema for my objects, giving my application a little bit more structure while still maintaining the flexibility of MongoDB.

### Nuxt.js
I started using Nuxt earlier this year because I wanted to build an application that has a backend server as well as a frontend. Instead of starting up a node server and having my frontend communicate with it, I decided to make my life a little easier by using Nuxt’s server side rendering mode. Nuxt takes the hard work off your shoulders by generating a folder for your server-side code, frontend code, plugins and anything else you can think of that you might want to add to your web application. I love how easy it makes my experience as a developer.

### Open Source
Without open source, I would have almost nothing to put on this A-Z list. Almost everything you’ve read about so far is open source or has some open sourced elements. So I am super thankful to all the developers who contribute to open source projects and continue to believe in it. In the upcoming years, I want to start contributing to more projects by looking at open issues on GitHub.

### Postman
API development has never been easier thanks to Postman. I will literally make an API just so I can test it out on Postman, that’s how much I love this platform. I especially love that if you are testing a certain API endpoint, for example GET posts, it will save your query so you can come back to it a week later to test the same thing.

### QR Codes
I thought it’d be really weird if I didn’t mention the coronavirus pandemic in an article about 2020 so I’m really thankful for QR codes since I can walk into a restaurant, scan a QR code and look at the menu. No more paper menus that get touched by every single visitor! Also, it’s really hard to find tech that starts with the letter Q so if you have any suggestions please let me know!

### Ruby on Rails
It was either React or Rails. The Rails framework is probably the easiest to work with, gives you the most out of the box functionality and has the NICEST community. Rails is for anyone who wants to build a web app, regardless of who you are and what your past is. The documentation is immaculate and there’s a built-in method for everything. Rails for president.

### Socket IO
Socket IO is a JavaScript library that provides an API implementation to exchange data between users and a server in real time. This library rides on the TCP, which makes this data exchange possible. Briefly, the Transmission Control Protocol is a standard for establishing a connection over a network and maintaining that connection in order to exchange data using sockets. I think the concept of seeing how many sockets are connected to a single node is kind of cool. Like, seeing how many people are online on a certain server. The Socket IO library makes this kind of simple and lets me build really cool real-time interaction applications.

### TypeScript
I never thought I’d be one to advocate for typed languages since they require more work (like knowing what type your data is going to be) but in the end, they do save you a lot of trouble. Less runtime errors due to wrong type casting and in general, better readability. When you look at a codebase, the method and variable names aren’t always the most thoughtful and in those cases, I wish I at least knew the return type or data type so I could further investigate.

### Unity
The only encounter I’ve had with Unity was in 2015 when I made a replica of the famous Pong game following a YouTube tutorial. It was kind of tedious and there was a lot more physics involved than I signed up for. I guess I’m thankful to Unity for allowing me to realize that game development was not really my thing and that there are other options when it comes to software development. That being said, I wouldn’t be opposed to trying again with a game idea that actually excites me (because Pong was not it).

### Vue.js
I started working with Vue.js last year when I joined the Delta Hacks technical team. I thought it was kinda nice that I could learn a new frontend framework, especially since we got access to a Vue course on Udemy. Since then I’ve used it for a lot of my projects and really like how easy it is to use. The built-in directives make for some clean code writing, for example, you can just have this in your template:
```html
<ul>
  <li v-for="item in items" :key="item.name">
    {{ item.name }}
  </li>
</ul>
```

Looks pretty easy right? 11/10 would recommend.

### World Wide Web
This one is self-explanatory — none of us would really be here without the web so, gotta give big thanks. The www is designed to be decentralized, but even with a handful of tech giants having control over it, I’m thankful that there are some pages on the web that I’ve contributed to.

### XML
I don’t actually have a lot of good things to say about XML. The only time I have to look at XML is if I make a SOAPful API request, which is almost never. I guess in a way, if you’re thankful for RESTful APIs, which standardize how information is passed over HTTP, you can be thankful for XML because without XML, we might not have had SOAP APIs and without SOAP APIs, we might not have had RESTful APIs.

### YAML
This is another hard one because there’s not a lot of cool tech that starts with the letter Y (not that I know of at least). I love YAML because it’s a recursive acronym — YAML Ain’t Markup Language. Other than that, it’s the file type for a lot of configuration files, and I love configuring things. Like docker-compose! In your `docker-compose.yml` you can define how you want your Docker containers to run. I’m glad YAML exists. For this reason.

### ZSH
After switching to Arch Linux I realized how important your shell and terminal is. I personally use `fish` for my shell but `zsh` is a pretty good competitor. Most environments come equipped with a default shell which lets you run commands and scripts. What I like about zsh (and fish) is that there is spelling correction support, auto complete, and searchable history. Not sure if zsh has this but fish also has syntax highlighting and really good predictive command listing.

#### Conclusion
That’s it! Thanks for reading and let me know what would be on your list :)
