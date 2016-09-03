---
layout: post
title:  "在cocos2d-x项目里获取iphone的转角信息（陀螺仪）"
date:   2016-08-29 11:03:40 +0900
categories: tech
---
> 今天尝试一下用英文写，代码多文章确实用英文直接表述更方便，可自己的英文叙事水平也比较弱(当然发现中文也没好到哪里去)，这点写博时很纠结。。。
> 本篇讲的是iOS中CoreMotion框架里的加速计(Accelerometer)和陀螺仪(Gyroscope)在cocos2d-x里用法。

### Gyroscope sensor

This time I encountered a request from a client that they want to get the gyro info in a cocos2d-x game. For this scenario we can use iphone's built-in sensor: gyroscope to measures the rate of rotation around the three axes.

### Accelerometer in cocos2d-x

In cocos2d-x there is public access to the accelerometer sensor, and we can find the example in `cpp-test/Class/PhysicsTest/PhysicsTest.cpp`, class name `PhysicsDemoClickAdd`:

```cpp
Device::setAccelerometerEnabled(true);
auto accListener = EventListenerAcceleration::create(CC_CALLBACK_2(PhysicsDemoClickAdd::onAcceleration, this));
_eventDispatcher->addEventListenerWithSceneGraphPriority(accListener, this);
```

However, this is dealing with the `accelerometer`, the acceleration of three axes, which is not I want. I want the access of gyroscope, measuring the attitudes of Yaw, Roll and Pitch, which is not provided in cocos2d-x's library.

The CMDeviceMotion class in iOS Core Motion framework encapsulates measurements of the attitude, and I decide to use directly in my cocos2d code.

### Euler Angles and Yaw, Roll, Pitch
These three terms come from flight dynamics, server as critical parameters of angles of rotation in three dimensions. 
<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/c/c1/Yaw_Axis_Corrected.svg/2000px-Yaw_Axis_Corrected.svg.png">

### Put them into cocos2d-x C++ code

Just take a sample cocos2d-x project for example, I am using version 3.8 and generated a project using:

```shell-session
cocos new my-gyro -l cpp
```

I am going to use `CMMotionManager` in the class of `HelloWorldScene.cpp`. The reference example of Apple's [Motion Event] is written is objective-c, how can it be put into our cocos2d-x C++ source?

Actually  Apple provides Objective-C++ as a convenient mechanism for mixing Objective-C code with C++ code! But wait, what I have is a C++ code `HelloWorldScene.cpp`, not compatible with objective-c... Solution is simple, just change the file name to `HelloWorldScene.mm` to make it become an objective-C++. Now it becomes an objective-C++ file only contains C++ code, and I can insert any objective-c logic into it.

Here is my `HelloWorldScene.mm`

```cpp
#include "HelloWorldScene.h"
#import <CoreMotion/CoreMotion.h>

USING_NS_CC;

Scene* HelloWorld::createScene()
{
    // 'scene' is an autorelease object
    auto scene = Scene::create();
    
    // 'layer' is an autorelease object
    auto layer = HelloWorld::create();

    // add layer as a child to scene
    scene->addChild(layer);

    // return the scene
    return scene;
}

// on "init" you need to initialize your instance
bool HelloWorld::init()
{
    //////////////////////////////
    // 1. super init first
    if ( !Layer::init() )
    {
        return false;
    }
    initDeviceMotion();
    schedule(CC_SCHEDULE_SELECTOR(HelloWorld::doStep));
    
    return true;
}

void HelloWorld::initDeviceMotion() {
    //////////////////////////////
    // Here is the usage of CMMotionManager in objective-c
    motionManager = [[CMMotionManager alloc] init];
    referenceAttitude = nil;
    
    CMDeviceMotion *deviceMotion = ((CMMotionManager*)motionManager).deviceMotion;
    CMAttitude *attitude = deviceMotion.attitude;
    referenceAttitude = [attitude retain];
    [(CMMotionManager*)motionManager startDeviceMotionUpdates];
}

void HelloWorld::doStep(float delta) {
    _count++;
    if (_count % 10 == 0) {
        CMDeviceMotion *deviceMotion = ((CMMotionManager*)motionManager).deviceMotion;
        CMAttitude *attitude = deviceMotion.attitude;
        cocos2d::log("gyro yaw: %f ", attitude.yaw);
    }
}
```

Notice in `HelloWorld::initDeviceMotion` I created a CMMotionManager to get the device attitude, which is used to get yaw value during frame update `HelloWorld::doStep`.

[Motion Event]: <https://developer.apple.com/library/ios/documentation/EventHandling/Conceptual/EventHandlingiPhoneOS/motion_event_basics/motion_event_basics.html#//apple_ref/doc/uid/TP40009541-CH6-SW23>