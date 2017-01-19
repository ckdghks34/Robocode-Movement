# Robocode-Movement

In this project, we come back to where we started in Battlefield_Basics, moving your robot. This project will show you how to circle in on your enemy and dodge bullets.

# Sideways Thinking

If you've played basketball before, you know that if you want to defend someone holding the ball, you want to maximize your lateral movement by always squaring off against them (facing them). The same is true with your robot as the following picture illustrates:

![Image of Sideways Robot]
(http://mark.random-article.com/robocode/square_off.jpg)

The yellow robot can easily move side to side to evade the blue robot and dodge the bullets he shoots. The blue robot, by contrast, doesn't have any good place to go: if he moves back, he gets shot at by the yellow bot, if he moves forward to try to ram, the yellow bot can just scoot out of the way.

## Squaring Off

To square off against an opponent, use the following code:

```java
setTurnRight(enemy.getBearing() + 90);
```

which will always place your robot perpendicular (90 degrees) to your enemy.
However, as we've seen before, adding a number to the existing bearing could give you a non-normalized bearing, which might make you turn the long way around. To fix that, just normalize the "square off" bearing like so:

```java
setTurnRight(normalizeBearing(enemy.getBearing() + 90));
```

using the function we learned about earlier in basic targeting.

## Forward or Backward?

When you're squared off against an opponent, the ideas of "forward" and "backward" become somewhat obsolete. You're probably thinking more in terms of "strafe left" or "strafe right". To keep track of the movement direction, just declare a variable like we did for oscillating the radar.

```java
class MyRobot extends AdvancedRobot {
	private byte moveDirection = 1;
 ```
 
then, when you want to move your robot, you can just say:

```java
setAhead(100 * moveDirection);
```

You can switch directions by changing the value of moveDirection from 1 to -1 like so:

```java
moveDirection *= -1;
```

## Switching Directions

The most intuative approach to switching directions is to just flip the move direction any time you hit a wall or hit another robot like so:

```java
public void onHitWall(HitWallEvent e) { 
    moveDirection *= -1; 
}
public void onHitRobot(HitRobotEvent e) { 
    moveDirection *= -1; 
}
```

However, you will find that if you do that, you'll end up doggedly pressing up against a robot that rams you from the side (not ideal). That's because onHitRobot() gets called so many times that the moveDirection keeps flipping and you never move away.

A better approach is to just test to see if your robot has stopped. If it has, it probably means you've hit something and you'll want to switch direction. You can do it with the code:

```java
if (getVelocity() == 0){
    moveDirection *= -1;
}
```

Put that into your `doMove()` method (or wherever else you're handling movement) and you can handle all wall-hit and robot-hit events. All the sample bots below use this technique.

# Shall We Dance?

All the sample robots that follow use the above techniques for moving around their enemies, with some minor variations. You can match them up against any of the sample robots.

## Circling

Circling your enemy can be done by simply using the above techniques:

```java
public void doMove() {
	// switch directions if we've stopped
	if (getVelocity() == 0) {
		moveDirection *= -1;
	}
	// always square off against our enemy
	setTurnRight(normalizeBearing(enemy.getBearing() + 90));
	// circle our enemy
	setAhead(1000 * moveDirection);
}
```
Note: be sure to put the 'if' test first or your bot will hug the wall.

**Sample robot:** [Circler](http://mark.random-article.com/robocode/lessons/Circler.java) circles his enemy using the above movement code, rather like a shark circling it's prey in the water.

## Strafing

One problem you might notice with Circler is that he is easy prey for predictive targeting because his movements are so... predictable. If you match Circler up against PredictiveShooter you'll see how quick Circler goes down.

To evade bullets more effectively, you should move side-to-side or "strafe". A good way to do this is to switch direction after a certain number of "ticks", like so:

```java
public void doMove() {

	// always square off against our enemy
	setTurnRight(enemy.getBearing() + 90);

	// strafe by changing direction every 20 ticks
	if (getTime() % 20 == 0) {
		moveDirection *= -1;
		setAhead(150 * moveDirection);
	}
}
```

Oddly, MyFirstRobot does something along these lines and can be surprisingly hard to hit.

**Sample robot:** [Strafer](http://mark.random-article.com/robocode/lessons/Strafer.java) rocks back and forth using the above movement code. Notice how nicely he dodges bullets.

## Closing In

You'll notice that both Circler and Strafer have another problem: they get stuck in the corners easy and end up just banging into the walls. An additional problem is that if their enemy is distant, they shoot a lot but don't hit a lot.

To make your robot close in on your enemy, just modify the "squaring off" code to make him turn in toward his enemy slightly, like so:

```java
setTurnRight(normalizeBearing(enemy.getBearing() + 90 - (15 * moveDirection)));
```

**Sample robot:** [Spiraler](http://mark.random-article.com/robocode/lessons/Spiraler.java) is a variation on Circler that uses the above code to spiral in toward his enemy.

**Sample robot:** [StrafeCloser](http://mark.random-article.com/robocode/lessons/StrafeCloser.java) is a variant on Strafer that uses the above code to strafe ever closer. He's a pretty good bullet-dodger, too.

Note that neither of the above robots gets caught in a corner for very long.

## Part I: MOVE Something

1. Use the above strategies to improve your robot's movement. Your robot should strafe, circle, and / or close in on enemies.

# Part II: PartsBot

You may find the following information useful. These are links to Sun's online Java tutorials.

[What Is an Interface?](https://docs.oracle.com/javase/tutorial/java/IandI/createinterface.html) - basic concept of what an interface is

[Using Interfaces](https://docs.oracle.com/javase/tutorial/java/IandI/usinginterface.html) - how to create and to use interfaces 

## Specifications

Write an interface called RobotPart and three classes that implement it called Radar, Gun, and Tank.

### The RobotPart Interface

Declare an interface called RobotPart. It should have two method stubs: init() and move().

### The Radar Class

Write a class called Radar that implements the RobotPart interface. You will need to override both of the interface's methods.

1. In the override of the init() method, you will probably want to call setAdjustRadarForGunTurn(true).
2. In the override of the move() method, you will want to move the radar somehow(It can be oscillating, tracking, or your own unique algorithm).

Additional convenience methods: You **must** override all the methods in interfaces that you implement, but you can certainly add additional methods if you want. In the test harness below I've added `shouldTrack()` and `wasTracking()` methods.

If you want to utilize the test harness add these methods to your Radar class

```java
public boolean shouldTrack(ScannedRobotEvent e) {
	// track if we have no enemy, the one we found is significantly
	// closer, or we scanned the one we've been tracking.
	return (enemy.none() || e.getDistance() < enemy.getDistance() - 70 ||
			e.getName().equals(enemy.getName()));
}

public boolean wasTracking(RobotDeathEvent e) {
	return (e.getName().equals(enemy.getName()));
}
```

### The Gun Class

Next, write a class called Gun that also implements the RobotPart interface. As before, you will need to override both of the interface's methods.

1. In the override of the init() method, you will probably want to call `setAdjustGunForRobotTurn(true)`.
2. In the override of the move() method, you will want to aim the gun and fire. Feel free to use the predictive targeting that you wrote from the Targeting assignment.

### The Tank Class

Lastly, write a class called Tank that also implements the RobotPart interface. If you haven't figured it out already, you will need to override both of the interface's methods in this class too.

1. In the override of the init() method, you will probably want to call setColors(...) with colors of your choice. (See the [online docs for Color](http://java.sun.com/j2se/1.3/docs/api/java/awt/Color.html) for ideas.)
2. In the override of the move() method, you will want to move your robot. You will probably want to look at Part I of this assignment for ideas.


### Test Harness

To test the various parts you wrote, you can start with the following skeleton code:

```java
//add your package name 
import robocode.*;

public class PartsBot extends AdvancedRobot {

	private AdvancedEnemyBot enemy = new AdvancedEnemyBot();

	private RobotPart[] parts = new RobotPart[3]; // make three parts
	private final static int RADAR = 0;
	private final static int GUN = 1;
	private final static int TANK = 2;

	public void run() {

		parts[RADAR] = new Radar();
		parts[GUN] = new Gun();
		parts[TANK] = new Tank();

		// initialize each part
		for (int i = 0; i < parts.length; i++) {
			// behold, the magic of polymorphism
			parts[i].init();
		}

		// iterate through each part, moving them as we go
		for (int i = 0; true; i = (i + 1) % parts.length) {
			// polymorphism galore!
			parts[i].move();
			if (i == 0){
				execute();
			}
		}
	}

	public void onScannedRobot(ScannedRobotEvent e) {
		Radar radar = (Radar)parts[RADAR];
		if (radar.shouldTrack(e)) {
			enemy.update(e, this);
		}
	}

	public void onRobotDeath(RobotDeathEvent e) {
		Radar radar = (Radar)parts[RADAR];
		if (radar.wasTracking(e)) {
			enemy.reset();
		}
	}   

}
```

# Part III: Improved Movement

Here you'll learn how to avoid walls and make a multi-mode bot.

## Avoiding Walls

A problem with all of the previous robots we've looked at is that they hit the walls a lot, and hitting the walls drains your energy. A better strategy would be to stop before you hit the walls. But how?

## Adding a Custom Event

The first thing you need to do is decide how close we will allow our robot to get to the walls:

```java
public class WallAvoider extends AdvancedRobot {
	...
	private int wallMargin = 60;
```

Next, we add a custom event that will be fired when a certain condition is true:

```java
// Don't get too close to the walls
addCustomEvent(new Condition("too_close_to_walls") {
		public boolean test() {
			return (
				// we're too close to the left wall
				(getX() <= wallMargin ||
				 // or we're too close to the right wall
				 getX() >= getBattleFieldWidth() - wallMargin ||
				 // or we're too close to the bottom wall
				 getY() <= wallMargin ||
				 // or we're too close to the top wall
				 getY() >= getBattleFieldHeight() - wallMargin)
				);
			}
		});
```

Note that we are creating an [anonymous inner class](https://docs.oracle.com/javase/tutorial/java/javaOO/anonymousclasses.html) with this call. We need to override the `test()` method to return a boolean when our custom event occurs.

## Handling the Custom Event

The next thing we need to do is handle the event, which can be done like so:

```java
public void onCustomEvent(CustomEvent e) {
	if (e.getCondition().getName().equals("too_close_to_walls"))
	{
		// switch directions and move away
		moveDirection *= -1;
		setAhead(10000 * moveDirection);
	}
}
```

The problem with that approach, though is that this event could get fired over and over, causing us to rappidly switch back and forth, never actually moving away.

**Sample robot:** [JiggleOfDeath](http://mark.random-article.com/robocode/lessons/JiggleOfDeath.java) demonstrates the flaw in the above approach. Match him up against Walls and watch him go down.

To avoid this "jiggle of death" we should have a variable that indicates that we're handling the event. We can declare another like so:

```java
public class WallAvoider extends AdvancedRobot {
	...
	private int tooCloseToWall = 0;
```

Then handle the event a little smarter:

```java
public void onCustomEvent(CustomEvent e) {
	if (e.getCondition().getName().equals("too_close_to_walls"))
	{
		if (tooCloseToWall <= 0) {
			// if we weren't already dealing with the walls,
			// we are now
			tooCloseToWall += wallMargin;
			setMaxVelocity(0); // stop!!!
		}
	}
}
```

## Handling the Two Modes

There are two last problems we need to solve. Firstly, we have a `doMove()` method where we put all our normal movement code. If we're trying to get away from a wall, we don't want our normal movement code to get called, creating (once again) the "jiggle of death". Secondly, we want to eventually return to "normal" movement, so we should have the `tooCloseToWall` variable "time out" eventually.

We can solve both these problems with the following `doMove()` implementation:

```java
public void doMove() {
	// always square off against our enemy, turning slightly toward him
	setTurnRight(enemy.getBearing() + 90 - (10 * moveDirection));

	// if we're close to the wall, eventually, we'll move away
	if (tooCloseToWall > 0) tooCloseToWall--;

	// normal movement: switch directions if we've stopped
	if (getVelocity() == 0) {
		setMaxVelocity(8);
		moveDirection *= -1;
		setAhead(10000 * moveDirection);
	}
}
```

**Sample robot:** [WallAvoider](http://mark.random-article.com/robocode/lessons/WallAvoider.java) uses all the above code to avoid running into the walls. Match him up against Walls and note how he gently glides toward the sides but never (well, rarely) hits them. Feel free to experiment perfecting it. 

## Multi-Mode Bot

Besides the colors you chose, the biggest part of your robot's personality is in his movement code. On the other hand, different situations call for different tactics. Using the wall-avoiding as an example, you may want to code your bot to change "modes" based on certain criteria. Using your PartsBot exercise as a starting point, I can picture a robot with a method like this in it:

```java
public void onRobotDeath(RobotDeathEvent e) {
	...
	if (getOthers() > 10) {
		// a large group calls for fluid movement
		parts[TANK] = new CirclingTank();
	} else if (getOthers() > 1) {
		// dodging is the best small-group tactic
		parts[TANK] = new DodgingTank();
	} else if (getOthers() == 1) {
		// if there's only one bot left, hunt him down
		parts[TANK] = new SeekAndDestroy();
	}
	...
}   
```

## Assignment Part III: Create a multi-mode robot

1. Use the above strategies so that your robot has at least 3 modes

**Hint:** You will need to create a new class for each type of Tank and have it inherit from the `Tank` class
