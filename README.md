# Stormworks Radar Documentation:


This is a collaborative documentation to clear up misconception, and provide information about the "new" radars in Stormworks.
Huge credit goes to L963 for setting the groundwork of how and when radars update their data, and providing the original idea of a documentation.

## table of contets:
1. [noise](#noise)
2. [radar range](#range)
3. [optimal rotation speed for search radars](#speed)
4. [rotation speed slider](#sweepSlider)
5. [update rate](#rate)
6. [radar to XYZ](#xyz)
7. [credits](#credits)




## noise: <a name="noise"></a>
	
New radar targets experience noise, wich is a random offset from thier true position, and is updated every tick.
It is split up into angular noise, and distance noise.
	
**angular nosie:** ±0.001 turns

**distance noise:** ±1% of the distance

It can be estimated by a simple function:

`` maxNoiseLength(dist) = 0.0134069*dist ``

providing the worst possible noise deviation from any target ``dist`` meters away from the radar.





## radar range: <a name="range"></a>

The maximum range of each radar type with respect to its FOV X and FOV Y can be calculated:

`` maxRange(baseRange, fovX, fovY) = round( baseRange / ( fovX * fovY * 10000) ) ``

``baseRange`` is the range with both fovX and fovY = 0.01:

|Radar Type   |baseRange|
|-------------|:-------:|
|Radar Missile|80000    |
|Radar Basic  |240000   |
|Radar Phalanx|400000   |
|Radar Dish   |800000   |
|Radar Awacs  |2000000  |


The range at which a vehicle is detected, depends on its weight with respect to the radars maximum range.




## optimal rotation speed for search radars: <a name="speed"></a>

In order to not miss any targets with a rotating search radar, while still getting the fastest rotation rate out of your radars,
you can use the following function:

``R(fovX) = fovX * 60`` where ``fovX`` is pulled from the radars "select" page slider and ``R(fovX)`` is the optimal rotation rate in RPS

Best results are achieved by rotating the radar slightly slower, in order to have overlapping cones, thus not missing any targets.




## rotation speed slider: <a name="sweepSlider"></a>

The built in sweep speed is different between radars. You can still achieve any desired RPS with the Sweep Speed slider, or the XML number:

``S(coefficient, rps) = 100 / coefficient * rps`` where ``S(coefficient, rps)`` is the "Sweep Speed" slider percentage and the ``coefficient`` is the RPS each the radar type when the speed is 100%

| Radar Type     | Coefficient |
|----------------|:-----------:|
| Radar Missile  | 0.2865      |
| Radar Basic    | 0.382       |
| Radar Phalanx  | 0.2865      |
| Radar Dish     | 0.1910      |
| Radar Awacs    | 0.1432      |





## update rate: <a name="rate"></a>

Radar noise is update every tick, but the actual target position is not. This is a problem for spinning radars with a maximum distance of 3km and above, that rely on 1 position update per tick.
	
The position update rate depends on the maximum range of the radar, and can be expressed as the following function:

``U(r) = round(r/2000)`` where ``r`` is the maximum radar range in meters and ``U(r)`` is time per update in ticks

    2999m radar updates every tick
    3000m radar updates every 2 ticks
    20km radar updates every 10 ticks

- When exactly the radar updates, is dictated by its update timer.

- It starts at 0 and counts up when the radar is turned on.

- It pauses when the radar is turned off, and continues when it is on again.

- It resets after it hits U(r)-1.

- The radar updates when it hits 0 and the radar is turned on, thus updating every U(r) ticks.

- Targets are not only detected once the timer has reset (not on start)

- Timer needs electricity to run
	


### There are 2 ways of countering this problem:

1. **Spinning the radars slower, with an updated RPS function:**

    ``R(fovX) = (fovX * 60) / U(r)``


2. **Using more Radars:**

    This method relies on the fact, that each radar has its own update timer.

	 Example: 3km radar

	- U(3000) = 2 -> each radar can provide one update every 2 ticks
	- You use 2 radars, turned on in sequence, such that we always have one radar updating on every tick.
	- To get useful data out of the radars, we have to alternate between their composite outputs using a composite switchbox.
	- You know this worked, if the 4th number channel of the output of the composite switchbox (time since detection) is constant, when looking at a target.
 	- Optimally, you want it to be 0, in order to have minimal delay.
	- It does not work together with the radars built in Sweep Modes, since all radars must be looking in the same direction
	- This method can be take further, by averaging the targets noisy positions between true position updates, to reduce noise



## radar to XYZ: <a name="xyz"></a>

Radars provide local spherical target data, which can easily be turned into a more useful global cartesian vector (XYZ world coordinates).
	
This can be done with a standalone Lua function:

```
function radarToGlobal(dist, az, el, pos, orth)
    local x=sin(az*pi2)*cos(el*pi2)*dist
    local y=cos(az*pi2)*cos(el*pi2)*dist
    local z=sin(el*pi2)*dist
    return vec(orth.r.x*x+orth.f.x*y+orth.u.x*z+pos.x,orth.r.y*x+orth.f.y*y+orth.u.y*z+pos.y,orth.r.z*x+orth.f.z*y+orth.u.z*z+pos.z)
end

-- parameters:

dist = distance (ch1)
az = azimuth (ch2)
el = elevation (ch3)
pos = radar coordinates in the form of a vector
orth = orthogonal unit vectors pointing {r=right, f=forward, u=up} of the vehicle
```

requires these variables to be set:

```
pi2=math.pi*2
sin=math.sin
cos=math.cos
vec=function(x,y,z)return{x=x or 0,y=y or 0,z=z or 0}end
```

	
orth:

```
x,y,z = -ign(6), -ign(4), ign(5) -- physics sensor
--------------------------------
cx, sx = m.cos(x), m.sin(x)
cy, sy = m.cos(y), m.sin(y)
cz, sz = m.cos(z), m.sin(z)
--------------------------------
right = vec(cx*cz, sx*sy-cx*cy*sz, cy*sx+cx*sz*sy)
fwd = vec(sx, cz*cy, -cz*sy)
up = vec(-cz*sx ,cx*sy+cy*sx*sz , cx*cy-sx*sz*sy)
```




# Credits: <a name="credits"></a>

editors: TSD

credits: L963, thatcoolcat1, judgementalbird, thephantomraptor, TSD
