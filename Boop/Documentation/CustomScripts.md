# Custom Scripts


## Setup

To use custom scripts, you need to tell Boop where to find them. Open the preferences Menu ( `Boop > Preferences...` or `⌘,` ) and go to the Scripts tab. You'll be able to set a custom path for Boop to look in.


## Writing Custom Scripts

You can easily extend Boop with custom scripts to add your own functionality. Each script is a self-contained Javascript file that is loaded at app launch. If you make something cool or useful, feel free to submit a PR and share it with everyone else!

### Meta info

Each script starts with a declarative JSON document, describing the contents of that file, a title, a description, which icon to use, and search tags. All that stuff is contained within a top level comment (with some extra asterisks) just like so:

```javascript
/**
  {
    "api":1,
    "name":"Add Slashes",
    "description":"Escapes your text",
    "author":"Ivan",
    "icon":"quote",
    "tags":"add,slashes,escape"
  }
**/
```

* `api` is not currently used, but is strongly recommended for potential backwards compatibility. You should set it to 1.
* `name`, `description` and `author` are exactly what you think they are.
* `icon` is a visual representation of your scripts' actions. You can see available icons in `Boop/Assets.xcassets/Icons/`. If you can't find what you like, feel free to create an issue and we'll make it work!
* `tags` are used by the fuzzy-search algorythm to filter and sort results.

### The Main Function

Your script must declare a top level `main()` function, that takes in a single argument of type `ScriptExecution`. This is where all of the magic happens.

Your script will only be executed once, just like a web browser would at page load. Anytime your script's services are requested, `main()` will be invoked with a new execution object. 

Your script will be kept alive in its own virtual machine, retaining context and any potential global variables/functions you define. This means you need to be mindful of potentially generated garbage.

### ScriptExecution

The script execution object is a representation of the current state of the editor. This is how you communicate with the main app. A new execution object will be created and passed down to your script every time the user needs it. Once your `main()` returns, values are extracted from the execution and put back in the editor.

Make sure you *do not store* the execution object. If you do so, the native ARC system will not release it, leading to memory leaks and a potential crash after 10-15 years of continuous use. All joking aside, this is not a good thing and if we can avoid we'll all be better off.

Script executions are not exactly full Javascript objects, instead they're a proxy to the native `ScriptExecution` Swift class that communicates with the editor. Properties are actually dynamic getters and setters rather than stored values, that way if the `fullText` is huge and you don't actually need it, we're not passing it around needlessly. Therefore, try to only use the values you need and store them in variables to avoid calling native code too often.

### Messaging

Script execution objects have additional functions to communicate with the user, called `postInfo()` and `postError()`. These functions take in a single string argument, that will be presented in the Boop toolbar.


## Limitations

### Performance

A few times in this document you may have seen tips about performance, memory leaks, etc. Truth is, this is not that big of a deal because we're running minimal snippets of code in headless sandboxed environment. That being said, even though we're not at webpage-in-a-native-container levels of performance concerns, keeping the interface snappy and memory footprint low should be a priority when developing scripts so that we don't get a whole bunch of angry tweets about how maybe native apps aren't that great after all. (for the record they are that great).

Including all of jQuery to save a few lines of code may not be the greatest move here, so always prefer vanilla javascript functions as your whole script will be kept alive as long as the app is being used. If you'd like some helper functions, consider extracting only the part you need from the library (and clealy mention where it comes from alongside the license) rather than pasting the full source in your script. Includes/requires are not currently supported and not currently planned. Plus, if you use Vanilla you get to brag about how you can do it with nothing but your sharp brain and isn't that what being a developer is all about anyway?
