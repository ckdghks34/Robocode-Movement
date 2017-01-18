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

## Assignment Part I: MOVE Something

1. Use the above strategies to improve your robot's movement. Your robot should strafe, circle, and / or close in on enemies.

