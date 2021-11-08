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
Before I begin, I do need to do a bit of plugging. I'm not going to dive too heavily into the programming side of things today, but if you are interested in learning more about the use of the Rust language in this game, I actually gave a talk about this for the RustCPlusPlus Cardiff group a while back. We're an informal group of Rust/CPP developers that meet roughly every month, and I'd encourage anyone -at any skill level with these languages- who's interested to come along to one of our -now online- meetups (Meetups.com) or to check out our youtube channel for previous recordings.    

## Why?

So you've seen the title, you know I set out to make a game with face detection and bongos. First question you might be asking yourself is why? I had a few different things motivating me.

Firstly, I like making small game prototypes. I don't really have time to work on a larger scale project, I guess you could say I sit more in the hobbyist camp nowadays. So when I do develop something, I want to be a bit creative with it. 

I also wanted to get more experience with the Rust programming language. I'd already spent a bit of time learning about it and I'd reached a point where I believed it was necessary to pick a project where the key aim wasn't necessarily to learn Rust and see what I picked up along the way

I wanted to experiment with Face detection because I think lots of game developers have ignored it for a while. After devices like the eye toy and the kinect came out, pricey peripherals that didn't offer hugely innovative gameplay, I think some appetite was lost. Well, now I think there's a lot of potential for camera based games to take off, now that most people have a phone in there pocket with a front facing camera that can play games. That potential market has increased in size, on a platform where games with a much quicker/shorter gameplay time are the norm, games are generally cheaper and there's no need to buy additional hardware.

And finally, I wanted to use the Donkey Kong Bongos because I own them, they've just sat on a shelf for the last fifteen years and its undeniably funny.

So to capture all this, I decided to make a "Space Invaders" style game that used head tracking to move the player character left and right and used the bongos to fire the gun. For the purposes of the prototype I just wanted the core mechanisms to work.

## Rust

So I said I'm not going to dive too deeply into the programming aspect of this, but I'd like to start with a brief overview of why Rust is worth considering for game development.

The Rust language is very fast and memory efficient, often scoring similar or better than C++ in benchmarks, and its designed to ensure memory and thread safety at compile time. Anyone who's written C++ code involving pointers will understand how powerful that is. Essentially you've got a language promising speeds comparable to C++, but with a compiler that guarantees you won't hit any null reference exceptions during runtime. The side effect to this is that the compiler ends up being very strict, but the error messages are much more helpful/descriptive.

Rust is quite a young language and the jury is still out as to whether it'll be widely adopted by the programming community (although there's lots of things to suggest that it will). This means there isn't a plethora of software libraries available for Rust in the same way there is for C++/C#/Python or whatever language you like to develop with. 
So you may end up having to write some libraries yourself that you'd usually import, you may end up trying to force a half-developed library to work for you, or you may have the halfway house solution of using a rust library which is essentially an interface to a library in some other language. (But of course, by doing that you lose that speed and memory safety guarantee).

I ultimately had to use such a binding to use a computer vision library called opencv to handle the face detection. But there was a rust library already available to handle the bongos. Developers have their priorities right!

## Means of Face detection

- Detect Facial landmarks using HOGs (Histogram of Oriented Gradients)
   - The idea is that feature descriptors can be extracted from an image, and these descriptors are derived from the distribution in intensity gradients within the image.
   - The intensity gradient being a directional change in colour intensity for a neighbourhood of pixels in an image. A descriptor is a concatenation of these.
   - Descriptors are passed through some machine learned model to find the key facial landmarks (using H.O.Gs is fairly agnostic with regards to the model type)
   - I believe that Support-Vector Machines were the first model used when H.O.G. facial detection although I've lost the link to wherever I read that, so citation needed. 

   - Dlib has provided this functionality for many years and is known to work
   - Unofficial rust libraries exist, but this just provides bindings, user has to install dlib
   - I had just come through the opencv set up at this point, so was eager to find a solution that didn't involve more libraries.

- Describe Haar-Feature classifier approach
   - I discovered this after implementing the DNN approach, so did not consider this at the time.
   - A predefined kernel/small matrix called a Haar-feature is applied across an image to detect the likelihood that some facial feature is within a particular subsection/window of the image.
   - Haar-features are used to detect lines and edges where a face would contrast locally between a darker and lighter areas. E.g. eyes vs bridge of the nose.
   - A Haar-feature classifier is a program constructed to group Haar-features into different stages and apply each stage to a window. If a stage fails, then the window is no longer considered for further evaluation. A window that passes all stages contains a facial landmark of some sort.

   - This is very fast, but from my reading it only works well for faces facing directly front.
   - OpenCV provides Haar-cascade Detection via its interface and even provides a weights file within its release files.
   - If I revisit this project, I'd like to experiment with a Harr-feature based implementation of the face tracker. 

- Describe DNN approach
   - I'll admit to some bias for this approach, I had used it before with Python when first experimenting with OpenCV a few years ago
   - For those that don't know, a neural network is a network of layers which uses weights to manipulate some given data. 
   - When training a Neural Net, the result (in our case a classification) is determined as right or wrong, and that information is fed back into the model to adjust the weights at each layer.
   - For face detection, we can feed in the pixel data of an image and have the model return an area supposedly containing the face.

   - OpenCV has a Deep Neural Network (DNN) Module, with an interface for loading a caffe model and weights.
   - It also hides the necessary model and weights for face detection in the source, which is now widely known online.
   - This approach is very fast and doesn't require the user to be directly front facing. 
   - This approach avoids the need to determine/define any facial landmark descriptors, but we can only obtain a bounding box around the face instead of precise detection of the face
      - Acceptable for our purposes   

- Picked NN because
  - prior experience
  - had the model and weights to hand
  - believed it would be more performant when running (best option for a game).

## Implementation - How does it work
- Runs in a loop
  - Obtaining an image/frame from the camera
  - Image is passed to the neural network
  - Neural network returns a bounding box, a set of points representing the area of the image that contains a face.
  - The game then transform that box from image space to game space 
    - For non mathematicians, your camera may grab the image in one size, say 400x400 pixels, but if your game is running in 800x600 then the bounding box will never be able to lie on the furthest edges of the game screen. So this transformation handles that by working out how much bigger the game space is from the image space, and then it applies that scale to the bounding box.
  - Once we have our scaled bounding box, the game uses the x component of the centre point of the box and sets the player character's x to match. (Remember, we're only moving along one axis, like in space invaders).

## How did it go?
Well it certainly works in the back room in my house. I've had a lot of fun playing it when testing.
I didn't consider a few things:

- Distance of player to camera (this only really occurred to me when preparing to demonstrate this here)
- 

## Lessons Learned

Lessons learned about Rust:
- Its fast
- Compiler guarantees memory safety
- Community is young, not lots of libraries to use

Lessons learned about face detection for games:
-

Lessons learned about Bongos:
- They're fun to slap
- Software developers like to write programs that use them fairly early on in a programming language's life.


## I need a volunteer
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