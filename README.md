# Firebase Functions Interop Library for Dart

This library provides JS bindings for Firebase Functions Node SDK.
It also exposes a convenience layer to simplify writing Cloud Functions 
applications in Dart.

> Please note that it's currently a proof-of-concept-preview version so it lacks 
> many features, tests and documentation. But, it's already fun to play with!

## Usage

> Make sure you have Firebase CLI installed as well as a Firebase account
> and a test app. 
> See [Getting started](https://firebase.google.com/docs/functions/get-started)
> for more details.

### 1. Create a new project directory and initialize functions:

```bash
$ mkdir myproject
$ cd myproject
$ firebase init functions
```

This creates `functions` subdirectory in your project's root which contains
standard NodeJS package structure with `package.json` and `index.js` files.

### 2. Initialize Dart project

Go to `functions` subfolder and add `pubspec.yaml` with following contents:

```yaml
name: myproject_functions
description: My project functions
version: 0.0.1

environment:
  sdk: '>=1.20.1 <2.0.0'

dependencies:
  # Firebase Functions bindings
  firebase_functions_interop: ^0.0.1 
  # Node bindings required to compile a nice-looking JS file for Node.
  # Also provides access to globals like `require` and `exports`.
  node_interop: ^0.0.1 

transformers:
  - $dart2js
  - node_interop # This transformer must go after $dart2js
```

Then run `pub get` to install dependencies.

### 3.1 Write a Web function

Create `bin/index.dart` and type in something like this:

```dart
import 'package:firebase_functions_interop/firebase_functions_interop.dart';

void main() {
  FirebaseFunctions functions = new FirebaseFunctions();
  var httpsFunc = functions.https.onRequest((request, response) {
    response.send('Hello from Firebase Functions Dart Interop!');
  });

  exports.setProperty('helloWorld', httpsFunc);
}
```

Copy-pasting also works.

### 3.2 Write a Realtime Database Function (optional)

Update `bin/index.dart` with following:

```dart
void main() {
  // ...Add after helloWorld function...

  // This implements makeUppercase function from the Getting Started tutorial:
  // https://firebase.google.com/docs/functions/get-started
  var dbFunc =
      functions.database.ref('/messages/{pushId}/original').onWrite((event) {
    String original = event.data.val();
    print('Uppercasing $original');
    String uppercase = original.toUpperCase();
    return event.data.ref.parent.child('uppercase').set(uppercase);
  });
  exports.setProperty('makeUppercase', dbFunc);
}
```

### 4. Build your function(s)

Building functions is as simple as running `pub build`. Note that Pub by 
default assumes a "web" project and only builds `web` folder so we need
to explicitly tell it about `bin`:

```bash
$ pub build bin
```

### 5. Copy and deploy

The result of `pub build` is located in `build/bin/index.dart.js`. Replace
default `index.js` with the built version:

```bash
$ cp build/bin/index.dart.js index.js
```

Deploy using Firebase CLI:

```bash
$ firebase deploy --only functions
```

### 6. Test it

You can navigate to the new HTTPS function's URL printed out by the deploy command. 
For the Realtime Database function, login to the Firebase Console and try
changing values under `/messages/{randomValue}/original`.

## Features and bugs

Please file feature requests and bugs at the [issue tracker][tracker].

[tracker]: https://github.com/pulyaevskiy/firebase-functions-interop/issues/new
