# Geolocation

## Methodology

### #1. Our eyes

Before we can apply a tool or a methodology for finding the location of an image, we should use our eyes to scan the image for important information. Extracting key data points from the image will allow you to apply the right tool, craft a good Google search or identify which part of the world the image might have been taken in.

There are 5 elements of IMINT that you should consider when looking at an image, according to Geoint expert Benjamin Strick:

* Context: In real-world cases, you usually have a context in which the image was produced or shared, usually called context clues.
* Foreground
* Background
* Map markings
* Trial and error

These are some of the questions we should be asking ourselves:

* Are there any obvious data in the image that reveals the location, like a street name or storefront signs?
* Can you determine the country or region of the image by, for instance, which side of the road they drive on, language or architectural characteristics that may reveal a country or continent/region?
* Do you recognize road sign styles, nature and environmental characteristics, or popular motor vehicle brands or vehicle types?
* What is the quality of any visible infrastructure like? Is the road paved or do you see gravel roads?
* Do you see any unique landmarks, buildings, bridges, statues or mountains that can help you geolocate the image?

### #2. Google

If you see anything in the image that can be extracted into a keyword, phrase, a company name, telephone number or any other question you may have as a result of scanning the image up and down: GOOGLE IT!

Do you see anything in the image that can be used in a search query or help you narrow down the possible location? Use "[Google Dorking](../../../pentesting-web/web-reconnaissance/google-dorking.md)" to get an answer.

When geolocating a picture finding the exact location is key, but we may need to answer other questions about the location or the image as well, usually referred to as context questions.

If we find an image of a restaurant (from the front) that has multiple branches throughout a country, we can search for it by Lens or Google Maps, one by one.

### #3. Reverse image search (RIS)

Today's search engines not only allow keyword-based queries, but all search engines (such as Google) also have a reverse image search (RIS) function.

We can use Google Lens or Yandex.

{% embed url="https://yandex.eu/images" %}

The method for doing this with Google (Lens) is as follows:

* If we have a local image (on our computer), we open a browser and type file:// followed by the absolute path of the image. Example:

![file:///C:/Users/\<USER>/Desktop/background1.jpg](../../../.gitbook/assets/file\_image\_browser.png)

* Now, as we have the image in the browser (regardless if it is local or online), right click and select the option "Search image with Lens".

![Lens also allows us to crop an image to our liking, to search for the part we are most interested in.](../../../.gitbook/assets/lens\_example.jpg)
