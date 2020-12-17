<template>
  <div class="home">
    <div id="canvas">

    </div>
  </div>
</template>

<script>
  import {defineComponent, onMounted} from 'vue'
  import * as PIXI from 'pixi.js'

  export default defineComponent({
    name: 'home',
    setup() {
      let app,
        explorerHit,
        blobs = [],
        treasure,
        door,
        explorer,
        gameScene,
        gameOverScene,
        message,
        state
      //Aliases
      const Application = PIXI.Application,
        Container = PIXI.Container,
        Graphics = PIXI.Graphics,
        TextureCache = PIXI.utils.TextureCache,
        Sprite = PIXI.Sprite,
        Text = PIXI.Text,
        TextStyle = PIXI.TextStyle

      //The `randomInt` helper function
      function randomInt(min, max) {
        return Math.floor(Math.random() * (max - min + 1)) + min
      }

      function hitTestRectangle(r1, r2) {

        //Define the variables we'll need to calculate
        let hit, combinedHalfWidths, combinedHalfHeights, vx, vy;

        //hit will determine whether there's a collision
        hit = false;

        //Find the center points of each sprite
        r1.centerX = r1.x + r1.width / 2;
        r1.centerY = r1.y + r1.height / 2;
        r2.centerX = r2.x + r2.width / 2;
        r2.centerY = r2.y + r2.height / 2;

        //Find the half-widths and half-heights of each sprite
        r1.halfWidth = r1.width / 2;
        r1.halfHeight = r1.height / 2;
        r2.halfWidth = r2.width / 2;
        r2.halfHeight = r2.height / 2;

        //Calculate the distance vector between the sprites
        vx = r1.centerX - r2.centerX;
        vy = r1.centerY - r2.centerY;

        //Figure out the combined half-widths and half-heights
        combinedHalfWidths = r1.halfWidth + r2.halfWidth;
        combinedHalfHeights = r1.halfHeight + r2.halfHeight;

        //Check for a collision on the x axis
        if (Math.abs(vx) < combinedHalfWidths) {

          //A collision might be occuring. Check for a collision on the y axis
          if (Math.abs(vy) < combinedHalfHeights) {

            //There's definitely a collision happening
            hit = true;
          } else {

            //There's no collision on the y axis
            hit = false;
          }
        } else {

          //There's no collision on the x axis
          hit = false;
        }

        //`hit` will be either `true` or `false`
        return hit;
      }

      // todo
      function contain(sprite, container) {
        let collision = undefined
        //  left
        if (sprite.x < container.x) {
          sprite.x = container.x
          collision = 'left'
        }
        if (sprite.y < container.y) {
          sprite.y = container.y
          collision = 'top'
        }
        //  right
        if (sprite.x + sprite.width > container.width) {
          sprite.x = container.width - sprite.width
          collision = 'right'
        }
        // bottom
        if (sprite.y + sprite.height > container.height) {
          sprite.x = container.height - sprite.height
          collision = 'bottom'
        }

        return collision
      }

      function keyboard(keyCode) {
        let key = {}
        key.code = keyCode;
        key.isDown = false;
        key.isUp = true;
        key.press = undefined;
        key.release = undefined;

        key.downHandler = function (event) {
          if (event.keyCode === key.code) {
            if (key.isUp && key.press) key.press()
          }
        }
      }

      // 游戏ing
      function play() {
        blobs.forEach((blob) => {
          blob.y += blob.vy
          let blockHitWall = contain(blob, {x: 28, y: 10, width: 488, height: 400});

          if (blockHitWall === 'top' || blockHitWall === 'bottom') {
            blob.vy *= -1;
          }

          if (hitTestRectangle(explorer, blob)) {
            explorerHit = true
          }
        })

        if (hitTestRectangle(treasure, door)) {
          state = end;
          message.text = "you lost"
        }

        // 移动探险者


      }


      function start() {
        gameScene = new Container()
        app.stage.addChild(gameScene)

        gameOverScene = new Container()
        app.stage.addChild(gameOverScene)
        gameOverScene.visible = false

        const loader = app.loader
        const resources = loader.resources

        const id = resources["images/treasureHunter.json"].textures

        const dungeon = new Sprite(id['dungeon.png'])
        gameScene.addChild(dungeon)

        door = new Sprite(id['door.png'])
        door.position.set(32, 0)
        gameScene.addChild(door)

        //  Explorer
        explorer = new Sprite(id['explorer.png'])
        explorer.x = 68
        explorer.y = gameScene.height / 2 - explorer.height / 2
        explorer.vx = 0
        explorer.vy = 0
        gameScene.addChild(explorer)

        // Treasure
        treasure = new Sprite(id['treasure.png'])
        treasure.x = gameScene.width - treasure.width - 48
        treasure.y = gameScene.height / 2 - treasure.height / 2
        gameScene.addChild(treasure)

        let numberOfBlobs = 6, spacing = 48, xOffset = 150, speed = 2, direction = 1


        for (let i = 0; i < numberOfBlobs; i++) {
          let blob = new Sprite(id['blob.png'])
          let x = spacing * i + xOffset
          let y = randomInt(0, app.stage.height - blob.height)

          blob.x = x
          blob.y = y
          // @ts-ignore
          blob.vy = speed * direction
          direction *= -1
          blobs.push(blob)
          gameScene.addChild(blob)
        }

        let left = keyboard(37)

        state = play
        // 游戏循环：
        app.ticker.add(delta => gameLoop(delta))
      }

      function gameLoop(delta) {
        state(delta)
      }


      function end() {
        gameScene.visible = false
        gameOverScene.visible = true
      }

      onMounted(() => {
        app = new Application({
          width: 512,
          height: 512,
          transparent: false,
          resolution: 1
        })
        app.loader
          .add("images/treasureHunter.json")
          .load(start)
        document.getElementById('canvas').appendChild(app.view)
      })
      return {}
    }
  })

</script>
