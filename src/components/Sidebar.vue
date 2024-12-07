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
        <h3 v-show="is_expanded">Cerca coordinate o indirizzo</h3>
        <input class="input" placeholder="e.g. 40.321388, 9.313577" v-on:keyup.enter="processInput($event)"
            v-show="is_expanded">
        <h3 v-show="is_expanded">Normative</h3>
        <div style="overflow-y: auto;">

            <ul v-for="law in laws">
                <div class="button" style="cursor: pointer;padding: 5px;"
                    v-on:click="law.shown = !law.shown; $forceUpdate(); store.law = law.name; store.lawValue = law.value"
                    v-show="is_expanded && law.enabled">
                    <span class="material-icons" style="vertical-align: middle">description</span>
                    <span class="text" style="vertical-align: middle">{{ law.name }}</span>
                </div>
                <ul v-for="fer in law.fer" style="padding-left: 10px;" v-show="law.shown && is_expanded">
                    <div class="button" style="cursor: pointer;padding: 5px;"
                        v-on:click="fer.shown = !fer.shown; $forceUpdate();">
                        <span class="material-icons" style="vertical-align: middle">eco</span>
                        <span class="text" style="vertical-align: middle">{{ fer.name }}</span>
                    </div>

                    <ul v-for="type in fer.types" style="padding-left: 10px;" v-show="fer.shown && is_expanded"
                        v-on:click="fer.shown = true;">
                        <div class="button" style="cursor: pointer;padding: 5px;"
                            v-on:click="store.layer = `${fer.name}_${type.type}`; console.log(store.layer); $forceUpdate();"
                            @click="ToggleMenuIfMobile">
                            <span class="material-icons" style="vertical-align: middle">category</span>
                            <span class="text" style="vertical-align: middle">{{ type.type }}</span>
                        </div>
                    </ul>

                </ul>
            </ul>
        </div>
        <div class="bottom-button" v-show="false">
            <div class="button" style="cursor: pointer; padding: 15px; text-align: center;border-radius: 5px;"
                @click="handleBottomButtonClick">
                <span class="material-icons" style="vertical-align: middle;" >question_mark</span><!-- Da sistemare con vshow = is_expanded-->
                <span class="text" style="vertical-align: middle;" v-show="false">Informazioni</span><!-- Da sistemare con vshow = is_expanded-->
            </div>
        </div>
        <div class="flex"></div>


    </aside>
</template>

<script setup>
import sidebarjson from '/src/components/sidebar.json'
var laws = sidebarjson
import { ref } from 'vue'
import logoURL from '/src/components/icons/italy.svg'
const is_expanded = ref(localStorage.getItem("is_expanded") === "true")
import { store } from '/src/components/store.js'
async function processInput(input) {
    ToggleMenu();
    console.log(input.target.value)
    // Regex per verificare se l'input sono coordinate
    const coordRegex = /^-?\d+(\.\d+)?\s*,\s*-?\d+(\.\d+)?$/;

    if (coordRegex.test(input.target.value)) {
        console.log("Coordinate rilevate: ", input.target.value);
        try {
            if (civico_marker != null) {
                map.removeLayer(civico_marker);
            }
        } catch (e) {

        }


        const [lat, lng] = input.target.value.split(',').map(Number);
        store.flyCoord = [lat, lng]

    } else {
        var precisionecivico
        console.log("Indirizzo rilevato: ", input.target.value);

        let nominatimData = "ko"
        let tipoosm
        const nominatimrequest = await fetch(`https://nominatim.openstreetmap.org/search?q=${input.target.value}&format=json&addressdetails=1&limit=1&polygon_svg=1`, { signal: AbortSignal.timeout(3000) })
            .catch(error => {
                console.log("nominatim errore")
            })
        if (nominatimrequest != null) {
            nominatimData = await nominatimrequest.json()
            if (nominatimData.length > 0) {
                tipoosm = nominatimData[0].addresstype
            }
        }

        if (nominatimData != "ko" && nominatimData.length > 0 && tipoosm === "place" || tipoosm === "house") {
            var latok = nominatimData[0].lat
            var lngok = nominatimData[0].lon
            precisionecivico = nominatimData[0].addresstype
            console.log("nominatim", latok, lngok, precisionecivico)
            var tipogeocoding = "OSM"
        } else {
            const arcgisrequest = await fetch(`https://geocode.arcgis.com/arcgis/rest/services/World/GeocodeServer/findAddressCandidates?f=pjson&outFields=Addr_type&forStorage=false&SingleLine=${input.target.value}`);
            const arcgisData = await arcgisrequest.json();


            if (arcgisData.candidates[0].attributes.Addr_type === "PointAddress" || arcgisData.candidates[0].attributes.Addr_type === "Subaddress") {
                var latok = arcgisData.candidates[0].location.y
                var lngok = arcgisData.candidates[0].location.x
                var precisionecivico = arcgisData.candidates[0].attributes.Addr_type
                tipogeocoding = "Esri"
                console.log("arcgis", latok, lngok, precisionecivico)
            } else {
                const googlerequest = await fetch(`https://maps.googleapis.com/maps/api/geocode/json?address=${input.target.value}&key=AIzaSyBUuDGQcuvj7tmrCZ3ZSVgxYn3ochO21YI`);
                const googleData = await googlerequest.json();


                if (googleData.results[0].geometry.location_type === "ROOFTOP") {
                    var latok = googleData.results[0].geometry.location.lat;
                    var lngok = googleData.results[0].geometry.location.lng;
                    precisionecivico = googleData.results[0].geometry.location_type;
                    var tipogeocoding = "Google"
                    console.log("google", latok, lngok, precisionecivico)
                }
                else {
                    if (nominatimData != "ko" && nominatimData[0].addresstype === "road") {
                        var latok = nominatimData[0].lat
                        var lngok = nominatimData[0].lon
                        precisionecivico = nominatimData[0].addresstype
                        console.log("nominatim", latok, lngok, precisionecivico)
                        var tipogeocoding = "Nominatim OpenStreetMap"
                    } else {
                        if (arcgisData.candidates[0].Addr_type === "StreetAddress" || arcgisData.candidates[0].Addr_type === "StreetAddressExt" || arcgisData.candidates[0].Addr_type === "StreetInt") {
                            var latok = arcgisData.candidates[0].location.y
                            var lngok = arcgisData.candidates[0].location.x
                            precisionecivico = arcgisData.candidates[0].attributes.Addr_type
                            var tipogeocoding = "Esri"
                            console.log("arcgis", latok, lngok, precisionecivico)
                        } else {
                            var latok = googleData.results[0].geometry.location.lat;
                            var lngok = googleData.results[0].geometry.location.lng;
                            precisionecivico = googleData.results[0].geometry.location_type;
                            var tipogeocoding = "Google"
                            console.log("google", latok, lngok, precisionecivico)
                        }
                    }
                }
            }
        }
        try {
            if (civico_marker != null) {
                map.removeLayer(civico_marker);
            }
        } catch (e) {

        }
        console.log(latok, lngok)
        store.flyCoord = [latok, lngok]
    }
}
const ToggleMenu = () => {
    is_expanded.value = !is_expanded.value
    localStorage.setItem("is_expanded", is_expanded.value)
}
const ToggleMenuIfMobile = () => {
    if (window.innerWidth < 750) {
        is_expanded.value = !is_expanded.value
        localStorage.setItem("is_expanded", is_expanded.value)
    }

}
</script>

<style lang="scss" scoped>
.button:hover {
    background-color: var(--dark-alt);

}

aside {
    display: flex;
    flex: 0 0 auto;
    flex-direction: column;
    background-color: var(--dark);
    color: var(--light);
    width: calc(2rem + 32px);
    /* Collapsed width */
    overflow: hidden;
    height: 100%;
    padding: 1rem;
    transition: width 0.5s ease-in-out;

    .flex {
        flex: 0 0 auto;
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
        animation: fadeIn 0.3s ease-in-out 0.3s forwards;
        /* Start fade-in after 3s */
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
                animation: fadeIn 0.3s ease-in-out 0.3s forwards;
                /* Delayed fade-in */
            }

            .text {
                color: var(--light);
                opacity: 0;
                visibility: hidden;
                transition: opacity 0.3s ease-in-out;
                animation: fadeIn 0.3s ease-in-out 0.3s forwards;
                /* Delayed fade-in */
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
        width: var(--sidebar-width);
        /* Expanded width with smooth transition */

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

::placeholder {
    color: rgb(101, 101, 119);
    opacity: 1;
    /* Firefox */
}

.input {

    height: 30px;
    border-radius: 6px;
    margin-bottom: 1rem;
    border-color: black;
    border-width: 0cap;
    color: white;
    background-color: var(--dark-alt);
    font-size: 0.875rem;
    text-transform: uppercase;
    text-indent: 0.5rem;
}

.bottom-button {
    margin-top: auto;
    /* Push this element to the bottom of the flex container */
    display: flex;
    justify-content: center;
    padding-top: 1rem;
}

.action-button {
    display: flex;
    align-items: center;
    justify-content: center;
    background-color: var(--primary);
    color: var(--light);
    border: none;
    border-radius: 4px;
    padding: 0.5rem 1rem;
    font-size: 0.875rem;
    cursor: pointer;
    transition: background-color 0.3s ease-in-out;

    .material-icons {
        margin-right: 0.5rem;
    }

    &:hover {
        background-color: var(--primary-alt);
    }
}

::-webkit-scrollbar {
    width: 14px;
}

::-webkit-scrollbar-thumb {
    border: 4px solid rgba(0, 0, 0, 0);
    background-clip: padding-box;
    border-radius: 9999px;
    background-color: #AAAAAA;
}

::v-deep(.leaflet-popup-content-wrapper) {
    background-color: #000;
    overflow: auto;
    /* Ensure content doesn't overflow improperly */
}
</style>