# ANIMINFO Specification Rev 1.0.0
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
ANIMINFO uses a parser similar to UMAPINFO, so implementing it into ports are rather simple.

It's implementation however, is a mix between the simplicity of UMAPINFO and structure of JSON. I attempted to see how this lump would be formatted in JSON itself, and found myself unhappy with the result in this particular case. While I can understand the benefits of JSON for some ID24 lumps, it is important for ANIMINFO to be extremely easy to edit.

Lump names and version strings require double quotes. Keywords, Boolean values, and numeric values are unquoted.

## Example
Here is an example of a full ANIMINFO lump:
```
metadata "ANIMINFO"
{
  version = "1.0.0";
}

lump "titlepic"
{
  animate = clear;
  widepic = clear;

  animate =
  {
    type = sequence;
    oscillate = true;
    pic = "S_TITLEP"; tics = 4;
    pic = "TITLEPIC"; tics = rand(8, 30);
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
    type = range;
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
    type = range;
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
    key = value;
    ...
}
```
Lump names and version strings require quotation marks (`"`). Keywords such as `range`, `sequence`, and `clear`, Boolean values, and numeric values are unquoted.

## Animate
```animate = clear;```

```
animate = 
{
  key = "value";
  key = value;
  ...
}
```
This is the main block where animations are defined. `animate = clear;` explicitly disables animation substitution for the specified lump. Note that `animate` definitions stack, and the final definition determines the behavior to use.

### Type
```type = range;```

Specifies the animation type. Currently there are two supported types: `range`, which specifies a start and end lump and animates the lumps in between; and `sequence`, which allows every frame of an animation to be defined.

### Oscillate
```oscillate = true;```

Optional Boolean property supported by both animation types. When `false` or omitted, the animation loops from the final frame directly to the first. When `true`, it plays forward and then backward in a loop.

### Pic
```pic = "TITLEP12";```

Only applicable when `type = sequence`, it adds the specified graphic as a frame of the animation. Every `pic` must be followed by its own `tics` property.

### Tics
```tics = #;```

```tics = rand(min, max);```

Required property in an animation definition. It is used by both `type = range` and `type = sequence` and specifies how long the graphic is shown before moving to the next frame. Range animations use this property once, while sequence animations must specify `tics` after every frame. Timing is not inherited from the previous frame.

If an animation is defined, this property is required. An error is shown at startup if the required timing is not specified.

Fixed durations are positive, unquoted integers. Random durations use the unquoted `rand(min, max)` form and select a value between `min` and `max` whenever the frame begins. Both values must be positive, `min` must not exceed `max`, and no tic value may exceed 65,535 (~30 minutes).

### Startpic
```startpic = "S_TITLEP";```

Only applicable to `type = range`, it sets the starting graphic for a range animation. The property is required for ranges and produces an error at startup if the lump is not found.

### Endpic
```endpic = "E_TITLEP";```

Only applicable to `type = range`, it sets the ending graphic for a range animation. The property is required for ranges and produces an error at startup if the lump is not found. The start lump must precede the end lump in WAD-directory order.

## Widepic
```widepic = clear;```

```widepic = "W_TITLEP";```

This is the main block where the widescreen asset is defined. `widepic = clear` explicitly means to not replace the specific lump with a widescreen graphic (mostly used to avoid port auto detecting that widescreen lump). Note that `widepic` blocks stack, and the final block will determine the behaviour to use.

## Default Handling
By default, Nyan Doom creates an animation database from the substituted lumps that exist. The purpose of `animate = clear;` and `widepic = clear;` is to prevent those automatic substitutions for a particular lump.

Regarding the default animation ranges and widepics, Nyan Doom skips animations and widepics it does not find. The default tic duration of automatically detected animations is `8`, and they loop without oscillation. To support custom lumps referenced by properties such as UMAPINFO's `enterpic`, Nyan Doom can update the animation database at run time to check whether an animation or widepic exists.

"ANIMINFO" parsing is very strict. Missing required properties, invalid timing values, or missing lumps will produce an error at startup, by design. Range animations only validate the starting and ending lumps, and do not validate every lump between them.
