---
published: true
title: Avata in the Dark
author: synvan
date: 2025-02-11 14:10:00 +0200
categories: [Product, Hobbies]
tags: [Drones, FPV, CAD, 3D Modeling, 3D Printing]
render_with_liquid: false
description: Turning night-flight headaches into a product-style side project. My journey designing, testing, and iterating an LED mount for the DJI Avata.
media_subpath: '/assets/img/posts/06_avata-in-the-dark'
image:
  path: /00_avata-in-the-dark.webp
  alt: 
  in_post: true
---

In 2022, DJI released the Avata drone. It burst onto the scene as a game-changer in the consumer FPV drone world, offering an immersive flight experience with DJI's great UX and safety features. The Avata truly shines when it comes to indoor flight – its compact size and built-in propeller guards make it nimble and durable, allowing confident navigation through tight spaces with low risk of damage from bumps and scrapes. The visual positioning system provides exceptional stability even without GPS signals, enabling it to maintain its position accurately in cluttered indoor environments, making it a favored choice for capturing dynamic interior footage.

## No Light

The problem starts at twilight or when you simply float into a window-less, light-less room – piloting the drone through dim corridors can feel as unpredictable as chasing fireflies. The optical sensors are starved of contrast and reference points, leaving the drone wobbling each time you try to hover between obstacles. Eventually it'll hit something and crash.

![Avata VPS components](05_avata-vps.png)
_VPS components of the DJI Avata_

The solution seemed obvious – more light equals better stability. By mounting LEDs on the drone, I could feed the vision sensors a steady stream of visual cues, transforming low-light missions into smooth cinematic flights.

I started off with some ad-hoc market research: I scoured forums, DJI groups, Thingiverse, and online shops for Avata LED mounting solutions. A few options existed but they were overpriced, bulky, or poorly reviewed. Classic "build vs buy" moment. With no viable off-the-shelf choice, I went hands-on – dusted off my 3D modeling skills and started designing a custom LED mount for my exact needs.

## Choosing the Right LED

First things first, I needed a capable LED light that I could design a mount around and attach to the drone. I scanned Amazon and AliExpress for candidates that comply with a few prioritized requirements:
1. **Illumination** – strong enough to light up dark halls and corridors.
2. **Lightweight** – minimal impact on drone flight time.
3. **Small profile** – maintain the drone's cross-section and have minimal effect on propeller thrust.
4. **Battery capacity** – enough to maintain illumination for at least one drone-battery cycle (~20 minutes).

I ended up with two main candidates to choose from:
- Firehouse ARC V
- BRDRC Strobe 3-LED

![LED options: BRDRC vs. Firehouse](10_brdrc-vs-firehouse.png){: w='500' h='400' }
_BRDRC (left) vs. Firehouse (right)_

As always, when tasked with a tough decision – the scorecard method is your friend. I listed all features that mattered to me in a table, assigned weight factors based on my priorities, and scored each candidate with a 1-5 rating based on its compliance/performance:

| Category            | Weight   | Firehouse       | Score | Weighted | BRDRC           | Score | Weighted |
| :------------------ | :------- | :-------------- | :---- | :------- | :-------------- | :---- | :------- |
| Price (1 unit)      | 30%      | US$ 33          | 2     | 0.60     | US$ 8           | 5     | 1.50     |
| Weight              | 20%      | 13 gr           | 3     | 0.60     | 6 gr            | 5     | 1.00     |
| Illumination        | 15%      | x5 White LEDs   | 5     | 0.75     | x3 White LEDs   | 3     | 0.45     |
| Size & profile      | 15%      | 38 x 25 x 13 mm | 3     | 0.45     | 28 x 16 x 10 mm | 5     | 0.75     |
| Battery (Steady-on) | 10%      | 120 mins        | 5     | 0.50     | 45 mins         | 2     | 0.20     |
| Charger socket      | 5%       | USB-C           | 5     | 0.25     | USB-C           | 5     | 0.25     |
| Charge indication   | 5%       | No              | 2     | 0.10     | Yes             | 5     | 0.25     |
| **Total**           | **100%** |                 | —     | **3.25** |                 | —     | **4.40** |

With everything factored in, going with **BRDRC** was a no-brainer. It's smaller, lightweight and offers better bang-for-the-buck – though I did sacrifice some illumination compared to the Firehouse. That's an acceptable trade-off given that most low-light flights are planned to be indoors.

## Figuring Out a Mounting Solution

Now that I've chosen an LED light for my solution (I ordered 6 strobes, more than enough to begin with), I could move ahead with figuring out how to actually mount the LED onto the drone in the most efficient way.

### The Requirements

Before opening my 3D-modeling app I wrote a mini-PRD – nothing fancy, just a bulleted list of "must haves" ranked by priority. That helped me organize my thoughts, stay honest about the use-case and cut down on 3D-printing iterations. Here's the shortlist my LED mount had to meet:
1. **3D-printable** – relatively simple design that can be 3D-printed easily at home.
2. **Durable** – just like the drone itself, it's expected to take a few hits.
3. **Lightweight** – minimal impact on drone flight time.
4. **Slim design** – to minimize protrusion outside of the drone's profile.
5. **Detachable** – for two main reasons:
   - Avoid flying daylight missions with extra dead weight (no flight-time reduction).
   - Enable flying long low-light missions by easily swapping LEDs when they're out of juice (improves field usability and reduces downtime).

_Mini-KPI plan: total added weight, impact on flight time, and whether the Avata can still hover stably in a dark hallway. Those would be my success metrics for this prototype._

### Mountable Surfaces

The initial step before getting down to actual design work would be deciding which part of the drone fuselage the LED mount should actually attach to. A quick look at the bottom showed me two potential spots where an LED could go:
1. Attached to the relatively-wide & flat surface right below the VPS components.
2. Attached to the diamond-shaped slot formed between the outer sides of the rotors. 

![Mounting options](20_avata-mount-options.png){: w='400' h='400' }
_Bottom view of the Avata drone with marked options for mounting_

I decided to move forward with option #2 for a few reasons:
- The diamond-shaped slot provides depth for more robust mounting methods.
- The depth also means that LEDs can be mounted closer to the fuselage, helping to keep the LEDs tighter with the assembly and preserve the drone's slim profile.
- The slots appear on both sides of the fuselage, allowing to mount x2 LEDs → safer to fly low-light missions. If one LED fails, you could still finish your mission as planned with the other functioning LED, or at the very least increase your odds of getting the drone back safely in the dark.

### 3D Measurements

Next, after deciding what part of the drone we'll attach to, we need to measure that area to properly design a mount that actually fits well. Normally I'd grab a caliper and start measuring, but finding a scanned 3D model online saved me a ton of trial-and-error. Luckily, a quick search in some 3D repositories landed me in this [Thingiverse webpage][dji-avata-model] featuring the DJI Avata body scanned for us by the good soul [goshes][goshes-thingiverse].

I loaded the scanned object into my 3D modeling app and extracted the exact contour I needed for my mounting solution to fit. Using scanned objects is super efficient and saved me at least a few iterations of caliper measuring, printing and testing the mount fit vs. real drone body.

![Avata VPS components](30_avata-measure-sketch.png)
_Measurement sketch drawn on a 3D model of the Avata drone_

### The Design

With the goal of meeting all requirements I specified above in my mini-PRD, I finally got down to the actual modeling step. Using the measurement sketch from before, I started by designing the **core** of the mount (diamond-like shape that fits in the propeller-guards slot), and around it I added a few key elements:
1. **Base plate** – this would be the surface area to which I'd attach the LED. I want it to be slim and match the actual surface area my LED takes (minimize impact on propeller thrust).
2. **Flexible arms** – these elements will continue upwards from the core, they will be relatively thin to enable flexibility as the mount is pushed into the slot in the drone body.
3. **Latch teeth** – these will merge to the top of the flexible arms and will allow locking the mount in place when fully inserted into the diamond-shaped slot.
4. **Arm extensions** – this is the top-most part of the flexible arms, it will extrude above the propeller-guards. It'll enable the pilot to flex the arms and detach the mount from the drone body.

![Avata VPS components](40_led-mount-part.png){: w='500' h='500' }
_LED mount design_

Here are a few screenshots of the mount attached to the scanned drone body:

![Design assembly view](70_assembly-diagonal.png)
_Mock BRDRC LED (green) attached to the drone with my custom mount (blue)_

![Design assembly front view](50_assembly-front.png)
_Front view showcasing the slim profile of a mock BRDRC LED (green) mounted on the drone_

![Design assembly top view](60_assembly-top.png){: w='500' h='500' }
_Top view showcasing the minimal surface area of the mount over the properllor-thrust direction_


## Printing the LED Mounts

With the 3D model wrapped up and saved as an STL, I headed over to a friend's place to fire up his 3D printer and crank out a few test pieces. At this stage there are two big decisions that can make or break the print – what material you're going to use, and how you're going to orient the part on the bed.

### Printing Material

When it comes to hobbyist 3D printing, it usually boils down to PLA or PETG. PLA is kind of the "default" – it prints easily, doesn't warp much, and is perfect for quick prototypes. PETG, on the other hand, is a bit tougher (literally). It needs higher temps and a bit more fine-tuning to avoid stringing, but it pays off with parts that are stronger, more flexible, and way better at handling moisture or outdoor use. 

For the first run I treated the print like an MVP: go fast, validate fit, then iterate. PLA gave me a quick, clean prototype. Once the geometry is proven, PETG can be the "production" version with more durability.

### Orientation
Since our design uses flexible arms, the bending stress runs along the length of those arms. That means the way you orient the part on the print bed can make a huge difference in how strong (or weak) it ends up. By lining up the arms so the layer lines run along them instead of across them, you give the print way more resistance to snapping when flexed. It's one of those little tweaks that doesn't take any extra time but pays off when you start actually using the part.

![3D printing orientation](80_print-orientation.png){: w='500' h='500' }
_Recommended orientation for 3D printing_

### Attachment
Now with a few printed models and the LEDs at hand, I needed to attach them together – this was easily done with outdoor-rated double-sided tape. Good quality tape designed to work in heat and moisture conditions will hold forever (not much stress on the bond since our parts are quite lightweight). The bond is actually very likely to outlast the drone itself.

Here are some final pictures of the LEDs and mounts:

![Drone with LEDs mounted](90_assembly-top.png){: w='500' h='500' }

![Drone with LEDs mounted](100_assembly-side.png)

![LEDs in action](110_leds-in-action.png)

## Final Thoughts

Like any good prototyping there's a plan for next version – testing PETG prints, different LED angles for better spread, and maybe a solution for controlling the LEDs remotely. You can find the STL file for 3D printing in my github repository [here][github-repo] (where I'll likely upload future revisions as well).

That's about it, enjoy flying!

[dji-avata-model]: https://www.thingiverse.com/thing:5693161
[goshes-thingiverse]: https://www.thingiverse.com/goshes/designs
[github-repo]: https://github.com/synvanalt/avata-led-mount
