NMMU Game Development Android (Carbon Avengers)

HEY GUYS!!

Please take the time to read this article by the next game.dev meeting so we can get this going :)



Android Game Development with libgdx – Prototype in a day, Part 1a

In this article I will take a detour from the building blocks of a game engine and components and I will demonstrate how to prototype a game quickly using the libgdx library.

What you will learn:

> Create a very simple 2D shooter platformer game. What a complete game architecture looks like. How to use 2D Graphics with OpenGL without knowing anything about OpenGL. What different entities make up a game and how they are tied together in a game world. How to add sound to your game. How to build your game on the desktop and deploy in onto Android – yes, it’s that magic.

Steps to create a game

> Have an idea for a game.

> 2. Draft up some scenarios on paper to resemble your vision and how will it look like. 3. Analyse the idea, iterate over a few versions by tweaking it and decide what the game will have in its initial version. 4. Pick a technology and start prototyping. 5. Start coding and creating the assets for the game. 6. Play-test, improve, and continuously make small steps towards finishing it. 7. Polish and release!

The Game Idea

Because this will be a one day project, there is very limited time at disposal and the goal is to learn the technology to make games, not the actual process. For this purpose I took the liberty to borrow ideas from other games and focus on the technical aspects of this process. I will be borrowing heavily from a game called Star Guard. It’s a little gem made by Vacuum Flowers. Go get the game and check it out. A very simple shooter platformer with a simplistic style and old school arcade feel. The idea is to guide our hero through the levels by killing enemies and and dodging everything that tries to kill us. The controls are simple, the arrow keys move the hero to the left or right, Z jumps and X shoots the laser. The longer the jump button is held, the higher the hero jumps. He can change direction in the air and also shoot. We’ll see how we can translate these controls to Android later on.

The next steps (2 and 3) can be skipped over as we will already have this taken care of because of the functioning game.

Start your Eclipse

This is where we start. I will be using libgdx library to create the game. Why libgdx? It’s the best library (in my opinion) that makes developing games easy without knowing much about the underlying technology. It allows developers to create their games on the desktop and deploy it to Android without any modification. It offers all the elements to use it in games and hides the complexity of dealing with specific technologies and hardware. It will become more obvious as we go along.

Setting up the project

By following the instructions from libgdx’s documentation we have to first download the library. Go to http://libgdx.badlogicgames.com/nightlies/ and download the libgdx-nightly-latest.zip file and unpack it.

Create a simple java project in eclipse. I will call it star-assault.

Leave the default settings and once the project was created, right click on it and select New->Folder and create a directory named libs. From the unpacked libgdx-nighly-latest directory, copy the gdx.jar file into the newly created libs directory. Also copy the the gdx-sources.jar file into the libs directory. It is in the sources sub-directory of the unpacked gdx directory. You can do this by simply dragging the jar files into your directories in eclipse. If you copy them using explorer, or finder or any other means, don’t forget to refresh your eclipse project after by pressing F5.

The structure should look like the following image:

Add gdx.jar as a dependency to the project. Do this by right-clicking the project title and select Properties. On this screen pick Java Build Path and click onto the Libraries tab. Click Add JARs…, navigate to the libs directory and select gdx.jar, then click OK.

In order to have access to the gdx source code and to be able to debug our game easily, it’s a good idea to add the sources to the gdx.jar file. To do this, expand the gdx.jar node, select Source attachment, click Edit…, then Workspace… and pick gdx-sources.jar then OK until all the pop-up windows are closed.

The complete documentation for setting up projects with libgdx can be found on the official wiki.

This project will be the core project for the game. It will contain the game mechanics, the engine, everything. We will need to create two more projects, basically launchers for the 2 platforms we are targeting. One for Android and one for Desktop. These projects will be extremely simple and will contain only the dependencies required to run the game on the respective platforms. Think of them as the class containing the main method.

Why do we need separate projects for these? Because libgdx hides the complexity of dealing with the underlying operating system (graphics, audio, user input, file i/o, etc.), Each platform has a specific implementation and we will need to include only those implementations (bindings) that are required targeted. Also because the application life-cycle, the asset loading (loading of images, sounds, etc) and other common aspects of an application are heavily simplified, the platform specific implementations reside in different JAR files and only those need to be included that are required for the platform we are targeting.

The Desktop Version

Create a simple java project as in the previous step and name it star-assault-desktop. Also follow the steps to create the libs directory. This time the required jar files from the downloaded zip file are: gdx-natives.jar, gdx-backend-lwjgl.jar, gdx-backend-lwjgl-natives.jar. Also add these jar files as dependencies to the project as in the previous project. (right click the project -> Properties -> Java Build Path -> Libraries -> Add JARs, select the three JARs and click OK.) We also need to add the star-assault project to the dependencies. To do this, click the Projects tab, click Add, check the star-assault project and click OK.

Important! We need to make the star-assault project a transitive dependency, meaning that dependencies for this project to be made dependencies of projects depending on this. To do this: right click on the main project -> Properties -> Java Build Path -> Order and Export -> check the gdx.jar file and click OK.

The Android Version

For this you will need the Android SDK installed. Create a new Android project in eclipse: File -> New -> Project -> Android Project. Name it star-assault-android. For build target, check “Android 2.3?. Specify a package name net.obviam or your own preference. Next to “Create Activity” enter StarAssaultActivity?. Click Finish.

Go to the project directory and create a sub-directory named libs (you can do this from eclipse). From the nightly zip, place gdx-backend-android.jar and the armeabi and armeabi-v7a directories in the newly created libs directory. In eclipse, right click the project -> Properties -> Java Build Path -> Libraries -> Add JARs, select gdx-backend-android.jar and click OK.

Click Add JARs again, select gdx.jar under the main project (star-assault) and click OK. Click the Projects tab, click Add, check the main project and click OK twice. This is how the structure should look like:

Important! For ADT release 17 and newer, the gdx jar files need to be explicitly marked to be exported. To do this Click on the Android Project Select Properties Select Java Build Path (step 1) Select Order and Export (step 2) Check all the references, e.g. the gdx.jar, the gdx-backend-android.jar, the main project etc. (step 3). The following image shows the new state.

Also, more information on this issue here.

Sharing the Assets (images, sounds and other data)

Because the game will be identical on both desktop and Android but each version requires to be built separately from different projects, we want to keep the images, sounds and other data files in a shared location. Ideally this would be in the main project as it is included in both Android and desktop bu because Android has a strict rule on where to keep all these files, we will have to keep the assets there. It is in the automatically created assets directory in the Android project. In eclipse there is the possibility to link directories as in symbolic links on linux/mac or shortcuts in windows. To link the assets directory from the Android project to the desktop project do the following: Right click the star-assault-desktop project -> Properties -> Java Build Path -> Source tab -> Link Source… -> Browse… -> browse to the asssets directory in the star-assault-android project and click Finish. You can also extend the Variables… instead of browsing to the assets directory. It is recommended as it makes the project file system independent.

Also make sure the assets directory is included as a source folder. To do that, right click on the assets directory in eclipse (the desktop project), select Build Path -> Use as Source Folder.

At this stage we are ready with the setup and we can go ahead to work on the game.

Creating the Game

A computer application is a piece of software that runs on a machine. It starts up, does something (even if that’s nothing) and stops in a way or another. A computer game is a specific type of application in which the “does something” part is filled with a game. The start and end is common to all applications. Also the game has a very straight forward architecture based around an continuous loop. You can find out more about the architecture and the loop here and here.

Thanks to libgdx our game can be pieced together as a staged play in a theatre. All you need to do is, to think of the game as a theatrical play. We will define the stages, the actors, their roles and behaviours, but we will delegate the choreography to the player.

So to set up our game/play we need to take the following steps:

> Start application.

> 2. Load up all the images and sounds and store them in memory. 3. Create the stages for our play along with the actors and their behaviours (rules for interactions between them). 4. Hand the control to the player. 5. Create the engine that will manipulate the actors on the stage based on the input received from the controller. 6. Determine when the play ends. 7. End the show.

It looks quite simple and it really is. I will be introducing the notions and elements as they appear.

To create the Game we simply need just one class. Let’s create StarAssault?.java in the star-assault project. Every class will be created in this project with 2 exceptions. 01 package net.obviam.starassault; 02 03 import com.badlogic.gdx.ApplicationListener?; 04 05 public class StarAssault? implements ApplicationListener? { 06 07 @Override 08 public void create() { 09 // TODO Auto-generated method stub 10 } 11 12 @Override 13 public void resize(int width, int height) { 14 // TODO Auto-generated method stub 15 } 16 17 @Override 18 public void render() { 19 // TODO Auto-generated method stub 20 } 21 22 @Override 23 public void pause() { 24 // TODO Auto-generated method stub 25 } 26 27 @Override 28 public void resume() { 29 // TODO Auto-generated method stub 30 } 31 32 @Override 33 public void dispose() { 34 // TODO Auto-generated method stub 35 } 36 }

Just implement ApplicationListener? from gdx and eclipse will generate the stubs for the methods needed to be implemented. These are all the methods that we need to implement from the application lifecycle. It is very simple considering all the setup code needed for Android, or on desktop to initialise the OpenGL context and all those boring (and difficult) tasks.

The method create() is called first. This happens when the application is ready and we can start loading our assets and create the stage and actors. Think of building the stage for the play in a theatre AFTER all the things have been shipped there and prepared. Depending on where the theatre is and how you get there, the logistic can be a nightmare. You can ship things by hand, or by plane or trucks…we don’t know. We’re inside and have the stuff ready and we can start to assemble it. This is what libgdx is doing for us. Shipping our stuff and delivering it regardless of the platform.

The method resize(int width, int height) is called every time the drawable surface is resized. This gives us the chance to rearrange the bits before we go on to start the play. It happens when the window (if the game runs in one) is resized for example.

The heart of every game is the render() method which is nothing more than the infinite loop. This gets called continuously until we decide that the game is over and want to terminate the program. This is the play in progress.

Note: For computers the game over is not equivalent of program over. So it’s just a state. The program is in a state of game over, but is still running.

Of course that the play can be interrupted bu pauses and they can be resumed. The pause() method will be called whenever the application enters into the background on the desktop or Android. When the application comes to the foreground it resumes and the resume() method is being called. When the game is done and the application is being closed, the dispose() is called and this is the time to do some cleanup. It’s similar when the play is over, spectators have left and the stage is being dismantled. No more coming back. More on the lifecycle here.

The Actors

Let’s start taking steps towards the actual game. The first mile-stone is to have a world in which our guy can move. The world is composed of levels and each level is composed of a terrain. The terrain is nothing more than some blocks through which our guy can’t go.

Identifying the actors and entities so far in the game is easy.

We have the guy (let’s call him Bob – libgdx has tutorials with Bob) and the blocks that make up the world.

Having played Star Guard we can see that Bob has a few states. When we don’t touch anything, Bob is idle. He can also move (in both directions) and he can also jump. Also when he’s dead, he can’t do anything. Bob can be in only one of the 4 identified states at any given time. There are other states as well but we’ll leave them out for now.

The states for Bob:

> Idle – when not moving or jumping and is alive Moving – either left or right at a constant speed. Jumping – also facing left or right and high or low. Dead – he’s not even visible and respawning.

The Blocks are the other actors. For simplicity we have just blocks. The level consists of blocks placed in a 2 dimensional space. For simplicity we will use a grid. Turn the start of Star Guard into a block and Bob structure, will look something like this:

The top one is the original and the bottom one is our world representation.

We have imagined the world but we need to work in a measure system that we can make sense of. For simplicity we will say that one block in the world is one unit wide and 1 unit tall. We can use meters to make it even simpler but because Bob is half a unit, it makes him half a meter. Let’s say 4 units in the game world make up 1 meter so Bob will be 2 meters tall.

It is important because when we will calculate the speed with which Bob runs for example, we need to know what we’re doing.

Let’s create the world.

Our main playable character is Bob. The Bob.java class looks like this: 01 package net.obviam.starassault.model; 02 03 import com.badlogic.gdx.math.Rectangle; 04 import com.badlogic.gdx.math.Vector2; 05 06 public class Bob { 07 08 public enum State { 09 IDLE, WALKING, JUMPING, DYING 10 } 11 12 static final float SPEED = 2f; // unit per second 13 static final float JUMP\_VELOCITY = 1f; 14 static final float SIZE = 0.5f; // half a unit 15 16 Vector2 position = new Vector2(); 17 Vector2 acceleration = new Vector2(); 18 Vector2 velocity = new Vector2(); 19 Rectangle bounds = new Rectangle(); 20 State state = State.IDLE; 21 boolean facingLeft = true; 22 23 public Bob(Vector2 position) { 24 this.position = position; 25 this.bounds.height = SIZE; 26 this.bounds.width = SIZE; 27 } 28 }

Lines #16-#21 define the attributes of Bob. The values of these attributes define Bob’s state at any given time. position – Bob’s position in the world. This is expressed in world coordinates (more on this later). acceleration – This will determine the acceleration when Bob jumps. velocity – Will be calculated and used for moving Bob around. bounds – Each element in the game will have a bounding box. This is nothing more than a rectangle, in order to know if Bob ran into a wall, got killed by a bullet or shot an enemy and hit. It will be used for collision detection. Think of playing with cubes. state – the current state of Bob. When we issue the walk action, the state will be WALKING and based on this state, we know what to draw onto the screen. facingLeft – represents Bob’s bearing. Being a simple 2D platformer, we have just 2 facings. Left and right.

Lines #12-#15 define some constants we will use to calculate the speed and positions in the world. These will be tweaked later on.

We also need some blocks to make up the world. The Block.java class looks like this: 01 package net.obviam.starassault.model; 02 03 import com.badlogic.gdx.math.Rectangle; 04 import com.badlogic.gdx.math.Vector2; 05 06 public class Block { 07 08 static final float SIZE = 1f; 09 10 Vector2 position = new Vector2(); 11 Rectangle bounds = new Rectangle(); 12 13 public Block(Vector2 pos) { 14 this.position = pos; 15 this.bounds.width = SIZE; 16 this.bounds.height = SIZE; 17 } 18 }

Blocks are nothing more than rectangles placed in the world. We will use these blocks to make up the terrain. We have one simple rule. Nothing can penetrate them.

You might have noticed that we are using the Vector2 type from libgdx. This makes our life considerably easier as it provides everything we need to work with Euclidean vectors. We will use vectors to position entities, to calculate speeds, and to move thing around.

About the coordinate system and units

As the real world, our world has dimensions. Think of a room in a flat. It has a width, height and depth. We will make it 2 dimensional and will get rid of the depth. If the room is 5 meters wide and 3 meters tall we can say that we described the room in the metric system. It is easy to imagine placing a table 1 meter wide and 1 meter tall in the middle. We can’t go through the table, to cross it, we will need to jump on top of it, walk 1 meter and jump off. We can use multiple tables to create a pyramid and create some weird designs in the room.

In our star assault world, the world represents the room, the blocks the table and the unit, the meter in the real world.

If I run with 10km/h that translates to 2.77777778 metres / second ( 10 1000 / 3600). To translate this to Star Assault world coordinates, we will say that to resemble a 10km/h speed, we will use 2.7 units/second.

Examine the following diagram of representing the bounding boxes and Bob in the world coordinate system.

The red squares are the bounding boxes of the blocks. The green square is Bob’s bounding box. The empty squares are just empty air. The grid is just for reference. This is the world we will be creating our simulations in. The coordinate system’s origin is at the bottom left, so walking left at 10.000 units/hour means that Bob’s position’s X coordinate will decrease with 2.7 units every second.

Also note that the access to the members is package default and the models are in a separate package. We will have to create accessor methods (getters and setters) to get access to them from the engine.

Creating the World

As a first step we will just create the world as a hard-coded tiny room. It will be 10 units wide and 7 units tall. We will place Bob and the blocks following the image shown below.

The World.java looks like this: 01 package net.obviam.starassault.model; 02 03 import com.badlogic.gdx.math.Vector2; 04 import com.badlogic.gdx.utils.Array; 05 06 public class World { 07 08 / The blocks making up the world / 09 Array



&lt;block&gt;



blocks = new Array



&lt;block&gt;



(); 10 / Our player controlled hero / 11 Bob bob; 12 13 // Getters ----------- 14 public Array



&lt;block&gt;



getBlocks() { 15 return blocks; 16 } 17 public Bob getBob() { 18 return bob; 19 } 20 // -------------------- 21 22 public World() { 23 createDemoWorld(); 24 } 25 26 private void createDemoWorld() { 27 bob = new Bob(new Vector2(7, 2)); 28 29 for (int i = 0; i < 10; i++) { 30 blocks.add(new Block(new Vector2(i, 0))); 31 blocks.add(new Block(new Vector2(i, 7))); 32 if (i > 2) 33 blocks.add(new Block(new Vector2(i, 1))); 34 } 35 blocks.add(new Block(new Vector2(9, 2))); 36 blocks.add(new Block(new Vector2(9, 3))); 37 blocks.add(new Block(new Vector2(9, 4))); 38 blocks.add(new Block(new Vector2(9, 5))); 39 40 blocks.add(new Block(new Vector2(6, 3))); 41 blocks.add(new Block(new Vector2(6, 4))); 42 blocks.add(new Block(new Vector2(6, 5))); 43 } 44 }

It is a simple container class for the entities in the world. Currently the entities are the blocks and Bob. In the constructor the blocks are added to the blocks array and Bob is created. It’s all hard-coded for the time being.

Remember that the origin is in the bottom left corner.