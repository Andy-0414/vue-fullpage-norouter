<template>
    <div id="app">
        <div id="virtual-scroll">
            <Main></Main>
            <Next></Next>
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
            currentY: 0
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
            if (!e.which) {
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
            if (gapX < -100) {
                this.currentX++;
            }
            if (gapX > 100) {
                this.currentX--;
            }
            var gapY = e.clientY - startPointY;
            if (gapY < -100) {
                this.currentY++;
            }
            if (gapY > 100) {
                this.currentY--;
            }
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
            if (gapX < -100) {
                this.currentX++;
            }
            if (gapX > 100) {
                this.currentX--;
            }
            var gapY = endPointY - startPointY;
            if (gapY < -100) {
                this.currentY++;
            }
            if (gapY > 100) {
                this.currentY--;
            }
            this.setPosition();
            mouseDownStart = false;
        });
    },
    methods: {
        setPosition(addX = 0, addY = 0) {
            this.virtualScroll.style.top = `${-this.currentY * innerHeight +
                addY}px`;
            this.virtualScroll.style.left = `${-this.currentX * innerWidth +
                addX}px`;
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
</style>
