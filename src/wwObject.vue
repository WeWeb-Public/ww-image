<template>
    <Hydrate ssr-only class="ww-image">
        <wwObjectImage class="ww-image" :ww-object-ctrl="wwObjectCtrl" :ww-attrs="wwAttrs"></wwObjectImage>
    </Hydrate>
</template>

<script>
import Hydrate from 'lazy-hydration';
import wwObjectImage from './wwObjectImage.vue';

export default {
    name: '__COMPONENT_NAME__',
    components: {
        Hydrate,
        wwObjectImage
    },
    props: {
        wwObjectCtrl: Object,
        wwAttrs: {
            type: Object,
            default: {}
        }
    },
    /* wwManager:start */
    computed: {
        wwObject() {
            return this.wwObjectCtrl.get();
        }
    },
    methods: {
        async changeImage() {
            wwLib.wwObjectHover.setLock(this);

            wwLib.wwPopups.addStory('WWIMAGE_SELECT', {
                title: {
                    en: 'Select image',
                    fr: 'Choisir une image'
                },
                type: 'wwPopupSelectImage'
            });

            let options = {
                firstPage: 'WWIMAGE_SELECT'
            };

            try {
                const result = await wwLib.wwPopups.open(options);
                this.wwObject.content.data.url = result.image.replace('.dev', '.space');

                this.wwObject.content.data.zoom = 1;
                this.wwObject.content.data.position = { x: 0, y: 0 };

                this.wwObjectCtrl.update(this.wwObject);
            } catch (error) {}

            wwLib.wwObjectHover.removeLock();
        },
        async edit() {
            wwLib.wwObjectHover.setLock(this);

            let editOptionsList = {
                EDIT_IMAGE_SELECT: {
                    separator: {
                        en: 'Image',
                        fr: 'Image'
                    },
                    title: {
                        en: 'Select image',
                        fr: 'Sélectionner une image'
                    },
                    desc: {
                        en: 'Accepted formats (Max 5Mb) : .png, .jpg, .gif',
                        fr: 'Formats acceptés (Max 5Mb) : .png, .jpg, .gif'
                    },
                    icon: 'wwi wwi-image',
                    shortcut: 'i',
                    next: 'WWIMAGE_SELECT'
                }
            };

            if (this.wwAttrs.wwCategory == 'background') {
                editOptionsList.WWIMAGE_FOCUSPOINT = {
                    title: {
                        en: 'Background focus point',
                        fr: 'Point focal pour font'
                    },
                    desc: {
                        en: 'Select a focus point that should always be visible.',
                        fr: 'Selectionner un point focal qui doit toujours être visible.'
                    },
                    icon: 'wwi wwi-show',
                    shortcut: 'f',
                    next: 'WWIMAGE_FOCUSPOINT'
                };
            }

            editOptionsList.EDIT_IMAGE_ALT = {
                title: {
                    en: "'Alt' text",
                    fr: "Texte 'Alt'"
                },
                desc: {
                    en: 'Description to help blind people and Google SOE',
                    fr: 'Description pour aider les personnes mal voyantes et le référencement de Google'
                },
                icon: 'wwi wwi-text',
                shortcut: 'a',
                next: 'WWIMAGE_ALT'
            };
            editOptionsList.EDIT_IMAGE_STYLE = {
                separator: {
                    en: 'Style',
                    fr: 'Style'
                },
                title: {
                    en: 'Change image style',
                    fr: "Changer l'apparence de l'image"
                },
                desc: {
                    en: 'Borders, shadow, ...',
                    fr: 'Bordures, ombres, ...'
                },
                icon: 'wwi wwi-edit-style',
                shortcut: 's',
                next: 'WWIMAGE_STYLE'
            };
            editOptionsList.EDIT_IMAGE_RATIO = {
                title: {
                    en: 'Change image ratio',
                    fr: "Changer le ratio de l'image"
                },
                desc: {
                    en: 'Portrait, square, landscape, ...',
                    fr: 'Portrait, carré, paysage, ...'
                },
                icon: 'wwi wwi-ratio',
                shortcut: 'r',
                next: 'WWIMAGE_RATIO'
            };
            editOptionsList.EDIT_IMAGE_MINWIDTH = {
                title: {
                    en: 'Set image minimum width',
                    fr: "Changer la largeur minimale de l'image"
                },
                desc: {
                    en: 'In pixels',
                    fr: 'En pixels'
                },
                icon: 'fas fa-arrows-alt-h',
                shortcut: 'm',
                next: 'WWIMAGE_MINWIDTH'
            };
            editOptionsList.EDIT_IMAGE_LINK = {
                separator: {
                    en: 'Link',
                    fr: 'Lien'
                },
                title: {
                    en: 'Change image link',
                    fr: "Changer le lien de l'image"
                },
                desc: {
                    en: 'External, internal, ...',
                    fr: 'Externe, interne, ...'
                },
                icon: 'wwi wwi-link-external',
                shortcut: 's',
                next: 'WWIMAGE_LINKS'
            };

            wwLib.wwPopups.addStory('WWIMAGE_EDIT', {
                title: {
                    en: 'Edit Image',
                    fr: "Editer l'image"
                },
                type: 'wwPopupEditWwObject',
                buttons: null,
                storyData: {
                    list: editOptionsList
                }
            });
            wwLib.wwPopups.addStory('WWIMAGE_SELECT', {
                title: {
                    en: 'Select image',
                    fr: 'Choisir une image'
                },
                type: 'wwPopupSelectImage',
                next: 'WWIMAGE_RATIO'
            });
            wwLib.wwPopups.addStory('WWIMAGE_RATIO', {
                title: {
                    en: 'Image Ratio',
                    fr: "Ratio de l'image"
                },
                type: 'wwPopupWwObjectRatio',
                buttons: {
                    NEXT: {
                        text: {
                            en: 'Next',
                            fr: 'Suivant'
                        },
                        next: 'WWIMAGE_STYLE'
                    }
                }
            });
            wwLib.wwPopups.addStory('WWIMAGE_STYLE', {
                title: {
                    en: 'Image style',
                    fr: "Style de l'image"
                },
                type: 'wwPopupWwObjectStyle',
                buttons: {
                    NEXT: {
                        text: {
                            en: 'Next',
                            fr: 'Suivant'
                        },
                        next: 'WWIMAGE_ALT'
                    }
                }
            });
            wwLib.wwPopups.addStory('WWIMAGE_LINKS', {
                title: {
                    en: 'Link',
                    fr: 'Lien'
                },
                type: 'wwPopupLinks',
                storyData: {
                    links: ['EXTERNAL', 'INTERNAL', 'SECTION', 'POPUP', 'DOWNLOAD', 'ZOOM', 'TOGGLE_NAVBAR', 'NO_LINK']
                }
            });
            wwLib.wwPopups.addStory('WWIMAGE_ALT', {
                title: {
                    en: "'Alt' text",
                    fr: "Text 'Alt'"
                },
                type: 'wwPopupForm',
                storyData: {
                    fields: [
                        {
                            label: {
                                en: "'Alt' text :",
                                fr: "Texte 'Alt' :"
                            },
                            type: 'text',
                            key: 'altText',
                            valueData: 'wwObject.content.data.alt',
                            desc: {
                                en: 'Short description of the image to help blind people and Google for SOE',
                                fr: "Déscription courte de l'image pour aider les personnes mal voyantes et Google (référencement)"
                            }
                        }
                    ]
                },
                buttons: {
                    OK: {
                        text: {
                            en: 'Ok',
                            fr: 'Valider'
                        },
                        next: false
                    }
                }
            });
            wwLib.wwPopups.addStory('WWIMAGE_MINWIDTH', {
                title: {
                    en: 'Minimum width',
                    fr: 'Largeur minimale'
                },
                type: 'wwPopupForm',
                storyData: {
                    fields: [
                        {
                            label: {
                                en: 'Min width (px) :',
                                fr: 'Largeur minimale (px) :'
                            },
                            type: 'text',
                            key: 'minWidth',
                            valueData: 'wwObject.content.data.style.minWidth'
                        }
                    ]
                },
                buttons: {
                    OK: {
                        text: {
                            en: 'Ok',
                            fr: 'Valider'
                        },
                        next: false
                    }
                }
            });
            wwLib.wwPopups.addStory('WWIMAGE_FOCUSPOINT', {
                title: {
                    en: 'Background focus point',
                    fr: 'Point focal pour le font'
                },
                type: 'wwImagePopupFocusPoint',
                storyData: {},
                buttons: {
                    OK: {
                        text: {
                            en: 'Ok',
                            fr: 'Valider'
                        },
                        next: false
                    }
                }
            });

            let options = {
                firstPage: 'WWIMAGE_EDIT',
                data: {
                    wwObject: this.wwObject
                }
            };

            try {
                const result = await wwLib.wwPopups.open(options);

                /*=============================================m_ÔÔ_m=============================================\
                  IMAGE
                \================================================================================================*/
                if (typeof result.image != 'undefined') {
                    this.wwObject.content.data.url = result.image;
                }
                if (typeof result.altText != 'undefined') {
                    this.wwObject.content.data.alt = result.altText;
                }
                if (typeof result.focusPoint != 'undefined') {
                    this.wwObject.content.data.focusPoint = result.focusPoint;
                }

                /*=============================================m_ÔÔ_m=============================================\
                  STYLE
                \================================================================================================*/
                if (typeof result.borderColor != 'undefined') {
                    this.wwObject.content.data.style.borderColor = result.borderColor;
                }
                if (typeof result.borderRadius != 'undefined') {
                    this.wwObject.content.data.style.borderRadius = result.borderRadius;
                }
                if (typeof result.borderRadiusUnit != 'undefined') {
                    this.wwObject.content.data.style.borderRadiusUnit = result.borderRadiusUnit;
                }
                if (typeof result.borderStyle != 'undefined') {
                    this.wwObject.content.data.style.borderStyle = result.borderStyle;
                }
                if (typeof result.borderWidth != 'undefined') {
                    this.wwObject.content.data.style.borderWidth = result.borderWidth;
                }
                if (typeof result.boxShadow != 'undefined') {
                    this.wwObject.content.data.style.boxShadow = result.boxShadow;
                }
                if (typeof result.filter != 'undefined') {
                    this.wwObject.content.data.style.filter = result.filter;
                }
                if (typeof result.overlay != 'undefined') {
                    this.wwObject.content.data.style.overlay = result.overlay;
                }
                if (typeof result.ratio != 'undefined') {
                    this.wwObject.ratio = result.ratio;
                }
                if (typeof result.maxHeight != 'undefined') {
                    this.wwObject.content.data.style.maxHeight = result.maxHeight;
                }
                if (typeof result.minWidth != 'undefined') {
                    this.wwObject.content.data.style.minWidth = result.minWidth;
                }

                this.$nextTick(() => {
                    this.wwObjectCtrl.update(this.wwObject);

                    this.wwObjectCtrl.globalEdit(result);
                });
            } catch (error) {
                console.log(error);
            }

            wwLib.wwObjectHover.removeLock();
        },
        async connectCms(){
            this.wwObjectCtrl.editCmsBindings();
        }
    }
    /* wwManager:end */
};
</script>
