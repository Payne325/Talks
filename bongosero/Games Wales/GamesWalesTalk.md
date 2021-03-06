# Rust, Face Detection and a pair of Bongos
A run through of lessons learned developing a game prototype with alternative input devices, programmed with the rust programming language.

## Who am I

I'm Alex, for those of you that don't know me, by day I'm Software Engineer at Esri and my background is in the Geographic, Engineering and Game development software.

Can find me on 
- Twitter: @Payne325
- LinkedIn: Alexander Payne
- GitHub: Payne325
- Discord : PaynoInsano

Today I'm going to be talking about a prototype game I developed in my spare time that uses face detection, the rust programming language, and this Gamecube Donkey Konga bongo controller.   

## Why?

So you've seen the title, you know I set out to make a game with face detection and bongos. First question you might be asking yourself is why? I had a few different things motivating me.

Firstly, I like making small game prototypes. I don't really have time to work on a larger scale project, I guess you could say I sit more in the hobbyist camp nowadays. So when I do develop something, I want to be a bit creative with it. 

I also wanted to get more experience with the Rust programming language. I'd already spent a bit of time learning about it and I'd reached a point where I believed it was necessary to pick a project where the key aim wasn't necessarily to learn Rust and see what I picked up along the way

I wanted to experiment with Face detection because I think lots of game developers have ignored it for a while. After devices like the eye toy and the kinect came out, pricey peripherals that didn't offer hugely innovative gameplay, I think some appetite was lost. Well, now I think there's a lot of potential for camera based games to take off, now that most people have a phone in there pocket with a front facing camera that can play games. That potential market has increased in size, on a platform where games with a much quicker/shorter gameplay time are the norm, games are generally cheaper and there's no need to buy additional hardware.

And finally, I wanted to use the Donkey Kong Bongos because I own them, they've just sat on a shelf for the last fifteen years and its undeniably funny. On a more seriousness point, they're much more tactile than a traditional controller, a keyboard or a phone screen and I thought it would be interesting to see if players become any more or less engaged when playing the game.

So to capture all this, I decided to make a "Space Invaders" style game that used head tracking to move the player character left and right and used the bongos to fire the gun. For the purposes of the prototype I just wanted the core mechanisms to work.

## Rust

The Rust language is very fast and memory efficient, often scoring similar or better than C++ in benchmarks, and its designed to ensure memory and thread safety at compile time. Anyone who's written C++ code involving pointers will understand how powerful that is. Essentially you've got a language promising speeds comparable to C++, but with a compiler that guarantees you won't hit any null reference exceptions during runtime. The side effect to this is that the compiler ends up being very strict, but the error messages are much more helpful/descriptive.

Rust is quite a young language and the jury is still out as to whether it'll be widely adopted by the programming community (although there's lots of things to suggest that it will). This means there isn't a plethora of software libraries available for Rust in the same way there is for C++/C#/Python or whatever language you like to develop with. 
So you may end up having to write some libraries yourself that you'd usually import, you may end up trying to force a half-developed library to work for you, or you may have the halfway house solution of using a rust library which is essentially an interface to a library in some other language. (But of course, by doing that you lose that speed and memory safety guarantee).

I ultimately had to use such a binding to use a computer vision library called opencv to handle the face detection. But there was a rust library already available to handle the bongos. Developers have their priorities right!

I'm not going to dive any deeper into the Rust programming aspect of this as I realise the audience aren't all programmers, but for those of you who are interested in learning more about the Rust language, I actually gave a talk about this project for the RustCPlusPlus Cardiff group a while back. We're an informal group of Rust/CPP developers that meet roughly every month, and I'd encourage anyone -at any skill level with these languages- who's interested to come along to one of our -now online- meetups (Meetups.com) or to check out our Youtube channel for previous recordings. 

## HOG Face Detection
So as far as face detection goes, I had to do a bit of reading around on the subject and found myself with three possible approaches to consider.

- Detect Facial landmarks using HOGs (Histogram of Oriented Gradients)
   - The idea is that feature descriptors can be extracted from an image, and these descriptors are derived from the distribution in intensity gradients within the image.
   - The intensity gradient being a directional change in colour intensity for a neighbourhood of pixels in an image. A descriptor is a concatenation of these.
   - Descriptors are passed through some machine learned model to find the key facial landmarks (using H.O.Gs is fairly agnostic with regards to the model type)
   - I believe that Support-Vector Machines were the first model used when H.O.G. facial detection although I've lost the link to wherever I read that, so citation needed. 

   - This functionality has been available for many years from several libraries and is known to work well.
   - This method can produce information to generate exact position of face in image and its pose (orientation) information about an object
   - Computationaly expensive at least for a computer game

## Haar-Feature classifier Face Detection
   - A predefined kernel/small matrix called a Haar-feature is applied across an image to detect the likelihood that some facial feature is within a particular subsection/window of the image.
   - Haar-features are used to detect lines and edges where a face would contrast locally between a darker and lighter areas. E.g. eyes vs bridge of the nose.
   - A Haar-feature classifier is a program constructed to group Haar-features into different stages and apply each stage to a window. If a stage fails, then the window is no longer considered for further evaluation. A window that passes all stages contains a facial landmark of some sort.

   - OpenCV provides Haar-cascade Detection via its interface and even provides a weights file within its release files.
   - Would provide exact positional information.
   - This is very fast, but from my reading it only works well for faces facing directly front.
    - Not ideal for a real time application (game)

## Neural Network Face Detection
   - A neural network is a network of layers which uses weights to manipulate some given data. 
   - When training a Neural Net, the result (in our case a classification) is determined as right or wrong, and that information is fed back into the model to adjust the weights at each layer.
   - For face detection, we can feed in the pixel data of an image and have the model return an area supposedly containing the face.

   - OpenCV has a Deep Neural Network (DNN) Module, with an interface for loading a caffe model and weights.
    - It also hides the necessary model and weights for face detection in the source, which is now widely known online.
    - If we didnt have this I'd have to generate my own model with my own dataset.
      - would introduce all kinds of extra concerns regarding diversity in face types, positions, orientations, image quality etc
   - This approach is very fast and doesn't require the user to be directly front facing. 
   - This approach avoids the need to determine/define any facial landmark descriptors, but we can only obtain a bounding box around the face instead of precise detection of the face
      - Acceptable for our purposes   

- Picked NN because
  - had the model and weights to hand
  - believed it would be more performant when running (best option for a game)
  - didn't need exact position of facial features, just rough location on screen

## Bongos
This might end up being the shortest section of the talk.

The Bongos are a genuine Nintendo Donkey Konga Bongo Controller from circa 2004. I also have an official Gamecube to USB adapter that I got with Smash Bros for Wii U. (Yes I realise this outs me as someone who actually bought a Wii U).

There's lots of documentation and support for the adapter online, particularly among the Dolphin emulator and Project M communities. If you're using a Unix based operating system, you just need to plug in the Bongos and poll the device as you would with any other USB device. I'm no hardware programmer though, and found a rust library to talk to the adapter device within 5 minutes. I guess nerds love the Gamecube.
For Windows, you need the extra step of installing a driver. There's no official one available online, but the dolphin emulator team provide a custom written one for installation. And it works excellently.

The buttons on the bongo correspond to buttons on a regular controller, the only thing that doesn't seem to work is the clap sensor. I imagine the adapter wire is missing a data line or something, but I wasn't able to confirm this.
It's a bit of a shame because I originally thought having a secondary attack triggered through clapping would've been very fun.

## Implementation - How does it work
- Runs in a loop
  - Obtaining an image/frame from the camera
  - Image is passed to the neural network
  - Neural network returns a bounding box, a set of points representing the area of the image that contains a face.
  - The game then transform that box from image space to game space 
    - For non mathematicians, your camera may grab the image in one size, say 400x400 pixels, but if your game is running in 800x600 then the bounding box will never be able to lie on the furthest edges of the game screen. So this transformation handles that by working out how much bigger the game space is from the image space, and then it applies that scale to the bounding box.
  - Once we have our scaled bounding box, the game uses the x component of the centre point of the box and sets the player character's x to match. (Remember, we're only moving along one axis, like in space invaders).

## How did it go?
Well it certainly works in the back room in my house. I've had a lot of fun playing it when testing... until I whacked my knee.
I did discover that I hadn't considered a few things:

- Distance of player to camera (this only really occurred to me when preparing to demonstrate this here)
  - If you move further away from the camera you end up being able to get more precise movement.
  - You can also reach the edges much easier due to the way the bounding box centre is used, the smaller the box, the easier the centre can reach the sides of the screen.
- If I also wanted to take this further, I'd need to think about setting up some sort of calibration routine to make sure the movement of the character is consistent for different screen size ratios and people sit at different distances away from the camera.

## Lessons Learned
Lessons learned about Rust:
- Its fast
- Compiler guarantees memory safety
- Community is young, not lots of libraries to use

Lessons learned about face detection for games:
- Multiple means of detecting faces with various trade offs in accuracy and speed
- Neural network approach excellent for using face to move a character in 2D
  - But without using a premade model, gathering relevant data could be potentially difficult 

Lessons learned about Bongos:
- They're fun to slap
- Software developers like to write programs that use them fairly early on in a programming language's life.
- Official adapter *probably* doesn't support the clap sensor.  
  
Lessons learned about camera based gameplay:
- Calibration is key to having a polished product
- For fast paced gameplay, you need space.
- Its relatively straightforward to add the controls to a game.

## Is it actually fun?

I think so, but I need to do some wider testing. Very few people have a Bongo controller though... that's where you come in.
 *Get someone to try the game*

## References
Thanks for watching/listening/reading.
I've uploaded these slides and some notes to my GitHub, where you can also find the f-trak and bongosero projects as well. 
The notes also contain references to the various articles I read and borrowed images from for this presentation.

https://github.com/not-yet-awesome-rust/not-yet-awesome-rust#computer-vision
https://towardsdatascience.com/whats-the-difference-between-haar-feature-classifiers-and-convolutional-neural-networks-ce6828343aeb
https://docs.opencv.org/3.4/db/d28/tutorial_cascade_classifier.html
https://www.analyticsvidhya.com/blog/2021/07/facial-landmark-detection-simplified-with-opencv/
https://learnopencv.com/facial-landmark-detection/
https://towardsdatascience.com/face-detection-models-which-to-use-and-why-d263e82c302c
https://en.wikipedia.org/wiki/Haar-like_feature
https://en.wikipedia.org/wiki/Histogram_of_oriented_gradients
https://www.eeweb.com/real-time-face-detection-and-recognition-with-svm-and-hog-features/
https://en.wikipedia.org/wiki/Support-vector_machine
https://medium.com/@goutam0157/haar-cascade-classifier-vs-histogram-of-oriented-gradients-hog-6f4373ca239b
https://en.wikipedia.org/wiki/Neural_network