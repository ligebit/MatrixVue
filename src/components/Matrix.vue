<script lang="ts">
//import img from './image1.jpg';
import img from './image.png';

interface ILine {
    xChar: number,

    yPixels: number,

    lengthChar: number,

    speed: number;
}

const PROBABILITY_MULTIPLIZATOR = 1;

//% creation in iter
const PROBABILITY_CREATE_LINE = 0.3;

//pixels in iter
const MIN_SPEED_LINE = 6;
const MAX_SPEED_LINE = 6;

const MIN_LENGTH_LINE = 5;
const MAX_LENGTH_LINE = 15;

const CHAR_SIZE = 14;

//let chars = [] as string[][];
let lines = [] as ILine[];


interface IChar {
    char: string,
    color: number,
    alphaArray: number[],
}

class Char implements IChar {
    char: string;
    color: number;

    alphaArray: number[];

    constructor(char: string) {
        this.char = char;
        this.color = 0;
        this.alphaArray = []
    }

    addAlpha(value: number) {
        this.alphaArray.push(value);
    }
    
    clearAlpha() {
        if(this.color != 0 ) this.alphaArray = [1];
        else this.alphaArray = [];
        // const alpha = (this.color/255)*this.color;
        // this.alphaArray = [alpha];
    }

    get alpha(): number {
        if(this.alphaArray.length == 0) return 0;
        return Math.max(...this.alphaArray);
    }
}


let chars = [] as Char[][];


export default {
    name: 'MatrixLoader',

    data() {
        return {
            characterList: 'QWERTYUIOPASDFGHJKLZXCVBNM~!@#$%^&*()_+`1234567890-=;:/?.>,<',
            interval: setInterval(this.iter, 1000 / 60),
            fps: 0,

            x: 0,
            initWidth: 0,
            initHeight: 0,
        }
    },
    computed: {
        canvas() {
            return this.$refs.canvas as HTMLCanvasElement;
        },
        ctx() {
            return this.canvas.getContext('2d') as CanvasRenderingContext2D;
        }
    },
    mounted() {
        this.initWidth = window.innerWidth;
        this.initHeight = window.innerHeight;

        this.canvas.width = window.innerWidth;
        this.canvas.height = window.innerHeight;

        window.onresize = this.resizeCanvas;
        this.resizeCanvas();

        lines = Array();

        for (let i = 0; i < Math.ceil(this.canvas.width / CHAR_SIZE); i++) {
            chars[i] = [];
            for (let j = 0; j < Math.ceil(this.canvas.height / CHAR_SIZE); j++) {
                this.setRandomChar(i, j);
            }
        }
    },

    created() {
        this.workImage();
    },
    methods: {
        resizeCanvas() {
            // if(window.innerWidth/window.innerHeight != this.initWidth/this.initHeight) {
            //     this.x = (window.innerWidth - this.initWidth) / 2;
            // }

            // this.canvas.width = window.innerWidth;
            // // this.canvas.height = window.innerHeight;

            // if(chars.length * CHAR_SIZE + (CHAR_SIZE * 2) < this.canvas.width) {
            //     chars.unshift([]);
            //     chars.push([]);
            //     for (let j = 0; j < Math.ceil(this.canvas.height / CHAR_SIZE); j++) {
            //         this.setRandomChar(0, j);
            //     }
            //     for (let j = 0; j < Math.ceil(this.canvas.height / CHAR_SIZE); j++) {
            //         this.setRandomChar(chars.length - 1, j);
            //     }
            // }

            // if(chars.length * CHAR_SIZE > this.canvas.width + (CHAR_SIZE * 2)) {
            //     chars.shift();
            //     chars.pop();
            // }

            this.ctx.font = `${CHAR_SIZE}px Inconsolata, monospace`;
        },
        iter() {
            const lastDate = Date.now();
            for (let i = 0; i < PROBABILITY_MULTIPLIZATOR; i++) {
                if (Math.random() < PROBABILITY_CREATE_LINE) {
                    const index = Math.floor(Math.random() * chars.length)

                    this.generateLine(index);
                }
            }

            this.clear();
            this.work();
            this.draw();

            this.fps = 1000/(Date.now() - lastDate)
        },
        generateLine(x: number) {
            lines.push({
                yPixels: 0,
                lengthChar: this.getRandomLengthLine(),
                xChar: x,
                speed: this.getRandomSpeedLine(),
            });
        },
        getRandomChar() {
            return this.characterList[Math.floor(Math.random() * this.characterList.length)];
        },
        setRandomChar(x: number, y: number) {
            chars[x][y] = new Char(this.getRandomChar());
        },
        getRandomLengthLine() {
            return Math.floor(Math.random() * (MAX_LENGTH_LINE - MIN_LENGTH_LINE) + MIN_LENGTH_LINE);
        },
        getRandomSpeedLine() {
            return Math.floor(Math.random() * (MAX_SPEED_LINE - MIN_SPEED_LINE) + MIN_SPEED_LINE);
        },
        work() {
            lines.map((line, index) => {
                this.workLine(line, index)
            });
        },
        draw() {
            for (let i = 0; i < chars.length; i++) {
                for (let j = 0; j < chars[i].length; j++) {

                    if(chars[i][j].alpha <= 0) continue;
                    
                    this.ctx.globalAlpha = chars[i][j].alpha;
                    this.ctx.fillStyle = `rgb(${chars[i][j].color}, ${chars[i][j].color}, ${chars[i][j].color})`;
                    // else
                    // this.ctx.fillStyle = `rgb(
                    //     ${yChar + yLine},
                    //     ${lengthLine * CHAR_SIZE},
                    // 0)`;

                    //this.ctx.fillRect(x*CHAR_SIZE, y*CHAR_SIZE, CHAR_SIZE, CHAR_SIZE);

                    //if(chars[i][j].color != 0) {
                        this.ctx.shadowColor = `rgb(${chars[i][j].color}, ${chars[i][j].color}, ${chars[i][j].color})`;
                        this.ctx.shadowBlur = 8;
                        this.ctx.lineWidth = 4;
                    //} else {
                        //this.ctx.shadowBlur = 0;
                    //}
                    this.ctx.fillText(chars[i][j].char, i * CHAR_SIZE + this.x, j * CHAR_SIZE);

                    // if(chars[i][j].color == 0)
                    // chars[i][j].alpha = 0;
                    // else
                    // chars[i][j].alpha = 1;

                    chars[i][j].clearAlpha();
                }
            }
        },
        workLine(line: ILine, index: number) {
            if (line.yPixels > this.canvas.height + line.lengthChar * CHAR_SIZE) return this.deleteLine(index);

            if(chars[line.xChar])
            chars[line.xChar].map((char, index) => this.workCharacter(char, line.xChar, index, line.yPixels, line.lengthChar));

            line.yPixels += line.speed;
        },
        clear() {
            this.ctx.clearRect(0, 0, this.canvas.width, this.canvas.height);
        },
        workCharacter(char: IChar, x: number, y: number, yLine: number, lengthLine: number) {
            const yChar = y * CHAR_SIZE;

            const distToHeadLine = yLine - yChar;
            const lengthLineInPixels = lengthLine * CHAR_SIZE;

            if (distToHeadLine < 0) return;
            if (distToHeadLine > lengthLineInPixels) return;
            if (yChar > this.canvas.height) return;

            //chars[x][y].alpha = 1 - distToHeadLine / lengthLineInPixels;
            chars[x][y].addAlpha(1 - distToHeadLine / lengthLineInPixels);
        },
        deleteLine(index: number) {
            lines.splice(index, 1);
        },

        workImage() {
            const image = new Image()
            image.src = img;

            image.onload = () => {
                const charsInWidth = Math.ceil(this.canvas.width / CHAR_SIZE);
                const charsInHeight = Math.ceil(this.canvas.height / CHAR_SIZE);

                const canvas = document.createElement('canvas');
                const ctx = canvas.getContext('2d') as CanvasRenderingContext2D;

                canvas.width = Math.ceil(charsInWidth);
                canvas.height = Math.ceil(charsInHeight);

                if(image.width/image.height > canvas.width/canvas.height) {
                    const heightImageInPixels = (canvas.width/image.width) * image.height;

                    const distHeigthImageInPixels = canvas.height - heightImageInPixels;

                    ctx?.drawImage(
                        image, 
                        0, 
                        0, 
                        image.width,
                        image.height,
                        0, 
                        distHeigthImageInPixels/2, 
                        canvas.width, 
                        heightImageInPixels
                    );
                } else {

                    const widthImageInPixels = (canvas.height/image.height) * image.width;

                    const distWidthImageInPixels = canvas.width - widthImageInPixels;

                    ctx?.drawImage(
                        image, 
                        0, 
                        0, 
                        image.width, 
                        image.height, 
                        distWidthImageInPixels/2, 
                        0, 
                        widthImageInPixels, 
                        canvas.height
                    );
                }

                const pixels = ctx?.getImageData(0, 0, canvas.width, canvas.height).data as Uint8ClampedArray;

                for (let i = 0; i < canvas.width; i++) {
                    for (let j = 0; j < canvas.height; j++) {
                        const red = pixels[(j * canvas.width + i) * 4];
                        const green = pixels[(j * canvas.width + i) * 4 + 1];
                        const blue = pixels[(j * canvas.width + i) * 4 + 2];
                        const black = pixels[(j * canvas.width + i) * 4 + 3];

                        if(red == green && red == blue)

                        chars[i][j].color = red;
                    }
                }
            }
        }
    }
}
</script>

<template>
    <!-- {{fps}} -->
    <canvas class="matrix" ref="canvas"></canvas>
    <canvas ref="second"></canvas>
</template>

<style>
.matrix {
    position: absolute;
    top: 0px;
    left: 0px;
    z-index: 9999;

    width: 100vw;
    height: 100vh;


}
</style>
