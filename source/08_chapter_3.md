# Method and Implementation

## Scene Preparation

Many 3D models can be found online. One of the popular websites is Free3D (https://free3d.com). In the furniture category, there are thousands of 3D models in commonly-used file formats, like *.obj*, *.dae*, *.blend*, etc. Besides, there are many sub-categories, such as lamps, desks, beds, etc. so it is user-friendly. I also used other websites like TurboSquid (https://www.turbosquid.com), CGTrader (https://www.cgtrader.com) which also provide many 3D models. 

In this project, I download all 3D models online in `Collada (.dae)` file format. Collada, which stands for Collaborative Design Activity, defines an XML-based schema. The XML-based schema states clearly the geometry information about each corresponding 3D model, such as width, height, depth, position, Euler, scale, material, etc. The XML-based schema follows an open standard, so it makes Collada an interchange file format for 3D applications. Collada can transport 3D assets between applications more easily. Figure 3.1 shows a Collada file.

![Collada file](https://lh3.googleusercontent.com/Eqhl5FbzwrgTl4FgOdWXJa6mukICJ-UiVeczaDTKpZvbiPwEQK-Zh8jAN5FGf5e28vmjTbpq4x0)

In this project, I use Xcode SceneKit Editor to edit the scenes. I convert the Collada file to a SceneKit file. I copy the Collada file to the `art.scnassets` folder, which is the default SceneKit Catalog. Then, I click the file, so the corresponding scene graph of the file will be displayed in the editor. In the menu bar, I click "Editor", then "Convert to SceneKit file format (.scn)", as shown in Figure 3.2, so a scene file will be generated.

![Convert the Collada file to a SceneKit file](https://lh3.googleusercontent.com/VHVtcisVLJhVAYuh5Md1_5jCmjcPxoAXTuYQd2S-Q5QKC-cAp89xhnA8cWMmj5slO1uj4yLJHhI)


After the conversion, first, I remove all the unused camera and light nodes. Then, I set the material of the nodes in the material inspector. The details of the material setting can be found below at "3.6.3 Physically Based Rendering". Next, in the scene graph, I create an empty node and the editor will add it to the center of our scene by default, as shown in Figure 3.3. Both the default position and Euler are (0, 0, 0) for (x, y, z). I rename it to "Body" and drag all other nodes underneath it. This newly-created node can act as a base node of the scene. It can group the nodes, so positioning or scaling the nodes will be easier as I do not need to perform those actions to the nodes one by one. Instead, I can perform those actions to the base node.

![The base node is added to the scene](https://lh3.googleusercontent.com/HQfJTt1Y1magF1xJxtUrEuHKWZL8vifCbHWby6rHg_2BdzR9NnPlCBCMyM1PQITJIbySQV5DUJw)

The bottom of a 3D object should be positioned exactly on top of the detected surface. SceneKit operates in a “Y-up” system as shown in Figure 3.4, which means that the y-axis is pointing up. Each node shows a coordinate system as shown in Figure 3.5. Dragging upwards or downwards the arrow pointing up is changing the y-value of the object's position. The object will move up or down. I select all the nodes at the same time and dragged them upwards or downwards until the bottom of the object is touching the grid line in the editor. Eventually, the scene file is now ready to be used in ARKit.

![SceneKit coordinate system](https://lh3.googleusercontent.com/JT85DQ8qf7UgfrvsyFyrn_Cj89jy4RMHnGYJweWFCJZZbuvGJSmncvXhnp-frlN4aJXK_v2dutQ)

![The coordinate of the base node](https://lh3.googleusercontent.com/VhXKTZ2kmsmy0_7xq0iAkylfD7rxCZMtswkw3XVqNX7SxNBHgNI6HXh3Vm1joP2qMykNzX9BowY)

## Plane Detection

Before augmenting the virtual content, the first step of this application is to detect any flat surface, i.e. plane, in the real world. ARKit not only detects planes in the real world, but also recognizes many types of real-world surfaces on supported iOS devices that have an A9 or later processor. I will utilize this feature in my application, as shown in Figure 3.6.

Related class of this function:

- `PlaneViewController` class
- `Plane` class

![The application is performing plane detection with "Seat" detected](https://lh3.googleusercontent.com/IP6iHTKlhMsk57spyEpGX6YCX397yIIwZAltJR_atD4MFQXBnnQcuNq5Ggx-312NHRJZcLcfP00)

### Session Configuration

`ARSession` class is the basis of controlling an AR experience. It is the main object for coordinating the major processes that ARKit performs to provide an AR experience.

First, in order to run an `ARSession` class, I provide a session configuration. In the `viewWillAppear(_ animated: Bool)` method of the `PlaneViewController` class, which is called every time the view is displayed, a session configuration is created using `ARWorldTrackingConfiguration`  class that is a sub-class of `ARConfiguration` class. `ARWorldTrackingConfiguration`  class can track the iOS device's position and orientation. The relationship between ARSession and ARConfiguration subclass is shown in Figure 3.7. 

![The relationship between ARSession and ARConfiguration subclass](https://lh3.googleusercontent.com/XqSTC25m2J6UWg-s8KARLm15Kbk7vH468H7nK7mbcE9CipL9HlI7NAfdOL1i0V6HdFd9tEE8a0E)

Then, the `planeDetection` property, which is a type property under `ARWorldTrackingConfiguration` class, is used. This type property can find the real-world horizontal or vertical surfaces, and add them to the session as the `ARPlaneAnchor` objects. 

Next, I define the value of the `planeDetection` property as both `.horizontal` and `.vertical`, so the session will attempt to automatically detect the horizontal and vertical flat planes in the live camera scene. 

At last, the view's session is run by calling the `run(_:options:)` instance method.

```java
override func viewWillAppear(_ animated: Bool) {

    super.viewWillAppear(animated)

    // Create a session configuration
    let configuration = ARWorldTrackingConfiguration()

    // Run the plane detection for horizontal and vertical planes
    configuration.planeDetection = [.horizontal, .vertical]

    // Enable the view's light estimation
    configuration.isLightEstimationEnabled = true

    // Run the view's session
    sceneView.session.run(configuration)

    // Set a delegate to track the number of plane anchors  
    // for providing UI feedback
    sceneView.session.delegate = self

    // Prevent the screen from being dimmed after a while
    UIApplication.shared.isIdleTimerDisabled = true

}
```

### Plane Visualization and Classification

To visualize the planes detected, that is what the user can see a rectangle, the `Plane` class is created, which inherits from the `SCNNode` class. 

In the `Plane` class's initializer `init(anchor: ARPlaneAnchor, in sceneView: ARSCNView)`, two nodes which are a mesh node to visualize the detected plane's estimated shape and an extent node to visualize the detected plane's bounding rectangle are created. They are the main components of plane visualization.
 The mesh node's geometry is the `ARSCNPlaneGeometry` class while the extent node's geometry is the `SCNPlane` class with the width of the x-value of the anchor's extent and the height of the z-value of the anchor's extent. 

Then, because the extent node's geometry, which is the `SCNPlane` class, is vertically oriented in its local coordinate system, so it is rotated to let the extent node match the orientation of `ARPlaneAnchor`. 

At last, the two nodes are added as the child nodes using an instance method, which is `addChildNode(_:)`, so they appear in the scene. 

```java
// Create a mesh node to visualize the plane's estimated shape

guard let meshGeometry = ARSCNPlaneGeometry(device: sceneView.device!)
    else { fatalError("Can't create plane geometry") }

meshGeometry.update(from: anchor.geometry)

meshNode = SCNNode(geometry: meshGeometry)

// Create an extent node to visualize the plane's bounding rectangle

let extentPlane: SCNPlane = SCNPlane(width: CGFloat(anchor.extent.x), 
                                height: CGFloat(anchor.extent.z))
extentNode = SCNNode(geometry: extentPlane)
extentNode.simdPosition = anchor.center
extentNode.eulerAngles.x = -.pi / 2

super.init()
self.setupMeshNodeVisualStyle()
self.setupExtentNodeVisualStyle()

// Add the mesh node and extent node as the child nodes 
// so they appear in the scene
addChildNode(meshNode)
addChildNode(extentNode)
```

Besides, a text node showing the plane type of each detected plane is created and added as a child node of the extent node. It is center-aligned in the bounding rectangle. 

The plane classification feature is only available in iOS 12.0 or above, so there is a version checking about the iOS device's version that is `if #available(iOS 12.0, \*), ARPlaneAnchor.isClassificationSupported`. `ARPlaneAnchor.Classification` is an enumeration in which the plane anchor's classification can be got. ARKit provides 7 types, which include *wall*, *floor*, *ceiling*, *table*, *seat*, *window* and *door*, for plane classification.

```java
// Create a text node to show the plane type

if #available(iOS 12.0, *), ARPlaneAnchor.isClassificationSupported {

    let classification = anchor.classification.description

    let textNode = self.setupTextNodeVisualStyle(classification)

    classificationNode = textNode

    // Change the pivot of the text node to its center
    textNode.centerAlign()

    // Add the text node as a child node of the extent node
    extentNode.addChildNode(textNode)
    
}
```

Also, there are three private methods for setting up the visual style of the above mesh node, extent node and text node. Mainly, the mesh node and extent node will be made as semi-transparent by lowering the opacity's value to 0.6 so that the real-world placement can be clearly shown.

Moreover, because of the session configuration, ARKit will automatically add and update anchors in a session. In the `PlaneViewController` class, there is an optional instance method called `renderer(_:didAdd:for:)`. The view calls this method once for each new *ARPlaneAnchor* which had been added to the scene. In the `renderer(_:didAdd:for:)` method, the `Plane` class as mentioned above is instantiated each time with the `ARPlaneAnchor` detected and the current scene view to visualize the plane's mesh and extent. Then, the plane created is added as the child node using an instance method, which is `addChildNode(_:)`, so they appear in the scene. Additionally, also in the `PlaneViewController` class, there is another optional instance method `renderer(_:didUpdate:for:)`. The view calls this method once for each updated `ARPlaneAnchor` in the scene. The method content is similar to the `renderer(_:didAdd:for:)` method above.

## Furniture Search
In this project, I developed three search methods, which are by keyword, by measurement and by plane selection, as shown in Figure 3.8-3.10.

![Search Furniture by Keyword](https://lh3.googleusercontent.com/LlIfwe1qnvEJKFysuTuRKetUlYUQfU3OMekbjKZNoGHeK30q-J9nvv8kdZXYXK92THmyk4Q3RbI)

![Search Furniture by Measurement](https://lh3.googleusercontent.com/AeGlI1bjqimEJoiRSGbmNieTl-dNZKNYX0b2x6DGJpvcDkwvhtlZoXF0Uc3NK6Rd7jSJ-_Q4Img)

![Search Furniture by Plane Selection](https://lh3.googleusercontent.com/H_9yLKpOhobY0rmWXqs-fEn-p6BOoBKk1gguWhrf_Y1qG3yNYwqcDXSEYjJJ1qdb4kE8JfXOfw8)

### Simple Search for Furniture by Keyword

Related class of this function:

- `HomeViewController` class
- `Furniture` class

This function is done by the `HomeViewController` class. First, I create a variable which inherits from `UISearchController` class. It is the sub-class of `UIViewController` class. I added a `searchBar` property, which is an instance property of `UISearchController` class, in the `tableHeaderView`. 

``` java
tableView.tableHeaderView = controller.searchBar
```

Then, I create a method called `updateSearchResults(for searchController: UISearchController)`. In this method, the Realm will retrieve the data in the `Furniture.realm` through the `Furniture` class, then the Realm result will be filtered by the text of the search bar.

``` java
// Read some data from the Realm

filteredRealmResults = realm.objects(Furniture.self)
        .filter("name CONTAINS[c] '\(searchController.searchBar.text!)'")
```

To show the filtered result, in the instance method `tableView(_:cellForRowAt:)`, there is a checking of whether the search controller is active, which is `if (resultSearchController.isActive)`. When it is active, the filtered result will be displayed.

### Search for Furniture by Measurement

Related class of this function:

- `MeasurePlaneViewController` class
- `MeasureHomeTableViewController` class
- `MeasureSearchResultTableViewController` class
- `Grid` class
- `Box` class
- `Measurement` class
- `Furniture` class

#### Session Configuration

First of all, plane detection would be enabled. Similar to the implementation of plane detection above, in the `viewWillAppear(_ animated: Bool)` function of the `MeasurePlaneViewController` class, a session with session configuration of plane detection will be created and run.

#### Grid Visualization

To visualize each plane detected as a grid, the `Grid` class is created, which inherits from the `SCNNode` class. It is similar to the `Plane` class as mentioned above. 

In the `Grid` class's initializer `init(anchor: ARPlaneAnchor)`, a grid node to visualize the detected plane's estimated shape is created. It is the main component of grid visualization. The grid node's geometry is the `SCNPlane` class with the width of the x-value of the anchor's extent and the height of the z-value of the anchor's extent. 

Also, I set up the grid's material using the `SCNMaterial` class. I set the diffuse contents of the material as an image of grid as it is more convenient for the users to make a point on a gird for the start and end position of the distance they want to measure.

Besides, the `physicsBody` of the grid node is the `SCNPhysicsBody` class. Its type is *.static*, and its shape is the `SCNPhysicsShape` class. Also, the `categoryBitMask` of the `physicsBody` of this node is equal to 2, which means that this body is static, as the default value of `categoryBitMask` is 1, which means that the body is dynamic.

Moreover, I will use `SCNVector3Make(_:_:_:)` to set the grid node's position. The x-value is the x-vlaue of the anchor's center, while the z-value is the z-vlaue of the anchor's center. The y-value is 0. It will return a three-component vector. 

Next, same as before, because the grid node's geometry, which is the `SCNPlane` class, is vertically oriented in its local coordinate system, so it is rotated to let the extent node match the orientation of `ARPlaneAnchor`.

At last, the grid node is ready and it is added as the child node using an instance method, which is `addChildNode(_:)`, so it appears in the scene.

```java
// Create a plane
gridPlane = SCNPlane(width: CGFloat(anchor.extent.x), 
                height: CGFloat(anchor.extent.z))

// Set up the grid's material
let material = SCNMaterial()
material.diffuse.contents = UIImage(named: "overlay_grid.png")

gridPlane.materials = [material]

// Create a grid node to visualize the plane's estimated shape
gridNode = SCNNode(geometry: gridPlane)

// Set up the physical body of the grid node
gridNode.physicsBody = SCNPhysicsBody(type: .static, 
        shape: SCNPhysicsShape(geometry: gridPlane, options: nil))
gridNode.physicsBody?.categoryBitMask = 2

// Set up the position of the grid node
gridNode.position = SCNVector3Make(anchor.center.x, 0, anchor.center.z)
gridNode.transform 
    = SCNMatrix4MakeRotation(Float(-Double.pi / 2.0), 1.0, 0.0, 0.0)

super.init()

// Add the grid node as the child node so it appears in the scene
addChildNode(gridNode)
```

Similar to the implementation of plane detection above, in the the `MeasurePlaneViewController` class, two optional instance methods, which are `renderer(_:didAdd:for:)` and `renderer(_:didUpdate:for:)`, will be run to add and update the grids in the scene view.

#### Measuring Path Visualization

The measuring path, as shown in Figure 3.11, will being drawn when the user is measuring the reality.

First, the `Box` class is created for visualization of the measuring path. It inherits from the `SCNNode` class. In this class, first, there is a method for creating a box node. The geometry of the box node is the `SCNBox` class, which is a sub-class of `SCNGeometry`. It is a six-sided polyhedron geometry whose faces are all rectangles. It is created with the specified width, height and length and chamfer radius. The chamfer radius is the radius of the curvature for the edges and corners of the box. It is 0 here, which means that the box will not have any round edge. The material of the box is assigned as uniform shading with `UIColor` of white. 

Then, the box node is ready and it is added as the child node using an instance method, which is `addChildNode(_:)`, so it appears in the scene.

```java
func makeBoxNode() -> SCNNode {

let box = SCNBox(width: 0.005, height: 0.005, length: 0.005, chamferRadius: 0)

    for material in box.materials {
        material.lightingModel = .constant
        material.diffuse.contents = UIColor.white
    }

    let node = SCNNode(geometry: box)
    self.addChildNode(node)

    return node
}
```

Besides, in the `viewDidLoad()` method of the `MeasurePlaneViewController` class, the box node is instantiated using the `Box` class and also added as the child node using an instance method, which is `addChildNode(_:)`, so it appears in the scene. But it is hidden before the user starts the measurement. 

To know when the user starts the measurement, I use a variable called `mode` to manage the state of waiting for measuring and measuring. I also create an enumeration, `Mode`, to contain two cases, `waitingForMeasuring` and `measuring`. The variable `mode` is `waiting for measuring` at the beginning. When the user starts the measurement by switching on the switch, the variable `mode` will change from `waiting for measuring` to `measuring`, so the property observer `didset` of `mode` will get called because new value is assigned to `mode`. Then, the private method `update(minExtents: SCNVector3, maxExtents: SCNVector3)` will call to update both the minimum and maximum extent position of the box to `SCNVector3Zero`, which means that every component of the vector is 0.0. It is to give an initial position to the box node. Also, the box, which is the measuring path, will become visible in the view.

```java
enum Mode {

    case waitingForMeasuring
    case measuring

}

var mode: Mode = .waitingForMeasuring {

    didSet {

        switch mode {

            case .waitingForMeasuring:
            status = "NOT READY"

            case .measuring:

            box.update(minExtents: SCNVector3Zero, maxExtents: SCNVector3Zero)
            box.isHidden = false

            startPosition = nil
            distance = 0.0

            setStatusText()
        }

    }

}
```

Moreover, the measuring path will move according to the device's movement when the user is measuring the reality. Therefore, `resizeTo(extent: Float)` method is needed to update the path's size, especially the length.

```java
func resizeTo(extent: Float) {

    var (min, max) = boundingBox

    max.x = extent

    update(minExtents: min, maxExtents: max)

}
```

The user will change direction when measuring the reality, the path's direction will also change, so `calculateAngleInRadians(from: SCNVector3, to: SCNVector3) -> Float` method is needed to method to get the direction changes and the rotation value of the path can be set with the angle in redials the method returned.

```java
func calculateAngleInRadians(from: SCNVector3, to: SCNVector3) -> Float {

    let x = from.x - to.x
    let z = from.z - to.z

    return atan2(z, x)

}
```  

![The measuring path and the distance result](https://lh3.googleusercontent.com/dzhwI5qQQUeyQL_RLlvJl2a28ye2zmQ_UT5NfQ7BnWbvBf-2hhhWa8XtdCAV1KYP8g0sk4Bw4lI)

#### Calculation of Distance

In the `MeasurePlaneViewController` class, I use the `measure(time: TimeInterval)` method to perform the measurement and distance calculation. The two variables, `startPosition` and `worldPosition` are the main parts of the measurement and distance calculation.

First of all, the `startPosition` is the screen center. The screen center can be found with `midX` and `midY` of scene view's bounds. A hit test is used to search for whether the screen center is within any existing plane's area. If the screen center is within any existing plane's area, given that the user has already switched on the switch and the `mode` is `measuring`, the measurement starts. 

Then, the `worldPosition` will be got by `SCNVector3Make(_:_:_:)`. As the user will keep on moving the iOS device, like the iPhone, when measuring, which means that the screen center will keep on moving corresponding to the movement of the device, so the hit test result will keep updating and setting the variable `worldPosition` as the hit test location simultaneously. Also, the distance will keep on being calculated and updated at the same time. The calculation is using the Pythagoras Theorem. When the user stops measuring by switching off the switch, the update of the hit test result and the distance calculation will stop. We can get the final distance eventually.

```java
func measure(time: TimeInterval) {

    let screenCenter: CGPoint = CGPoint(x: self.sceneView.bounds.midX,   
        y: self.sceneView.bounds.midY)

    let planeTestResults = sceneView.hitTest(screenCenter,   
        types: [.existingPlaneUsingExtent])

    if planeTestResults.first != nil {

        if let result = planeTestResults.first {

            status = "READY"

            if mode == .measuring {

                status = "MEASURING"

                let worldPosition =  
                    SCNVector3Make(result.worldTransform.columns.3.x,   result.worldTransform.columns.3.y,  
                    result.worldTransform.columns.3.z)

                if startPosition == nil {
                    startPosition = worldPosition
                    box.position = worldPosition
                }

                distance =  
                    calculateDistance(from: startPosition!,  
                    to: worldPosition)
                    
                box.resizeTo(extent: distance)

                let angleInRadians =  
                    calculateAngleInRadians(from: startPosition!,  
                    to: worldPosition)
                    
                box.rotation =  
                    SCNVector4(x: 0, y: 1, z: 0,  
                    w: -(angleInRadians + Float.pi))

            }

        } else {

            status = "NOT READY"

        }

    }

    setStatusText()

}
```

```java
func calculateDistance(from: SCNVector3, to: SCNVector3) -> Float {

    let x = from.x - to.x
    let y = from.y - to.y
    let z = from.z - to.z

    return sqrtf( (x * _x) + (y_ * y) + (z * z))
}
```

The measure() method is called in the optional instance method, `renderer(_:updateAtTime:)` in the `MeasurePlaneViewController` class. SceneKit calls this method exactly once per frame so that the distance will be updated per frame. Additionally, this method is called asynchronously using `DispatchQueue.main.async` to perform the heavy calculation task without slowing down the UI.

```java
func renderer(_ renderer: SCNSceneRenderer, updateAtTime time: TimeInterval) {

    // Call the heavy method asynchronously without slowing down the UI
    DispatchQueue.main.async {
        self.measure(time: time)
    }
}
```

#### Search Result Filtering

The final measured distance will be saved to the temporary Realm using the `Measurement` class which is the Realm data model, and passed to the `MeasureHomeTableViewController` class using `segue.destination`. The `MeasureHomeTableViewController` class manages the scene showing a table view of the "Search", "Length", "Width", "Height" button. The user tells the application which type of distance he is going to measure by clicking different buttons.

![The home page of measuring](https://lh3.googleusercontent.com/AXGpMWgH4Yn02w_LhxXyXDWWNGPt-5lxs-DHoim0ONr9-A461gM7T3hO5Tw4YO4PTCfGfmaTlCM)

When the user clicks the "Search" button, as shown in Figure 3.12, and given that there is at least one distance of length, width and height are saved into Realm, the distance(s) will be the filtering parameter for searching for furniture and passed using `segue.destination` to the `MeasureSearchResultTableViewController` class as the measurement for searching for furniture. The Realm will retrieve the data in the `Furniture.realm` through the `Furniture` class, and the Realm result will be filtered by the length, width and/or height and the final result will be displayed.

### Search for Furniture by Plane Selection

Related class of this function:

- `PlaneViewController` class
- `PlaneTypeSearchResultTableViewController` class
- `Plane` class
- `Furniture` class

As mentioned above, ARKit provides plane classification for the real-world surfaces. ARKit can recognize 7 surfaces in total, namely *wall*, *floor*, *ceiling*, *table*, *seat*, *window* and *door*. When the plane anchor information is not enough for classification, the classification will return *none*. This feature is only available in iOS 12.0 or above, so in the `Plane` class, there is a version checking about the iOS device's version that is `@available(iOS 12.0, *)` before the extension of enumeration `ARPlaneAnchor.Classification`.

```java
@available(iOS 12.0, *)
extension ARPlaneAnchor.Classification {

    var description: String {

        switch self {

            case .wall:
            return "Wall"

            case .floor:
            return "Floor"

            case .ceiling:
            return "Ceiling"

            case .table:
            return "Table"

            case .seat:
            return "Seat"

            case .window:
            return "Window"

            case .door:
            return "Door"

            case .none(.unknown):
            return "Unknown"

            default:

            return ""

        }
    }
}
```

When the user taps the scene view, the touch location will be got in the `PlaneViewController` class. The hit test of `existingPlaneUsingExtent` will search for any point which lies within the area defined by the plane's center and extent properties in the touch location. If the hit test successfully gets a point, which means that the user has tapped an existing plane area, an object is created using the `Plane` class and the anchor as `ARPlaneAnchor` of the hit test result will be got.

```java
if let planeAnchor = result.anchor as? ARPlaneAnchor {

    if planeAnchor.classification.description != "Unknown" &&  
    
       planeAnchor.classification.description != "" {

        type = planeAnchor.classification.description

        // Show the plane type
        planeType.text = "This is \(type)"

        // Show the search button
        search.isHidden = false

    }

}
```

Each `ARPlaneAnchor` has its classification description as specified in the `Plane` class. The classification description will be passed to the `PlaneTypeSearchResultTableViewController` class as the plane type for searching for furniture using `segue.destination`. 

When the user clicks the "Search" button as shown in Figure 3.13 in the  
`PlaneTypeSearchResultTableViewController` class, the Realm will retrieve the data in the  
`Furniture.realm` through the `Furniture` class, then the Realm result will be filtered by the plane type and the final result will be displayed.

![The "Search" button is located at the top right corner](https://lh3.googleusercontent.com/F00RIQVEWooVwUD6j9-i-FH9W10scFRUvJRyKmhZM8wlrSR94JjBL5q_pfTcnEJfjZByAqJLmac)

```java
// Read some data from the Realm

realmResults =  
    realmFurniture.objects(Furniture.self).filter("type == '\(self.type)'")
```

## Virtual Content Augmentation

### Insertion of Virtual Content into the Camera Scene

Related class of this function:

- `PlaneViewController` class
- `DetailViewController` class

First of all, when the user has already searched for at least one furniture object, he can go to the corresponding detail page of that object, as shown in Figure 3.14. In the detail page, he can click the "Place" button at the bottom of the furniture photo, and in the `DetailViewController` class the furniture ID will be passed to the `PlaneViewController` class using `segue.destination`.

In `Main.StoryBoard`, the `Tap Gesture Recognizer` object is added to the `Plane Scene`. It is also added to the referencing outlet collection of the scene view . A sent action `@IBAction`, which is called `screenTapped`, is created to send an action to the corresponding custom class of the `Plane Scene`, which is the `PlaneViewController` class, as a method. In the `PlaneViewController`, the `sender` of the `@IBAction` is `UITapGestureRecognizer`.

In the `@IBAction`, which is the `screenTapped(_ sender: UITapGestureRecognizer)` method, when the user taps the scene view, the touch location will be got. The hit test of `existingPlaneUsingExtent` will search for any point which lies within the area defined by the plane's center and extent properties in the touch location. If the hit test successfully gets a point, which means that the user has tapped an existing plane area, the anchor as `ARPlaneAnchor` of the hit test result will be got and the function `addFurniture(result: ARHitTestResult)` will be run. In this function, a container called `furnitureScene`, which inherits from the `SCNScene` class, is created to specify the 3D content's location. Here, the location is `art.scnassets/scene/\(sceneName).scn`, in which the scene name is the furniture name actually, and it is got using the furniture ID passed from the `DetailViewController` class. When the 3D content is got, a variable called `furnitureNode` is created with the `Body` node of the 3D content that the `furnitureScene` got.

Next, the position of the `furnitureNode` will be set by using the above hit test result to get the `worldTransform`, which is a matrix showing the position and orientation of the hit test result relative to the world coordinate system. The position of the `furnitureNode` is the x-value, y-value and z-value of column 3 of the matrix got.

Finally, the `furnitureNode` is ready and it is added as the child node using an instance method, which is `addChildNode(_:)`, so it appears in the scene. Figure 3.15 shows the insertion of the virtual content into the camera scene.

```java
func addFurniture(result: ARHitTestResult) {

    let sceneName = realmResults![0].name!

    let furnitureScene =  
        SCNScene(named: "art.scnassets/scene/\(sceneName).scn")

    guard let furnitureNode =   
        furnitureScene?.rootNode.childNode  
        (withName: "Body", recursively: true)   
        else { fatalError("Unable to find the node") }

    let planePosition = result.worldTransform.columns.3

    furnitureNode.position =  
        SCNVector3(planePosition.x, planePosition.y,  
        planePosition.z)

    sceneView.scene.rootNode.addChildNode(furnitureNode)

    virtualObject = furnitureNode

}
```

![The detail page of the bed cabinet](https://lh3.googleusercontent.com/x99zmkvgNiB0e8VjVTz6E9SvhZFhq0T-m9zpefIsNitppGzVrqUieDIrTvsYcA-7qhF43chlqh4)

![A pillow is placed on the bed](https://lh3.googleusercontent.com/BtWOle5OxeZ6vQf5GPzj5pbmSkOtUT4Q1AlCRoxxb1DfDLQQfSF5Zgy5iCeR4ROeZqXkwDOQjSg)

### Deletion of Virtual Content from the Camera Scene

Related class of this function:

- `PlaneViewController` class

When the user clicks the "Delete" button, given that he has already placed at least one furniture object in the scene view, all the nodes will be deleted.

First, when the user adds any node into the scene view, I store those nodes into an array of `SCNNode` called `nodes`. Therefore, when he clicks the "Delete" button, the corresponding method will call `removeFromParentNode()` of each member in the array. Then, `removeAll()` is called to remove the array. Finally, all the nodes are removed.

``` java
for nodeAdded in nodes {
    nodeAdded.removeFromParentNode()
}

nodes.removeAll()
```

## User Interaction with Virtual Content

Related class of this function:

- `PlaneViewController` class

### Pan Gesture for Transformation

In `Main.StoryBoard`, the `Pan Gesture Recognizer` object is added to the `Plane Scene`. It is also added to the referencing outlet collection of the scene view . A sent action `@IBAction`, which is called `screenPanned`, is created to send an action to the corresponding custom class of the `Plane Scene`, which is the `PlaneViewController` class, as a method. In the `PlaneViewController`, the `sender` of the `@IBAction` is `UIPanGestureRecognizer`.

In the `screenPanned(_ sender: UIPanGestureRecognizer)` method, first, the sender's location is got, which means that the location of the continuous movement of the user's finger on the screen is got. Because this gesture recognizer only recognizes continuous gestures, any discrete event, such as a tap, does not report change. 

Then, a switch statement is used. When the `sender's state` is `began`, the `selectedNode` is set to be the virtual object's node, which means that the user selects the virtual object as the node to move. On the other hand, when the `sender's state` is `changed`, which means the touches have been recognized as a continuous change, a hit test is called. The type of hit test is set to be the existing plane, so the hit test will search for the AR anchors and real-world objects of the existing plane detected in each hit. Next, `worldTransform` is used. It is a matrix, showing the position and orientation of the hit test result relative to the world coordinate system. The new position is set to be the x-value, y-value and z-value of column 3 of the matrix got. Each node has its `simdPosition`. At last, the `simdPosition` of the virtual object is set to be the new position.

```java
@IBAction func screenPanned(_ sender: UIPanGestureRecognizer) {

    // Find the location in the view
    let location = sender.location(in: sceneView)

    switch sender.state {

        case .began:

        // Choose the node to move
        selectedNode = virtualObject

        case .changed:

        // Move the node based on the real world translation
        guard let result =  
            sceneView.hitTest(location,   
            types: .existingPlane).first  
            else { return }

        let transform = result.worldTransform

        let newPosition =  
            SIMD3<Float>(transform.columns.3.x,  
            transform.columns.3.y, transform.columns.3.z)
        
        selectedNode?.simdPosition = newPosition

        default:

        // Remove the reference to the node
        selectedNode = nil
    }
}
```

Therefore, when user pans the virtual object, the position of the virtual object will change according to the panning direction.

### Pinch Gesture for Enlargement or Reduction

In `Main.StoryBoard`, the `Pinch Gesture Recognizer` object is added to the `Plane Scene`. It is also added to the referencing outlet collection of the scene view . A sent action `@IBAction`, which is called `screenPinched`, is created to send an action to the corresponding custom class of the `Plane Scene`, which is the `PlaneViewController` class, as a method. In the `PlaneViewController`, the `sender` of the `@IBAction` is `UIPinchGestureRecognizer`.

In the `screenPinched(_ sender: UIPinchGestureRecognizer)` method, when the `sender's state` is `changed`, which means that the user is pinching against the screen, either outwards or inwards, the virtual object's `simdScale` is got. Each object has its `simdScale`, which is the scale factor. At the same time, it is multiplied with the `sender's scale`, and this calculation result is set to be the new scale.  Then, the `simdScale` of the virtual object is set to be a new scale.

```java
@IBAction func screenPinched(_ sender: UIPinchGestureRecognizer) {

    guard let node = virtualObject, sender.state == .changed

    else { return }

    let newScale = node.simdScale * Float(sender.scale)

    node.simdScale = newScale

    sender.scale = 1.0

}
```

Therefore, when the user pinches outwards against the virtual object, it is enlarged by the pinching scale; when the user pinches inwards against the virtual object, it is reduced by the pinching scale.

### Rotation Gesture for Rotation

In `Main.StoryBoard`, the `Rotation Gesture Recognizer` object is added to the `Plane Scene`. It is also added to the referencing outlet collection of the scene view . A sent action `@IBAction`, which is called `screenRotated`, is created to send an action to the corresponding custom class of the `Plane Scene`, which is the `PlaneViewController` class, as a method. In the `PlaneViewController`, the `sender` of the `@IBAction` is `UIRotationGestureRecognizer`.

In the `screenRotated(_ sender: UIRotationGestureRecognizer)` method, a switch statement is used. When the `sender's state` is `began`, the `originalRotation` is set to be the Euler angle of the virtual object's node. On the other hand, when the `sender's state` is `changed`, which means the user is performing rotation gesture against the virtual object, the y-value of the `originalRotation` is subtracted by the float of the `sender's rotation`. Then, the Euler angle is set to be the rotation value after subtraction.

```java
@IBAction func screenRotated(_ sender: UIRotationGestureRecognizer) {

    guard let node = virtualObject else { return }

    switch sender.state {

        case .began:
        originalRotation = node.eulerAngles

        case .changed:
        guard var originalRotation = originalRotation else { return }
        originalRotation.y -= Float(sender.rotation)
        node.eulerAngles = originalRotation

        default:
        originalRotation = nil
    }

}
```

Therefore, when user rotates the virtual object, the Euler angle of the virtual object will change according to the rotation gesture.

## Virtual Content Realism

### Estimated Ambient Lighting

Related class of this function:

- `PlaneViewController` class

First, in the `viewDidLoad()` method of the `PlaneViewController` class, all the default lights that SceneKit adds would be turned off, including `autoenablesDefaultLighting` and `automaticallyUpdatesLighting`.

Then, the `lightingEnvironment` of the scene is set with an image. This process is to create a general lighting environment for the scene. This method applies Image Based Lighting (IBL). The concept is using the images taken in the real world to be the light source of the environment. Therefore, the 3D objects will become much more realistic and accurate. Mostly, the environment map is spherical because the sphere is used to surround the 3D objects. Therefore, the environment map can illuminate them. Also, the environment map will be reflected by shiny materials, like metals. But without the environment map, the metals will not be shiny and reflective, and will become faked. Figure 3.16-3.17 shows the reflective feature of metals of using an environment image to set up the lighting in the scene.

![The nuts of the bed cabinet are made of streaked metal which is shiny, so they reflects the environment map](https://lh3.googleusercontent.com/Vvzzgk8lyxMwu4zQr9k9Oee-e-OSAI6v7IrSE2yBE6Wa89ykOm7g2lF0_a1pw0J85NUXXlQSSWU)

![Without environment map, the nuts become black in color and they look faked](https://lh3.googleusercontent.com/_1rr8QjDc02hp8924OzWqf-XFTZ9l1U-M6iXTAt1Ki5v4P1VtwbMS6wR6Dd5vzGA7EMeWO-VbsY)

Many types of spherical environment maps can be found online, like Lugher Texture (http://www.lughertexture.com). `spherical.jpg` shows an ordinary home lighting environment, so I will use this image as an environment lighting in this project. Figure 3.18 shows the environment image used.

![Using this image to create a lighting environment](https://lh3.googleusercontent.com/tFvE4N7dFovaxbDoysFiwFNcm-s6htyjULQiSmyEq5N40LCsfB0GeHx6U8kJtArmWe-yF5LouME)

```java
func setupLights() {

    // Turn off all the default lights SceneKit adds  
    // since we are handling it ourselves
    sceneView.autoenablesDefaultLighting = false
    sceneView.automaticallyUpdatesLighting = false

    let env = UIImage(named: "./Assets.scnassets/spherical.jpg")
    sceneView.scene.lightingEnvironment.contents = env

}
```

In the `viewWillAppear(_ animated: Bool)` method of the `PlaneViewController` class, the Boolean value `isLightEstimationEnabled` is `true` in the scene configuration, which enables the light estimation. 

```java
configuration.isLightEstimationEnabled = true
```

Then, the method `renderer(_:updateAtTime:)`, which is called exactly once per frame will adjust the light intensity of the scene. I get the `lightEstimate` of the current frame, and modify the scene's intensity of `lightingEnvironment` to be the ambient intensity of the `lightEstimate`. The estimated ambient intensity is normalized by dividing with 1000.0 because 1000.0 intensity is neutral. Figure 3.19-3.21 shows the change of the lighting environment intensity.

```java
func renderer(_ renderer: SCNSceneRenderer, updateAtTime time: TimeInterval) {

    let estimate = sceneView.session.currentFrame?.lightEstimate

    if estimate == nil {
        return
    }

    let intensity = (estimate?.ambientIntensity ?? 0.0) / 1000.0
    sceneView.scene.lightingEnvironment.intensity = intensity

}
```

![Under brighter lights](https://lh3.googleusercontent.com/z8KM0e04oxg_Zz01maoORl5KnAWr1t75Pd82YPpToDvinM-s8HwpqohgTyzW6JpOcgBo-HTXMFM)

![Under dimmer lights](https://lh3.googleusercontent.com/gR2XKDMAsc-mZpD2XWv6-Bg8WHuE_UZle9qoDhQh2khGYheTJ1JO_ZJvg_TTPEvFW_mYzvGNNy4)

![The object will even disappear in the dark as the light intensity is very low](https://lh3.googleusercontent.com/xOIMvZNxTLDLJTTBPB86NyHr9qKsVm-RN0cc9BVZLOCzttC02qXiPzVerrHtn1dQoy9s1pns-Eg)

### Shadows

#### Directional Lighting

Related class of this function:

- `PlaneViewController` class

Shadows can make the 3D objects looks more realistic. I add shadows to the scene by adding directional light. Figure 3.22 shows the 4 commonly used lights in SceneKit.

![Common light types in SceneKit](https://lh3.googleusercontent.com/r94ExP6iJRoLwHraoUen4xhBoOju6uSnlDKlk9xPsbJsPulv6kBrkVxqWxKQ_bkK_sHFJLLJC80){width=100%}

By adding directional light to the scene, the shadows will be cast on the objects and they will only be cast on the scene geometry, so there will not be "extra shadows" above or below the objects. The *addDireactionalLight()* method is called in the *viewDidLoad()* method of the *PlaneViewController* class. In this method, first, I create a light and set its type to *.directional* because only directional and spot lights can cast shadows and the directional lights are more natural than spot lights. The added light does not have any effect on the lighting environment, so I set the *intensity* to zero. Then, I set  *castsShadow* to *true* and  *shadowMode*  to  *.deferred*,  so that the shadows are not applied when rendering the objects. Instead, it would be applied after finishing the rendering.

Next, I create a black color with 50% opacity and set it as the *shadowColor*. This will make the casting shadows look more grey and realistic. I also increase its *shadowSampleCount* to create smoother shadows with and higher resolution. Finally, I create a node called "directionalLightNode", attach the above light to it, and rotate it so that it is facing the floor at a slightly downward angle. The "directionalLightNode" is ready and it is added as the child node using an instance method, which is *addChildNode(_:)*, so it appears in the scene.

Figure 3.23-3.25 show that, the spot light does not provide a more natural view than directional light and that is why I choose the directional lighting. Moreover, Figure 3.25-3.26 show that, the bed cabinet with directional lighting will have a shadow inside the drawer. In comparison, the bed cabinet without any lighting does not have a shadow inside the drawer.

```java
func addDirectionalLight() {

    // Create a directional light
    let directionalLight = SCNLight()

    directionalLight.type = .directional
    directionalLight.intensity = 0
    directionalLight.castsShadow = true

    directionalLight.shadowMode = .deferred
    directionalLight.shadowColor =  
        UIColor(red: 0, green: 0, blue: 0, alpha: 0.8)
    directionalLight.shadowSampleCount = 10

    // Create a directional light node
    let directionalLightNode = SCNNode()
    directionalLightNode.light = directionalLight
    directionalLightNode.rotation = SCNVector4Make(1, 0, 0, -Float.pi / 3)

    sceneView.scene.rootNode.addChildNode(directionalLightNode)

}
```


![With spot light: the bed cabinet is less natural and its surface has higher contrast](https://lh3.googleusercontent.com/hbG-RLr20e9KZ9pPT_7I3veMS0UHjbUwRpyzN0_9Y-c3ByovKdpGB5fhxHktODZLBgPI5XYHRK4)

![With spot light: the bed cabinet's surface becomes even darker with dimmer ambient lighting in the real environment](https://lh3.googleusercontent.com/PZYIFg-vQTt6J8zUIDh0eihWqZvEfSWB5Z6AiuqXLnUkY62mh-sVbz_n-t1I0xjBOVN79OpPNmY)

![With directional lighting](https://lh3.googleusercontent.com/gzcNFZh043JedsPN9O0LTAR8x23zD_1n6GRvMcftBToHEJhhwDViLzI_w-jkZ1Ulxfn2itBlnwU)

![Without any lighting](https://lh3.googleusercontent.com/wpJCbIEMQPH-5ilRrfHG44eqBDo_1Z-uP5s9luVuUcFawg4UblqTVOy6jAlZaRn7oF6TQG0Ls0E)


#### Shadow Baking uisng Blender

Normally, there are shadows below each objects in the daily life. If there is not any shadow below an object, the object will be like it is floating in the air. I used Blender, which is a free 3D computer graphics software, to bake a shadow for each furniture object. Figure 3.27 shows the Blender software. 

![Blender software](https://lh3.googleusercontent.com/UxWKGu0hGCYPM4Hq1vouFDtVljot6xbK99yukkKRa7jJw6r0n5GtyLurTzAYYLGyQT1_sqFDT9k)

According to the Figure 3.27, in the left editor, which is the default scene, I create a plane below the object first. The bottom editor is UV/Image Editor, where I can render a shadow on it. And I save the rendering result as Figure 3.28.

![Rendering shadow](https://lh3.googleusercontent.com/nR7OHcP7ZbBUdcA2cj4vgOOZxfE6HRZNnnABk60f09M8njn6e7RHepoQG70KWaZX3Qq7QS7mr0U)

But I only want the shadow on the plane. The right editor is node editor, where I can set the image texture when I bake the shadows. I set the image texture to be black color.

![The shadow baked by Blender](https://lh3.googleusercontent.com/7Yu9jsBPZPrR0-gdSK70KEQs-oukuzpLPJl3oVtKs5HNV8gOCORTWCOle4dmQLtNtVCThOS2zAg)

After I got the shadow image, which is Figure 3.29. As shown in Figure 3.30, I go to Xcode SceneKit Editor and create a plane below the 3D object. Then, I go to the material inspector of that plane, and set its diffuse map to the shadow image. And I change the opacity of the shadow image to 0.5-0.7 to make it semi-transparent. Figure 3.31-3.32 show the result of with or without a shadow below the object. The one with a shadow below will be more realistic and it blends more into the environment than the one without a shadow below.

![Adding a shadow below the tea table](https://lh3.googleusercontent.com/2jlmDsSogAmixD4Lwos8NFax84UKo2_l1sIxywdxz_1lj2CxkyfTThrAaoVK8MW9zj3kNfLSj8c)

![With a shadow below](https://lh3.googleusercontent.com/NhyPNIwTvGgk7lr8AfFW3Uiw5PbGRE0czMIIFm02vBMAazNytsY0wfxpf7nDc3OIXEmCeq-L7P4)

![Without a shadow below](https://lh3.googleusercontent.com/EjpuHlceVJ3Wm9zqZfslpJipuEYYyLYZjPrzAXDx_QLoPG1V6CaunGeK_vkqdC64BD_16H4wQrM)

### Physically Based Rendering

Physically Based Rendering (PBR) is a collection of render techniques to generate objects which look more realistic. When we add material to an object using PBR, we usually provide the following information as parameters:

- Albedo \newline
It is also known as the diffuse map. It is one of the factors that determines the amount of light reflected is material because it measures how much light that hits a surface is reflected without being absorbed. The light reflected would become visible. Figure 3.33 shows an example.

    ![A diffuse map of bamboo wood](https://lh3.googleusercontent.com/vJs_Ltqm_3R5jz6B3iAeeWdNA2aDUPH9Xgdtck-ocoNQyhS9gNGf1zdt3jCnsMVJGGX9Ghys7yg)

- Roughness \newline
It describes how rough the material will be. The rougher the material, the dimmer the reflection, vice versa. Figure 3.34 shows an example. 

    ![A roughness map of bamboo wood](https://lh3.googleusercontent.com/4eRq9vmDQjMiUtP4j-tt8lXR3nxE5fii5a0EwS8T4PUnhUV910M9bsjqPZET8AzIwcm944hWxjI)

- Metalness \newline
    It describes how metallic the material will be. It is an important parameter in different rendering systems. Firstly, metals (conductors) are much more reflective than insulators (non-conductors). The high reflectivity prevents most light from reaching the interior and scattering, so metals look shiny in general. The more metallic the material, the shinier the reflection, vice versa.
    In SceneKit, the meatless property can be set with a color vale. Lower values, which means darker colors, will make the material to less metallic, such as wood. Higher values, which means brighter colors, will cause the surface to appear more metallic, like iron.
    Figure 3.35 shows the streaked metal material of the nuts. The metalness property is white color, so the material is very shiny.

    ![The nuts of the bed cabinet are using streaked metal as the material](https://lh3.googleusercontent.com/Oxy0xRFlCX3LDzmdynZuvtddG5n2DlU6eKJ4EXblb4dZRAWc3Da9AoVUREMvugS-gcj52d9XRF0)

- Normal \newline
It describes the nominal orientation of a surface at each point for use in lighting. In general, we use a texture image as a normal map for the normal property. When SceneKit uses the texture image for the normal property, it treats the R, G, and B components of each pixel of the image as the X, Y, and Z components of a surface normal vector. Because a normal map can store much more detailed surface information than a geometry, we can set a texture image to the normal property to simulate rough surfaces such as stone and wood by adding protrusion in the normal map. Figure 3.36-3.37 show the effects of a normal map.

    ![Adding protrusion by a normal map to the diffuse content](https://lh3.googleusercontent.com/ZPOeuYVoNFO5PGQmwij8klkDcduk3CefYBeHueh6mfk_ez-ujOAG1p0NTBUOdk0teuPlHagAiAo)

    ![A normal map of bamboo wood with some protrusion on it](https://lh3.googleusercontent.com/U1-vqoaNnTiziRoXnIecZdGlkHi1C5yGAVuCD9weSXNSiGO4O18_kQqFCgf7EbPeBzjNqKGHFoE)

In Xcode SceneKit Editor, we can select a specific node and set the node's material in the material inspector as shown in Figure 3.38. We can select an image for each property, such as diffuse, metalness, roughness, normal, etc.

![The material inspector of the bed cabinet is mainly using the PBR model of bamboo wood](https://lh3.googleusercontent.com/nh21s6GS1TrKGEvH_VR6hIMUXT4xqCzlZXSVnIlzB5wjG1o_MFsz39hZ1AtOyEApvo9_Fz9FE-Y)
