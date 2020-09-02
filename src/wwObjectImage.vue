<template>
    <div class="ww-image" :class="{ bg: wwAttrs.wwCategory == 'background' }" :style="c_styles.wrapper">
        <!-- Test: {{data.position}} -->
        <!-- wwManager:start -->
        <div class="controls-desktop" v-if="isControlsDisplayed">
            <div class="zoom-bar">
                <div class="zoom-line"></div>
                <div class="zoom-handle" :style="{ top: c_zoomPercentY + '%' }" @mousedown="startZoomDesktop($event)">
                    <div></div>
                </div>
            </div>
        </div>
        <div class="reset-zoom" @mousedown="resetZoom($event)" v-if="isControlsDisplayed">
            <i class="fa fa-expand" aria-hidden="true"></i>
        </div>
        <div class="format" :style="c_styles.format">
            <!-- Background -->
            <div v-if="wwAttrs.wwCategory == 'background'" class="image bg" :style="c_styles.image"></div>

            <!-- Normal Image -->
            <img v-if="wwAttrs.wwCategory != 'background'" draggable="false" class="image" :src="data.url" :alt="data.alt" :style="c_styles.image" />
        </div>
        <!-- wwManager:end -->

        <!-- wwFront:start -->
        <div class="format" :style="c_styles.format">
            <!-- Background -->
            <template v-if="wwAttrs.wwCategory == 'background'">
                <!-- No Twicpics -->
                <div v-if="wwAttrs.wwNoTwicPics" class="image bg" :style="c_styles.image"></div>

                <!-- Twicpics -->
                <div
                    v-else-if="!c_imgSrcSet"
                    class="image bg twic"
                    :data-background="'url(' + data.url + ')'"
                    data-background-step="400"
                    :data-background-focus="c_focusPoint"
                    data-background-transform="auto/quality=85"
                    :style="c_styles.image"
                ></div>

                <!-- SRCSET -->
                <div v-else class="image bg" :style="c_styles.image" v-style="c_imgSrcSet"></div>
            </template>

            <!-- Normal Image -->
            <template v-else>
                <!-- No Twicpics -->
                <img v-if="wwAttrs.wwNoTwicPics" class="image" :src="data.url" :alt="data.alt" :style="c_styles.image" :loading="d_isLazy ? 'lazy' : 'auto'" />

                <!-- Twicpics -->
                <img v-else-if="!c_imgSrcSet" class="image twic" :src="d_preview" :data-src="data.url" data-src-transform="quality=85/auto" data-src-step="10" :alt="data.alt" :style="c_styles.image" />

                <!-- SRCSET -->
                <img
                    class="image"
                    :style="c_styles.image"
                    :srcset="c_imgSrcSet"
                    :src="`https://weweb.twic.pics/${data.url}${data.url.indexOf('?') !== -1 ? '&' : '?'}twic=v1/quality=85/resize=1024`"
                    :alt="data.alt"
                    loading="lazy"
                />
            </template>
        </div>
        <!-- wwFront:end -->

        <div class="border" :style="c_styles.border"></div>
    </div>
</template>

<script>
/* wwManager:start */
import wwImagePopupFocusPoint from './wwImagePopupFocusPoint.vue';
wwLib.wwPopups.addPopup('wwImagePopupFocusPoint', wwImagePopupFocusPoint);
/* wwManager:end */

export default {
    name: '__COMPONENT_NAME__',
    props: {
        id: Number,
        data: Object,
        wwAttrs: {
            type: Object,
            default: {}
        },
        /* wwManager: start */
        isSelected: Boolean,
        is3d: Boolean,
        /* wwManager: end */
    },
    data() {
        return {
            d_zoomMin: 0.2,
            d_zoomFactor: 1,
            d_el: null,
            d_preview: 'data:image/gif;base64,R0lGODlhAQABAAAAACH5BAEKAAEALAAAAAABAAEAAAICTAEAOw==',
            d_imageSizes: {
                lg: 0,
                md: 0,
                sm: 0,
                xs: 0
            },

            /* wwManager:start */
            d_lastMovePosition: { x: 0, y: 0 },
            d_moving: false,
            d_lastTouchDist: 0,
            d_zoomBarElement: null,
            d_moveDirection: null,
            d_imgSize: {}
            /* wwManager:end */
        };
    },
    computed: {
        c_styles() {
            if (!this.d_el) {
                return {};
            }

            const styles = {
                image: {
                    backgroundImage: '',
                    width: '100%',
                    height: '100%',
                    minHeight: 'none',
                    top: '50%',
                    left: '50%',
                    filter: 'none'
                },
                format: {
                    paddingBottom: '',
                    borderRadius: 0,
                    boxShadow: '',
                    filter: '',
                    minWidth: '40'
                },
                border: {
                    borderRadius: 0,
                    borderWidth: 0,
                    borderColor: null,
                    borderStyle: null,
                    background: null
                },
                wrapper: {
                    maxWidth: null
                }
            };

            const styleData = this.data.style || {};

            //IMAGE
            styles.image.filter = styleData.filter || null;

            if (this.wwAttrs.wwCategory == 'background' && !this.c_imgSrcSet) {
                styles.image.backgroundImage = `url(${this.wwAttrs.wwNoTwicPics ? styleData.url : this.d_preview})`;
            }

            /* wwManager:start */
            this.wwAttrs.wwCategory == 'background' && (styles.image.backgroundImage = 'url(' + styleData.url + ')');
            /* wwManager:end */
            styles.image.height = this.wwAttrs.wwCategory == 'background' ? '100%' : 'auto';
            styles.image.width = (styleData.zoom > 0 ? styleData.zoom : 1) * 100 + '%';
            if (this.wwAttrs.wwCategory != 'background') {
                let position = this.data.position || { x: 0, y: 0 };
                styles.image.left = '50%';
                styles.image.top = '50%';
                styles.image['-webkit-transform'] = 'translate(' + (position.x - 50) + '%, ' + (position.y - 50) + '%)';
                styles.image['-moz-transform'] = 'translate(' + (position.x - 50) + '%, ' + (position.y - 50) + '%)';
                styles.image['-ms-transform'] = 'translate(' + (position.x - 50) + '%, ' + (position.y - 50) + '%)';
                styles.image['-o-transform'] = 'translate(' + (position.x - 50) + '%, ' + (position.y - 50) + '%)';
                styles.image.transform = 'translate(' + (position.x - 50) + '%, ' + (position.y - 50) + '%)';
            }

            //FORMAT
            styles.format.boxShadow = this.getShadow();
            styles.format.minWidth = styleData.minWidth ? styleData.minWidth + 'px' : '20px';

            if (this.wwAttrs.wwCategory != 'background') {
                styles.format.paddingBottom = this.getRatio() + '%';
            } else {
                styles.format.paddingBottom = 0;
            }

            if (styleData.maxHeight) {
                styles.wrapper.maxWidth = (parseInt(styleData.maxHeight) / this.getRatio()) * 100 + 'px';
            } else {
                styles.wrapper.maxWidth = null;
            }

            //BORDER
            const unit = styleData.borderRadiusUnit || '%';
            const borderRadius = (styleData.borderRadius / (unit == '%' ? 2 : 1) || 0) + unit;
            styles.border.borderRadius = borderRadius;
            styles.format.borderRadius = borderRadius;

            styles.border.borderWidth = (styleData.borderWidth || 0) + 'px';

            styles.border.borderColor = styleData.borderColor || 'black';
            styles.border.borderStyle = styleData.borderStyle || 'none';
            styles.border.background = styleData.overlay || '';

            return styles;
        },
        c_focusPoint() {
            let focusPoint = this.data.focusPoint || [50, 50];
            return focusPoint[0] + 'px' + focusPoint[1] + 'p';
        },
        c_editing() {
            return this.isSelected && !this.is3d;
        },
        c_imgSrcSet() {
            /* wwFront:start */
            if (this.data.url.indexOf('.gif') !== -1) {
                return null;
            }

            const lg = this.d_imageSizes.lg ? this.d_imageSizes.lg[0] : 0;
            const md = this.d_imageSizes.md ? this.d_imageSizes.md[0] : 0;
            const sm = this.d_imageSizes.sm ? this.d_imageSizes.sm[0] : 0;
            const xs = this.d_imageSizes.xs ? this.d_imageSizes.xs[0] : 0;

            const sum = lg + md + sm + xs;
            if (!sum) {
                return null;
            }

            const screenBreakpoints = {
                xs: 768,
                sm: 992,
                md: 1200,
                lg: 1920
            };

            if (this.wwAttrs.wwCategory !== 'background') {
                const sources = [];

                for (const screen in screenBreakpoints) {
                    if (this.d_imageSizes[screen]) {
                        sources.push(
                            `https://weweb.twic.pics/${this.data.url}${this.data.url.indexOf('?') !== -1 ? '&' : '?'}twic=v1/quality=85/resize=${this.d_imageSizes[screen][0]} ${screenBreakpoints[screen]}w`
                        );
                    }
                }

                return sources.join(', ');
            } else {
                const sources = {};

                for (const screen in screenBreakpoints) {
                    if (this.d_imageSizes[screen]) {
                        if (!this.d_imageSizes[screen][0] || !this.d_imageSizes[screen][1]) {
                            return null;
                        }
                        sources[screen] = {
                            backgroundImage: `url(https://weweb.twic.pics/${this.data.url}${this.data.url.indexOf('?') !== -1 ? '&' : '?'}twic=v1/quality=85/focus=${this.c_focusPoint}/cover=${
                                this.d_imageSizes[screen][0]
                            }x${this.d_imageSizes[screen][1]}/resize=${this.d_imageSizes[screen][0]})`
                        };
                    }
                }
                return sources;
            }
            /* wwFront:end */
            return null;
        },
        c_screenSize() {
            return wwLib.$store.getters['front/getScreenSize'];
        },

        /* wwManager:start */
        c_zoomPercentY() {
            return 100 - this.d_zoomFactor * Math.sqrt(Math.max(this.data.zoom, 0) - this.d_zoomMin);
        },
        isControlsDisplayed() {
            return this.isSelected && this.wwAttrs.wwCategory !== 'background'
        }
        /* wwManager:end */
    },
    watch: {
        /* wwFront:start */
        c_screenSize() {
            if (window.__WW_IS_PRERENDER__ && this.$el) {
                const elemOk = (this.wwAttrs.wwCategory !== 'background' && this.$el.querySelector('.image')) || (this.wwAttrs.wwCategory === 'background' && this.$el.querySelector('.format'));

                if (!elemOk) return null;

                if (this.wwAttrs.wwCategory !== 'background') {
                    let width = this.$el.querySelector('.image').getBoundingClientRect().width;
                    width += width * 0.2;
                    this.d_imageSizes[this.c_screenSize] = [Math.floor(width), 0];
                } else {
                    this.d_imageSizes[this.c_screenSize] = [Math.floor(this.$el.querySelector('.format').getBoundingClientRect().width), Math.floor(this.$el.querySelector('.format').getBoundingClientRect().height)];
                }

                let imgSizeElm = document.getElementById(`ww-image-size`);
                if (!imgSizeElm) {
                    imgSizeElm = document.createElement('script');
                    imgSizeElm.setAttribute('id', `ww-image-size`);
                    document.head.append(imgSizeElm);
                }
                imgSizeElm.innerText += `window.wwg_imageSize_${this.c_screenSize}_${this.id} = ${JSON.stringify(this.d_imageSizes[this.c_screenSize])};`;
            }
        }
        /* wwFront:end */
    },
    methods: {
        init() {
            this.d_zoomFactor = Math.sqrt((100 * 100) / (10 - this.d_zoomMin));
        },
        preventEvent(event) {
            if (!this.isTouch(event)) {
                event.preventDefault();
                event.stopPropagation();
            }

            return false;
        },

        /*=============================================m_ÔÔ_m=============================================\
          GET PARAMS
        \================================================================================================*/
        getRatio() {
            //If ratio is fixed in ww-object directive, override it here
            if (this.wwAttrs.wwFixedRatio) {
                try {
                    var ratio = parseFloat(this.wwAttrs.wwFixedRatio);
                    if (ratio) {
                        return ratio;
                    }
                } catch (error) {
                    console.log('wwRatio error', error);
                }
            }

            // if (!this.wwObject.ratio || this.wwObject.ratio < 0) {
                if (this.wwAttrs.wwDefaultRatio) {
                    return this.wwAttrs.wwDefaultRatio;
                } else {
                    return (100 / 3) * 2;
                }
            // }

            // return this.wwObject.ratio;
        },
        getShadow() {
            const styleData = this.data.style ||{}
            const shadow = styleData.boxShadow || {};
            if (shadow.x || shadow.y || shadow.b || shadow.s || shadow.c) {
                return shadow.x + 'px ' + shadow.y + 'px ' + shadow.b + 'px ' + shadow.s + 'px ' + shadow.c;
            }
            return '';
        },

        /* wwManager:start */
        /*=============================================m_ÔÔ_m=============================================\
          IMAGE ZOOM
        \================================================================================================*/
        resetZoom(event) {
            this.preventEvent(event);

            //Reset zoom
            const rectImg = this.$el.querySelector('.image').getBoundingClientRect();
            const imgSize = {
                w: rectImg.width,
                h: rectImg.height
            };

            const ratio = imgSize.h / imgSize.w;
            let zoom;

            if (this.data.zoom !== 1) {
                zoom = 1;
            } else {
                const rectEl = this.$el.getBoundingClientRect();
                const ratioContainer = rectEl.height / rectEl.width;

                zoom = ratioContainer / ratio;
            }

            this.$emit('update', {zoom, position: { x: 0, y: 0 }})

            return false;
        },
        startZoomDesktop(event) {
            event.preventDefault();
            event.stopPropagation();

            wwLib.wwManagerUI.lockSelection();

            this.d_zoomBarElement = this.$el.querySelector('.zoom-bar');

            window.addEventListener('mousemove', this.zoomDesktop);
            window.addEventListener('mouseup', this.stopZoomDesktop);

            window.document.body.classList.add('ww-image-dragging');

            return false;
        },
        zoomDesktop(event) {
            let zoomPositionY = ((event.clientY - this.d_zoomBarElement.getBoundingClientRect().top) * 100) / this.d_zoomBarElement.getBoundingClientRect().height;
            zoomPositionY = Math.min(Math.max(zoomPositionY, 0), 100);

            const zoom = Math.pow((100 - zoomPositionY) / this.d_zoomFactor, 2) + this.d_zoomMin;

            this.preventEvent(event);

            this.$emit('update', {zoom})

            return false;
        },
        stopZoomDesktop(event) {
            wwLib.wwManagerUI.unlockSelection();

            window.removeEventListener('mousemove', this.zoomDesktop);
            window.removeEventListener('mouseup', this.stopZoomDesktop);

            window.document.body.classList.remove('ww-image-dragging');

            return false;
        },

        /*=============================================m_ÔÔ_m=============================================\
          IMAGE MOVE
        \================================================================================================*/
        getEventPosition(event) {
            var position = { x: 0, y: 0 };

            if (this.isTouch(event)) {
                position.x = 0;
                position.y = 0;

                for (let touch of event.touches) {
                    position.x += touch.clientX;
                    position.y += touch.clientY;
                }

                position.x = position.x / event.touches.length;
                position.y = position.y / event.touches.length;
            } else {
                position.x = event.clientX;
                position.y = event.clientY;
            }

            return position;
        },
        getTouchDist(event) {
            if (event.touches && event.touches.length >= 2) {
                var touch0 = {
                    x: event.touches[0].clientX,
                    y: event.touches[0].clientY
                };

                var touch1 = {
                    x: event.touches[1].clientX,
                    y: event.touches[1].clientY
                };

                var dist = Math.sqrt((touch0.x - touch1.x) * (touch0.x - touch1.x) + (touch0.y - touch1.y) * (touch0.y - touch1.y));
                return dist;
            }
            return 0;
        },
        isTouch(event) {
            return event.touches && event.touches.length;
        },
        startMove(event) {
            if (!this.isSelected || this.is3d || this.wwAttrs.wwCategory == 'background') {
                return;
            }

            if (event.ctrlKey || event.button == 2) {
                return;
            }

            this.d_moving = false;
            wwLib.wwManagerUI.lockSelection();

            this.d_lastMovePosition = this.getEventPosition(event);
            if (this.d_lastMovePosition) {
                wwLib.getFrontDocument().addEventListener('mousemove', this.move);
                wwLib.getManagerDocument().addEventListener('mousemove', this.move);
                wwLib.getFrontDocument().addEventListener('click', this.stopMove);
                wwLib.getManagerDocument().addEventListener('click', this.stopMove);

                wwLib.getFrontDocument().addEventListener('touchmove', this.move);
                wwLib.getManagerDocument().addEventListener('touchmove', this.move);
                wwLib.getFrontDocument().addEventListener('touchend', this.stopMove);
                wwLib.getManagerDocument().addEventListener('touchend', this.stopMove);

                document.body.classList.add('ww-image-dragging');
                return false;
            }
        },
        move(event) {
            let position = this.getEventPosition(event);

            if (!position) {
                return;
            }

            var offsetXpx = position.x - this.d_lastMovePosition.x;
            var offsetYpx = position.y - this.d_lastMovePosition.y;

            if (!this.d_moving && Math.abs(offsetXpx) + Math.abs(offsetYpx) < 4) {
                return;
            }

            this.d_moving = true;

            if (this.d_moveDirection == 'x') {
                offsetYpx = 0;
            } else if (this.d_moveDirection == 'y') {
                offsetXpx = 0;
            } else if (wwLib.wwShortcuts.hasModifiers('SHIFT') && Math.abs(offsetXpx) > Math.abs(offsetYpx)) {
                offsetYpx = 0;
                this.d_moveDirection = 'x';
            } else if (wwLib.wwShortcuts.hasModifiers('SHIFT')) {
                offsetXpx = 0;
                this.d_moveDirection = 'y';
            }

            const rectEl = this.$el.getBoundingClientRect();
            const rectImg = this.$el.querySelector('.image').getBoundingClientRect();
            var offsetXpercent = (offsetXpx * 100) / rectImg.width;
            var offsetYpercent = (offsetYpx * 100) / rectImg.height;

            const currentPosition = this.data.position
            const newPosition = {
                x: currentPosition.x + offsetXpercent,
                y: currentPosition.y + offsetYpercent
            }

            let newZoom;

            if (this.isTouch(event)) {
                const touchDist = this.getTouchDist(event);

                newZoom = this.data.zoom + ((touchDist - this.d_lastTouchDist) / 100) * (this.data.zoom === 0 ? 1 : this.data.zoom);

                if (newZoom < this.d_zoomMin) {
                    newZoom = this.d_zoomMin;
                }
                if (newZoom > 10) {
                    newZoom = 10;
                }

                this.d_lastTouchDist = touchDist;
            }

            this.d_lastMovePosition.x = position.x;
            this.d_lastMovePosition.y = position.y;

            this.preventEvent(event);
            const update = { position: newPosition }
            if (newZoom) { update.zoom = newZoom }
            this.$emit('update', update);
        },
        stopMove(event) {
            this.d_moving = false;

            wwLib.getFrontDocument().removeEventListener('mousemove', this.move);
            wwLib.getManagerDocument().removeEventListener('mousemove', this.move);
            wwLib.getFrontDocument().removeEventListener('click', this.stopMove);
            wwLib.getManagerDocument().removeEventListener('click', this.stopMove);

            wwLib.getFrontDocument().removeEventListener('touchmove', this.move);
            wwLib.getManagerDocument().removeEventListener('touchmove', this.move);
            wwLib.getFrontDocument().removeEventListener('touchend', this.stopMove);
            wwLib.getManagerDocument().removeEventListener('touchend', this.stopMove);

            this.d_moveDirection = null;
            window.document.body.classList.remove('ww-image-dragging');

            wwLib.wwManagerUI.unlockSelection();

            return false;
        }
        /* wwManager:end */
    },
    created() {
        /* wwManager:start */
        let formattedUrl = this.data.url;

        //Replace for prod
        formattedUrl = formattedUrl.replace('weweb.twic.pics', 'cdn.weweb.app');
        formattedUrl = formattedUrl.replace('wewebapp.s3.eu-west-3.amazonaws.com', 'cdn.weweb.app');

        //Replace for preprod
        formattedUrl = formattedUrl.replace('weweb.twic.pics/preprod', 'cdn.weweb.dev');
        formattedUrl = formattedUrl.replace('wewebapp-preprod.s3.eu-west-3.amazonaws.com', 'cdn.weweb.dev');

        //Replace for staging
        formattedUrl = formattedUrl.replace('weweb.twic.pics/staging', 'cdn.weweb.space');
        formattedUrl = formattedUrl.replace('wewebapp-dev.s3.eu-west-3.amazonaws.com', 'cdn.weweb.space');

        if (formattedUrl != this.data.url) {
            this.$emit('update', {url: formattedUrl})
        }
        /* wwManager:end */

        /* wwFront:start */
        const screens = ['lg', 'md', 'sm', 'xs'];
        for (const screen of screens) {
            if (window[`wwg_imageSize_${screen}_${this.id}`]) {
                this.d_imageSizes[screen] = window[`wwg_imageSize_${screen}_${this.id}`];
                this.d_imageSizes[screen][0] = this.d_imageSizes[screen][0] || 1024;
            }
        }
        /* wwFront:end */
        console.log(this.id, this.data)
    },
    mounted() {
        this.init();

        this.d_el = this.$el;

        /* wwManager:start */
        if (this.wwAttrs.wwCategory != 'background') {
            this.$el.addEventListener('touchstart', this.startMove);
            this.$el.addEventListener('mousedown', this.startMove);
        }
        /* wwManager:end */
    },
    beforeDestroy() {
        /* wwManager:start */
        if (this.wwAttrs.wwCategory != 'background') {
            this.$el.removeEventListener('mousedown', this.startMove);
            this.$el.removeEventListener('touchstart', this.startMove);
        }
        /* wwManager:end */
    }
};
</script>

<style scoped lang="scss">
.ww-image {
    position: relative;
    user-select: none;
    display: flex;
    justify-content: center;

    .border {
        position: absolute;
        top: -1px;
        left: -1px;
        right: -1px;
        bottom: -1px;
        pointer-events: none;
    }

    .format {
        width: 100%;
        height: 100%;
        position: relative;
        overflow: hidden;
        min-width: 40px;

        .image {
            position: absolute;
            width: 100%;
            height: 100%;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);

            img {
                width: 100%;
                height: 100%;
            }

            &.bg {
                top: auto;
                left: auto;
                transform: 'none';
                background-repeat: no-repeat;
                background-position: center;
                background-size: cover;
            }

            // &.twic {
            //     filter: blur(5px);
            //     will-change: filter;

            //     &.twic-done,
            //     &.twic-background-done {
            //         transition: filter 0.3s ease;
            //         filter: none;
            //     }
            // }
        }
    }
}

/* wwManager:start */

.ww-edit-mode-content {
    .ww-image:not(.bg) {
        cursor: move;
        cursor: grab;
    }

    .controls-desktop {
        position: absolute;
        right: 4px;
        min-height: 20px;
        bottom: 5px;
        top: 5px;
        width: 20px;
        z-index: 5;
        display: block;
        transition: opacity 0.15s ease;

        .zoom-bar {
            position: relative;
            width: 100%;
            height: 100%;

            .zoom-line {
                position: absolute;
                left: 50%;
                top: 0;
                bottom: 3px;
                background-color: white;
                width: 3px;
                border-radius: 50px;
                transform: translateX(-50%);
                box-shadow: 0px 0px 2px 0px rgba(0, 0, 0, 0.5);
            }

            .zoom-handle {
                position: absolute;
                left: 50%;
                transform: translate(-50%, -50%);
                top: 50%;
                background-color: #e73055;
                width: 20px;
                height: 20px;
                border-radius: 100%;
                cursor: move;
                cursor: grab;
                box-shadow: 0px 2px 2px 0px rgba(0, 0, 0, 0.5);
            }
        }
    }

    .reset-zoom {
        position: absolute;
        z-index: 1;
        left: 5px;
        bottom: 5px;
        width: 20px;
        height: 20px;
        background-color: white;
        line-height: 25px;
        font-size: 15px;
        text-align: center;
        cursor: pointer;
        color: #757575;
        box-shadow: 0px 1px 1px 0px rgba(0, 0, 0, 0.5);
        display: block;
        transition: opacity 0.15s ease;
    }
}
/* wwManager:end */
</style>

<style>
.ww-image-dragging {
    cursor: move;
    cursor: grab;
    user-select: none;
}
</style>
