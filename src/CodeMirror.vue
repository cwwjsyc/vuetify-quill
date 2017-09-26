<template>
    <div class="codemirror-container">
        <v-card flat class="grey lighten-4">
            <textarea class="codemirror" ref="codemirror"></textarea>
        </v-card>
    </div>
</template>

<script>
    import CodeMirror from 'codemirror'
    import emmet from '@emmetio/codemirror-plugin'
    import 'codemirror/mode/htmlmixed/htmlmixed'

    export default {
        name: 'v-codemirror',
        model: {
            prop: 'content'
        },
        props: {
            content: null,
            options: { type: Object, default: () => { return {} } },
            theme: { type: String, default: "monokai" }
        },

        data () {
            return {
                codemirror: {
                    content: null,
                    instance: null,
                    model: false,
                    options: {
                        lineNumbers: true,
                        lineWrapping: true,
                        matchBrackets: true,
                        styleActiveLine: true,
                        mode: 'text/html',
                        extraKeys: {
                            'Tab': 'emmetExpandAbbreviation',
                            'Enter': 'emmetInsertLineBreak'
                        }

                    }
                }
            }
        },

        methods: {
            init () {
                this.codemirror.options = Object.assign(this.codemirror.options, this.options, {theme: this.theme})

                this.codemirror.instance = CodeMirror.fromTextArea(this.$refs.codemirror, this.codemirror.options)
            },

            emmetify () {
                emmet(CodeMirror)
            }
        },

        mounted () {
            this.emmetify()
            this.init()
        },

        watch: {
            'content': function (value) {
                this.codemirror.content = value
            },
            'codemirror.content': function (value) {
                this.codemirror.instance.setValue(value)
            }
        }
    }
</script>

<style lang="scss">
    @import '~codemirror/lib/codemirror.css';
    @import '~codemirror/theme/monokai.css';

    .codemirror-container, .codemirror-container .codemirror textarea {
        font-family: 'Ubuntu Mono', monospace;
    }
</style>