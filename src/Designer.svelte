<script>
import Debug from './Debug.svelte';
import { createEventDispatcher } from 'svelte';
const dispatch = createEventDispatcher();
const CANVAS_X_DIM = 600;
const CANVAS_Y_DIM = 600;
const TO_CM = 2.54
let imageLayer = {
    image: null,
    rect : {
        x: 0,
        y: 0,
        w: 0,
        h: 0,
    },
    aspectRatio: 0,
}
const LAYOUT = {
    'NONE': 0,
    'NORMAL': 1,
    'VERTICAL_OFFSET': 2,
    'HORIZONTAL_OFFSET': 3,
    'MIRROR_REFLECTION': 4,
    'DOUBLE_MIRROR_REFLECTION': 5,
}

let state = {
    canvasSizeCM: 20,
    dpi: 150,
    dpiStepSize: 15,
    minDpi: 150,
    selectedLayout: LAYOUT.NONE
}


function OnChange(ev) {
    console.log(ev)

    const file = ev.target.files[0];
    console.log(file)
    const reader = new FileReader()
    imageLayer.image = new Image()
    reader.onload = (res) => {
        const cvs = document.getElementById('canvas')
        const ctx = cvs.getContext('2d')
        imageLayer.image.onload = function(){
            imageLayer.aspectRatio = imageLayer.image.width/imageLayer.image.height;
            imageLayer.rect.w = imageLayer.image.width
            imageLayer.rect.h = imageLayer.image.height
            // ctx.drawImage(imageLayer.image, imageLayer.rect.x, imageLayer.rect.y , imageLayer.image.width, imageLayer.image.height);
            Resize()
            Draw()
        }
        imageLayer.image.src = res.target.result;
    }
    reader.readAsDataURL(ev.target.files[0]); 

}

function Draw() {
    if (imageLayer.image === null) {
        return
    }
    const cvs = document.getElementById('canvas')
    const ctx = cvs.getContext('2d')
    ctx.clearRect(0, 0, cvs.width, cvs.height)
    if (state.selectedLayout === LAYOUT.NONE) {
        console.log('DRAW NONE')
        imageLayer.rect.x = cvs.width / 2 - imageLayer.rect.w / 2
        imageLayer.rect.y = cvs.height / 2 - imageLayer.rect.h / 2
        ctx.drawImage(imageLayer.image, imageLayer.rect.x, imageLayer.rect.y , imageLayer.rect.w, imageLayer.rect.h);
    } else if (state.selectedLayout === LAYOUT.NORMAL) {
        console.log('DRAW NORMAL')
        let y = 0
        while (y < CANVAS_Y_DIM) {
            let x = 0
            while (x < CANVAS_X_DIM) {
                ctx.drawImage(imageLayer.image, x, y, imageLayer.rect.w, imageLayer.rect.h)
                x += imageLayer.rect.w
            }
            y += imageLayer.rect.h
        }
    } else if (state.selectedLayout === LAYOUT.VERTICAL_OFFSET) {
        console.log('DRAW VERTICAL OFFSET')
        let x = 0
        let oddColumn = false
        while (x < CANVAS_X_DIM) {
            let y = 0 
            if (oddColumn) {
                y -= imageLayer.rect.h / 2
            }
            while (y < CANVAS_Y_DIM) {
                ctx.drawImage(imageLayer.image, x, y, imageLayer.rect.w, imageLayer.rect.h)
                y += imageLayer.rect.h
            }
            oddColumn = !oddColumn
            x += imageLayer.rect.w
        }
    } else if (state.selectedLayout === LAYOUT.HORIZONTAL_OFFSET) {
        console.log("DRAW HORIZONTAL OFFSET")
        let y = 0
        let oddRow = false
        while (y < CANVAS_Y_DIM) {
            let x = 0
            if (oddRow) {
               x -= imageLayer.rect.w / 2 
            }
            while (x < CANVAS_X_DIM) {
                ctx.drawImage(imageLayer.image, x, y, imageLayer.rect.w, imageLayer.rect.h)
                x += imageLayer.rect.w
            }
            oddRow = !oddRow
            y += imageLayer.rect.h
        }
    } else if (state.selectedLayout === LAYOUT.MIRROR_REFLECTION) {
        console.log("DRAW MIRROR REFLECTION")
        let y = 0
        let mirrorRow = false
        while (y < CANVAS_Y_DIM) {
            let x = 0
            ctx.save()
            if (mirrorRow) {
                ctx.scale(1, -1)
            }
            while (x < CANVAS_X_DIM) {
                ctx.drawImage(imageLayer.image, x, mirrorRow ? -y + 1 : y, imageLayer.rect.w, mirrorRow ? -imageLayer.rect.h : imageLayer.rect.h)
                x += imageLayer.rect.w
            }
            ctx.restore()
            mirrorRow = !mirrorRow
            y += imageLayer.rect.h
        }
    } else if (state.selectedLayout === LAYOUT.DOUBLE_MIRROR_REFLECTION) {
        console.log("DRAW DOUBLE MIRROR REFLECTION")
        let y = 0
        let mirrorRow = false
        while (y < CANVAS_Y_DIM) {
            let x = 0
            ctx.save()
            if (mirrorRow) {
                ctx.transform(1, 0, 0, -1, 0, 0)
            }
            let mirrorColumn = false
            while (x < CANVAS_X_DIM) {
                ctx.save()
                if (mirrorColumn) {
                    ctx.transform(-1, 0, 0, 1, 0, 0)
                }
                ctx.drawImage(imageLayer.image, mirrorColumn ? -x : x, mirrorRow ? -y + 1 : y, mirrorColumn ? -imageLayer.rect.w : imageLayer.rect.w, mirrorRow ? -imageLayer.rect.h : imageLayer.rect.h)
                x += imageLayer.rect.w
                mirrorColumn = !mirrorColumn
                ctx.restore()
            }
            ctx.restore()
            mirrorRow = !mirrorRow
            y += imageLayer.rect.h
        }
    }
}

function Resize() {
    const sizeInCm = imageLayer.image.width / state.dpi * TO_CM
    const w = sizeInCm * CANVAS_X_DIM / state.canvasSizeCM
    const factor = w / imageLayer.rect.w

    imageLayer.rect.w = w
    imageLayer.rect.h *= factor 
}

function OnMakeSmaller(ev) {
    state.dpi += state.dpiStepSize
    Resize()
    Draw()
}

function OnMakeBigger(ev) {
    if (state.dpi <= state.minDpi) {
        // 150 DPI is the minimum DPI to ensure decent print quality
        return
    }
    state.dpi -= state.dpiStepSize
    Resize()
    Draw()
}

function OnNormal() {
    state.selectedLayout = LAYOUT.NORMAL
    Draw()()
}
</script>

<div id='container'>
    <canvas id='canvas' width="{CANVAS_X_DIM}px" height="{CANVAS_Y_DIM}px"
    ></canvas><br>
    <input type='file' id='file-input' on:change={OnChange}>
    <button class='size' on:click={OnMakeSmaller}>-</button>
    <button class='size' on:click={OnMakeBigger}>+</button>
    <br>
    <button on:click={() => { state.selectedLayout = LAYOUT.NONE; Draw();}}>None</button>
    <button on:click={() => { state.selectedLayout = LAYOUT.NORMAL; Draw();}}>Normal</button>
    <button on:click={() => { state.selectedLayout = LAYOUT.VERTICAL_OFFSET; Draw();}}>Vertical Offset</button>
    <button on:click={() => { state.selectedLayout = LAYOUT.HORIZONTAL_OFFSET; Draw();}}>Horizontal Offset</button>
    <button on:click={() => { state.selectedLayout = LAYOUT.MIRROR_REFLECTION; Draw();}}>Mirror Reflection</button>
    <button on:click={() => { state.selectedLayout = LAYOUT.DOUBLE_MIRROR_REFLECTION; Draw();}}>Double Mirror Reflection</button>
    <br>
    <button on:click={() => { state.canvasSizeCM = 100; Resize(); Draw(); }}>Linear meter</button>
    <button on:click={() => { state.canvasSizeCM = 20; Resize(); Draw(); }}>Sample</button>
    <Debug {state} {imageLayer}/>
</div>

<style>
#container {
    display: inline-block;
}

.size {
    width: 40px;
    height: 40px;
    color: #fff;
    font-size: 30px;
    line-height: 34px;
    text-align: center;
    background: #434244;
    margin-right: 10px;
    font-family: Arial;
    text-decoration: none;
}

#canvas {
    border: 1px black dashed;
}

</style>