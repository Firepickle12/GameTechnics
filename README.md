# Game Programming Mechanics

* **Vector Normalisation**
* **Acceleration / Friction Movement**
* **Tilemaps**
* **Entity getter** with tile-based entity recognition → `getEntity();`
  Useful for creating hitboxes for any entity.
* **Sprite animation patterns** with life and duration
* **onGround** variable when y-acceleration is 0
* Prevent repeated button input every tick when held (see **old button** example)
* Always build in small movement gimmicks

---

## Vector Normalisation

```java
double dx = targetX - x;
double dy = targetY - y;
double dist = Math.sqrt(dx * dx + dy * dy);
if (dist != 0) {
    dx /= dist;
    dy /= dist;
}
```

## Acceleration / Friction Movement

```java
public void tick() {
    onGround = false;

    x += xa;
    y += ya;

    xa *= Level.FRICTION;
    ya *= Level.FRICTION;
    ya += Level.GRAVITY;
}
```

## Nested Loops
* if you create a nested for-loop for tilerendering you can specify the area with x0, x1, y0, y1
* these are the coordinates and if you render it with these for loops you specify it like this

```java
	for (int x = x0; x < x1; x++) {
		for (int y = y0; y < y1; y++) {
			g.setColor(new Color(Math.random() * 0xffffff));
			g.fillRect(x * 8, y * 8, 8, 8) // Example
		}
	}

	// specified area width (x1 - x0)
	// specified area height (y1 - y0)
```

## Tilemaps

```java
byte[] walls = new byte[width * height];
walls[x + y * width] = 1; // 1 = Wall, 0 = empty

public boolean isFree(double x, double y, int w, int h) {
    int x0 = (int)(x / 10);
    int y0 = (int)(y / 10);
    int x1 = (int)((x + w - 0.1) / 10);
    int y1 = (int)((y + h - 0.1) / 10);

    for (int xx = x0; xx <= x1; xx++)
        for (int yy = y0; yy <= y1; yy++)
            if (walls[xx + yy * width] != 0) return false;
    return true;
}
```

## Entity Getter (Tile-based Entity Recognition)

```java
public List<Entity> getEntities(int xc, int yc, int w, int h) {
    hits.clear();
    int r = 20;
    int x0 = (int)((xc - r) / 10);
    int y0 = (int)((yc - r) / 10);
    int x1 = (int)((xc + w + r) / 10);
    int y1 = (int)((yc + h + r) / 10);

    for (int x = x0; x <= x1; x++)
        for (int y = y0; y <= y1; y++)
            if (x >= 0 && y >= 0 && x < width && y < height)
                for (Entity e : entityMap[x + y * width]) {
                    double xx0 = e.x,
					double yy0 = e.y;
                    double xx1 = e.x + e.w,
					double yy1 = e.y + e.h;

                    if (xx0 > xc + w || yy0 > yc + h || xx1 < xc || yy1 < yc) continue;
                        hits.add(e);
                }

    return hits;
}
```

## Sprite Animation Patterns

```java
int frame = (elapsed * totalFrames) / totalDuration;
g.drawImage(spriteSheet[frame], x, y, null);

int frame = (time / slowness) % totalFrames;
g.drawImage(spriteSheet[spriteId + frame], x, y, null);
```

## onGround Variable

```java
if (ya == 0) {
    onGround = true;
} else {
    onGround = false;
}
```

## Button Input Tweaks

```java
public boolean[] buttons = new boolean[64];
public boolean[] oldButtons = new boolean[64];

public void tick() {
    for (int i = 0; i < buttons.length; i++) {
        oldButtons[i] = buttons[i];
    }
}

if (buttons[JUMP] && !oldButtons[JUMP]) jump();
```

## Movement Gimmicks & Mini-Mechanics

```java
if (onIce) xa *= 0.995;
else xa *= Level.FRICTION;
```

## tilemap mechanics
* To get Tile coords divide sheetwidth and sheet height through tilewidth and tileheight
* You can iterate over these tilecoords and make an individual array for each tile

```java
// Division
int tileIndex = sheetWidth() / tilewidth;
int tileIndex = sheetHeight() / tileheight;

int[] tilePixels = new int[tilewidth * tileheight];

// get Tilevalues with
getRGB(x, y, xc, yc, array, start, scanlinesize);

// set Tilevalues with
setRGB(x, y, xc, yc, array, start, scanlinesize);
```

## tilebased Lighting
* To get tilebased lighting you need to create an array which saves the light values for your tiles

```java
	int[] seen = new int[width * height];

	// you can make light rays from different ponts with this design
	private void lightRay(int x0, int y0, int x1, int y1, int r) {

		// Optional for circular lighting
		int rd = r;
		{
			double xd = (x1 - x0);
			double yd = (y1 - y0);
			double dist = Math.sqrt(xd * xd + yd * yd) / r;
			rd = (int) (r / dist);
		}
		
		
		for (int i = 0; i <= rd; i++) {
			int x = x0 + (x1 - x0) * i / r;
			int y = y0 + (y1 - y0) * i / r;
			
			if (isFree(x, y)) {
				return;
			}

			// Optional for smooth circular lighting 
			for (int xx = -1; xx <= 1; xx++) {
				for (int yy = -1; yy <= 1; yy++) {
					if (x >= 0 && y >= 0 && x < width && y < height) {
						seen[x + y * width] = 2; // Your light value
					}
				}
			}
		}
	}
```

## Color Interpolation
* You can interpolate different color channels of your graphcics.
* To Extract these you can use these Variables:

```java
int r = ((color >> 16) & 0xff);
int g = ((color >> 8) & 0xff);
int b = ((color) & 0xff);

// Here you can moify the strength of each color channel 0xff(255) bright | 0x00(0) dark
r = r * rCol / 255;
b = b * bCol / 255;
g = g * gCol / 255;

// you can use this system for linear contrast lowering

// Lower every color by ~60% darker tones wont be affected as much beacuse they can't get darker
r = r * (255 - 60) / 255;
g = g * (255 - 60) / 255;
b = b * (255 - 60) / 255;

// Add back these colors and the bright colors turn normal and dark colors that werent affected by the color lowering bevause they're now proportionaly brighter than before
r += 60;
g += 60;
b += 60;
```



* to merge them back you can use the bitwise or operator:

```java
int newColor = (r << 16) | (g << 8) | (b);
```

## Tile Scrolling
* If you made a few tiles you can scroll them if you render them with nested loops (x and y)
* You add the offset to the nested for loop to extend the tile canvas in which these are rendered and you subtract this offset on the tiles you actually render.
* So for example you add an xOffset to the horizontal Size of your level or tilemap the Level gets expanded on the x-Axis. If you subtract this Offset from the xCoord from your tile it doesn't get rendered anymore on the left side and this offsetted tile gets moved to its original position

```java
		for (int x = x0 + xo; x < x1 + xo; x++) {
			for (int y = y0 + yo; y < y1 + yo; y++) {

				setTile(x - xo, y - yo);
			}
		}
	}
```
