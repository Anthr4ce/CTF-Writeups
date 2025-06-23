# Operation Silent Hunt

Description of the challenge:

During the theft of a hard drive containing sensitive data, the attacker made a mistake. They lost their phone at the scene.  
Your mission: follow the clues and locate the exact address where the hard drive is hidden.  

But be careful... getting caught could compromise the entire operation.  

It is necessary to conduct online research only on the following websites:

- [https://www.google.com/](https://www.google.com/)
- [https://www.youtube.com/](https://www.youtube.com/)
- [https://fr.pinterest.com/](https://fr.pinterest.com/)
- [https://www.google.com/maps](https://www.google.com/maps)
- [https://x.com/](https://x.com/)
- [https://www.instagram.com/](https://www.instagram.com/)
- [https://github.com/](https://github.com/)
- [https://www.reddit.com/](https://www.reddit.com/)
- [https://linkedin.com/](https://linkedin.com/)

  
Virtual Phone : https://message-app.midnightflag.fr/  
  
Format : **MCTF{housenumber_street_city} (all lowercase)**  
Exemple: **MCTF{66_rue_des_fleurs_paris}**  

We are given a virtual phone and to unlock it we need a code.
![alt text](images/image.png)
![alt text](images/image2.png)

As we can see in the first image, there are some informations about an instagram account.
![alt text](images/image3.png)
![alt text](images/image4.png)

By inspecting closely the photos, we notice some patters that could be used as code to unlock the phone.
![alt text](images/image5.png)
![alt text](images/image6.png)

Since he seemed pretty attached to his cat, i noticed this date.
![alt text](images/image7.png)

I tried this as code
```text
020425
```
And we cna finally unlock the phone!

As soon as we unlock it, we have this conversation
![alt text](images/image8.png)

I then tried to gather some info and i tried every possible input and when we select "I lost the website link", we get a reply with a website
![alt text](images/image9.png)

And while asking the credentials we get asked a question about the name of the cat
![alt text](images/image10.png)

Luckily we know the name thanks to the photos that our "target" shared with Eloan FRANK, we know the name of the cat
![alt text](images/image11.png)
![alt text](images/image12.png)

We then get to the website where the hard drive was sold and we log in with the credentials.
![alt text](images/image13.png)

While checking the the websitei looked at the json that supposedly charged the data and found the nickname of the buyer
```js
}]
          , m = [{
            id: 5,
            title: "External Hard Drive",
            description: "2TB external hard drive",
            price: 2e5,
            seller: "tech_seller",
            buyer: "Darkythedark42",
            deliveryDate: "22/06/2025 at 16:00",
            image: "/External_Hard_Drive.webp"
        }, {
```

And i got a X account.
https://x.com/Darkythedark42

Checking the account, we see only 1 post 
![alt text](images/image14.png)

Trynna figure out where the location would it  be i noticed the first letters of the place
![alt text](images/image15.png)

I tried to google reverse image search and i found a result kinda sus
![alt text](images/image16.png)

We are competing in Rennes, so it wouldn't be too shocking if the place we are looking for would be here, no?
![alt text](images/image17.png)

Finally we found it. So now we just have to get the route number and name and we have the flag.

```text
MCTF{48_rue_de_saint-brieuc_rennes}
```
