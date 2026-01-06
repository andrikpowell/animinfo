# ANIMINFO Specification Rev 0.9.0
Contents:
- [Preamble](#preamble)
- [Lump Format](#lump-format)
- [Example](#example)
- [Metadata Entry](#metadata-entry)
- [Lump Entry](#lump-entry)
- [Default Handling](#default-handling)

## Preamble
Doom has had it's advancements over the years regarding features, from new complevels (MBF21), to new map formats (UMAPINFO), to the latest new customisation lumps (ID24). However the most shocking thing to me is that with these advancements, no one ever thought of adding UI animations. Pretty much everyone agrees, an animated TITLEPIC would be sick! Having an animated menu and UI graphics would breathe new life for Doom modders. And now I can finally present this spec for the ANIMINFO lump!

Before we get to the actual specs, I would like to clarify the goals I had when designing this feature. I expect some discussion about its implementation, and I just wanna make clear this spec's goals and expectations.

First off, this is an animation substitution system. This is very important to clarify. The idea is that it indexes default lumps, checks whether an animation or widescreen asset exists, and then based on the port / port's settings will substitute the main lump with the new lump. The main reason I made it this way was to make it easy for users to disable animations (for accessibility reasons). But also it makes the wad backwards compatible with older ports that don't support it's features.

The inclusion of widescreen asset replacement was in direct response to wads crashing in DOS (since they don't support menu graphics larger than 320). The main idea is to avoid including extra "widescreen" wads, which clutter up wad directories. Whether you think this is an extraneous feature, I can understand, but since it follows the same "substitution" method, it was easy and made sense to integrate.

I have had some discussions regarding this lump, saying that animations can be included in some ID24 lumps like DEMOLOOP or SBARINFO. While this is indeed true, I feel that this would fragment animations to a dozen separate lumps. I feel it makes more sense to have a single lump governing animations similar to ANIMATED or ANIMDEFS. The other downside to such a method, is that just to add simple animations for a lump, they would have to create an entire layout lump (which could overwrite other lumps), just to get a simple animation. I feel this the anthesis of what ANIMINFO strives to do. Nyan Doom includes a set of [default animation ranges](https://github.com/andrikpowell/nyan-doom/blob/master/docs/animbg.md) and [widescreen names](https://github.com/andrikpowell/nyan-doom/blob/master/docs/ws.md) that when found, will automatically substitute the lumps. This makes it so easy, that all you would need to know about doom modding was to open a wad file and add graphics in.

## Lump Format
ANIMINFO uses the same parser as UMAPINFO, so implementing it into ports are rather simple.

It's implementation however, is a mix between the simplicity of UMAPINFO and structure of JSON. I attempted to see how this lump would be formatted in JSON itself, and found myself unhappy with the result in this particular case. While I can understand the benefits of JSON for some ID24 lumps, it is important for ANIMINFO to be extremely easy to edit.

Unlike UMAPINFO, ANIMINFO includes semicolons `;` similar to JSON to indicate the end of a property. I haven't added semicolons after brackets `{ }` currently, but I have no problem including them, if you think it'll be better in the long run.

## Example
Here is an example of a full ANIMINFO lump:
```
metadata "ANIMINFO"
{
  version = "0.9.0";
}

lump "titlepic"
{
  animate = clear;
  widepic = clear;

  animate =
  {
    type = "sequence";
    pic = "S_TITLEP"; tics = 4;
    pic = "TITLEPIC"; tics = rand(8,30);
    pic = "TITLEP2"; tics = 4;
    pic = "TITLEP3"; tics = 7;
    pic = "TITLEP4"; tics = 22;
    pic = "TITLEP5"; tics = 45;
    pic = "TITLEP6"; tics = 5;
    pic = "TITLEP10"; tics = 8;
    pic = "E_TITLEP"; tics = 10;
  }
  widepic = "W_TITLEP";
}

lump "HELP"
{
  animate =
  {
    type = "range";
    tics = 4;
    startpic = "S_HELP";
    endpic = "E_HELP";
  }
  widepic = "W_HELP";
}

lump "CREDIT"
{
  animate =
  {
    type = "range";
    tics = 4;
    startpic = "S_CREDIT";
    endpic = "E_CREDIT";
  }
  widepic = "W_CREDIT";
}
```

## Metadata Entry
As an attempt to remedy the issues of UMAPINFO, ANIMINFO includes a metadata block. Currently there is only one key, but in the future we can add more.
```
metadata "ANIMINFO"
{
  key = "value";
  ...
}
```

### Version
`version = "#.#.#"`
Specifies the version of the lump. This I feel is very important when it comes to making revisions to a lump. I think all future lumps should use this.

## Lump Entry
```
lump "LUMPNAME"
{
    key = value
    key = value1, value2,...
    ...
}
```
Values will be treated like strings, even if numbers, requiring quotation marks (`"`). An exception to this rule is the value `clear` which "clears" out the value.

## Animate
```animate = clear```

```
animate = 
  key = value;
  key = value; key = value;
  key = value; key = rand(min, max);
  ...
}
```
This is the main block where animations are defined. `animate = clear` explicitly means to not replace the specific lump with an animation. Note that `animate` blocks stack, and the final block will determine the behaviour to use.

### Type
```type = "range";```

Specifies the animation type. Currently there are 2 support types: "range" which specifies a start and end lump, animating the lumps in-between; and "sequence" which allows every frame of an animation to be defined.

### Pic
```pic = "TITLEP12";```

Only applicable to when `type = "sequence"`, it allows for a specific graphic to be shown during that frame of the animation.

### Tics
```tics = "#";```

```tics = rand(min #, max #);```

Required key, unless `clear` is used. Used for both `type = "range"` and `type = "sequence"`. Specifies how long the graphic should show before moving to the next frame. `"range"` animations only have to to use this key once, while `"sequence"` animations must specify `tics` after every frame. There is no inheriting tics from the previous frame.

Note that if an animation is defined, this key is required. A error will show at startup, if tics are not specified.

Random duration tics can be used for the value, but require the format `rand(min, max)`. The min is the lowest frame duration, with max as the highest frame duration. These tics are truly random and will change everytime the animation plays.

### Startpic
```startpic = "S_TITLEP";```

Only applicable to `type = "range"`, it sets the starting graphic for a range animation. Key is required for `"range"`, and will throw an error at startup, if not found.

### Endpic
```endpic = "E_TITLEP";```

Only applicable to `type = "range"`, it sets the end graphic for a range animation. Key is required for `"range"`, and will throw an error at startup, if not found.

## Widepic
```widepic = clear;```

```widepic = "W_TITLEP";```

This is the main block where the widescreen asset is defined. `animate = clear` explicitly means to not replace the specific lump with a widescreen graphic (mostly used to avoid port auto detecting that widescreen lump). Note that `widepic` blocks stack, and the final block will determine the behaviour to use.

## Default Handling
By default, Nyan Doom will create an animation database of the substituted lumps that exist. The main purpose of `animate = clear;` and `widepic = clear;` to tell the port to mark those lumps to not be substituted, and ignore any of those names.

Regarding the default animation ranges and widepics, Nyan Doom will skip animations and widepics it does not find. This works well as the default as only wads to want to use this functionality, will be able to just load a wad and have the animations happen. Note that the default tic duration of autodetected lumps is `8`. Allowing support for custom lumps specified in UMAPINFO's `enterpic`, Nyan Doom can and will update the animation database during run-time to check whether an animation or widepic exists.

"ANIMINFO" is much different when it comes to this behaviour. "ANIMINFO" parsing is very strict, and if a `tics` block or a `pic`/`enterpic`/`exitpic` lump specified doesn't exist, it will throw an error at startup. Currently due to how animation "ranges" work, it does not check frames in-between at the moment.
