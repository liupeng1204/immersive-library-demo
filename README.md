# SuperViz Immersive Library Demo

## About
Proof of concept app that uses @superviz/immersive-library.


---
## Required dependencies

* Node.js v16.x
* Yarn

---

## Quick Start


Clone the project repository:

```bash
git@github.com:SuperViz/immersive-poc.git
```

From the project root run `yarn` to install the dependencies:

```bash
yarn
```

After that, the demo is ready to be used. To run the development environment, from the root run:

```bash
yarn dev
```

## Initializing the library

To start the library you must fill in the fields below:

* `userName`: name of the user who will enter the 3D environment;
* `userId`: id of the user who will enter the 3D environment;
* `roomId`: id of the room that the user will enter;
* `contentData`: id of the content that will be displayed in the 3D environment;
* `contentType`: type of content that will be displayed in the 3D environment;


### Example:

![image](https://user-images.githubusercontent.com/49524331/175338753-9851f0ee-ba69-49e1-a914-90324725718f.png)



You can also use query params in the URL to make testing easier, the fields are:

  * userName: username;
  * userId: userid;
  * roomId: roomid;

Example URL: `http://localhost:3000/?roomId=1PGES2&userId=440f57e1-f574-4961-a3b0-e9ecaf80096e&userName=Steve%20Roges`




It is also possible to change the content of the room, you can do this via the `contentData` and `contentType` field.

* `contentData`: content that should be displayed in the room;
* `contentType`: type of content that should be displayed in the room;

List of projects that can be used for testing:

* matteport: `9yaHdtwoXg5`;
* forge: `urn:adsk.objects:os.object:e8d17563-1a4e-4471-bd72-a0a7e8d719bc/Normal%20neonatal%20heart.fbx`;
* sketchfab: `7b02c861d6384fb58ac14fcd54d98475`

---
### Useful links

* [Immersive library documentation](https://www.npmjs.com/package/@superviz/immersive-library)