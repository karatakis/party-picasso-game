<template>
  <div>
    <div ref="parent"></div>
  </div>
</template>

<script>
// const Phaser = window.Phaser
import * as Phaser from 'phaser/dist/phaser-arcade-physics'
import _ from 'lodash'

export default {
  name: 'Demo',
  data () {
    return {
      game: null
    }
  },
  mounted () {
    /**
     * image compents
     */
    const parts = [
      'ear',
      'left_eye',
      'left_eyebrow',
      'mouth',
      'nose',
      'right_eye',
      'right_eyebrow'
    ]

    /**
     * LOAD ASSETS
     */
    function preload () {
      this.load.setBaseURL('/game_assets/woman-in-hat-and-fur-collar/')

      this.load.image('base', 'base.png')

      parts.forEach((part) => {
        this.load.image(part, part + '.png')
      })
    }

    /**
     * Create scene
     */
    function create () {
      /**
       * Used for double click gesture
       */
      let lastClicked = null

      // add base image
      this.add.image(247, 400, 'base')

      // used to indicate delete part area
      const deleteRect = this.add.rectangle(247, 750, 494, 100, 0xFF8800)

      // used to indicate resize part area
      const increaseSizeRect = this.add.rectangle(150, 50, 100, 100, 0x00FF00)
      // used to inidcate resize part area
      const decreaseSizeRect = this.add.rectangle(50, 50, 100, 100, 0xFF00000)

      // used to inidcate rotate part area
      const rotateLeftRect = this.add.rectangle(350, 50, 100, 100, 0x00EEFF)
      // used to inidcate rotate part area
      const rotateRightRect = this.add.rectangle(450, 50, 100, 100, 0x0000FF)

      /**
       * Register rectangles to physics engine
       */
      this.physics.add.existing(deleteRect)

      this.physics.add.existing(increaseSizeRect)
      this.physics.add.existing(decreaseSizeRect)
      this.physics.add.existing(rotateLeftRect)
      this.physics.add.existing(rotateRightRect)

      /**
       * Used to keep the user defined parts
       * Will be kept after submission
       */
      const componentsGroup = this.physics.add.group()

      /**
       * Used to make tranformations @part
       */
      const transform = _.throttle((transformType, rect, part) => {
        if (!rect.getBounds().contains(part.x, part.y)) {
          return
        }
        switch (transformType) {
          case 'increaseSizeRect':
            part.setDisplaySize(part.width += 5, part.height += 5)
            break
          case 'decreaseSizeRect':
            part.setDisplaySize(part.width -= 5, part.height -= 5)
            break
          case 'rotateLeftRect':
            part.setAngle(part.angle -= 4)
            break
          case 'rotateRightRect':
            part.setAngle(part.angle += 4)
            break
          default:
            console.error('Unknown tranform type ', transformType)
            break
        }
      }, 100, {
        leading: true
      })

      // used to detect if item of componentsGroup overlaps delete area
      // and deletes area
      this.physics.add.overlap(deleteRect, componentsGroup, (rect, part) => {
        componentsGroup.remove(part, true, true)
      })

      /**
       * Used to detect if item of componentsGroup overlaps trasformation area
       * and then call transformation function
       */
      this.physics.add.overlap(increaseSizeRect, componentsGroup, (rect, part) => {
        transform('increaseSizeRect', rect, part)
      })
      this.physics.add.overlap(decreaseSizeRect, componentsGroup, (rect, part) => {
        transform('decreaseSizeRect', rect, part)
      })
      this.physics.add.overlap(rotateLeftRect, componentsGroup, (rect, part) => {
        transform('rotateLeftRect', rect, part)
      })
      this.physics.add.overlap(rotateRightRect, componentsGroup, (rect, part) => {
        transform('rotateRightRect', rect, part)
      })

      /**
       * Usedd to create cloned copy of @part
       */
      const createPart = (x, y, part) => {
        const partObj = this.physics.add.image(x, y, part)

        partObj.setInteractive()

        this.input.setDraggable(partObj)

        partObj.on('pointerdown', () => {
          const timeNow = new Date().getTime()
          if (timeNow - lastClicked <= 200) {
            partObj.setFlipX(!partObj.flipX)
          }
          lastClicked = timeNow
        })

        partObj.once('pointerdown', (pointer) => {
          componentsGroup.add(partObj)
          createPart(x, y, part)
        })
      }

      /**
       * Generate painting parts on toolbox bar
       */
      let yDiff = 150
      let xDiff = 50
      parts.forEach((part) => {
        const x = 494 + xDiff
        const y = yDiff

        console.log('add part', part, x, y)

        createPart(x, y, part)

        yDiff += 90
        if (yDiff >= 700) {
          yDiff = 150
          xDiff += 100
        }
      })

      /**
       * Used to enable drag of @part
       */
      this.input.on('drag', function (pointer, gameObject, dragX, dragY) {
        gameObject.x = dragX
        gameObject.y = dragY
      })
    }

    const config = {
      parent: this.$refs.parent,
      type: Phaser.AUTO,
      width: 494 + 200,
      height: 800,
      physics: {
        default: 'arcade',
        arcade: {
          gravity: 0
        }
      },
      scene: {
        preload: preload,
        create: create
      }
    }

    const game = new Phaser.Game(config)

    this.game = game
  },
  destroyed () {
    this.game.destroy(false, false)
  }
}
</script>
