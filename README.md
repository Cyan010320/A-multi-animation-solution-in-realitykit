# A-multi-animation-solution-in-realitykit
Offering a solution to USDZ's multiple animation in RealityKit.

As you tried, entity.availableAnimations.count is always 1. The README offers a solution to play multiple animations for a USDZ model in RealityKit. The article based on a fbx model with multiple animations.

1. Import the fbx model to blender, then switch to Dope Sheet > Action Editor. At here, you can see all animations in the fbx model.
2. Choose an animation as your first model, then select the second animation, press crtl+c to copy all the keyframe, then go back to the first animation, select the keyframe which is the end frame of the first animation +1. Then press ctrl+v to paste. You can see, the first animation contains two merged animations now.
3. Repeat step 2, paste all the animations you need into the first animation.
4. Now you get a very long animation. press shift+F9, click `Blender file`, in `action` directory, delete all animation files, except the first animation. (tips: you can use shift+left click to multiple select)
5. Now the fbx model just have a long animation. Export the model to glTF(leave texture alone temporarily). Then you'll get a model with blank texture.
6. Open Reality Converter, drag the model into it, then add texture from your texture directory, at last, export it as USDZ file.
7. Open your RealityKit project, in Experience.rcproject, import your USDZ model. You can set trigger to play animation. (not necessary)
8. Go back to xcode code editing view. Now solve the avaiableAnimations problem. 

First cut the long animation:
```swift
public enum PetAnimation: Int{
    case 伸懒腰 = 0, 趴着, 坐着伸懒腰, 喝水, 吃东西, 什么都不做, 向左看一眼, 趴下伸懒腰, 短跳, 跳下去, 着陆, 助跑大跳, 先走后跳, 蹦高, 蹦更高, 趴下长待机起来, 左转180, 右转180, 左转90, 右转90, 左转45, 右转45, 大跳, 左跃, 右跃, 跳跃收尾, 坐下待机后起身, 坐下左右看, 坐后躺后起身, 坐后舔手, 趴后躺, 向左走, 向右走
}


let FrameFor: [PetAnimation : (Double, Double)] =
    [
        PetAnimation.伸懒腰: (1, 61),
        PetAnimation.趴着: (62, 112),
        PetAnimation.坐着伸懒腰: (113, 173),
        PetAnimation.喝水: (174, 274),
        PetAnimation.吃东西: (275, 376),
        PetAnimation.什么都不做: (377, 510),
        PetAnimation.向左看一眼: (511, 577),
        PetAnimation.趴下伸懒腰: (578, 678),
        PetAnimation.短跳: (679, 714),
        PetAnimation.跳下去: (715, 745),
        PetAnimation.着陆: (768, 782),
        PetAnimation.助跑大跳: (783, 818),
        PetAnimation.先走后跳: (819, 860),
        PetAnimation.蹦高: (861, 896),
        PetAnimation.蹦更高: (897, 944),
        PetAnimation.趴下长待机起来: (945, 1095),
        PetAnimation.左转180: (1096, 1140),
        PetAnimation.右转180: (1141, 1185),
        PetAnimation.左转90: (1186, 1211),
        PetAnimation.右转90: (1212, 1237),
        PetAnimation.左转45: (1238, 1254),
        PetAnimation.右转45: (1255, 1271),
        PetAnimation.大跳: (1272, 1286),
        PetAnimation.左跃: (1287, 1301),
        PetAnimation.右跃: (1302, 1316),
        PetAnimation.跳跃收尾: (1317, 1337),
        PetAnimation.坐下待机后起身: (1338, 1478),
        PetAnimation.坐下左右看: (1479, 1580),
        PetAnimation.坐后躺后起身: (1581, 1630),
        PetAnimation.坐后舔手: (1631, 1731),
        PetAnimation.趴后躺: (1732, 2034),
        PetAnimation.向左走: (2035, 2059),
        PetAnimation.向右走: (2060, 2084)
    ]


public func getAnimationStartAndEndTime(_ animationType: PetAnimation) -> (Double, Double) {
    let frameInterval = FrameFor[animationType]
    let startFrame = frameInterval!.0
    let endFrame = frameInterval!.1
    let startTime = startFrame / 24 + 0.02
    let endTime = endFrame / 24 - 0.02
    return (startTime, endTime)
}

```
the numbers represents start frame and end frame. They can be seen in blender. Then use getAnimationStartAndEndTime to return a tuple.

Then in ViewController:

```swift
        let boxAnchor = try! Experience.loadBox()
        arView.scene.anchors.append(boxAnchor)
        let cat = boxAnchor.findEntity(named: "cat")!.children[0]
        let animation = cat.availableAnimations[0]
        let animationView = AnimationView(source: animation.definition,
                                          name: "ani",
                                          bindTarget: nil,
                                          blendLayer: 0,
                                          repeatMode: .none,
                                          fillMode: [],
                                          trimStart: getAnimationStartAndEndTime(PetAnimation.大跳).0,
                                          trimEnd: getAnimationStartAndEndTime(PetAnimation.大跳).1,
                                          trimDuration: nil,
                                          offset: 0,
                                          delay: 0,
                                          speed: 1.0)
        let resource = try! AnimationResource.generate(with: animationView)
        cat.playAnimation(resource, transitionDuration: 0, startsPaused: false)



```

Debug on your iPhone, you can see the model is playing animation you want to play!

Find me from my profile if you have more problems.
