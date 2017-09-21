<template>
    <div class="quill-container" :class="paper ? 'ql-paper grey lighten-4' : ''">
        <v-card flat>
            <div ref="toolbar" v-if="toolbar">
                <slot name="toolbar"></slot>
            </div>

            <div ref="quill" class="quill-editor" :class="paper ? 'elevation-1' : ''"></div>
            <template v-if="statusBar">
                <v-card-actions class="grey--text">
                    <v-spacer></v-spacer>
                    <small>Words: <span v-html="quill.status.words"></span></small>
                </v-card-actions>
            </template>
            <v-codemirror v-model="quill.content.source"></v-codemirror>
        </v-card>
    </div>
</template>

<script>
    import Quill from 'quill'
    import CodeMirror from './CodeMirror.vue'
    import Countable from 'countable'

    export default {
        name: 'Quill',
        model: {
            prop: 'content',
        },
        components: { 'v-codemirror': CodeMirror },
        props: {
            content: {},
            toolbar: { type: Boolean, default: false },
            options: { type: Object, default: () => { return { theme: 'snow', placeholder: 'Write something', modules: {} } } },
            fonts: { type: Array, default: () => { return [] } },
            statusBar: { type: Boolean, default: true },
            paper: { type: Boolean, default: false }
        },
        data () {
            return {
                quill: {
                    model: false,
                    instance: null,
                    content: {},
                    status: {}
                }
            }
        },
        methods: {
            register () {
                let Font = Quill.import('formats/font')
                let Size = Quill.import('attributors/style/size')

                Font.whitelist = this.fonts

                Quill.register(Font, true)
                Quill.register(Size, true)

                if (this.toolbar) {
                    this.$props.options.modules.toolbar = this.$refs.toolbar
                }
            },

            prototypes () {
                Quill.prototype.getHTML = function() {
                    return this.container.querySelector('.ql-editor').innerHTML;
                }

                Quill.prototype.getWordCount = function() {
                    return this.container.querySelector('.ql-editor').innerText.length;
                }
            },

            init () {
                this.quill.instance = new Quill(this.$refs.quill, this.options)

                this.$emit('init', this.quill.instance);
            },

            listen () {
                let self = this

                self.quill.instance.on('text-change', function (delta, oldDelta, source) {
                    self.$emit('text-change', delta, oldDelta, source, self.quill.instance);
                    self.$emit('input', self.quillbox().getContents());
                })

                Countable.on(self.quillbox().getContainer(), counter => {
                    self.quill.status = counter
                }, { stripTags: false, ignore: [] })
            },

            quillbox () {
                let self = this

                return {
                    setContents (content) {
                        self.quill.instance.setContents(content)
                    },

                    getContents () {
                        return {
                            html: self.quill.instance.getHTML(),
                            source: self.quill.instance.getHTML(),
                            delta: self.quill.instance.getContents()
                        }
                    },

                    getContainer () {
                        return self.quill.instance.container.querySelector('.ql-editor')
                    }
                }
            }
        },
        mounted () {
            this.register()
            this.prototypes()
            this.init()
            this.quillbox().setContents(this.content.delta ? this.content.delta.ops : [])
            this.listen()
        },
        watch: {
            'content': function (value) {
                this.quill.content = value
            }
        }
    }
</script>

<style lang="styl">
    @import "~quill/assets/snow";

    .ql-snow.ql-toolbar, .ql-snow.ql-container {
        border: none;
    }
    .ql-container {
        min-height: 300px;
    }

    .ql-paper {
        .quill-editor {
            background-color: white;
            margin: 2rem 4rem 0;
        }
        .ql-editor {
            padding: 3rem 2rem 1rem;
        }
        .ql-editor.ql-blank::before {
            left: 30px;
        }
    }
</style>