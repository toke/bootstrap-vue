<template>
    <div class="tabs">
        <div :class="{'card-header': card}">
            <ul :class="['nav','nav-' + navStyle, card? 'card-header-'+navStyle: null]">
                <li class="nav-item" v-for="(tab, index) in tabs">
                    <a :class="['nav-link',{small: small, active: tab.localActive, disabled: tab.disabled}]"
                       :href="tab.href"
                       @click.prevent.stop="setTab(index)"
                       v-html="tab.title"
                    ></a>
                </li>
            </ul>
        </div>
        <div :class="['tab-content',{'card-block': card}]">
            <slot></slot>
        </div>
    </div>
</template>

<script>
    export default {
        data() {
            return {
                currentTab: this.value || 0,
                tabs: []
            };
        },
        props: {
            noFade: {
                type: Boolean,
                default: false
            },
            card: {
                type: Boolean,
                default: false
            },
            small: {
                type: Boolean,
                default: false
            },
            value: {
                type: Number,
                default: 0
            },
            pills: {
                type: Boolean,
                default: false
            },
            lazy: {
                type: Boolean,
                default: false
            }
        },
        watch: {
            currentTab(val, old) {
                if (val === old) {
                    return;
                }

                this.$root.$emit('changed::tab', this, val, this.tabs[val]);
                this.$emit('input', val);
            },
            value(val, old) {
                if (val === old) {
                    return;
                }

                this.setTab(val);
            },
            fade(val, old) {
                if (val === old) {
                    return;
                }

                this.tabs.forEach(item => {
                    this.$set(item, 'fade', val);
                });
            }
        },
        computed: {
            fade() {
                return !this.noFade;
            },
            navStyle() {
                return this.pills ? 'pills' : 'tabs';
            }
        },
        methods: {
            /**
             * Move to next tab
             */
            nextTab() {
                this.setTab(this.currentTab + 1);
            },

            /**
             * Move to previous tab
             */
            previousTab() {
                this.setTab(this.currentTab - 1);
            },

            /**
             * Set active tab on the tabs collection and the child 'tab' component
             */
            setTab(index, force) {
                // Prevent setting same tab!
                if (!force && index === this.currentTab) {
                    return;
                }

                const tab = this.tabs[index];

                // Don't go beyond indexes!
                if (!tab) {
                    return;
                }

                // Ignore disabled
                if (tab.disabled) {
                    return;
                }

                // Deactivate previous active tab
                if (this.tabs[this.currentTab]) {
                    this.$set(this.tabs[this.currentTab], 'localActive', false);
                }

                // Set new tab as active
                this.$set(tab, 'localActive', true);

                // Update currentTab
                this.currentTab = index;
            }
        },
        mounted() {
            // Probe tabs
            if (this.$slots.default) {
                this.tabs = this.$slots.default.filter(tab => tab.componentInstance || false)
                    .map(tab => tab.componentInstance);
            }

            this.tabs.forEach(tab => {
                this.$set(tab, 'fade', this.fade);
                this.$set(tab, 'lazy', this.lazy);
            });

            // Set initial active tab
            let tabIndex = this.currentTab;

            this.tabs.forEach((tab, index) => {
                if (tab.active) {
                    tabIndex = index;
                }
            });

            this.setTab(tabIndex, true);
        }
    };

</script>
