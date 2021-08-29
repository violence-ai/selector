<template>
    <div class="selector" :class="css" @mouseover="mouseOver = true" @mouseout="mouseOver = false">
        <!-- Current value -->
        <div class="value" @click="clickValue()">
            <!-- Template Current value -->
            <slot name="value" :data="currentOption"></slot>
        </div>
        <!-- Drop menu -->
        <div class="drop-menu" v-show="dropMenuShow">
            <!-- Scrollbar for Options -->
            <vue-custom-scrollbar :settings="scrollSettings">
                <!-- Options -->
                <div class="option"
                     :class="{ active: data.id === currentId }"
                     v-for="data in options"
                     v-if="showSelectedOption || (!showSelectedOption && data.id !== currentId)"
                     @click="changeValue(data.id)">
                    <!-- Template Option inner -->
                    <slot name="option" :data="data"></slot>
                </div>
            </vue-custom-scrollbar>
        </div>
    </div>
</template>

<script lang="ts">
import {PropType} from "vue"
import Component from "vue-class-component"
import {Prop, Vue} from "vue-property-decorator"
import { Option } from './index'
// @ts-ignore
import vueCustomScrollbar from 'vue-custom-scrollbar'

@Component({
    components: { vueCustomScrollbar }
})
export default class Selector extends Vue {

    @Prop(String) readonly css!: string
    @Prop({type: Array as PropType<Option[]>}) readonly options!: Option[]
    @Prop(Number) currentId!: number
    @Prop(Boolean) showSelectedOption!: boolean

    private dropMenuShow: boolean = false
    private mouseOver: boolean = false

    private scrollSettings = {
        suppressScrollX : true,
        wheelPropagation : false
    }

    get currentOption(): Option {
        let findOption = this.options.find(option => {
            return option.id === this.currentId
        })
        if ( findOption ) {
            return findOption
        }
        return {
            id: null,
            value: 'Select an option'
        }
    }

    clickValue() {
        this.dropMenuShow = !this.dropMenuShow
    }

    changeValue(newId: number) {
        this.$emit('update:currentId', newId)
        this.dropMenuShow = false
    }

    clickDocument() {
        if ( this.mouseOver ) return
        this.dropMenuShow = false
    }

    mounted() {
        document.addEventListener('click', this.clickDocument)
    }

    destroyed() {
        document.removeEventListener('click', this.clickDocument)
    }
}
</script>

<style scoped>

</style>
