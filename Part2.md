Android Game Development with libgdx – Animation, Part 2

This is the second part of the Building a Game with LibGdx series. Make sure you read the first part before starting on the second. There will be a lot of stuff covered in the following articles so I will try to break them down in more digestible sizes. We left off with a basic world and Bob gliding back on forth when using the arrow keys or touching the screen. Let’s add some realism to the movement and animate the character whenever it is moving.
Character Animation

To animate Bob, we will use the simplest technique called sprite animation. The animation is nothing more than a sequence of images shown at a set interval to create

the illusion of movement. The following sequence of images is used to create the running animation.

bob running sprite sheet

. I have used Gimp to create the running character by playing Star Guard a lot and analysing its running sequence. To create animations is quite simple. We want to display each frame for a certain amount of time and then switch to the next image. When we reached the end of the sequence we start again. This is called looping. We need to determine the frame duration which the amount of time the frame will be displayed for. Let’s say we are rendering the game at 60 FPS, meaning we render a frame every 1 / 60 = 0.016 s. We have just 5 frames to animate a full step. Considering a typical athlete’s cadence of 180, we can work out how much time we show each frame to make the running realistic.
The math of running

A cadence of 180 means 180 steps per minute. To calculate the number of steps per second we have 180 / 60 = 3. So every second we have need to take 3 steps. Our 5 frames make up one full step so we need to show 3 **5 = 15 frames every second to simulate a professional athlete’s running. This is not sprinting by the way. Our frame duration will be 1 / 15 = 0.066 second. That is 66 ms.
Optimising the images**

Before we add it to the game, we will optimise the images. Currently the project has the images as separate png files under the assets/images directory in the star-assault-android project. We are currently using block.png and bob\_01.png. There are a few more images, namely bob\_02 - 06.png. These images make up the animation sequence. Because libgdx is using OpenGL under the hood, it’s not optimal to give the framework lots of images as textures to work with. What we will do, is to create a so called Texture Atlas. A texture atlas is just an image which is big enough to fit all the images on it and it has a descriptor holding each individual image’s name, position in the atlas and size. The individual images are called regions in the atlas. It there are many images, the atlas can have multiple pages. Each page gets loaded into the memory as a singe image and the regions are used as individual images. Don’t need to know all this but this makes the application more optimal, will load more quickly and will run more smoothly. Libgdx has a utility called TexturePacker2 to create these atlases. It can be run from the command line or used programatically. To run it from Java use the following program:
01	package net.obviam.starassault.utils;
02
03	import com.badlogic.gdx.tools.imagepacker.TexturePacker2;
04
05	public class TextureSetup {
06
07	    public static void main(String[.md](.md) args) {
08	        TexturePacker2.process('/path-to-star-guard-assets-images/', 'path-to-star-guard-assets-images', 'textures.pack');
09	    }
10	}

Make sure that you add gdx-tools.jar to your libs directory. Change the attributes of the process method to point to the directory where the assets are located.
Note:

Also rename the files that contain the underscore “_” character because the TexturePacker2 uses it as a delimiter and we currently don’t need that. Replace the underscore character with the hyphen “-” character. Processing the images in our directory should produce 2 files: textures.png and textures.pack. The texture atlas should look similar to the following image_

texture atlas

The directory structure I am using is the following: Assets Directory Structure

Now that we have worked out what our animation will be, the frame duration and optimised the assets, let’s add it to the game. We will modify Bob.java first as this is the smallest bit
01	public class Bob {
02
03	    // ... omitted ... //
04
05	    float       stateTime = 0;
06
07	    // ... omitted ... //
08
09	    public void update(float delta) {
10	        stateTime += delta;
11	        position.add(velocity.tmp().mul(delta));
12	    }
13	}

We added an attribute called stateTime. This will track Bob’s time in a particular state. We will be using it to provide the time spent by Bob in the game. It is important for the animation to work out which frame to show. Don’t worry about it now. If you really want to understand, think of each frame of the animation as a state. Bob goes through state\_frame\_1, state\_frame\_2 and so on. Each one of these states lasts for 0.066 seconds. Once the state time exceeded 0.066 seconds, Bob goes into the next state. The animation class knows which image to provide to be displayed for the current state. It is also called the key frame. The WorldRenderer.java suffers the most changes. The following snippet contains all the changes.
01	public class WorldRenderer {
02
03	    // ... omitted ... //
04
05	    private static final float RUNNING\_FRAME\_DURATION = 0.06f;
06
07	    / Textures /
08	    private TextureRegion bobIdleLeft;
09	    private TextureRegion bobIdleRight;
10	    private TextureRegion blockTexture;
11	    private TextureRegion bobFrame;
12
13	    / Animations /
14	    private Animation walkLeftAnimation;
15	    private Animation walkRightAnimation;
16
17	    // ... omitted ... //
18
19	    private void loadTextures() {
20	        TextureAtlas atlas = new TextureAtlas(Gdx.files.internal('images/textures/textures.pack'));
21	        bobIdleLeft = atlas.findRegion('bob-01');
22	        bobIdleRight = new TextureRegion(bobIdleLeft);
23	        bobIdleRight.flip(true, false);
24	        blockTexture = atlas.findRegion('block');
25	        TextureRegion[.md](.md) walkLeftFrames = new TextureRegion[5](5.md);
26	        for (int i = 0; i < 5; i++) {
27	            walkLeftFrames[i](i.md) = atlas.findRegion('bob-0' + (i + 2));
28	        }
29	        walkLeftAnimation = new Animation(RUNNING\_FRAME\_DURATION, walkLeftFrames);
30
31	        TextureRegion[.md](.md) walkRightFrames = new TextureRegion[5](5.md);
32
33	        for (int i = 0; i < 5; i++) {
34	            walkRightFrames[i](i.md) = new TextureRegion(walkLeftFrames[i](i.md));
35	            walkRightFrames[i](i.md).flip(true, false);
36	        }
37	        walkRightAnimation = new Animation(RUNNING\_FRAME\_DURATION, walkRightFrames);
38	    }
39
40	    private void drawBob() {
41	        Bob bob = world.getBob();
42	        bobFrame = bob.isFacingLeft() ? bobIdleLeft : bobIdleRight;
43	        if(bob.getState().equals(State.WALKING)) {
44	            bobFrame = bob.isFacingLeft() ? walkLeftAnimation.getKeyFrame(bob.getStateTime(), true) : walkRightAnimation.getKeyFrame(bob.getStateTime(), true);
45	        }
46	        spriteBatch.draw(bobFrame, bob.getPosition().x **ppuX, bob.getPosition().y** ppuY, Bob.SIZE **ppuX, Bob.SIZE** ppuY);
47	    }
48	    // ... omitted ... //
49	}

#05 – declaring the RUNNING\_FRAME\_DURATION constant which controls how long a frame in the running/walking cycle will be displayed
#08 – #11 – TextureRegions for Bob’s different states. bobFrame – will hold the region that will be displayed in the current cycle.
#14 – #15 – the two Animation objects that are used to animate Bob when walking/running.

Loading the images from a TextureAtlas

#19 – the new loadTextures() method
#20 – loading the TextureAtlas form the internal file. This is the .pack file resulted from TexturePacker2.
#21 – assigning the region named “bob-01? (this is the actual png name without extension – see TexturePacker2) to the bobIdleLeft variable.
#22 – #23 – creating a new TextureRegion (note the use of copy constructor, we need a copy, not a reference) and flipping it on the X axis so we have the same image but mirrored for Bob’s idle state but when facing right. The flipping is very useful as we don’t need to load an extra image, we create one from an existing one.
#24 – assign the corresponding region to the block
#25 – #28 – we create an array of TextureRegions that will make up the animation. We know that there are 5 frames and their names: bob-02, bob-03, bob-04, bob-05 and bob-06. We use a for loop for convenience.
#29 – This is where the animation for the walking left state is defined. The first parameter is the duration of each frame from the sequence expressed in seconds (0.06) and the second parameter takes the ordered list of frames making up the animation.
#31 – #38 – Creating the animation for the walking right state. It is a copy of the animation for the walking left state but each frame is flipped. It is important to make a copy of the frames and not flipping them as the originals get flipped too.
#40 – the changed drawBob() method.
#42 – setting the active frame to one of the idle frames depending on Bob’s facing
#44 – in case Bob is in a walking state, we extract the corresponding frame for one of the walking sequences based on Bob’s current state time and assign it to bobFrame which will be drawn to the screen. Running the StarAssaultDesktop application, we should see the Bob animation in action. It should look something like this:

The source code for this project can be found here: https://github.com/obviam/star-assault. You need to checkout the branch part2. To check it out with git: git clone -b part2 git@github.com:obviam/star-assault.git. You can also download it as a zip file.