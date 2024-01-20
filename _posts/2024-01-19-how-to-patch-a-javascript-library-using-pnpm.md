---
layout: post
title:  "How to patch a javascript library using pnpm"
author: wijione
categories: [ pnpm, Javascript, Library ]
image: assets/images/how-to-patch-a-javascript-library-using-pnpm/patch.png
featured: true
---

In this article we'll explore how we can fix a broken javascript library quickly and efficiently with pnpm built-in feature. Due to its efficiency and speed, I use `pnpm` package manager for every new project I start.

If you would like to find out more, please refer to [this article](https://www.kochan.io/nodejs/why-should-we-use-pnpm.html) or to their [official documentation](https://pnpm.io/).

## Problem

It's not possible to find a javascript project with no dependencies. Almost every application or even library has a bunch of items in `dependencies` or `devDependencies` in its `package.json` file. And it's no surprise, bacause using libraries saves us lots of efforts, time, and of course money.

However, there are times when we install or update a particular library, and then suddenly we realize that it breaks our application.

It happens for multiple reasons. For example, developers don't always test every edge case, they might incorrectly follow Semantic Versioning rules or they just made a mistake (after all, we are humans rather than robots, we all make mistakes).

## Possible solutions

There are at least a few solutions to our problem. Each of them (as everything else) has its own pros and cons.

### Find an alternative

The easiest way is to throw away this shitty library and find something new. Voila 😅

But if you've already been there, you probably know that finding a new library is not as simple as it might seem. Sometimes there is just no good alternatives, and sometimes there are so many alternatives that you are not able to pick one to cover your needs.

### Downgrade a library version

If the issue arises after upgrading a library version, first of all, you need to check if there were any breaking changes and adjust your code accordingly. If there's no changes that might expectedly break your app, then you can downgrade the library back if it had worked as expected before. It's not always an option though. The new version might include some important security fixes or new features that your project requires.

### Ask maintainers to fix the issue

Another way to get a library fixed is to open a library repository and create an issue under the **Issues** tab, and then wait for a new version. This is a reliable and common way for at least two reasons:

- the maintainers know their code better than anyone else and can fix the code quickly and release a new version;

- other users probably experience the same issue and they might've already found a solution, so someone will reply and help you to find a temporary workaround.

However, this method might require a considerable amount of time, maintainers might have other priorities.

### Fork a library repo, fix the issue, create PR

This method is similar to the one above, but in this case you are the one who finds and fixes the issue. The exact process might vary from library to library, depending on the repository rules, but the overall idea is:

1. Fork the repository and clone it to your local machine.
2. Explore the code and find where the issues comes from.
3. Fix the issue and test operability.
4. Push the changes to the remote.
5. Go to the original repository and create a pull request.
6. Wait untill it's approved and merged.

This way might be a little faster than the previous one, yet it requires some time for maintainers to review and merge your PR.


## Patch a library on your own

But what if the release is planned for tommorow or even worse, it's in a few hours? Obviously, you don't have the time to wait for the fix. Your last option is to fix the library on your own.

### Begginer's approach

If you're a beginner you might be tempted to fix a library right in your `node_modules` folder. There is still a problem. How would your colleagues or production environment get the same fix?

Of course you could create a guide how to apply your fix and repeat the same process on every fresh `pnpm install`, but doesn't it look too tedious and unreliable?

### Professional approach using pnpm built-in feature

For this section I've created and published a very simple [library](https://www.npmjs.com/package/wijione-add-two-numbers) for adding two numbers. However, it has a little error that you will see further.

#### Setup

To start, let's initialize a project locally where we'll be installing our library. To do so we need to run the following command in the terminal:

```plain
pnpm init
```

Pnpm will only generate the `package.json` file for us with the following structure:

```json
{
  "name": "pnpm-patch-example",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "type": "module",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "dependencies": {}
}

```

Please, make sure you have `"type": "module"` in the `package.json` file.


Let's now install the library into our newly created project. The following command will generate `pnpm-lock.yaml` file, add the library into `dependencies` object in the `package.json`, create `node_modules` folder, and add the actual library there.

```plain
pnpm add wijione-add-two-numbers
```

Once everything is set up, we can start utilizing the library in our project. Let's create our main `index.js` file with very simple program.

```js
// index.js
import addNumbers from "wijione-add-two-numbers";

const sum = addNumbers(20, 10);

console.log(sum);
```

#### Bug

The program we've written is supposed to print **30** in the terminal. Let's run the application and see what happens.

![Wrong result](/assets/images/how-to-patch-a-javascript-library-using-pnpm/wrong-result.png)

Unexpectedly, we get **10** instead of **30**. Hmm, seems like the library does subtraction instead of addition. Let's open the source code of the library inside `node_modules` and find our why we're getting a wrong result.

![Broken library](/assets/images/how-to-patch-a-javascript-library-using-pnpm/broken-library.png)

As you can see, we were right, a wrong operator has been used. Let's fix it together. 

#### Patch

To start patching process we need to run the command below, which creates a draft folder and copies the source code of the library so that we can edit it:

```plain
pnpm patch wijione-add-two-numbers@1.0.0
```

By default pnpm creates the folder in a special path like in the screenshot below. The path is instantly printed to the terminal. However, using `--edit-dir` flag you can specify the folder as you see fit.

![Patch folder](/assets/images/how-to-patch-a-javascript-library-using-pnpm/patch-folder.png)

Once the folder is created, we can open it and edit the source code however we like. In our example we must replace subtraction operator (-) with addition operation (+) in `/<generated_path_to_the_patch_folder>/index.js` file.

Final result should look like this:

```js
// <generated_path_to_the_patch_folder>/index.js

export default (a, b) => {
  return a + b;
}
```

Once we've done editing the source code, we need to commit the changes we've made. To do so we have to run command below. As an argument we pass a path to the folder that was generated by `pnpm patch` command:

```plain
pnpm patch-commit <generated_path_to_the_patch_folder>
```

When you commit the changes, pnpm enters the information about the patched package into `package.json` and `pnpm-lock.yaml` files, creates a special file `wijione-add-two-numbers@1.0.0.patch` under `patches` folder in the root of your project and applies changes to the library source code in `node_modules`. In the screenshot below you can see what information the patch file contains.

![Patch file](/assets/images/how-to-patch-a-javascript-library-using-pnpm/patch-file.png)

In general, every time you run `pnpm install`, `pnpm add`, `pnpm update` and so on, pnpm will automatically traverse `patches` folder and apply changes on top of the installed packages based on patch files contents.

Now, to test the theory, remove the `node_modules` folder and run `pnpm install`. Eventually, in the `node_modules` you will see addition operator in the library source code according to our patch.

Note that if you update the library version (for example to `1.0.1`), changes from patch won't be applied, since the patch is associated with a particular version. In our case it's `1.0.0`.

Also, don't forget to commit and push `patches` folder to the remote. This way your colleagues will have access to the patch as well.

#### Patch deletion

Once the library maintainers have fixed the issue, we can upgrade the version and get rid of the patch we've created.

Simply deleting the patch file is not enough, because like I said earlier, pnpm also adds some information into `package.json` and `pnpm-lock.yaml` files.

Don't worry, removing a patch is much simpler than creating it. To delete the patch just type the following command into the terminal and it will do everything for you:

```plain
pnpm patch-remove wijione-add-two-numbers@1.0.0
```

Voila! The patch has been completely removed from the project.

## Conclusion

That's basically it. Although the article turned out to be a little longer than I'd been planning, now you know what to do when you run into a broken library. I'll try to keep my future posts as short as possible to include only necessary information.