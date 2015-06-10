---
title: "Move on with this tutorial!"
slug: coding-simple-timberman
---

**Create the Code Files**

We'll need three Swift classes for our game right now:

1. `MainScene.swift`
2. `Character.swift`
3. `Piece.swift`

`MainScene.swift` has already been created for us by SpriteBuilder so only need to create two.

If you haven't already, open up `SushiNeko.xcodeproj` -- it should be in your `SushiNeko.spritebuilder` folder.

In Xcode, go to `File > New > File`. Select `iOS Source` from the left side and then `Swift File`. Press `Next` and save it as `Character` in your `Source` folder.

Define the `Character` class as a subclass of `CCSprite` in the newly created file.

> [solution]
> It should look like this:
>
>       class Character: CCSprite {
>
>       }

Follow the same steps to create `Piece.swift` and define the `Piece` class as a subclass of `CCNode` in the newly create file.

> [solution]
> It should look like this:
>
>       class Piece: CCNode {
>
>       }

**Testing the project**

You should now be able to run the project in Xcode. If you had tried earlier it would have crashed as soon as it launched because it could not find the `Character` and `Piece` classes.

Hit the run button to try it out. You should launch to the `MainScene` we created in SpriteBuilder. Nothing will work yet and there will be a few complaints in the console output but it should launch without crashing.

Its time to start filling in our code!

**What's next**

We'll be building the game's code piece by piece. Our plan is to:

1. Build the sushi tower of obstacles
2. Add touch controls to move the character
3. Randomize the obstacle side
4. Move down the sushi tower with every touch
5. Detect collisions & trigger a game over
6. Get the timer working
7. Update the score

**Building the Sushi Tower**

We'll build the sushi tower up in the `didLoadFromCCB` method so that it's added before `MainScene` is shown to the player.

Our goal is to create a tower ten pieces high. When we start moving that tower down, we'll actually move the bottom piece back to the top of the tower instead of removing it from the scene. This will save a bit of processing power since we are reusing the same objects.

Start off by defining `didLoadFromCCB`.

> [solution]
> It should look like:
>
>       func didLoadFromCCB() {
>
>       }

We'll also need access to one of those code connections we added in SpriteBuilder. Add an instance variable to `MainScene` for `piecesNode`. You should also create an empty Piece array called `pieces`.

> [solution]
> After the opening curly brace in `class MainScene: CCNode {` add:
>
>       var piecesNode: CCNode!
>       var pieces: [Piece] = []
>
> Remember, we always use *implicitly unwrapped optionals* (the !) for code connections from SpriteBuilder.

Inside the `didLoadFromCCB` method, load in ten instances of `Piece.ccb` and position them so they build up a tower. Add each piece as a child of `piecesNode` and use the `contentSizeInPoints` of the piece to calculate the offset in its y-position. Ten instances of `Piece.ccb` should be enough to cover the screen of any device you build on.

Give it a shot then run the project to see if your solution worked.

> [solution]
> One possible way to write `didLoadFromCCB` is:
>
>       for i in 0..<10 {
>           var piece: Piece = CCBReader.load("Piece") as! Piece
>
>           var yPos = piece.contentSizeInPoints.height * CGFloat(i)
>           piece.position = CGPoint(x: 0, y: yPos)
>           piecesNode.addChild(piece)
>           pieces.append(piece)
>       }
>
> In each iteration of the for-loop we load in a new `Piece` from `Piece.ccb`. We then set its x-position to 0 and y-position is set to the iteration number multiplied by `piece.contentSizeInPoints`.
>
> Remember how we manually changed the anchor point and content size of the root node in `Piece.ccb`? Now you may be able to see how those values are making our life a whole lot easier on the code side. The lowest piece is added as a child of `piecesNode` with a position of `(0, 0)` and its exactly where we want it. Each subsequent piece is offset appropriately in the y-direction so they line up into a nice tower.
>
> If your solution is different, make sure it matches the requirements above.

Your game should now look like this when you run it:

![](./Simulator_Pieces_Generated.png)

Once you have it working, move onto adding touch controls.

**Add Touch Controls**

To add touch controls we first need a method we can call to move the `Character` to the right side of the screen and back. We'll be relying on a little trick to make this easy -- setting the `scaleX` to -1 flips a sprite horizontally around its anchor point. Since `Character` already has an anchor point at the center of the screen, we can us this trick to flip it to the other side.

Your job is to create two methods in Character.swift:

1. `left` - move the character to the left side of the screen
2. `right` - move the character to the right side of the screen

We'll trigger these methods from `MainScene` when we add in the touch controls.

> [solution]
> You should have added the following methods to the `Character` class:
>
>       func left() {
>           scaleX = 1
>       }
>
>       func right() {
>           scaleX = -1
>       }

Before we can actually detect touches, we need to complete the code connection for `character` and set `userInteractionEnabled` in the `MainScene` class.

> [action]
> Add this near where you declared the other instance variables:
>       var character: Character!
>
> Add this to the end of `didLoadFromCCB` in `MainScene.swift`:
>
>       userInteractionEnabled = true

Now we can override `touchBegan` and add in our code.

> [action]
> Create and empty `touchBegan` method in `MainScene.swift`:
>
>       override func touchBegan(touch: CCTouch!, withEvent event: CCTouchEvent!) {
>
>       }

Inside `touchBegan`, try using `touch.locationInWorld().x` and `CCDirector.sharedDirector().viewSize().width` to determine which side of the screen a touch is on. Call `character.left()` and `character.right()` to trigger the correct movements.

> [solution]
> The `touchBegan` method should look like:
>
>       var xTouch = touch.locationInWorld().x
>       var screenHalf = CCDirector.sharedDirector().viewSize().width / 2
>       if xTouch < screenHalf {
>           character.left()
>       } else {
>           character.right()
>       }

You should now be able to move the character from one side to another and back:

![](./Simulator_Touch_Detect.gif)

**Randomize Each Obstacle's Side**

Before we start randomizing the obstacle side, we should declare an enum to add uniformity to checking sides in code.

> [action]
> At the top of `MainScene.swift` (before the class declaration) add:
>
>       enum Side {
>           case Left, Right, None
>       }

Now we can refer to the type `Side` with three different values: Left, Right, and None. This will make our lives a lot easier throughout our code.

Let's setup the `Piece` class. We need to complete the code connections to `left` and `right` (our chopstick sprites from SpriteBuilder) and add a `side` variable to keep track of which obstacle is showing. We want this variable to trigger the correct visibility of `left` and `right` each time it changes.

> [action]
> Open up `Piece.swift` and add this to the top of the class declaration:
>
>       var left: CCSprite!
>       var right: CCSprite!
>
>       var side: Side = .None {
>           didSet {
>               left.visible = false
>               right.visible = false
>               if side == .Right {
>                   right.visible = true
>               } else if side == .Left {
>                   left.visible = true
>               }
>           }
>       }

> [info]
> The `didSet` property observer gets called each time the `side` variable is set. This allows us to trigger the correct visibility of `left` and `right`.

We want our obstacles (chopsticks) to randomly choose a side but we need to have a few rules to make sure the game is fair and we do not produce an impossible to pass sequence. Our rules for obstacle generation are:

1. a piece has no obstacle if the previous piece had one
2. right should appear 45% of the time an obstacle can be added
3. left should appear 45% of the time an obstacle can be added
4. no obstacle should appear 10% of the time an obstacle can be added

To make sure our pieces follow these rules, we'll pass in the obstacle side of the previous piece every time we randomize a piece. Time to implement the rules!

> [action]
> Create a method in the `Piece` class as follows:
>
>       func setObstacle(lastSide: Side) -> Side {
>
>       }

Now fill it in according to our four rules. Remember to return the side that is randomly chosen.

> [info]
> You can call `CC_RANDOM_0_1()` to randomly generate a number between 0 and 1.

> [solution]
> The body of `setObstacle` should look like:
>
>       if lastSide != .None {
>           side = .None
>       } else {
>           var rand = CCRANDOM_0_1()
>           if rand < 0.45 {
>               side = .Left
>           } else if rand < 0.9 {
>               side = .Right
>           } else {
>               side = .None
>           }
>       }
>       return side

The `Piece` class is all ready so let's go back to `MainScene.swift`. We need an instance variable `lastPieceSide` to keep track of the last piece's side. This will be especially important when we start moving the sushi tower down with each tap.

> [action]
> Add this near your other instance variables in `MainScene`:
>
>       var pieceLastSide: Side = .Left

We're setting `pieceLastSide` to `Left` so that the bottom piece always has a side of `None`.

> [action]
> Add this below the line with `var piece: Piece = CCBReader.load("Piece") as! Piece` in `didLoadFromCCB`:
>
>       pieceLastSide = piece.setObstacle(pieceLastSide)

This line is pretty powerful. It both randomizes the current piece's obstacle side and updates `pieceLastSide` since we return the chosen side.

Run the game. It should have randomized obstacles now!

![](./Simulator_Random_Obstacles.png)

**Move the Sushi Tower**

Our next goal is to get the sushi tower to move downward with each of the player's taps. Once we get that working we'll be able to add in collision detection!

Early we mentioned that we want the tower to cycle between the ten pieces we have already created. We want to create a `stepTower` method in the `MainScene` class that does the following:

1. moves current piece to top of tower
2. increase its `zOrder` by one
3. randomize its obstacle
4. moves `piecesNode` down by the content size of a piece

We'll add more to the `stepTower` method later on but go ahead and give this version a shot. Remember that you have a `pieces` array. You'll probably want to also create a new index variable, `pieceIndex`, to track the index of the current piece in `pieces`.

> [info]
> Do not rely on `piecesNode.children`. The order of this array is not guaranteed. It is currently optimized for drawing the contents to the screen but could change in the future.

> [solution]
> You should have added a method like this to the `MainScene` class:
>
>       func stepTower() {
>           var piece = pieces[pieceIndex]
>
>           var yDiff = piece.contentSize.height * 10
>           piece.position = ccpAdd(piece.position, CGPoint(x: 0, y: yDiff))
>
>           piece.zOrder = piece.zOrder + 1
>
>           pieceLastSide = piece.setObstacle(pieceLastSide)
>
>           piecesNode.position = ccpSub(piecesNode.position,
>                                        CGPoint(x: 0, y: piece.contentSize.height))
>
>           pieceIndex = (pieceIndex + 1) % 10
>       }
>
> You might have noticed that `pieceIndex` hasn't be declared yet. Make sure to add the following near your other instance variables:
>
>       var pieceIndex: Int = 0

Launch the game and play around a bit. You should have an infinitely looping tower of sushi with randomized obstacles!

![](./Simulator_Randon_Obstacles.gif)

**Detect Collisions and Trigger Game Over**

We already have a variable storing a `Side` in the `Piece` class but we need one in character before we get started. Add a `side` variable to the `Character` class. Also update the `left()` and `right()` methods accordingly.

> [solution]
> Your `Character` class should now look like this:
>
>       var side: Side = .Left
>
>       func left() {
>           side = .Left
>           scaleX = 1
>       }
>
>       func right() {
>           side = .Right
>           scaleX = -1
>       }

Now checking for collisions between the character and obstacles is as easy as comparing their `side` properties!

> [action]
> Add a `gameOver` instance variable to `MainScene` and complete the code connection to `restartButton`:
>
>       var gameOver = false
>       var restartButton: CCButton!
>
> Create a `isGameOver() -> Bool ` method in `MainScene`:
>
>       func isGameOver() -> Bool {
>           var newPiece = pieces[pieceIndex]
>
>           if newPiece.side == character.side { triggerGameOver() }
>
>           return gameOver
>       }
>
> Also create `triggerGameOver()` in `MainScene`:
>
>       func triggerGameOver() {
>           gameOver = true
>           restartButton.visible = true
>       }

The last step is to call `isGameOver()` at the appropriate times. Once a game over is triggered, the restart button will appear!

There are three types of collisions that trigger a game over in Timberman:

1. switch into - player switched into base obstacle
2. head-on - player continued into next obstacle
3. switch into, head-on - player switched into next obstacle

We can catch the first one by checking right after moving the character (in `touchBegan`) and we can catch the others by checking right after the tower is stepped (in `stepTower`).

> [action]
> Add the line below to `touchBegan` right before `stepTower` is called:
>
>       if isGameOver() { return }
>
> Also add it at the end of `stepTower`.

Try out the game now. You'll realize that a restart button appears, but you can still continue to play and the restart button doesn't actually work.

> [action]
> Add the following line to the beginning of `touchBegan`:
>
>       if gameOver { return }
>
> We also need to define the `restart()` method for `restartButton`.
>
>       func restart() {
>           var scene = CCBReader.loadAsScene("MainScene")
>           CCDirector.sharedDirector().replaceScene(scene)
>       }

Now that we short-circuit out of `touchBegan` after a game over, the player can no longer continue playing after a collision. `restart()` has also been defined so we can try again after losing.

The core gameplay is pretty close to completion. The only thing left is the timer and score!

**Get the Timer Working**

In Timberman there is a timer constantly counting down. Every successful move adds a little bit of time. You'll trigger a game over for running out of time if you don't play fast enough.

We are going to use property observers to update the `scaleX` of our `lifeBar` and an `update()` loop to decrement it.

> [action]
> First we need to complete the code connection for `lifeBar`. Add the following to `MainScene` near your instance variables:
>
>       var lifeBar: CCSprite!
>
> We also want to create an instance variable to track time left and setup a property observer on it to change `scaleX` on `lifeBar` whenever it changes. Add the following near your other instance variables:
>
>       var timeLeft: Float = 5 {
>           didSet {
>               timeLeft = max(min(timeLeft, 10), 0)
>               lifeBar.scaleX = timeLeft / Float(10)
>           }
>       }

The `didSet` property observer for `timeLeft` clamps the time between 0 and 10. After that, it sets the `scaleX` of `lifeBar` as a percentage of time left divided by 10 (the maximum amount of buffer the player can build).

> [action]
> Add the following to the end of `stepTower()`:
>
>       timeLeft = timeLeft + 0.25
>
> And add an `update` loop to decrement `timeLeft` and trigger game over if the player runs out of time:
>
>       override func update(delta: CCTime) {
>           if gameOver { return }
>           timeLeft -= Float(delta)
>           if timeLeft == 0 {
>               triggerGameOver()
>           }
>       }

> [info]
> It's safe to check if `timeLeft` is equal to zero since we clamped it in the `didSet` property observer.

The only thing left to do in core gameplay is implementing the score!

**Update the Score**

The score implementation is pretty similar the timer. We'll complete the `scoreLabel` code connection. Then we'll set up an instance variable with a `didSet` property observer to track the score and update the `scoreLabel`. Finally, we'll increment `score` at the end of `stepTower`.

Try and see if you can implement it on your own!

> [solution]
> Add the following to complete the code connection for `scoreLabel`:
>
>       var scoreLabel: CCLabelTTF!
>
> Create a `score` instance variable with a `didSet` property observer to update `scoreLabel`:
>
>       var score: Int = 0 {
>           didSet {
>               scoreLabel.string = "\(score)"
>           }
>       }
>
> Increment score at the end of `touchBegan`:
>
>       score++

Congrats! You have completed the core gameplay for Sushi Neko, a Timberman clone! Continue onto part two to polish up the gameplay :)

![](./Simulator_MVP.gif)
