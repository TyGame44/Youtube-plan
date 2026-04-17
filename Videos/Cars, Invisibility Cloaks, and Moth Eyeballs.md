# Cars, Invisibility Cloaks, and Moth Eyeballs



What if we combined Neural Radiance fields and depth illusions and we combined that with the idea of the cones in moth eyeballs.



Let's start by describing moth eyeballs. They are made up of many crystalline cones. These cones reduce reflections and they guide light from multiple facets onto a single point of the retina.



Now imagine we did a depth projection onto a car from every different angle. These depth projections onto the car would create a guideline grid-like pattern of these cone structures.



Then we can use IR reflectors off of peoples eyes to position them in 3d space.



So we take their 3D gaze data and we create a nerf of a desired model of car. This would let you disguise a truck like the cybertruck into a plethora of different body shapes.



Some flaws you might notice is if you look through any myopic feature, you can see distortions where the depth illusion fails. This also happens when you first look towards them and they have to briefly update the nerf with your gaze data vs their estimate.



I also noticed a problem with them. This is a common computer graphics problem where sharp angles and edges need more sampling (anti-aliasing) or they will look blurry.

I noticed sampling errors on the sharp-angled surfaces of these vehicles.



I commented on this and noticed it stopped being an issue.



This same technology can be used for an invisibility cloak but that would require a pre-scanned area, or a realtime nerf.



If you just were using the lighting data applied to a car nerf, you could simplify the process with much smaller realtime updates.

