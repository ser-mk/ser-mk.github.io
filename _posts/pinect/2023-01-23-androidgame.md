---
title: Chapter 6. Android Software. Game
date: 2023-01-23
categories: [Pinect]
tags: [Pinect, android, aidl, game, libgdx, kryonet, java]     # TAG names should always be lowercase
image: http://ser-mk.github.io/assets/Pinect/ch5/preview.png
description: Game developing for Pinect hand-free console. Game engine. Android interface used for capturing the game stream position. 
---

### Linked repositories:
 * [PiLauncher](https://github.com/ser-mk/pilauncher)
 * [PiBall Game](https://github.com/ser-mk/piball)
 * [Build script](https://github.com/ser-mk/pibuild)


In our [previous post](https://ser-mk.github.io/pinect/Pinect-android1nput/), we explored the intricacies of the control service in the Pinect hand-free console and how it expertly captures and processes the gamer's stream position. But what's a control service without a game to put it to use?

<style>
    video {
        height: 100%;
        width: 100%;
        object-fit: cover; // use "cover" to avoid distortion
                position: absolute;

    }
</style>
<center>
<video autoplay loop muted playsinline>
    <source src="/assets/Pinect/ch5/demo2players-cut.webm" type="video/webm">
    Your browser does not support HTML5 video.       
</video>
</center>
    
    

That's why in this post, we're excited to introduce the [2D arcade game for our console](https://github.com/ser-mk/piball) named [**PiBall**](https://github.com/ser-mk/piball). This is a unique blend of football and table tennis, allowing for a fast-paced and exciting two-player local network match. Challenge your friends and see who comes out on top in this thrilling game. The best part is that, on the Pinect console, developing it is just as simple as creating a typical Android application, but what sets it apart is its seamless integration with the control service through the use of [AIDL](https://developer.android.com/guide/components/aidl) (Android Interface Definition Language). This allows for a fast and efficient interaction between the game and the controller service, ensuring a smooth and lag-free gaming experience.

![aidl interface](/assets/Pinect/ch5/aidl.webp)

[Implementing AIDL](https://github.com/ser-mk/pilib/blob/master/src/main/aidl/sermk/pipi/pilib/Pinterface.aidl) in the [controller service is simple](https://github.com/ser-mk/pilauncher/blob/master/app/src/main/java/sermk/pipi/pilauncher/PInterface_Impl.java#L14), it's as easy as creating a Java interface with the **[getPosition](https://github.com/ser-mk/pilauncher/blob/master/app/src/main/java/sermk/pipi/pilauncher/PInterface_Impl.java#L19)** method that returns the gamer's stream position. Once that's done, game developers can easily integrate this interface into their game, allowing them to access the stream position in real-time.

![call method](/assets/Pinect/ch5/call.webp)

[PInterface_Impl.java](https://github.com/ser-mk/pilauncher/blob/master/app/src/main/java/sermk/pipi/pilauncher/PInterface_Impl.java)

[PII_Stub.java](https://github.com/ser-mk/piball/blob/master/android/src/main/ser/pipi/piball/PII_Stub.java)

However, there's more to it than just returning a simple set of coordinates. The "getPosition" method not only returns a value from 0 to 1000 in relative coordinates, but it also includes additional information by returning negative values as well. These negative values convey the current state of the controller service, without the need for extra methods.

Creating a game for the Pinect console is a fairly simple task, as the console runs on the Android operating system. One of the most popular ways to create a game for the Pinect console is to use a game engine or framework. There are many options available, including Unity, Godot, Defold, Corona, Cocos2D, LibGDX etc.

I chose to use the [LibGDX](https://libgdx.com/) framework for my game development, as I am familiar with the Java programming language and all android services in the Pinect project were written by Java. LibGDX being a Java-based framework, it was a natural choice for me to make as it integrates seamlessly with the existing codebase of the Pinect project. Additionally, it provides a comprehensive set of tools for game development, including a 2D renderer, physics engine, and many other features. This made it easy for me to create a high-quality 2D arcade game for the Pinect console.

The architecture of my Piball game is similar to that of other 2D games developed using the LibGDX framework. It consists of several main components, including the game loop and renderer.

In my Piball game, the game loop updates the position, velocity and other attributes of the objects on the screen like the ball and paddles. It also checks if the ball collides with the paddles or the walls, and updates the game state accordingly. The game loop also handles the scoring and the game over conditions. It keeps track of the score, and when the ball goes out of bounds, the game loop updates the score and determine if the game is over.

The game loop source code is located in the **[ser.pipi.piball.engine](https://github.com/ser-mk/piball/tree/master/core/src/java/ser/pipi/piball/engine)** package. This package contains the main game loop and other related classes that are responsible for running the game.

To updating the game state based on user input provided by the controller, the game loop also updates the game state based on the input received from the controller service. For example, if the player's "stream" moves the paddle right or left, the game loop updates the position of the paddle accordingly.

![pinect neighboring](/assets/Pinect/ch5/neighbor.webp)

In the case of my Piball game, one of the neighboring Pinect consoles acts as the server. This device is responsible for maintaining the game state and sending it to the other devices that connect as clients. The clients, in this case, the other neighboring Pinect consoles, are responsible for displaying the game state and sending input back to the server.

The LibGDX framework is a comprehensive toolset for game development, but it does not have built-in tools for network interaction. However, it allows the use of external libraries, such as [kryonet](https://github.com/EsotericSoftware/kryonet), to handle network communication.

In the case of my Piball game, I used the [kryonet](https://github.com/EsotericSoftware/kryonet) library to handle network communication between the neighboring Pinect consoles acting as the server and clients.

Setting up the server or client is done through the game settings, rather than requiring code changes or recompiling the game. This allows for easy and convenient setup without the need for technical knowledge or programming skills.

![flags order](/assets/Pinect/ch5/flags.webp)

To access the game settings menu in the game, players can press the next flag button on the screen in the following sequence: Brazil->Croatia->Japan->Poland.

![piball settings](/assets/Pinect/ch5/settings.webp)

In the settings menu of the PiBall game, players can access many other interesting options that allow them to customize their gaming experience. These options include setting the ball and paddle velocity, ball acceleration, and much more. To learn more about these options, you can check the source code of the [ser.pipi.piball.settings](https://github.com/ser-mk/piball/tree/master/core/src/java/ser/pipi/piball/settings) package.

A match is finished when one of the players press the back button on the navigation bar or leaves. Once the match is finished, the result score will be displayed on the screen.

![piball score of match](/assets/Pinect/ch5/score.webp)

If a player's urination had ended, the player does not have to finish the match. The player can continue to play with the help of the touchscreen. This feature ensures that the game does not end prematurely and that the player can still enjoy the game, even if their’s urination has ended.

Overall, the Piball game serves as a useful template for creating your own game for the Pinect console. The game can also run on desktop, and developers can develop, run and debug their games on their own PC by [ser.pipi.piball.desktop](https://github.com/ser-mk/piball/tree/master/desktop) package. This feature allows for a more convenient and efficient development process, and allows developers to easily test their games on different platforms. It provides a solid foundation and a starting point for your own game development, and the game's features can serve as a guide for your own game development.