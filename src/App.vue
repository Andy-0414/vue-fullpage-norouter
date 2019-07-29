<template>
    <div id="app">
        <div id="virtual-scroll">
            <Main></Main>

            <div class="column">
                <Next></Next>
                <Main></Main>
            </div>
        </div>
    </div>
</template>

<script>
import Main from "@/components/Main";
import Next from "@/components/Next";
export default {
    components: {
        Main,
        Next
    },
    data() {
        return {
            app: null,
            virtualScroll: null,

            clear: true,

            currentX: 0,
            currentY: 0,

            routerData: [[{}], [{}, {}]]
        };
    },
    mounted() {
        this.app = document.getElementById("app");
        this.virtualScroll = document.getElementById("virtual-scroll");

        this.app.addEventListener("wheel", e => {
            if (e.deltaY && this.clear) {
                this.clear = false;
                setTimeout(() => {
                    this.clear = true;
                }, 800);
                this.currentY += e.deltaY > 0 ? 1 : -1;
            } else if (e.deltaX && this.clear) {
                this.clear = false;
                setTimeout(() => {
                    this.clear = true;
                }, 800);
                this.currentX += e.deltaX > 0 ? 1 : -1;
            }
            this.setPosition();
        });
        var mouseDownStart = false;
        var startPointX = 0;
        var startPointY = 0;
        var endPointX = 0;
        var endPointY = 0;
        this.app.addEventListener("mousedown", e => {
            mouseDownStart = true;
            startPointX = e.clientX;
            startPointY = e.clientY;
        });
        this.app.addEventListener("mousemove", e => {
            if (!e.which && mouseDownStart) {
                mouseDownStart = false;
                this.setPosition();
            }
            if (mouseDownStart) {
                this.setPosition(
                    e.clientX - startPointX,
                    e.clientY - startPointY
                );
            }
        });
        this.app.addEventListener("mouseup", e => {
            var gapX = e.clientX - startPointX;
            var gapY = e.clientY - startPointY;
            this.calcGap(gapX, gapY);

            this.setPosition();
            mouseDownStart = false;
        });

        this.app.addEventListener("touchstart", e => {
            mouseDownStart = true;
            startPointX = e.touches[0].clientX;
            startPointY = e.touches[0].clientY;
        });
        this.app.addEventListener("touchmove", e => {
            if (mouseDownStart) {
                endPointX = e.touches[0].clientX;
                endPointY = e.touches[0].clientY;
                this.setPosition(
                    endPointX - startPointX,
                    endPointY - startPointY
                );
            }
        });
        this.app.addEventListener("touchend", e => {
            var gapX = endPointX - startPointX;
            var gapY = endPointY - startPointY;
            this.calcGap(gapX, gapY);

            this.setPosition();
            mouseDownStart = false;
        });
    },
    methods: {
        calcGap(gapX, gapY) {
            if (gapY < -100 && this.currentY < this.getCurrentLineY) {
                this.currentY++;
            } else if (gapY > 100 && this.getCurrentLineY) {
                this.currentY--;
            } else if (gapX < -100 && this.currentX < this.getCurrentLineX) {
                this.currentX++;
            } else if (gapX > 100 && this.currentX > 0) {
                this.currentX--;
            }
        },
        setPosition(addX = 0, addY = 0) {
            this.virtualScroll.style.top = `${-this.currentY * innerHeight +
                addY}px`;
            this.virtualScroll.style.left = `${-this.currentX * innerWidth +
                addX * this.getCurrentLineX}px`;
        }
    },
    computed: {
        getCurrentLineX() {
            return this.routerData[this.currentY].length - 1;
        },
        getCurrentLineY() {
            return (
                this.routerData.filter(x => {
                    return x.length > this.currentX;
                }).length - 1
            );
        }
    }
};
</script>

<style>
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
    user-select: none;
}

::-webkit-scrollbar {
    display: none;
}

#app {
    position: relative;

    width: 100vw;
    height: 100vh;

    overflow: hidden;
}
#virtual-scroll {
    position: absolute;

    top: 0;

    transition: 1s cubic-bezier(0.175, 0.885, 0.32, 1);
}
.column {
    display: flex;
}
</style>
