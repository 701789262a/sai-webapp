<template>
    <aside :class="`${is_expanded ? 'is-expanded' : ''}`">
        <div class="logo">
            <img :src="logoURL" alt="Vue" />
        </div>

        <div class="menu-toggle-wrap">
            <button class="menu-toggle" @click="ToggleMenu">
                <span class="material-icons">keyboard_double_arrow_right</span>
            </button>
        </div>
        <h3 v-show="is_expanded">Normative</h3>
        <ul v-for="law in laws">
            <div class="button" style="cursor: pointer;padding: 5px;" v-on:click="law.shown=!law.shown;this.$forceUpdate();" v-show="is_expanded">
                <span class="material-icons" style="vertical-align: middle">description</span>
                <span class="text" style="vertical-align: middle">{{ law.name }}</span>
            </div>
            <ul v-for="fer in law.fer" style="padding-left: 10px;" v-show="law.shown&&is_expanded" >
                <div class="button" style="cursor: pointer;padding: 5px;" v-on:click="fer.shown=!fer.shown;this.$forceUpdate();">
                    <span class="material-icons" style="vertical-align: middle">eco</span>
                    <span class="text" style="vertical-align: middle">{{ fer.name }}</span>
                </div>

                <ul v-for="type in fer.types" style="padding-left: 10px;" v-show="fer.shown&&is_expanded" v-on:click="fer.shown=true;">
                    <div class="button" style="cursor: pointer;padding: 5px;" v-on:click="store.layer=`${fer.name}_${type.type}`;console.log(store.layer);this.$forceUpdate;">
                        <span class="material-icons" style="vertical-align: middle">category</span>
                        <span class="text" style="vertical-align: middle">{{ type.type }}</span>
                    </div>
                </ul>

            </ul>
        </ul>
        <div class="flex"></div>

    </aside>
</template>

<script setup>
var laws = [{ name: 'Delib.G.R. n. 59/90 del 2020', shown: false,fer: [{ name: "Fotovoltaico e Termodinamico",  shown: false,types: [{ type: "Piccola taglia" }, { type: "Media taglia" }, { type: "Grande taglia" }] }, { name: "Eolico",  shown: false,types: [{ type: "Micro eolico" }, { type: "Mini eolico" }, { type: "Grande eolico (Eolico)" }] }, { name: "Biomasse",  shown: false,types: [{ type: "Piccola taglia" }, { type: "Media taglia" }, { type: "Grande taglia" }] }, { name: "Geotermico",  shown: false,types: [{ type: "Bassa entalpia" }, { type: "Media entalpia" }, { type: "Alta entalpia" }] }, { name: "Idroelettrico",  shown: false,types: [{ type: "Micro taglia" }, { type: "Mini taglia" }, { type: "Grande taglia" }] }] }]
import { ref } from 'vue'
import logoURL from '/src/components/icons/italy.svg'
const is_expanded = ref(localStorage.getItem("is_expanded") === "true")
import { store } from '/src/components/store.js'

const ToggleMenu = () => {
    is_expanded.value = !is_expanded.value
    localStorage.setItem("is_expanded", is_expanded.value)
}

</script>

<style lang="scss" scoped>
.button:hover{
    background-color: var(--dark-alt);

}
aside {
    display: flex;
    flex-direction: column;
    background-color: var(--dark);
    color: var(--light);
    width: calc(2rem + 32px); /* Collapsed width */
    overflow: hidden;
    min-height: 100vh;
    padding: 1rem;
    transition: width 0.5s ease-in-out;

    .flex {
        flex: 1 1 0%;
    }

    .logo {
        margin-bottom: 1rem;

        img {
            width: 2rem;
        }
    }

    .menu-toggle-wrap {
        display: flex;
        justify-content: flex-end;
        margin-bottom: 1rem;
        position: relative;
        top: 0;
        transition: 0.1s ease-in-out;

        .menu-toggle {
            transition: 0.1s ease-in-out;

            .material-icons {
                font-size: 2rem;
                color: var(--light);
                transition: 0.2s ease-out;
            }

            &:hover {
                .material-icons {
                    color: var(--primary);
                }
            }
        }
    }

    h3,
    .button .text,
    .button .material-icons {
        opacity: 0;
        visibility: hidden;
        transition: opacity 0.3s ease-in-out;
        animation: fadeIn 0.3s ease-in-out 0.3s forwards; /* Start fade-in after 3s */
    }

    h3 {
        color: var(--grey);
        font-size: 0.875rem;
        margin-bottom: 0.5rem;
        text-transform: uppercase;
    }

    .menu {
        margin: 0 -1rem;

        .button {
            display: flex;
            align-items: center;
            text-decoration: none;
            transition: 0.2s ease-in-out;
            padding: 0.5rem 1rem;

            .material-icons {
                font-size: 2rem;
                color: var(--light);
                opacity: 0;
                visibility: hidden;
                transition: opacity 0.3s ease-in-out;
                animation: fadeIn 0.3s ease-in-out 0.3s forwards; /* Delayed fade-in */
            }

            .text {
                color: var(--light);
                opacity: 0;
                visibility: hidden;
                transition: opacity 0.3s ease-in-out;
                animation: fadeIn 0.3s ease-in-out 0.3s forwards; /* Delayed fade-in */
                display: inline-block;
            }

            &:hover {
                cursor: pointer;
                background-color: var(--dark-alt);

                .material-icons,
                .text {
                    color: var(--primary);
                }
            }

            &.router-link-exact-active {
                background-color: var(--dark-alt);
                border-right: 5px solid var(--primary);

                .material-icons,
                .text {
                    color: var(--primary);
                }
            }
        }
    }

    .footer {
        opacity: 0;
        transition: opacity 0.3s ease-in-out;

        p {
            font-size: 0.875rem;
            color: var(--grey);
        }
    }

    &.is-expanded {
        width: var(--sidebar-width); /* Expanded width with smooth transition */

        .menu-toggle-wrap {

            .menu-toggle {
                transform: rotate(-180deg);
            }
        }



        .button .material-icons {
            margin-right: 1rem;
        }

        .footer {
            opacity: 1;
        }
    }

    @media (max-width: 7000px) {
        position: absolute;
        z-index: 1000;
    }
    
    .material-icons {
        transition: opacity 0.3s ease-in-out;
    }
}

/* Keyframes for fading in elements after a delay */
@keyframes fadeIn {
    from {
        opacity: 0;
        visibility: hidden;
    }
    to {
        opacity: 1;
        visibility: visible;
    }
}



</style>