---
title: 'Resources in MonoTouch assembly'
date: 2012-5-12 23:47
draft: false
tags: ['MonoTouch', 'DotNet']
summary: 'In my experience, I found that adding resources into a MonoTouch project can be slightly confusing - if you have not done it before.'
---

In my experience, I found that adding resources into a MonoTouch project can be slightly confusing - if you have not done it before.

I had a requirement to embed images into a project... easy you may say. Add a solution folder into the solution, add your images into the folder and give them a build action of 'Content'. Duh... Why bother with a blog post? I hear the crowd mutter.

Well... It on the surface it is easy. The process is:

1. Right click on the solution, and select "Add > Add Files"
2. Select your image and click "open"
3. Choose to "Copy the file to the directory"
4. Set the Build Action to "Content" (In the properties panel)

![](http://i.imgur.com/twNFE.png)

Done... You can access the image by loading it using a directive which is relative to the root of the app. In this case it would be:

`_Resources\Xamarin_Logo.png`

Meaning I can access the image with the following code:

```csharp
UIImage myImage = UIImage.FromFile("_Resources\Xamarin_Logo.png");
```

## A short interlude

At this point, it is probably wise to diverge from the main point and do a little explaining... You see that I have named the folder "\_Resources" and not "Resources" - there is a reason, I am not just being awkward. The folder name "Resources" is a reserved name, and MonoTouch will throw the following error at compile time if you name the folder "Resources":

Resources/Xamarin_Logo.png: Error: The folder name 'Resources' is reserved and cannot used for Content files (ResourcesExample)

Doh... I solve this by adding an underscore to the beginning of the folder name. You could change this to "Images" or "[Boyakasha](http://www.urbandictionary.com/define.php?term=boyakasha&defid=1486528)" or, indeed, anything else you wanted, it would still work... Just don't use a reserved name! (Boyakasha hasn't been coined by the Mono compiler... yet)

## Back to the point

My point is that, I am used to embedding images as resources into my .Net projects and accessing them as objects rather than as a string location to a directory. I think this is better because:

- Strongly typed
- I can change the Resource file, and all associated images will be changed at the same time
- I can transform the image before showing it (e.g. resize, remove sharp corners to please the office health and safety officer etc.)
- I do this by creating a static class in my project named "Resources", which has the definition of each Image which I will use in my project as a static property of type UIImage.

```csharp
public static UIImage XamarinLogo {
	get {
		return UIImage.FromFile("_Resources/Xamarin_Logo.png");
	}
}
```

This now means that, as long as I include the namespace of the static "Resources" class in my code, I can reference that image using "Resources.XamarinLogo". Magic!

![](http://i.imgur.com/F9MYQ.png)

Check out the [source code](https://blazeware.kilnhg.com/Code/Open-Source/Group/Examples/Files/ResourcesExample) for an example of this being used in a MonoTouch.Dialog sample app.
