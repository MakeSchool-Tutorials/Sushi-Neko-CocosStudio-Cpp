---
title: "Finishing up the game"
slug: game-over-screen
---     

#Designing the Game Over Popup

The last thing left to do is create a game over popup. This will help tie the whole game together.

##Laying it out

We'll want to create a new CCB for our game over screen. This will make it a easy to create an independent animation.

> [action]
> Create a new `node` file in SpriteBuilder named `GameOver`. Set its root node's *content size* to `(320, 480)`. Set its *custom class* to `GameOver`.

Now let's get our resources into this new CCB.

> [action]
> Drag `mat.png` into `GameOver.ccb`. Set its *position* to `(50%, 50%)` with a *position reference corner* of `top left`
> 
> Drag in an effect node as a child of `mat`. Set the node's *content size* to `(100%, 100%)` and *position* to `(0, 0)`. Add a pixelate effect with a *value* of `3`.
> 
> Drag in two LabelTTFs as children of the effect node. For the first label, set its:
> 
> - *positon* to `(50%, 483)`
> - *label text* to `Game`
> - *font name* to `Game of Three.ttf`
> - *font size* to `92`
> - *draw color* to `white`
> - *outline color* to `black`
> - *outline width* to `6`
> 
> For the second LabelTTF, set its:
> 
> - *positon* to `(50%, 415)`
> - *label text* to `Over`
> - other font properties to match the previous label
> 
> Drag two more LabelTTFs in as children of the effect node. For the first, set its:
> 
> - *positon* to `(50%, 299)`
> - *label text* to `Score`
> - *font name* to `Game of Three.ttf`
> - *font size* to `42`
> - *draw color* to `white`
> - *outline color* to `black`
> - *outline width* to `5`
> 
> For the second LabelTTF, set its:
> 
> - *positon* to `(50%, 249)`
> - *label text* to `0`
> - *font name* to `Game of Three.ttf`
> - *font size* to `64`
> - *draw color* to `white`
> - *outline color* to `black`
> - *outline width* to `5`
> - `doc root var` code connection to `scoreLabel`
> 
> Drag in a button as a child of the root node. Set its:
> 
> - *position* to `(50%, 42)`
> - *preferred size* to `(101, 63)`
> - *sprite frame* to `button.png` for ALL states
> - *selector* to `restart` with a *target* of `owner`

> [info]
> Setting the *target* to `owner` allows you to call selectors on a class that is different than of the root node's class. You set the owner when you load in the CCB from code.

##Drop down animation

Now its time to setup our down down animation.

> [action]
> Create position keyframes on the `mat` at `00:00:00` and `00:00:10`. Change the first keyframe's *value* to `(50%, -50%)`. Add `Ease Out` interpolation to the animation.

Finally, we want the game over popup to animate correctly on all screen sizes so we'll have to change the root node's *content size*. We waited until the end because the animation will no longer preview correctly in SpriteBuilder once we do this.

> [action]
> Change the root node's *content size* to `(100%, 100%)`.

The content size will now be inherited from its parent. This will be the full screen size once we load it in from code.

#Coding the Game Over Popup

##Triggering the popup
First we'll need to create a GameOver class. 

> [action]
> Create a new swift file named `GameOver`.
> 
> Add the following code:
> 
>       class GameOver: CCNode {
>           var scoreLabel: CCLabelTTF!
>           var score: Int = 0 {
>               didSet {
>                   scoreLabel.string = "\(score)"
>               }
>           }
> 
>           var restartButton: CCButton!
> 
>           func didLoadFromCCB() {
>               restartButton.cascadeOpacityEnabled = true
>               restartButton.runAction(CCActionFadeIn(duration: 0.3))
>           }
>       }

This code sets up the code connections we added in SpriteBuilder. It also creates a score property that we will be able to set from `MainScene` to update the `scoreLabel`. We also fade in the `restartButton` for a bit of an added effect. Again, we would do this from SpriteBuilder if it was possible.

> [action]
> Open `MainScene.swift`.
> 
> Change `triggerGameOver` to:
> 
>       func triggerGameOver() {
>           gameState = .GameOver
>
>           var gameOverScreen: GameOver = CCBReader.load("GameOver", owner: self) as! GameOver
>           gameOverScreen.score = score
>           self.addChild(gameOverScreen)
>       }

> [info]
> The restart button will trigger `restart()` in `MainScene` since we set its owner as `self`.

Congrats! You have finished a fully polished clone of Timberman!