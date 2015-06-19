---
title: "Finishing up the game"
slug: game-over-screen
---     

Designing the Game Over Dropdown
=============================

The last thing left to do is create a game over dropdown. Like a nice rug in a room, this will help tie the whole game together.

##Laying it out

In Cocos Studio, open *MainScene.csd*. 

> [action]
> First, make sure that *automatic frame recording* is off. Drag *mat.png* on to the scene. Set its *name* to mat. Set the *anchor point* to (0.5, 1.0). For now, so we can see it, set its position to (50%, 100%).

> [action]
Next, drag *gameOver.png* on to the scene. Make it a child of mat by dragging the sprite in the timeline on top of mat. Set the:
- *anchor point* to (0.5, 1.0)
- *position* to (50%, 90%)

> [action]
> 
Now, from the *Widgets* panel, drag two labels (*not* BitmapLabel!). *Name* one gameOverScore, and the other gameOverScoreLabel. Make them both children of mat also.

> [action]
> 
For gameOverScore, set:
- *position* to (50%, 52%)
- *Text* to score
- *Font Size* to 100
- *Font File* to Game of Three.ttf

> [action]
> 
For gameOverScoreLabel, set:
- *position* to (50%, 44%)
- *Text* to 0
- *Font Size* to 100
- *Font File* to Game of Three.ttf

Now drag *button.png* on to the scene. *Name* it play. Set its *position* to (50%, 8.75%).

##Drop down animation

Now its time to set up our down down animation.

> [action]
> With *automatic frame recording* enabled, move the scrubber to frame 0. Uncheck *visibility* for both mat and play.
> 
> Now add a new animation called gameOver. It should start on frame 121, and end on frame 151.
> 
> Move the scrubber to frame 121. Re-enable *visibility* for both mat and play. Set the *position* of mat to (50%, 200%). It should be off the screen.
> 
> Now, on frame 150, set the *position* of mat to (50%, 100%).
> 
> Go back to frame 121 and click the mat keyframe. Set the *animation curve* to Circ_EaseOut.

Try playing it! We have a nice animation.

> [action]
> 
Don't forget to **save and publish** before moving on.

Coding the Game Over Dropdown
==========================

##Triggering the dropdown
Thankfully, triggering the game over animation is as easy as adding some lines of code to the end of `triggerGameOver()`.

Add the following:

    // get a reference to the top-most node
    auto scene = this->getChildByName("Scene");
    
    // get a reference to tha mat sprite
    auto mat = scene->getChildByName("mat");
    
    // get a reference to the game over score label
    cocos2d::ui::Text* gameOverScoreLabel = mat->getChildByName<cocos2d::ui::Text*>("gameOverScoreLabel");
    
    // set the score label to the user's score
    gameOverScoreLabel->setString(std::to_string(this->score));
    
    // load and run the game over animations
    cocostudio::timeline::ActionTimeline* gameOverTimeline = CSLoader::createTimeline("MainScene.csb");
    this->stopAllActions();
    this->runAction(gameOverTimeline);
    gameOverTimeline->play("gameOver", false);

Congrats! You have finished a fully polished clone of Timberman!

<video width="100%" controls>
	<source src="https://s3.amazonaws.com/mgwu-misc/Sushi+Neko+Cpp/finalGameOver.mov" type="video/mp4">
</video>
