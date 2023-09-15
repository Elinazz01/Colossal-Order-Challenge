# Colossal-Order-Challenge

I started with making a shader for sliding rain on objects
I made the shader with code instead of shader graph, because coding gives more control over the parameters and it's easier to make it lightweight since shader graph can make some shaders bloated.

The shader takes the uv and divides it into grids which the raindrops are in, the grids work kind of like cages that restrict where the raindrops can go:
"
 float t = fmod(_Time.y + _T, 7200);
 float4 col = 0;
 float2 aspect = float2(2, 1);
 
 float2 uv = i.uv * _Size * aspect;
 uv.y += t * 0.25f;
 float2 gv = frac(uv)-.5;
 float2 id = floor(uv);
"
Then after that we create a rain drop which moves down in random directions(N21 is a function that generates a random value betwen two large numbers):
"
float n = N21(id); 
t += n*6.2831; 

float w = i.uv.y * 10;
float x = (n - 0.5) * 0.8;
x += (0.4 - abs(x)) * sin(3 * w) * pow(sin(w), 6) * 0.45;

float y = -sin(t + sin(t + sin(t) * 0.5)) * 0.45;
y -= (gv.x -x)  * (gv.x -x);

"

the rain drop creates a trail of smaller rain drops that gets smaller the farther the raindrop gets:
"
float2 dropPos = (gv - float2(x, y)) / aspect;
float drop = S(0.05, 0.03, length(dropPos));

float2 trailPos = (gv - float2(x, t * 0.25f)) / aspect;
trailPos.y = (frac(trailPos.y * 8)-0.5)/8;

float trail = S(0.03, 0.01, length(trailPos));

float fogTrail = S(-0.05, 0.05, dropPos.y);
fogTrail *= S(0.5f, y, gv.y);

trail *= fogTrail;
fogTrail *= S(0.05, 0.04, abs(dropPos.x));
"

Then after all of that is done we apply these changes to the col variable:
"
col += fogTrail * 0.5;
col += trail;
col += drop;

float2 offs = drop * dropPos + trail*trailPos;
col = tex2D(_MainTex, i.uv+offs * _Distortion);
return col;
"


After I made the shader I made a rain effect to the world with VFX Graph.

In the Spawn Node I set the spawn rate to constant and to around 100000, which should be enough particles.

After that In the System Node I set the bounds size to 100 each.

I then set a random velocity to each particle, from -24 to 30, which seemed to mimic rain pretty well.

I set a life time of 7, so the raindrops don't exist forever and they can be destroyed.

I set a random position of "-bound size, bound size" to the x and z coordinates, but for the y coordinates I set the spawn point to 10, 
which is the start of the bounds.

I set a random scale for the rain drops, I made them skinny but tall, to mimic raindrops.

Then in the update particle node I set turbulence to even further randomize the rain. I set intensity to 4 and octaves to 2.

And lastly, in the Output Particle Node I changed the base color map to another unity particle texture which worked pretty well.


If I had more time:
- I would've tried to add a normal map to the rain shader, but I didn't have enough time to implement that.
- I would've tried to make more variables into the inspector of the shader like the size of the raindrops.

-
