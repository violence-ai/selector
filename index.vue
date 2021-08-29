<template>
    <div class="selector" :class="css" @mouseover="mouseOver = true" @mouseout="mouseOver = false">

        <!-- Текущие выбранные значения (когда включен мульти-выбор) -->
        <div class="multiple-values"
             v-show="multiple && activeOptionsAvailable"
             @click="clickValue()">
            <slot name="multiplevalues" :data="{ multipleValues, unselectId }"></slot>
        </div>

        <!-- Текущее значение (если выбрано) -->
        <div class="value"
             v-show="!multiple && activeOptionsAvailable && !showSearch"
             @click="clickValue()">
            <slot name="value" :data="currentValue"></slot>
        </div>

        <!-- Текст по умолчанию (если не выбран не один вариант) -->
        <div class="default-value"
             v-if="!activeOptionsAvailable && !showSearch"
             @click="clickValue()">
            <slot name="defaultvalue" :data="defaultText"></slot>
        </div>

        <!-- Поле ввода для поиска (появляется при клике) -->
        <div class="search-value"
             v-show="allowSearch && showSearch && multiple"
             ref="search">
            <slot name="search-value"></slot>
        </div>

        <!-- Drop menu -->
        <div class="drop-menu" v-show="dropMenuShow">
            <!-- Scrollbar for Options -->
            <vue-custom-scrollbar :settings="scrollSettings">
                <!-- Опция выбора (в выпадающем списке) -->
                <div class="option"
                     :class="{ active: data.isActive }"
                     v-for="data in optionsFiltered"
                     v-show="showSelectedOption || (!showSelectedOption && !data.isActive)"
                     @click="changeValue(data.id)">
                    <slot name="option" :data="data"></slot>
                </div>
            </vue-custom-scrollbar>
        </div>
    </div>
</template>

<script lang="ts">
import { PropType } from "vue"
import Component from "vue-class-component"
import { Prop, Vue } from "vue-property-decorator"
import { Option } from './index'
// @ts-ignore
import vueCustomScrollbar from 'vue-custom-scrollbar'

@Component({
    components: { vueCustomScrollbar }
})
export default class Selector extends Vue {

    @Prop(String) readonly css!: string
    @Prop({type: Array as PropType<Option[]>}) readonly options!: Option[]
    @Prop(Boolean) showSelectedOption!: boolean
    @Prop(Boolean) allowSearch!: boolean
    @Prop(String) searchValue!: string
    @Prop(Boolean) multiple!: boolean
    @Prop(String) defaultText!: string

    $refs!: {
        search: HTMLElement
    }

    private dropMenuShow: boolean = false
    private mouseOver: boolean = false
    private showSearch: boolean = false

    private scrollSettings = {
        suppressScrollX : true,
        wheelPropagation : false
    }

    get optionsFiltered(): Option[] {

        if ( !this.searchValue )
        {
            return this.options
        }
        else
        {
            return this.options.filter( option => new RegExp(`^${this.searchValue}`, 'i').test(option.value) )
        }
    }

    get multipleValues(): Option[] {
        return this.options.filter(item => item.isActive)
    }

    get currentValue(): Option {
        let findOption = this.options.find(option => {
            return option.isActive
        })
        if ( findOption ) {
            return findOption
        }
        return {
            id: null,
            value: 'Select an option',
            isActive: true
        }
    }

    get activeOptionsAvailable() {
        return this.options.filter(item => item.isActive).length > 0
    }

    clickValue() {
        this.dropMenuShow = !this.dropMenuShow
        this.showSearch = true

        requestAnimationFrame(() => {
            if ( this.$refs.search ) {
                let inputElement = this.$refs.search.getElementsByTagName('input')[0]

                if ( inputElement ) {
                    inputElement.focus()
                }
            }
        })
    }

    changeValue(newId: number) {
        this.dropMenuShow = false
        this.showSearch = false
        this.$emit('update:searchValue', '')
        this.selectId(newId)
    }

    clickDocument() {
        if ( this.mouseOver ) return
        this.dropMenuShow = false
        this.showSearch = false
        this.$emit('update:searchValue', '')
    }

    selectId(id: number) {

        this.options.forEach(option => {

            if ( !this.multiple ) {
                option.isActive = option.id === id;
            } else if ( this.multiple && option.id === id ) {
                option.isActive = true
            }
        })

        this.$emit('update:options', this.options)
    }

    unselectId(id: number) {

        this.dropMenuShow = false
        this.showSearch = false

        this.options.forEach(option => {

            if ( option.id === id ) {
                option.isActive = false
            }
        })

        this.$emit('update:options', this.options)
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
