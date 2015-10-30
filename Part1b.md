Android Game Development with libgdx – Prototype in a day, Part 1b

Creating the Game and Displaying the World

To render the world onto the screen, we need to create a screen for it and tell it to render the world. In libgdx there is a convenience class called Game and we will rewrite the StarAssault class a subclass of the Game class provided by libgdx.

About Screens

A game can consist of multiple screens. Even our game will have 3 basic screens. The Start Game screen, the Play Screen and the Game Over screen. Each screen is concerned with the things happening on it and they don’t care about each other. The Start Game screen for example will contain the menu options Play and Quit. It has two elements (buttons) and it is concerned about handling the clicks/touches on those elements. It renders tirelessly those two buttons and if the Play button is clicked/touched, it notifies the main Game to load the Play Screen and get rid of the current screen. The Play Screen will run our game and will handle everything regarding the game. Once the Game Over state is reached, it tells the main Game to transition to the Game Over screen, whose sole purpose is to display the high scores and listen to clicks on the Replay button.

Let’s refactor the code and create just the main screen for the game for the time being. We will skip the start and game over screens.

The GameScreen.java
01	package net.obviam.starassault.screens;
02
03	import com.badlogic.gdx.Screen;
04
05	public class GameScreen implements Screen {
06
07	 @Override
08	 public void render(float delta) {
09	  // TODO Auto-generated method stub
10	 }
11
12	 @Override
13	 public void resize(int width, int height) {
14	  // TODO Auto-generated method stub
15	 }
16
17	 @Override
18	 public void show() {
19	  // TODO Auto-generated method stub
20	 }
21
22	 @Override
23	 public void hide() {
24	  // TODO Auto-generated method stub
25	 }
26
27	 @Override
28	 public void pause() {
29	  // TODO Auto-generated method stub
30	 }
31
32	 @Override
33	 public void resume() {
34	  // TODO Auto-generated method stub
35	 }
36
37	 @Override
38	 public void dispose() {
39	  // TODO Auto-generated method stub
40	 }
41	}

The StarAssault.java will become very simple.
01	package net.obviam.starassault;
02
03	import net.obviam.starassault.screens.GameScreen;
04
05	import com.badlogic.gdx.Game;
06
07	public class StarAssault extends Game {
08
09	 @Override
10	 public void create() {
11	  setScreen(new GameScreen());
12	 }
13	}

GameScreen implements the Screen interface which is very much like an ApplicationListener but it has 2 important methods added.
show() – this is called when the main game makes this screen active
hide() – this is called when the main game makes another screen active

StarAssault has just one method implemented. The create() does nothing more than to activate the newly instantiated GameScreen. In other words, it creates it, calls the show() method and will subsequently call its render() method every cycle.

The GameScreen becomes our focus for the next part as it is where the game will live. Remember that the game loop is the render() method. But to have something to render we first need to create the world. The world can be created in the show() method as we don’t have any other screens that can interrupt our gameplay. Currently, the GameScreen is shown only when the game starts.

We will add two members to the class and implement the render(float delta) method.
01	private World world;
02	private WorldRenderer renderer;
03
04	/ Rest of methods ommited /
05
06	@Override
07	public void render(float delta) {
08	 Gdx.gl.glClearColor(0.1f, 0.1f, 0.1f, 1);
09	 Gdx.gl.glClear(GL10.GL\_COLOR\_BUFFER\_BIT);
10	 renderer.render();
11	}

The world attribute is the World instance which holds the blocks and Bob.
The renderer is a class which will draw/render the world onto the screen (I will reveal it shortly).
The render(float delta)
Let’s create the WorldRenderer class.
WorldRenderer.java
01	package net.obviam.starassault.view;
02
03	import net.obviam.starassault.model.Block;
04	import net.obviam.starassault.model.Bob;
05	import net.obviam.starassault.model.World;
06	import com.badlogic.gdx.graphics.GL10;
07	import com.badlogic.gdx.graphics.OrthographicCamera;
08	import com.badlogic.gdx.graphics.glutils.ShapeRenderer;
09	import com.badlogic.gdx.graphics.glutils.ShapeRenderer.ShapeType;
10	import com.badlogic.gdx.math.Rectangle;
11
12	public class WorldRenderer {
13
14	 private World world;
15	 private OrthographicCamera cam;
16
17	 / for debug rendering /
18	 ShapeRenderer debugRenderer = new ShapeRenderer();
19
20	 public WorldRenderer(World world) {
21	  this.world = world;
22	  this.cam = new OrthographicCamera(10, 7);
23	  this.cam.position.set(5, 3.5f, 0);
24	  this.cam.update();
25	 }
26
27	 public void render() {
28	  // render blocks
29	  debugRenderer.setProjectionMatrix(cam.combined);
30	  debugRenderer.begin(ShapeType.Rectangle);
31	  for (Block block : world.getBlocks()) {
32	   Rectangle rect = block.getBounds();
33	   float x1 = block.getPosition().x + rect.x;
34	   float y1 = block.getPosition().y + rect.y;
35	   debugRenderer.setColor(new Color(1, 0, 0, 1));
36	   debugRenderer.rect(x1, y1, rect.width, rect.height);
37	  }
38	  // render Bob
39	  Bob bob = world.getBob();
40	  Rectangle rect = bob.getBounds();
41	  float x1 = bob.getPosition().x + rect.x;
42	  float y1 = bob.getPosition().y + rect.y;
43	  debugRenderer.setColor(new Color(0, 1, 0, 1));
44	  debugRenderer.rect(x1, y1, rect.width, rect.height);
45	  debugRenderer.end();
46	 }
47	}

The WorldRenderer has only one purpose. To take the current state of the world and render its current state to the screen. It has a single public render() method which gets called by the main loop (GameScreen). The renderer needs to have access to the world so we will pass it in when we instantiate the renderer. For the first step, we will render the bounding boxes of the elements (blocks and Bob) to see what we have so far. Drawing primitives in OpenGL is quite tedious but libgdx comes with a ShapeRenderer which makes this task very easy.
The important lines are explained.

#14 – Declares the world as a member variable.
#15 – We declare an OrthographicCamera. We will use this camera to “look” at the world from an orthographic perspective. Currently the world is very small and it fits onto one screen, but when we will have an extensive level and Bob moves around in it, we will have to move the camera following Bob. It’s analogous to a real life camera. More on orthographic projections can be found here.
#18 – The ShapeRenderer is declared. We will use this to draw primitives (rectangles) for the entities. This is a helper renderer that can draw primitives like lines, rectangles, circles. For anyone familiar with canvas based graphics, this should be easy.
#20 – The constructor which takes the world as the parameter.
#22 – We create the camera with a viewport of 10 units wide and 7 units tall. This means that filling up the screen with unit blocks (width = height = 1) will result in showing 10 boxes on the X axis and 7 on the Y.
Important: This is resolution independent. If the screen resolution is 480×320, that means that 480 pixels represent 10 units, so a box will be 48 pixels wide. It also means that 320 pixels represent 7 units so the boxes on the screen will be 45.7 pixels tall. It won’t be a perfect square. This is due to the aspect ratio. The aspect ratio in our case is 10:7.
#23 – This lines positions the camera to look at the middle of the room. By default it looks at (0,0) which is the corner of the room. The camera’s (0,0) is in the middle as you would expect from a normal camera. The following image shows the world and camera set-up coordinates.

#24 – The internal matrices of the camera are updated. The update method must be called every time the camera is acted upon (move, zoom, rotate, etc). OpenGL hidden beautifully.
The render() method:
#29 – We apply the matrix from the camera to the renderer. This is necessary as we positioned the camera and we want them to be the same.
#30 – We tell the renderer that we want to draw rectangles.
#31 – We will draw the blocks so we iterate through all of them in the world.
#32 – #34 – Extract the coordinates of the each block’s bounding rectangle. OpenGL works with vertices (points) so for it to draw a rectangle have to know the coordinates for the starting point and the width. Notice that we work in camera coordinates which coincides with the world coordinates.
#35 – Set the color of the rectangles to red.
#36 – Draw the rectangle at the x1, y1 with the given width and height.
#39 – #44 – We do the same with Bob, but this time the rectangle is green.
#45 – We let the renderer know that we’re done drawing rectangles.

We need to add the renderer and the world to the GameScreen (main loop) and see it in action.
Modify the GameScreen like this:
01	package net.obviam.starassault.screens;
02
03	import net.obviam.starassault.model.World;
04	import net.obviam.starassault.view.WorldRenderer;
05
06	import com.badlogic.gdx.Gdx;
07	import com.badlogic.gdx.Screen;
08	import com.badlogic.gdx.graphics.GL10;
09
10	public class GameScreen implements Screen {
11
12	 private World world;
13	 private WorldRenderer renderer;
14
15	 @Override
16	 public void show() {
17	  world = new World();
18	  renderer = new WorldRenderer(world);
19	 }
20
21	 @Override
22	 public void render(float delta) {
23	  Gdx.gl.glClearColor(0.1f, 0.1f, 0.1f, 1);
24	  Gdx.gl.glClear(GL10.GL\_COLOR\_BUFFER\_BIT);
25	  renderer.render();
26	 }
27
28	 / ... rest of method stubs omitted ... /
29
30	}

The render(float delta) method has 3 lines. The first 2 lines clear the screen with black and the 3rd line simply calls the renderer’s render() method.
The World and WorldRenderer are created when the screen is shown.

To test it on both the desktop and Android, we have to create the launchers for both platforms. Creating the Desktop and Android Launchers

We have created 2 more projects in the beginning.
star-assault-desktop and star-assault-android, the latter being an Android project.
For the desktop project is dead simple. We need to create a class with a main method in it which instantiates an application provided by libgdx.
Create the StarAssaultDesktop.java class in the desktop project.
1	package net.obviam.starassault;
2
3	import com.badlogic.gdx.backends.lwjgl.LwjglApplication;
4
5	public class StarAssaultDesktop {
6	 public static void main(String[.md](.md) args) {
7	  new LwjglApplication(new StarAssault(), "Star Assault", 480, 320, true);
8	 }
9	}

This is it. Line #7 is where everything happens. It instantiates a new LwjglApplication application passing in a new StarAssault instance which is a Game implementation. The 2nd and 3rd parameters tell the window’s dimension. I opted for 480×320 because it is a resolution supported on many Android phones and I want to resemble it on the desktop. The last parameter tells libgdx to use OpenGL ES 2.
Running the application as a normal Java program should produce the following result:

If you are getting some errors, track back and make sure the set-up is correct and all the steps are followed, including checking gdx.jar at the export tab on the star-guard Project properties -> Build Path.


The Android Version

In the star-assault-android project there is a single java class called StarAssaultActivity.
Change it to:
StarAssaultActivity.java
01	package net.obviam.starassault;
02
03	import android.os.Bundle;
04
05	import com.badlogic.gdx.backends.android.AndroidApplication;
06	import com.badlogic.gdx.backends.android.AndroidApplicationConfiguration;
07
08	public class StarAssaultActivity extends AndroidApplication {
09	    / Called when the activity is first created. **/
10	    @Override
11	    public void onCreate(Bundle savedInstanceState) {
12	        super.onCreate(savedInstanceState);
13	  AndroidApplicationConfiguration config = new AndroidApplicationConfiguration();
14	  config.useAccelerometer = false;
15	  config.useCompass = false;
16	  config.useWakelock = true;
17	  config.useGL20 = true;
18	  initialize(new StarAssault(), config);
19	    }
20	}**

Pay attention that the new activity extends AndroidApplication.
In line #13 an AndroidApplicationConfiguration object is created. We can set all types of configurations regarding the Android platform. They are self explanatory but note that if we want to use the Wakelock, the AndroidManifest.xml file also needs to be modified. This asks permission from Android to keep the device on and to prevent dimming the screen if we don’t touch it.
Add the following line to the AndroidManifest.xml file somewhere inside the 

&lt;manifest&gt;

tags.
1	<uses-permission android:name="android.permission.WAKE\_LOCK"/>

Also in line #17 we tell Android to use OpenGL ES 2. This means we will be able to test it only on a device as the emulator does not support OpenGL ES 2. In case there is a problem with it, switch this to false.
Line #18 initialises the Android application and launches it.
Having a device connected to eclipse, it gets directly deployed and below you can see a photo of the application running on a nexus one. It looks identical to the desktop version.

The MVC Pattern

It’s quite impressive how far we came in such a short time. Note the use of the MVC pattern. It’s very efficient and simple. The models are the entities we want to display. The view is the renderer. The view draws the models onto the screen. Now we need to interact with the entities (especially Bob) and we will introduce some controllers too.
To read more on the MVC pattern, check out my other article or search for it on the net. It’s very useful.

Adding Images

So far it’s all nice but definitely we want to use some proper graphics. The power of MVC comes in handy and we will modify the renderer so it will draw images instead of rectangles.
In OpenGL to display an image is quite a complicated process. First it needs to be loaded, turned into a texture and then mapped to a surface which is described by some geometry. libgdx makes this extremely easy. To turn an image from the disk into a texture is a one liner.
We will use 2 images hence 2 textures. One texture for Bob and one for the blocks. I have created two images, a block and Bob. Bob is a copycat of the Star Guard chap. These are simple png files and I will copy them into the assets/images directory. I have two images: block.png and bob\_01.png. Eventually Bob will become an animated character so I suffixed it with a number (panning for the future).
First let’s clean up the WorldRenderer a bit, namely, to extract the drawing of rectangles into a separate method as we will be using it for debug purposes.
We will need to load the textures and render them accordingly to the screen.
Take a look at the new WorldRenderer.java
01	package net.obviam.starassault.view;
02
03	import net.obviam.starassault.model.Block;
04	import net.obviam.starassault.model.Bob;
05	import net.obviam.starassault.model.World;
06	import com.badlogic.gdx.Gdx;
07	import com.badlogic.gdx.graphics.Color;
08	import com.badlogic.gdx.graphics.OrthographicCamera;
09	import com.badlogic.gdx.graphics.Texture;
10	import com.badlogic.gdx.graphics.g2d.SpriteBatch;
11	import com.badlogic.gdx.graphics.glutils.ShapeRenderer;
12	import com.badlogic.gdx.graphics.glutils.ShapeRenderer.ShapeType;
13	import com.badlogic.gdx.math.Rectangle;
14
15	public class WorldRenderer {
16
17	 private static final float CAMERA\_WIDTH = 10f;
18	 private static final float CAMERA\_HEIGHT = 7f;
19
20	 private World world;
21	 private OrthographicCamera cam;
22
23	 / for debug rendering /
24	 ShapeRenderer debugRenderer = new ShapeRenderer();
25
26	 / Textures /
27	 private Texture bobTexture;
28	 private Texture blockTexture;
29
30	 private SpriteBatch spriteBatch;
31	 private boolean debug = false;
32	 private int width;
33	 private int height;
34	 private float ppuX; // pixels per unit on the X axis
35	 private float ppuY; // pixels per unit on the Y axis
36	 public void setSize (int w, int h) {
37	  this.width = w;
38	  this.height = h;
39	  ppuX = (float)width / CAMERA\_WIDTH;
40	  ppuY = (float)height / CAMERA\_HEIGHT;
41	 }
42
43	 public WorldRenderer(World world, boolean debug) {
44	  this.world = world;
45	  this.cam = new OrthographicCamera(CAMERA\_WIDTH, CAMERA\_HEIGHT);
46	  this.cam.position.set(CAMERA\_WIDTH / 2f, CAMERA\_HEIGHT / 2f, 0);
47	  this.cam.update();
48	  this.debug = debug;
49	  spriteBatch = new SpriteBatch();
50	  loadTextures();
51	 }
52
53	 private void loadTextures() {
54	  bobTexture = new  Texture(Gdx.files.internal("images/bob\_01.png"));
55	  blockTexture = new Texture(Gdx.files.internal("images/block.png"));
56	 }
57
58	 public void render() {
59	  spriteBatch.begin();
60	   drawBlocks();
61	   drawBob();
62	  spriteBatch.end();
63	  if (debug)
64	   drawDebug();
65	 }
66
67	 private void drawBlocks() {
68	  for (Block block : world.getBlocks()) {
69	   spriteBatch.draw(blockTexture, block.getPosition().x **ppuX, block.getPosition().y** ppuY, Block.SIZE **ppuX, Block.SIZE** ppuY);
70	  }
71	 }
72
73	 private void drawBob() {
74	  Bob bob = world.getBob();
75	  spriteBatch.draw(bobTexture, bob.getPosition().x **ppuX, bob.getPosition().y** ppuY, Bob.SIZE **ppuX, Bob.SIZE** ppuY);
76	 }
77
78	 private void drawDebug() {
79	  // render blocks
80	  debugRenderer.setProjectionMatrix(cam.combined);
81	  debugRenderer.begin(ShapeType.Rectangle);
82	  for (Block block : world.getBlocks()) {
83	   Rectangle rect = block.getBounds();
84	   float x1 = block.getPosition().x + rect.x;
85	   float y1 = block.getPosition().y + rect.y;
86	   debugRenderer.setColor(new Color(1, 0, 0, 1));
87	   debugRenderer.rect(x1, y1, rect.width, rect.height);
88	  }
89	  // render Bob
90	  Bob bob = world.getBob();
91	  Rectangle rect = bob.getBounds();
92	  float x1 = bob.getPosition().x + rect.x;
93	  float y1 = bob.getPosition().y + rect.y;
94	  debugRenderer.setColor(new Color(0, 1, 0, 1));
95	  debugRenderer.rect(x1, y1, rect.width, rect.height);
96	  debugRenderer.end();
97	 }
98	}

I’ll point out the important lines:
#17 & #18 – Declared constants for the viewport’s dimensions. It’s used for the camera.
#27 & #28 – Declare the 2 textures that will be used for Bob and the blocks.
#30 – The SpriteBatch is declared. The SpriteBatch takes care of all the texture mapping, displaying and so on for us.
#31 – It’s an attribute set in the constructor to know if we need to render the debug screen too or not. Remember, the debug rendering will just render the boxes for the game elements.
#32 – #35 – these variables are necessary to correctly display the elements. The width and height hold the screen size in pixels and are passed in from the operating system at the resize step. The ppuX and ppuY are the number of pixels per unit.
Because we set the camera to have a view port of 10×7 in world coordinates (meaning we can display 10 boxes horizontally and 7 boxes vertically) and we are dealing with pixels on the end result, we need to map those values to the actual pixel coordinates. We have chosen to work in a 480×320 resolution. That means that 480 pixels horizontally are equivalent of 10 units, meaning a unit will consists of 48 pixels on the screen.
If we try to use the same unit for the height (48 pixels) we get 336 pixels (48 **7 = 336). But we have only 320 pixels available and we want to show the whole 7 blocks height. Doing the same for the vertical part, we get that 1 unit vertically will be 320 / 7 = 45.71 pixels. We need to distort every image a bit to fit in our world.
It’s perfectly fine and OpenGL does that very easily. This happens when we change the aspect ratio on our TV set and sometimes the image gets elongated or squashed to fit everything on the screen, or we just simply choose the option to cut the image off but maintain the aspect ratio.
Note: we use float for this, even if the screen resolution deals with ints, OpenGL prefers floats and so do we. OpenGL will work out the dimensions and where to place pixels.
#36 – The setSize (int w, int h) method will be called every time the screen is resized and it simply (re)calculates the units in pixels.
#43 – The constructor changed just a little but it does very important things. It instantiates the SpriteBatch and loads the textures (line #50).
#53 – loadTextures() does what it says: loads the textures. Look how incredibly simple it is. To create a texture, we need to pass in a file handler and it creates a texture out of it. The file handlers in libgdx are very helpful, as we don’t differentiate between Android or deskop, we just specify that we want to use an internal file and it knows how to load it. Note that for the path we skipped assets because assets is used as a source directory, meaning everything from that directory gets copied into the root of the final bundle. So assets acts as a root directory.
#58 – the new render() method contains just a few lines.
#59 & #62 – enclose a SpriteBatch drawing block/session. Every time we want to render images in OpenGL through the SpriteBatch we have to call begin(), draw our stuff and end() when we’re done. It is important to do that, otherwise it won’t work. You can read more on SpriteBatch here.
#60 & #61 – simply call 2 methods to render first the blocks and then Bob.
#63 & #64 – if debug is enabled, call the method to render the boxes. The drawDebug method was detailed previously.
#67 – #76 – the drawBlocks and drawBob methods are similar. Each method calls the draw method of the spriteBatch with a texture. It is important to understand this.
The first parameter is the texture (the image loaded from the disk).
The second and third parameters tell the spriteBatch where to display the image. Note that we use the conversion of coordinates from world coordinates to screen coordinates. Here is where the ppuX and ppuY are used. You can do the calculations by hand and see where the images get displayed. SpriteBatch by default uses a coordinate system with the origin (0, 0) in the bottom left corner.**

That’s it. Just make sure you modify the GameScreen class so the resize gets called on the renderer and also to set the renderer’s debug to true.
The modified bits of GameScreen
01	/ ... omitted ... /
02	 public void show() {
03	  world = new World();
04	  renderer = new WorldRenderer(world, true);
05	 }
06
07	 public void resize(int width, int height) {
08	  renderer.setSize(width, height);
09	 }
10	/ ... omitted ... /

Running the application should produce the following result:
without debug

and with debug rendering

Great! Give it a try on Android too and see how it looks.

Processing Input – on Desktop & Android

We’ve come a long way but so far the world is static and nothing interesting is going on. To make it a game, we need to add input processing, to intercept keys and touches and create some action based on those.
The control schema on the Desktop is very simple. The arrow keys will move Bob to the left and right, z will make Bob jump and x will fire the weapon. On Android we will have a different approach. We will designate some buttons for these functions and will lay it down on the screen and by touching the respective areas we will consider one of the keys pressed.

To follow the MVC pattern, we’ll separate the class that controls Bob and the rest of the world from the model and view classes. Create the package net.obviam.starassault.controller and all controllers will go there.
For the start we will control Bob by key presses. To play the game we to track the status of 4 keys: move left, move right, jump and fire. Because we will use 2 types of input (keyboard and touch-screen), the actual events need to be fed into a processor that can trigger the actions.
Each action is triggered by an event.
The move left action is triggered by the event when the left arrow key is pressed or a certain area of the screen is touched.
The jump action is triggered when the z key is pressed and so on.
Let’s create a very simple controller called WorldController.
WorldController.java
01	package net.obviam.starassault.controller;
02
03	import java.util.HashMap;
04	import java.util.Map;
05	import net.obviam.starassault.model.Bob;
06	import net.obviam.starassault.model.Bob.State;
07	import net.obviam.starassault.model.World;
08
09	public class WorldController {
10
11	 enum Keys {
12	  LEFT, RIGHT, JUMP, FIRE
13	 }
14
15	 private World  world;
16	 private Bob  bob;
17
18	 static Map<Keys, Boolean> keys = new HashMap<WorldController.Keys, Boolean>();
19	 static {
20	  keys.put(Keys.LEFT, false);
21	  keys.put(Keys.RIGHT, false);
22	  keys.put(Keys.JUMP, false);
23	  keys.put(Keys.FIRE, false);
24	 };
25
26	 public WorldController(World world) {
27	  this.world = world;
28	  this.bob = world.getBob();
29	 }
30
31	 //  Key presses and touches  //
32
33	 public void leftPressed() {
34	  keys.get(keys.put(Keys.LEFT, true));
35	 }
36
37	 public void rightPressed() {
38	  keys.get(keys.put(Keys.RIGHT, true));
39	 }
40
41	 public void jumpPressed() {
42	  keys.get(keys.put(Keys.JUMP, true));
43	 }
44
45	 public void firePressed() {
46	  keys.get(keys.put(Keys.FIRE, false));
47	 }
48
49	 public void leftReleased() {
50	  keys.get(keys.put(Keys.LEFT, false));
51	 }
52
53	 public void rightReleased() {
54	  keys.get(keys.put(Keys.RIGHT, false));
55	 }
56
57	 public void jumpReleased() {
58	  keys.get(keys.put(Keys.JUMP, false));
59	 }
60
61	 public void fireReleased() {
62	  keys.get(keys.put(Keys.FIRE, false));
63	 }
64
65	 / The main update method /
66	 public void update(float delta) {
67	  processInput();
68	  bob.update(delta);
69	 }
70
71	 / Change Bob's state and parameters based on input controls /
72	 private void processInput() {
73	  if (keys.get(Keys.LEFT)) {
74	   // left is pressed
75	   bob.setFacingLeft(true);
76	   bob.setState(State.WALKING);
77	   bob.getVelocity().x = -Bob.SPEED;
78	  }
79	  if (keys.get(Keys.RIGHT)) {
80	   // left is pressed
81	   bob.setFacingLeft(false);
82	   bob.setState(State.WALKING);
83	   bob.getVelocity().x = Bob.SPEED;
84	  }
85	  // need to check if both or none direction are pressed, then Bob is idle
86	  if ((keys.get(Keys.LEFT) && keys.get(Keys.RIGHT)) 
87	    (!keys.get(Keys.LEFT) && !(keys.get(Keys.RIGHT)))) {
88	   bob.setState(State.IDLE);
89	   // acceleration is 0 on the x
90	   bob.getAcceleration().x = 0;
91	   // horizontal speed is 0
92	   bob.getVelocity().x = 0;
93	  }
94	 }
95	}

#11 – #13 – define an enum for the actions Bob will perform. Each keypress/touch can trigger one action.
#15 – declare the World that is in the game. We will be controlling the entities found in the world.
#16 – declare Bob as a private member and it is just a reference to Bob in the game world, but we will need it as it’s easier to refer to it than retrieving it every time we need him.
#18 – #24 – it’s a static HashMap of the keys and their statuses. If the key is pressed, it’s true, false otherwise. It is statically initialised. This map will be used in the controller’s update method to work out what to do with Bob.
#26 – This is the constructor that takes the World as the parameter and gets the reference to Bob as well.
#33 – #63 – These methods are simple callbacks that are called whenever an action button was pressed or a touch on a designated area happened. These methods are the ones that get called from whatever input we’re using. They simply set the the value of the respective pressed keys in the map. As you can see, the controller is a state machine too and its state is given by the keys map.
#66 – #69 – the update method which gets called every cycle of the main loop. currently it does 2 things: 1 – processes the input and 2 – updates Bob. Bob has a dedicated update method which we will see later.
#72 – #92 – the processInput method polls the keys map for the keys and sets the values on Bob accordingly. For example lines #73 – #78 check if the key is pressed for the movement to the left and if so, then sets the facing for Bob to the left, his state to State.WALKING and his velocity to Bob’s speed but with a negative sign. The sign is because on the screen, left the negative direction (origin is in the bottom left and points to the right).
The same thing for the right. There are some extra checks if both keys are pressed or none and in this case, Bob becomes State.IDLE and his horizontal velocity will be 0.

Let’s see what changed in Bob.java.
1	public static final float SPEED = 4f; // unit per second
2
3	public void setState(State newState) {
4	 this.state = newState;
5	}
6
7	public void update(float delta) {
8	 position.add(velocity.tmp().mul(delta));
9	}

Just changed the SPEED constant to 4 units (blocks) per second.
Also added the setState method because I forgot it before.
The most interesting is the newly acquired update(float delta) method, which is called from the WorldController. This method simply updates Bob’s position based on his velocity. For simplicity we do only that without checking his state and because the controller takes care to set the velocity for Bob according to his facing and state. We use vector math here and libgdx helps a lot.
We simply add the distance travelled in delta seconds to Bob’s current position. We use velocity.tmp() because the tmp() creates a new object with the same value as velocity and we multiply that object’s value with the elapsed time delta. In Java we have to be careful on how we’re using references as velocity and position are both Vector2 objects. More on vectors here http://en.wikipedia.org/wiki/Euclidean_vector.

We have almost everything, we just need to call the correct events when they happen. libgdx has an input processor which has a few callback methods. Because we are using the GameScreen as the playing surface, it makes sense to use it as the input handler too. To do this, the GameScreen will implement the libgdx InputProcessor.
The new GameScreen.java
001	package net.obviam.starassault.screens;
002
003	import net.obviam.starassault.controller.WorldController;
004	import net.obviam.starassault.model.World;
005	import net.obviam.starassault.view.WorldRenderer;
006
007	import com.badlogic.gdx.Gdx;
008	import com.badlogic.gdx.Input.Keys;
009	import com.badlogic.gdx.InputProcessor;
010	import com.badlogic.gdx.Screen;
011	import com.badlogic.gdx.graphics.GL10;
012
013	public class GameScreen implements Screen, InputProcessor {
014
015	 private World    world;
016	 private WorldRenderer  renderer;
017	 private WorldController controller;
018
019	 private int width, height;
020
021	 @Override
022	 public void show() {
023	  world = new World();
024	  renderer = new WorldRenderer(world, false);
025	  controller = new WorldController(world);
026	  Gdx.input.setInputProcessor(this);
027	 }
028
029	 @Override
030	 public void render(float delta) {
031	  Gdx.gl.glClearColor(0.1f, 0.1f, 0.1f, 1);
032	  Gdx.gl.glClear(GL10.GL\_COLOR\_BUFFER\_BIT);
033
034	  controller.update(delta);
035	  renderer.render();
036	 }
037
038	 @Override
039	 public void resize(int width, int height) {
040	  renderer.setSize(width, height);
041	  this.width = width;
042	  this.height = height;
043	 }
044
045	 @Override
046	 public void hide() {
047	  Gdx.input.setInputProcessor(null);
048	 }
049
050	 @Override
051	 public void pause() {
052	  // TODO Auto-generated method stub
053	 }
054
055	 @Override
056	 public void resume() {
057	  // TODO Auto-generated method stub
058	 }
059
060	 @Override
061	 public void dispose() {
062	  Gdx.input.setInputProcessor(null);
063	 }
064
065	 // **InputProcessor methods**//
066
067	 @Override
068	 public boolean keyDown(int keycode) {
069	  if (keycode == Keys.LEFT)
070	   controller.leftPressed();
071	  if (keycode == Keys.RIGHT)
072	   controller.rightPressed();
073	  if (keycode == Keys.Z)
074	   controller.jumpPressed();
075	  if (keycode == Keys.X)
076	   controller.firePressed();
077	  return true;
078	 }
079
080	 @Override
081	 public boolean keyUp(int keycode) {
082	  if (keycode == Keys.LEFT)
083	   controller.leftReleased();
084	  if (keycode == Keys.RIGHT)
085	   controller.rightReleased();
086	  if (keycode == Keys.Z)
087	   controller.jumpReleased();
088	  if (keycode == Keys.X)
089	   controller.fireReleased();
090	  return true;
091	 }
092
093	 @Override
094	 public boolean keyTyped(char character) {
095	  // TODO Auto-generated method stub
096	  return false;
097	 }
098
099	 @Override
100	 public boolean touchDown(int x, int y, int pointer, int button) {
101	  if (x < width / 2 && y > height / 2) {
102	   controller.leftPressed();
103	  }
104	  if (x > width / 2 && y > height / 2) {
105	   controller.rightPressed();
106	  }
107	  return true;
108	 }
109
110	 @Override
111	 public boolean touchUp(int x, int y, int pointer, int button) {
112	  if (x < width / 2 && y > height / 2) {
113	   controller.leftReleased();
114	  }
115	  if (x > width / 2 && y > height / 2) {
116	   controller.rightReleased();
117	  }
118	  return true;
119	 }
120
121	 @Override
122	 public boolean touchDragged(int x, int y, int pointer) {
123	  // TODO Auto-generated method stub
124	  return false;
125	 }
126
127	 @Override
128	 public boolean touchMoved(int x, int y) {
129	  // TODO Auto-generated method stub
130	  return false;
131	 }
132
133	 @Override
134	 public boolean scrolled(int amount) {
135	  // TODO Auto-generated method stub
136	  return false;
137	 }
138	}

The changes:
#13 – the class implements the InputProcessor
#19 – the width and height of the screen used by the Android touch events.
#25 – instantiate the WorldController with the world.
#26 – set the this screen as the current input processor for the the application. libgdx treats this as a global input processor so each screen has to set a different one if they don’t share the same. In this case the screen itself handles the input.
#47 & #62 – we set the active global input processor to null just for cleanup.
#68 – the method keyDown(int keycode) is triggered whenever a key is pressed on the physical keyboard. The parameter keycode is the value of the pressed key and this way we can poll it and in case it’s a desired key, do something. This is exactly what’s happening. Based on the keys we want, we pass on the event to the controller. The method also returns true to let the input processor know that the input was handled.
#81 – the keyUp is the exact inverse of the keyDown method. When the key is released, it simply delegates to the WorldController.
#111 – #118 – this is where it gets interesting. This happens only on touch-screens and the coordinates are passed in along with the pointer and button. The pointer is for multi-touch and represents the id of the touch it captures.
The controls are utterly simple and are made just for simple demo purposes. The screen is divided into 4 and if the touch falls int to lower left quadrant, it is treated as a move left action trigger and passes the same event as the desktop to the controller.
Exactly the same thing for the touchUp.

Warning: – This is very buggy and unreliable, as the touchDragged is not implemented and whenever the finger is dragged across quadrants it will mess up things. This will be fixed of course, the purpose is to demonstrate the multiple hardware inputs and how to tie them together.

Running the application on both desktop and Android will demonstrate the controls. On desktop the arrow keys and on Android by touching the lower corners of the screen will move Bob.
On desktop you will notice that using the mouse to simulate touches will also work. This is because touchXXX also handles mouse input on desktop. To fix this add the following line to the beginning of touchDown and touchUp methods:
1	if (!Gdx.app.getType().equals(ApplicationType.Android))
2	 return false;

This returns false if the application is not Android and does not execute the rest of the method. Remember that false means that the input was not handled.

As we can see, Bob has moved.

Short Recap

So far we have covered quite a bit of game development and we already have something to show.
Gradually we have introduced working pieces into our app and step by step we have achieved something.

We still need to add:

> Terrain interaction (block collision, jump)
> Animation
> A big level and camera to follow Bob
> Enemies and a gun to blast them
> Sounds
> Refined controls and fine tuning
> More screens for game over and start
> Have more fun with libgdx