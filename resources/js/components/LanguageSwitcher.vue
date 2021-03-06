<template>
    <div class="relative">
        <button
            href="#"
            class="flex items-center"
            @click="toggleVisibility"
            @keydown.space.exact.prevent="toggleVisibility"
            @keydown.esc.exact="hideDropdown"
            @keydown.shift.tab="hideDropdown"
            @keydown.up.exact.prevent="startArrowKeys"
            @keydown.down.exact.prevent="startArrowKeys"
        >
            <img :src="`/flags/${$i18n.locale}.svg`" alt="flag" class="fill-current h-4 w-4">
            <span class="ltr:ml-2 rtl:mr-2 text-sm whitespace-no-wrap">{{ $i18n.locale.toUpperCase() }}</span>
            <svg fill="currentColor" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" width="24" height="24"><path class="heroicon-ui" d="M15.3 9.3a1 1 0 0 1 1.4 1.4l-4 4a1 1 0 0 1-1.4 0l-4-4a1 1 0 0 1 1.4-1.4l3.3 3.29 3.3-3.3z"></path></svg>
        </button>
        <transition name="dropdown-fade">
            <ul v-on-clickaway="hideDropdown" v-if="isVisible" ref="dropdown" class="absolute normal-case z-30 font-normal xs:left-0 lg:right-0 bg-white shadow overflow-hidden rounded w-48 border mt-2 py-1 lg:z-20">
                <li v-for="locale in locales" :key="locale.code">
                    <a href="#"
                        @click.prevent="setLocale(locale.code)"
                        ref="account"
                        class="flex items-center px-4 py-2 text-sm text-gray-700 hover:bg-gray-100 whitespace-no-wrap"
                        @keydown.up.exact.prevent=""
                        @keydown.tab.exact="focusNext(false)"
                        @keydown.down.exact.prevent="focusNext(true)"
                        @keydown.esc.exact="hideDropdown">
                        <img :src="`/flags/${locale.code}.svg`" class="h-4 w-4">
                        <span class="ltr:ml-2 rtl:mr-2">{{locale.name}}</span>
                    </a>
                </li>
            </ul>
        </transition>
    </div>
</template>

<script>
    import { mixin as clickaway } from 'vue-clickaway';
    import { getSupportedLocales } from "~/utils/i18n/supported-locales"
    import i18n, {loadLocaleMessagesAsync} from "../i18n";
    import {setDocumentDirectionPerLocale, setDocumentLang, setDocumentTitle} from "../utils/i18n/document";

    export default {
        name: "language_switcher",
        mixins: [ clickaway ],
        data() {
            return {
                isVisible: false,
                focusedIndex: 0,
                locales: getSupportedLocales()
            }
        },
        methods: {
            toggleVisibility() {
                this.isVisible = !this.isVisible;
            },
            hideDropdown() {
                this.isVisible = false;
                this.focusedIndex = 0
            },
            startArrowKeys() {
                if (this.isVisible) {
                    // this.$refs.account.focus()
                    this.$refs.dropdown.children[0].children[0].focus()
                }
            },
            focusPrevious(isArrowKey) {
                this.focusedIndex = this.focusedIndex - 1;
                if (isArrowKey) {
                    this.focusItem()
                }
            },
            focusNext(isArrowKey) {
                this.focusedIndex = this.focusedIndex + 1;
                if (isArrowKey) {
                    this.focusItem()
                }
            },
            focusItem() {
                this.$refs.dropdown.children[this.focusedIndex].children[0].focus()
            },
            setLocale(locale) {
                loadLocaleMessagesAsync(locale).then(() => {
                    setDocumentLang(locale);
                    setDocumentDirectionPerLocale(locale);
                });
                this.$inertia.hardVisit(true,'/lang/'+locale)
                this.hideDropdown()
            }
        }
    }
</script>

<style scoped>
    .dropdown-fade-enter-active, .dropdown-fade-leave-active {
        transition: all .1s ease-in-out;
    }
    .dropdown-fade-enter, .dropdown-fade-leave-to {
        opacity: 0;
        transform: translateY(-12px);
    }
</style>
