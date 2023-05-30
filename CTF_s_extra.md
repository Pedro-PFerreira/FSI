# Extra Challenges

## British punctuality

* In this challenge, we have to capture the flag that is hidden the server ctf-fsi.fe.up.pt 4006.

* First, we started to run the command ```nc ctf-fsi.fe.up.pt 4006``` in the terminal to access the server, and started to explore the existent files with ```ls```

![](https://i.imgur.com/KRcFtT2.png)


![](https://i.imgur.com/6jK2H71.png)

* We ran ```cat main.c``` to see its content and analysed main.c. We discovered that it had a function main which verifies if the file flag.txt exists or not.If the file exists, it would print the flag and String "File exists"; else, the file only prints "Files doesn´t exist."

* After that, we ran the command ```cat my_script.sh``` and we analysed its code:

![](https://i.imgur.com/lNQuRBl.png)

* We discovered that this script was exporting the content contained in "env" file of the "tmp" folder. We also knew that this folder contains files that are generated temporarily.

* We've also noticed that the script was being run periodically, since after some time, the commands were not working and terminated the connection with the server:

![](https://i.imgur.com/oPNqreN.png)

* With that information, we could conclude that were restricted to a short period of time to explore the server. So, we rapidly ran the commands ```cd /tmp/``` and ```ls``` to explore the files that were contained in this folder. We've also noticed that we were using a read-only directory, as we could see in the failure of the command ```mkdir boas```.

* Then, we wanted to see the content of the last_log file, in order to see the what happened in previous log that the script was run, and curiously, we noticed that we obtained the file:

![](https://i.imgur.com/nD4ptTy.png)

* This meant that, fortunately, we were at the right time when the script was run and generated the flag.


## Apply for flag II

* This challenge was quite similar to the CTF "Apply for Flag" of XSS Lab.

* Firstly, we started to analyse the main page of the website. We noticed that there was an ID for the request that is generated periodically and an HTML input form.

![](https://i.imgur.com/6eST2JF.png)

* We tried to inject HTML code on the form and we noticed that appeared in the next page:

![](https://i.imgur.com/OJt7Vgh.png)

* So, we could conclude that we could inject malicious scripts to obtain the flag. However, we needed more information at the time. So, we started to explore the page that was redirected by te "page" link:

![](https://i.imgur.com/J0k0ErE.png)

* We noticed that we have access to the admin page, where it evaluates the justification decides if gives the flag or not.

![](https://i.imgur.com/4SoICIg.png)

* We started to inspect the HTML code of the website, and we noticed that the buttons were disable. We also notice that it redirects to the API that deals with the request, using the generated ID:

![](https://i.imgur.com/IlG8we1.png)

* So, we just needed to inject a script that generates the form, but will be enabled and also simulate the click, as we did in the "Apply for Flag CTF".

* This is our malicious script:

> Script
```javascript
<form method="POST" action="http://ctf-fsi.fe.up.pt:5005/request/<id-of-the-request>/approve" role="form">     
    <div class="submit">         
        <input type="submit" id="giveflag" value="Give the flag">    
    </div> 
</form>  

<script type="text/javascript">     
    document.querySelector('#giveflag').click(); 
</script>
```

* We injected the this code, and put the ID of the request given in the main page and click on the submit button:

![](https://i.imgur.com/CamVO4a.png)

* We've also had to disable the javascript extensions in order to complete the attack:

![](https://i.imgur.com/5jxNQp7.png)


* Now, we just needed to refresh the page to obtain the flag:

![](https://i.imgur.com/TkJ1QSx.png)

![](https://i.imgur.com/c1UmdPM.png)

