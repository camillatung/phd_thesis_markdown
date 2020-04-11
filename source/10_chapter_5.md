# Discussions and Conclusions

## Difficulties and Limitations
During the developement of Funiture Application, I have a deep revision about the pan and rotation gesture. I have tried what Apple Developer provided which is `raycast(_:)`, but it is complicated. According to the Apple Developer, ray casting a enables one-time hit test for pan gesture, so we do not need refined the position results over time for the continuous requests. Also, ray casting gives us the orientation information about the surface at a given screen point. Therefore, we do not need to keep on subtracting the gesture’s rotation from the current object rotation for rotation gesture. The `raycast(_:)` method can provide a smoother interaction with the objects, but due to time limit of this project, I use the ordinary method to perform the pan and rotation gesture activities.

## Further Development

### Scanning and Detecting 3D Objects

Apple Developer provides us a resource to scan and record the spatial features of the real-world objects, and the records can be saved as an `.arobject` file. It can be used as an after-sale service of a furniture shop. For example, after the user buys a denim chair from the shop and put it at home, he wants to know the information of this denim chair, like the maintenance period, he can scan the denim chair. As the denim chair has a record as a reference object before he buys it, so when he scans the chair, the application can compare the scanning object with the reference object, and find out the related information. 

### User Community
A user community can be a platform for different users to share the furniture objects. They can make comments on other shared posts too.

### User Favorite
In the first progress report of this project, I wanted to add a page for the users to save their favorite furniture objects. However, due to the time limit, I decided to drop this function. I still believe that it can help the users to record what they like, and the preference information can provide great help for search suggestions by the application.

### Material Customization
To make the application more interactive, the material customization can be added to the functions. The users can change the material of an object. They can also adjust different properties, like adjusting the reflection, metalness, etc.

