<template>
    <div class="ww-popup-focuspoint">
        <div class="content">
            <div class="description">{{wwManagerLang.getText(description)}}</div>
            <div class="image-container" @click="setFocusPoint($event)">
                <img class="image" :src="wwObject.content.data.url">
                <div class="cursor" :style="cursorPosition">+</div>
            </div>
        </div>
    </div>
</template>

<script> 
export default {
    name: "wwImagePopupFocusPoint",
    props: {
        options: {
            type: Object,
            default: function () {
                return {}
            }
        },
    },
    data() {
        return {
            wwObject: this.options.data.wwObject,
            wwManagerLang: wwLib.wwManagerLang,
            description: {
                en: 'Select a focus point that should always be visible',
                fr: 'Selectionner un point focal qui devrait toujours être visible'
            },
            selectedPoint: {
                x: 50,
                y: 50
            }
        };
    },
    computed: {
        cursorPosition() {
            return {
                top: this.selectedPoint.y + '%',
                left: this.selectedPoint.x + '%'
            };
        }
    },
    watch: {
    },
    methods: {
        init() {
            this.options.result.focusPoint = this.options.data.wwObject.content.data.focusPoint || [50, 50];
            this.selectedPoint = {
                x: this.options.result.focusPoint[0],
                y: this.options.result.focusPoint[1]
            }
        },
        setFocusPoint(event) {
            let image = this.$el.querySelector('.image').getBoundingClientRect();
            let position = this.getEventPosition(event);
            let relativePosition = { x: position.x - image.left, y: position.y - image.top };

            this.selectedPoint = {
                x: relativePosition.x * 100 / image.width,
                y: relativePosition.y * 100 / image.height
            }

            this.options.result.focusPoint = [this.selectedPoint.x, this.selectedPoint.y];
        },

        isTouch(event) {
            return event.touches && event.touches.length;
        },
        getEventPosition(event) {
            var position = { x: 0, y: 0 };

            if (this.isTouch(event)) {

                position.x = 0;
                position.y = 0;

                for (let touch of event.touches) {
                    position.x += touch.clientX
                    position.y += touch.clientY
                }

                position.x = position.x / event.touches.length;
                position.y = position.y / event.touches.length;

            } else {
                position.x = event.clientX;
                position.y = event.clientY;
            }

            return position;
        }
    },
    mounted() {
        this.init()
    },
    created() {
    },
    beforeDestroy() {
    }
};
</script>

<style scoped lang="scss">
.ww-popup-focuspoint {
    display: -webkit-box;
    display: -ms-flexbox;
    display: flex;
    -webkit-box-pack: center;
    -ms-flex-pack: center;
    justify-content: center;
    .content {
        width: 100%;
        padding-top: 20px;

        @media (min-width: 768px) {
            width: 90%;
        }
        @media (min-width: 992px) {
            width: 60%;
        }
        @media (min-width: 1200px) {
            width: 600px;
        }

        .description {
            padding-bottom: 20px;
        }

        .image-container {
            width: 100%;
            position: relative;

            .image {
                width: 100%;
                cursor: pointer;
            }

            .cursor {
                position: absolute;
                color: white;
                top: 50%;
                left: 50%;
                -webkit-transform: translate(-50%, -50%);
                transform: translate(-50%, -50%);
                cursor: pointer;
                font-size: 30px;
                text-shadow: 0 0 5px black;
            }
        }
    }
}
</style>
