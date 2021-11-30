# Testing npm packages before publishing

Now, I use npm pack.

## npm pack

[The pack command](https://docs.npmjs.com/cli/pack) creates a `.tgz` file exactly the way it would if you were going to publish the package to npm. It pulls the name and version from `package.json`, resulting in a file like `package-name-0.0.0.tgz`. This can be copied, uploaded, or sent to a coworker. It's exactly the file that would be uploaded to npm if you published it.

```sh
~/workspace/package-name $ npm pack
```

Once you have the file, you can install it. npm install can install from a lot more sources than just npm, and I highly suggest [skimming through the docs](https://docs.npmjs.com/cli/install). We have to specify the full path to the file, so I usually copy it to my home directory first for convenience.

```sh
~/workspace/package-name $ npm pack
~/workspace/package-name $ cp package-name-0.0.0.tgz ~
~/workspace/some-application $ npm install ~/package-name-0.0.0.tgz
```

You could probably set up an alias or function in your terminal to automate this, but I don't do it frequently enough to bother. `npm pack | tail -n 1` will output just the filename to standard out.

This runs through a complete publish cycle—it even runs the publish npm script and the associated pre-scripts and post-scripts. Packing it up and installing it is an excellent way to simulate publishing a package, and it avoids all of the quirks and problems of symlinking.

I know when I was first trying to publish packages to npm, one of the hurdles I faced was figuring out if it would actually work. Publishing feels so final; you put it up for the world to see and that version number can never be used again. npm pack helps me be more confident that it's going to work the way I expect it to.