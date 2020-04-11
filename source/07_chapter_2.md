# System Overview

## System Development Tool

"Funiture" is an iOS AR application developed with Apple’s frameworks. It runs on iOS devices that support ARKit and SceneKit.

### ARKit

ARKit includes many different tools related to AR. Below are some features related to my project. First, ARKit can perform world tracking. ARKit can track the iOS device's position and orientation, so that we can augment the environment in front of the device. 

Second, ARKit provides us with visual-inertial odometry. It acquires information from the motion-sensing device with the live camera scene first, then combines different pieces of information together to provide a precise position of the virtual 3D objects in the live camera scene.

Third, ARKit can recognize many types of surfaces in the real environment, like walls, floors, seats, etc. It first detects the feature points aligned in a real plane, then returns this piece of information as an object with basic geometry and type of plane for further usage.

### SceneKit

SceneKit can add 3D objects to applications. It is convenient to use because it only requires the developers to provide descriptions of the scene’s actions they want. It does not require the developers to provide details of the rendering algorithms of 3D objects.

### Scripting

In this project, I will program in the native language, Swift. And I will use Realm Studio to manage my database.

## System Design

### Use Case Diagram
![enter image description here](https://lh3.googleusercontent.com/cmDAwWjea-SnnQVu_Jk0O6b3iEBjWSKLy7b6fzg4UYAFwxqOO2ZXw9Q_htXSSZW6cuyRju0Qm38) \

#### Search Furniture by Keyword
|  |  |
|--|----|
| Actor | User |
| Brief Description | The user needs to search for a furniture object before placing the object into the camera scene. |
| Preconditions | The user has entered the search page. |
| Flow of Events | 1. The user enters the keyword(s) in the search bar. \newline 2. A search result will be shown. |
| Postconditions | A search result is shown. |
| Alternative Flows and Exception | If the keyword(s) does not match any furniture object's name in the database, the system will show "No Result". |

#### Search Furniture by Measurement
|  |  |
|--|----|
| Actor | User |
| Brief Description | The user needs to search for a furniture object before placing the object into the camera scene. |
| Preconditions | The user has entered the measuring home page. |
| Flow of Events | 1. The user clicks the "Length"/"Width"/"Height" button. <\newline 2. The user measures the reality for the selected type of distance. \newline 3. The user repeats step 1-2 to measure other types of distance. \newline 4. The user clicks the "Search" button. \newline 5. A search result will be shown. |
| Postconditions | A search result is shown. |
| Alternative Flows and Exception | 1. If the measurement(s) does not match any furniture object's measurement in the database, the system will show "No Result". \newline 2. If the user does not measure any distance then clicks the "Search" button, the system will show a warning and nothing will be searched.  |

#### Search Furniture by Plane Selection
|  |  |
|--|----|
| Actor | User |
| Brief Description | The user needs to search for a furniture object before placing the object into the camera scene. |
| Preconditions | The user has entered the live camera scene showing each plane detected with its type. |
| Flow of Events | 1. The user selects a plane detected with its type in the live camera scene. \newline 2. The user clicks the "Search" button. \newline 3. A search result will be shown. |
| Postconditions | A search result is shown. |
| Alternative Flows and Exception | 1. If the plane type selected does not match any furniture object's plane type in the database, the system will show "No Result". \newline 2. The "Search" button will only be shown after the user has selected a plane. |

#### View Furniture Details
|  |  |
|--|----|
| Actor | User |
| Brief Description | The user can view each furniture object's details. |
| Preconditions | The user has searched for at least one furniture object. |
| Flow of Events | 1. The user clicks one of the furniture objects searched. \newline 2. A detail page will be shown, displaying the corresponding details. |
| Postconditions | A detail page is shown, displaying the corresponding details. |
| Alternative Flows and Exception | If the furniture object does not have any detail in the database, the system will show "No Result". |

#### Place Furniture into Camera Scene
|  |  |
|--|----|
| Actor | User |
| Brief Description | The user can place a furniture object into the camera scene. |
| Preconditions | The user has entered the detail page of a furniture object. |
| Flow of Events | 1. The user clicks the "Place" button below the furniture photo. \newline 2. The system will go to the page of the live camera scene. \newline 3. The system will detect existing planes in the live camera scene. \newline 4. The user selects one of the detected planes to place the object. \newline 5. The furniture object will be placed into the camera scene. |
| Postconditions | The furniture object is placed into the camera scene. |
| Alternative Flows and Exception | 1. If the system fails to detect any existing plane in the live camera scene, the object cannot be placed there. \newline 2. If the user does not select any plane detected, the object cannot be placed there. |

#### Delete Furniture from Camera Scene
|  |  |
|--|----|
| Actor | User |
| Brief Description | The user can delete the furniture objects from the camera scene. |
| Preconditions | The user has already entered the live camera scene and placed at least one furniture object into the camera scene. |
| Flow of Events | 1. The user selects a furniture object. \newline 2. The user clicks the "Delete" button. \newline 3. The furniture object will be deleted. |
| Postconditions | The furniture object is deleted. |
| Alternative Flows and Exception | 1. The "Delete" button will only be shown after the user has selected a furniture object in the camera scene. |

#### Interact with Furniture in Camera Scene
|  |  |
|--|----|
| Actor | User |
| Brief Description | The user can move/scale/rotate the furniture objects in the camera scene. |
| Preconditions | The user has already entered the live camera scene and placed at least one furniture object into the camera scene. |
| Flow of Events | 1. The user performs a pan/pinch/rotation gesture against one furniture object in the camera scene. \newline 2. The system will move/scale/rotate the object according to the gesture. \newline 3. The object will be moved/scaled/rotated in the camera scene. |
| Postconditions | The object is moved/scaled/rotated in the camera scene. |

### Activity Diagram

#### Searching
![enter image description here](https://lh3.googleusercontent.com/ctxtHyP0lxZa2SxzTx7GnuGwA1l8M2IdPTqIqjXRpBtoXho0xXwFuxV7d2BtB5GkJaz3FLQcvfk) \

![enter image description here](https://lh3.googleusercontent.com/Jewb7skpj86-7eP8_9gyeWsL-aHfcSwpfyNVhUhWrCzUQrZOqQd15uAELDGrX5zV__hSzz_ubIM) \

![enter image description here](https://lh3.googleusercontent.com/Hljc8uk9RpdbL_f2PNhWidbbPx1iUOFerh2n7aTyDh24vbnjaFx0Hww2ILMxF9fItT20gBTR9h8) \

#### Viewing Details
![enter image description here](https://lh3.googleusercontent.com/b2pq5QTkVQv4HFp6wouV2UNdwCSZtw3Z9A4qJGyQoN-iVTIiJvaaRBkcROS3HWfuatDA5t8VgqQ) \

#### Placing Furniture
![enter image description here](https://lh3.googleusercontent.com/_wCeSqLUZMRBUydsULMa-K8dtce8RpWypH-JXKKPwmmiBJGxpjWZOL3nmliBc3y3f_E1W8GxOn0) \

#### Interacting with Furniture
![enter image description here](https://lh3.googleusercontent.com/VjMDwfuMvoGmdDNRQhdJC3woJuwq6nflOaaRR8bTP5TzB7VhMpd-TNi3rlbA7alHY8UNabDcM7U) \

### State Diagram
![enter image description here](https://lh3.googleusercontent.com/FfgfRHZiENdWnJKf-bdhid8CdQXv492OUjEX8xAekDnKd4WeGOt78YWbBzPJ_vSWG9B82eMoELw) \

![enter image description here](https://lh3.googleusercontent.com/kni86JfMlP0nKB6Z3519qwDhcPPvazNcSeVyUS_DhjWDq8FdF4nrOVCgwQddWt2lZ_eWMlDAedo) \

### Class Diagram
![enter image description here](https://lh3.googleusercontent.com/uNAPBB9UDNSK28d3MmiKMbujPx_cu4WUZfM_pIlVSetv-6aNV6hL8H37nHGMNa4Etv9KfRZYEZU) \

### Database

#### Realm Data Model

To show the furniture information and make use of it, I add a local database to this application. The `RealmSwift` library can be installed through CocoaPods, which is a dependency manager for iOS projects. I edit the Podfile to include `RealmSwift` as a dependency for my project.

I have prepared a Realm database file `Furniture.realm`. The model definition is as below:

```java
class Furniture: Object {

    @objc dynamic var id: String? = nil
    @objc dynamic var type: String? = nil
    @objc dynamic var name: String? = nil

    let length = RealmOptional<Double>()
    let height = RealmOptional<Double>()
    let width = RealmOptional<Double>()
    let price = RealmOptional<Double>()

    @objc dynamic var material: String? = nil
    @objc dynamic var desc: String? = nil

}
```

Besides, I use the Realm for the temporary storage of measurement because the user can search after measuring all the length, width and height, more than one distance. In this process, the user will enter different views and the views will reload. To prevent the loss of the measurement when the views are reloading, I need the temporary storage to store the measured distances. The model definition is as below:

``` java
class Measurement: Object {

    @objc dynamic var id: String? = nil
    @objc dynamic var length: String? = nil
    @objc dynamic var width: String? = nil
    @objc dynamic var height: String? = nil

}
```
