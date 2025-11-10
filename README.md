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

## Color Interpolation
* You can interpolate different color channels of your graphcics.
* To Extract these you can use these Variables:

```java
int r = ((color >> 16) & 0xff);
int g = ((color >> 8) & 0xff);
int b = ((color) & 0xff);
```

* to merge them back you can use the bitwise or operator:

```java
int newColor = (r << 16) | (g << 8) | (b);
```


## Summary Table

| Topic                   | Description                                  |
| ----------------------- | -------------------------------------------- |
| Vector Normalisation    | Calculate direction independent of length    |
| Acceleration / Friction | Smooth movement with realistic acceleration  |
| Tilemaps                | Level structure and collision detection      |
| Entity Getter           | Efficient tile-based entity hitbox detection |
| Sprite Animation        | Control frame display over time              |
| onGround Variable       | Detect ground contact                        |
| Input Tweaks            | Prevent repeated input from held buttons     |
| Movement Gimmicks       | Small mechanics to make movement feel alive  |
