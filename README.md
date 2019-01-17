# class Painting
## Description
This class works with a P5 canvas to paint an image. The way it does this is by having 100s of objects that move about in circles over the canvas and they draw lines based up on the colour of the image at that particular point. This movement combined with the large number of objects creates a cool graphic. This class describes a single painting which is given default parameters to start off with, such as the image which it is painting and also the number of objects that move across the canvas.
## Attributes
### imageName
This attribute is a string that is used to store the URL link to the image used in the painting.
```javascript
painting1.imageName = "https://www.openprocessing.org/sketch/392202/files/portrait2.jpg"
```
### numOfCircles
This is an integer that is used to store the number of objects or 'circles' that would traverse across the canvas painting.
```javascript
painting1.numOfCircles = random(150,350)
```
### backgroundColour
This is an array of integers in the format [R,G,B] that represents a colour that the background should be. It is used to feed into the background() function in p5.
```javascript
painting1.backgroundColour = [231, 178, 129]
background(painting1.backgroundColour)
```
### randomStartingPosition
This is a boolean that is used to dictate whether the circles start from the middle or a random place on the canvas. 
```javascript
painting1.randomStartingPosition = True
```
### greyscaleEnabled
Another boolean that is used to dictate whether the image is painted in greyscale or full colour
### image
This is of type P5.Image() and is used to hold the image that is being painted. 
```javascript
painting1.image = loadImage("example.jpg")
```
### notLoaded
This is a boolean attribute that is used to record whether the image has loaded when using the loadImage() function 
### circles
This is an important attribute. It is an array that stores every circle object. Therefore, it has length numOfCircles. Each element in this array is an object with its own properties such as `pos`, `previousPos`, `dir`, `radius` and `angle`.
### lightLevel
This is a float that is used for the movement of the objects. It is used to create randomness in the circles' movement

## Methods
### constructor()
```javascript
 // constructor
	constructor(gImage,gNumOfCircles, gBackgroundColour, gRandomStartingPosition, gGreyscaleEnabled ){

		// sets starting values at initialisation with alternate defaults 
		this.imageName = gImage || "https://www.openprocessing.org/sketch/392202/files/portrait1.jpg";
		this.numOfCircles = gNumOfCircles || 100;
		this.backgroundColour = gBackgroundColour || 80;
		this.randomStartingPosition = gRandomStartingPosition || false;
		this.greyscaleEnabled = gGreyscaleEnabled || false;
		
      // loads the image 
		this.image = loadImage(this.imageName, function(loadedImage){
			return loadedImage
		});
		
		this.notLoaded = true;
	}
```
As you can see, the constructor takes some parameters that are given (denoted by the prefix g) during the declaration of an object with the `Painting` class. It sets the appropriate attributes to the given values and if no values are given, there are default values which will be used instead. After this, the image is loaded using the `loadImage()` function combined with the attribute `imageName`. The result is stored in the `image` attribute. The attribute `notLoaded` is also set to `true`. 

### draw()
The draw method can be thought of as two parts. The first part accomodates for the time before the image loads and also just straight after it. The first part can be seen below
``` javascript
      // if image hasn't loaded yet		
  		if(this.image.width < 10){
			// creates loading screen
			
			background(30);
			if(frameCount%2) text("loading", 300, 300);
				this.notLoaded = true;
				return;

		}

		// the moment that the image loads
		if (this.notLoaded){
			// creates canvas
			createCanvas(this.image.width, this.image.height);
			background(this.backgroundColour)
			this.notLoaded = false;

			// creates all the circles that are used to paint the picture
			this.circles = [];		
			for(var i = 0; i < this.numOfCircles; i++){
				if (this.randomStartingPosition == true){
					this.startingX = random(1, 600)
					this.startingY = random(1, 600)
				
			    }else{
					this.startingX = this.image.width/2
					this.startingY = this.image.height/2
				}

				this.circles[i] = {};
				this.circles[i].prevPos = {x: this.startingX, y: this.startingY}
				this.circles[i].pos = {x: this.startingX, y: this.startingY}
				this.circles[i].dir = random() > 0.5 ? 1 : -1
				this.circles[i].radius = random(3, 10)
				this.circles[i].angle = 0
			}
			return
		}

```
As you can see, the first if statement is run if image is not loaded yet. It displays a loading screen and has the return keyword. This means that the `draw()` function is started again without going to the rest of the code. This means that the minute the if statement is made false (aka when the image loads) it runs the if statement directly after it. This is where the canvas is created using the `createCanvas()` method from p5. The size of it matches the image. This means the sketch can now accomodate for pictures of any size, not just set sizes. The background colour is also set and the attribute `notLoaded` is set to false so that this if statement doesn't run again. 

Finally, the script iterates through the array `circles` populating each object with a starting value for all its properties (outlined above in the `circles` description)

The second part is where the objects are seen to move across the canvas:
``` javascript
// algorithm for circles to move and draw
		var lightLevel = (random()*70);
		for(var i = 0; i < this.numOfCircles; i++){
			
			this.circle = this.circles[i]
			this.circle.angle += 1/this.circle.radius*this.circle.dir
			this.circle.pos.x += sin(this.circle.angle) * this.circle.radius
        	this.circle.pos.y += cos(this.circle.angle) * this.circle.radius
        	
        	if( brightness(this.image.get(round(this.circle.pos.x), round(this.circle.pos.y))) > lightLevel || this.circle.pos.x < 0 || this.circle.pos.x > this.image.width || this.circle.pos.y < 0 || this.circle.pos.y > this.image.height){
				this.circle.dir *= -1;
				this.circle.radius = random(3, 10);
				this.circle.angle += PI;
			 }
			
			// colour is selected and line is created 
			var colours = this.image.get(this.circle.pos.x, this.circle.pos.y)
			
			
			if (this.greyscaleEnabled){
				
				var colourVal = (colours[0] + colours[1] + colours[2])/3
				stroke([colourVal,colourVal,colourVal,255])
			}else{
				stroke(colours);
			}
			
			line(this.circle.prevPos.x, this.circle.prevPos.y, this.circle.pos.x, this.circle.pos.y);

			this.circle.prevPos.x = this.circle.pos.x
			this.circle.prevPos.y = this.circle.pos.y
		}
```
As you can see, the attribute `lightLevel` is declared using the random() function to make the sketch look more unpredictable. The script then iterates back through the `circles` array and uses some maths involving the angle, radius and direction to work out what the next position of that object is. Once it has done this, it records the colour of the pixel at that position on the image being painted. The `stroke()` function is used to change the colour of pen to the colour the object has just worked out. It then draws a line of that colour from the point where it was (`prevPos`) to where it is now (`pos`). NOTE: The script checks whether `greyscaleEnabled` is true and if so, manipulates the RGB values so that a greyscale representative of the colour is used. 


### getImage()
This is a simmple getter method that returns the P5.image that is currently being painted.

### getImageName()
This is another getter than returns the string containing the URL to the image being painted.

### getGreyscaleBoolean()
This is also another getter which retrieves the value of the `greyscaleEnabled` attribute.

### getRandomStartBoolean()
This getter returns the boolean value of `randomStartingPosition`

### setCircleNum()
This is a setter that changes the value of the `numOfCircles` attribute after checking some certain rules

### setImageName()
This setter changes the `imageName` attribute which allows you to change the photo being displayed

### setGreyscale()
This setter changes the value of the `greyscale` attribute

### refreshImage()
This method is very important as it is what you use if you want to restart the painting without having to declare another object. What it does is it gets rid of the canvas, changes the value of attributes to whatever has been given in the arguments and reloads the image so the whole process is started again as if someone had run the constructor again. s

# Example Webpage
The example webpage showcases the parameterisation of the class. It shows how the class can easily handle any image it wants as opposed to just the one that was in the original source. Furthermore, it shows extension of scope as I have added features such as being able to make it greyscale or even have the ability to make it start from random points on the canvas. I used Bootstrap to make it more visually appealing. 

# Original Code
The original sketch that I adapted is this:
https://www.openprocessing.org/sketch/624879

It is covered under the Creative Commons Attribution-ShareAlike 3.0 Unported (CC BY-SA 3.0) License which can be found here:
https://creativecommons.org/licenses/by-sa/3.0/legalcode.txt

