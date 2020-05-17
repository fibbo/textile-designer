<script>
import Debug from './Debug.svelte';
import { createEventDispatcher } from 'svelte';
const dispatch = createEventDispatcher();
const CANVAS_X_DIM = 800;
const CANVAS_Y_DIM = 600;

let imageLayer = {
    img: new Image(),
    rect : {
        x: 30,
        y: 30,
        w: 220,
        h: 60,
    },
    aspectRatio: 0,
}

let state = {
    canvasSize: 0.2,
    dpi: 150
}

// export let pointerInfo = {
//     dragging: false,
//     pointerInImage: false,
//     canvasX: 0,
//     canvasY: 0,
//     imageOffsetX: 0,
//     imageOffsetY: 0,
// }

// function clampToCanvas() {
//     if (imageLayer.position.x < 0) {
//         imageLayer.position.x = 0;
//     }
//     if (imageLayer.position.x + imageLayer.position.w > CANVAS_X_DIM) {
//         imageLayer.position.x = CANVAS_X_DIM - imageLayer.position.w;
//     }
//     if (imageLayer.position.y < 0) {
//         imageLayer.position.y = 0;
//     }
//     if (imageLayer.position.y + imageLayer.position.h > CANVAS_Y_DIM) {
//         imageLayer.position.y = CANVAS_Y_DIM - imageLayer.position.h;
//     }
// }

// function updatePointerInfo(ev) {
//     const cvs = document.getElementById('canvas')
//     const ctx = cvs.getContext('2d')
//     const rect = cvs.getBoundingClientRect();
//     pointerInfo.pointerInImage = pointerInImage()
//     pointerInfo.canvasX = ev.clientX - rect.x;
//     pointerInfo.canvasY = ev.clientY - rect.y; 
// }

// function pointerInImage() {
//     return imageLayer.img && imageLayer.position.x <= pointerInfo.canvasX &&
//            imageLayer.position.x + imageLayer.position.w >= pointerInfo.canvasX &&
//            imageLayer.position.y <= pointerInfo.canvasY &&
//            imageLayer.position.y + imageLayer.position.h >= pointerInfo.canvasY
// }

// function OnPointerDown(ev) {
//     const cvs = document.getElementById('canvas');
//     cvs.setPointerCapture(ev.pointerId);
//     ev.preventDefault();
//     updatePointerInfo(ev);
//     if (pointerInImage()) {
//         pointerInfo.dragging = true;
//     }
//     pointerInfo.imageOffsetX = pointerInfo.canvasX - imageLayer.position.x;
//     pointerInfo.imageOffsetY = pointerInfo.canvasY - imageLayer.position.y;
// }


// function OnPointerMove(ev) {
//     ev.preventDefault();
//     updatePointerInfo(ev);
//     if (pointerInfo.dragging) {
//         const cvs = document.getElementById('canvas')
//         const ctx = cvs.getContext('2d')
//         imageLayer.position.x = pointerInfo.canvasX - pointerInfo.imageOffsetX;
//         imageLayer.position.y = pointerInfo.canvasY - pointerInfo.imageOffsetY;
//         imageLayer.position.h = imageLayer.position.w / imageLayer.aspectRatio;
//         clampToCanvas();
//         ctx.clearRect(0, 0, cvs.width, cvs.height);
//         ctx.globalAlpha = 0.4;
//         ctx.drawImage(imageLayer.img, imageLayer.position.x, imageLayer.position.y, imageLayer.position.w, imageLayer.position.h)
//     }
// }

// function OnPointerUp(ev) {
//     const cvs = document.getElementById('canvas')
//     cvs.releasePointerCapture(ev.pointerId);
//     ev.preventDefault();
//     const ctx = cvs.getContext('2d')
//     ctx.globalAlpha = 1.0;
//     if (pointerInfo.dragging) {
//         pointerInfo.dragging = false;
//         ctx.drawImage(imageLayer.img, imageLayer.position.x, imageLayer.position.y, imageLayer.position.w, imageLayer.position.h)
//     }
// }

function OnChange(ev) {
    console.log(ev)

    const file = ev.target.files[0];
    console.log(file)
    const reader = new FileReader()

    reader.onload = (res) => {
        const cvs = document.getElementById('canvas')
        const ctx = cvs.getContext('2d')
        imageLayer.img.onload = function(){
            imageLayer.aspectRatio = imageLayer.img.width/imageLayer.img.height;
            imageLayer.rect.x = cvs.width / 2 - imageLayer.img.width / 2
            imageLayer.rect.y = cvs.height / 2 - imageLayer.img.height/ 2
            imageLayer.rect.w = imageLayer.img.width
            imageLayer.rect.h = imageLayer.img.height
            ctx.drawImage(imageLayer.img, imageLayer.rect.x, imageLayer.rect.y , imageLayer.img.width, imageLayer.img.height);
        }
        imageLayer.img.src = res.target.result;
    }
    reader.readAsDataURL(ev.target.files[0]); 

}

function draw() {
    const cvs = document.getElementById('canvas')
    const ctx = cvs.getContext('2d')
    imageLayer.rect.x = cvs.width / 2 - imageLayer.rect.w / 2
    imageLayer.rect.y = cvs.height / 2 - imageLayer.rect.h / 2
    console.log(imageLayer.rect.w, imageLayer.rect.h)
    ctx.clearRect(0, 0, cvs.width, cvs.height)
    ctx.drawImage(imageLayer.img, imageLayer.rect.x, imageLayer.rect.y , imageLayer.rect.w, imageLayer.rect.h);
}

function OnMakeSmaller(ev) {
    imageLayer.rect.w *= 0.9
    imageLayer.rect.h *= 0.9
    draw()
}

function OnMakeBigger(ev) {
    imageLayer.rect.w *= 1.1
    imageLayer.rect.h *= 1.1
    draw()
}
</script>

<div id='container'>
    <canvas id='canvas' width="{CANVAS_X_DIM}px" height="{CANVAS_Y_DIM}px"
    ></canvas><br>
    <input type='file' id='file-input' on:change={OnChange}>
    <button on:click={OnMakeSmaller}>Smaller</button>
    <button on:click={OnMakeBigger}>Bigger</button>
</div>

<style>
#container {
    display: inline-block;
}

#canvas {
    border: 1px black dashed;
}

</style>