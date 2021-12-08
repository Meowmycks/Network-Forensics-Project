# Network-Forensics-Project

## Objective
> We have found a secret website where the famous hacker group called 'Inc08itos' keeps their members' data. The problem is, we don't know how to access the page. However, the DoD was able to sniff out some interesting packet captures that might give you some clues. One thing that we know about Inc08itos is that every member has a unique codename based on famous malware.

As soon as we try to access the site, we are told ```You do not have the correct key to access this page```.

![sharkbite-1](https://user-images.githubusercontent.com/45502375/145147627-d312a648-8ab5-4ba0-b285-da51246c8154.png)

So let's get the correct key.


## Getting the Key

We were given two packet captures that have information on it we can possibly use. Let's take a look at the first one.

### Conversation 1:

- There's a *lot* of data to sift through. Fortunately, we don't have to. We'll filter for HTTP traffic and see what we find.
- There's a file called ```pages``` and some cookie data we can steal labeled ```PHPSESSID``` and ```_ga```. These cookies will be useful later.

![sharkbite-2](https://user-images.githubusercontent.com/45502375/145147645-5dd3ab7d-5709-433d-9a81-c8349bb0fb85.gif)

- The file we downloaded contains a long base64 encoded string. Decoding it reveals an RSA private key we can use.
- We save the key we found into a file with the ```.pfx``` extension. This allows us to use it in Wireshark as an RSA key.

![sharkbite-3](https://user-images.githubusercontent.com/45502375/145147656-9527763f-6925-49d9-a8f7-1da59e320836.png)
![sharkbite-4](https://user-images.githubusercontent.com/45502375/145147662-b3943ded-b688-4892-9843-624b4ed7c4f2.png)

### Conversation 2:

- Now that we have an RSA private key, we can decrypt the TLS traffic found in the second packet capture.
- To do that, we go to the TLS protocol settings in Wireshark and add the key file ```sharkbite.pfx``` to the list of RSA keys.
- Upon successful decryption, we find that another "key" has been revealed.
- Combining the cookie data we found with the key we found, we are now able to access the website.

![sharkbite-5](https://user-images.githubusercontent.com/45502375/145147860-4c5e42df-edcd-4e54-bf04-453dcd987f00.gif)


## Gaining Access

All we really need to do to access the website is put all that stuff we found into our own cookies.

- Open Inspect Element
- Go to the ```Storage``` tab and create three cookies.
- Rename the three of them to ```PHPSESSID```, ```_ga```, and ```key```, and change their values to the ones we found.
- Upon refreshing the page, we're greeted like one of their own.

![sharkbite-6](https://user-images.githubusercontent.com/45502375/145148232-ea87cd25-3147-4444-902d-4d7b2266326c.gif)


## The Last Mile

Now that we have access to the website, we have to find the data of one of Inc08itos' members. But we're told that there are ```No registered members...```. Strange.

Let's take a closer look at the request we're sending to the site.

- Using Burp Suite, we can see that there's a new parameter being included in the request called ```codename```
- From what we know, these members use the names of famous malware as their aliases.
- Using [this](https://attacksimulator.com/blog/10-famous-malware-examples-in-history/) website, we can guess that the person we're looking for is named "cryptolocker" (after guessing a bunch of other names, of course :) ).
- Therefore, we will send the request to the Repeater and assign ```codename``` the value "cryptolocker" and resend the request.

![sharkbite-8](https://user-images.githubusercontent.com/45502375/145148799-d2aa51b7-625c-4407-8dc5-e0a431bdad08.gif)

Done.
