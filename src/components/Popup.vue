                     <template>
    <div class="popup-container" style="width: 260px;">
        <div style="overflow: auto;" v-bind:style="[this.isPopupOpened ? 'height:400px;padding-left:15px;margin-top:20px;' : '']">
            <ul v-for="vincolo in this.vincoli">
                <p style="text-align:justify;color: white;" v-text="vincolo['tema_di_riferimento']"></p>
                <p style="text-align:justify;color: white;" v-text="vincolo['tipologie_area']"></p>
                <p style="text-align:justify;color: white;" v-text="vincolo['riferimento_normativo_area']"></p>
                <p style="text-align:justify;color: white;" v-text="vincolo['codice']"></p>
                <p style="text-align:justify;color: white;" v-text="vincolo['elementi_considerati']"></p>
                <p style="text-align:justify;color: white;" v-text="vincolo['disposizioni_tutela']"></p>
                <hr>
            </ul>
        </div>
        <div class="image-container" style="padding-top: 8px; padding-bottom: 10px;" v-bind:style="[this.isPopupOpened? 'padding-top:15px':'']">
            <img v-for="(image, index) in this.mapsLink" :src="image" :alt="'Image ' + (image + 1)"
                :class="this.redIndexes[index] ? 'pointer' : ''" @click="showVincoli(index,this.redIndexes[index])" />
        </div>

    </div>
</template>
<script>
import g1 from '/src/assets/svg/g1.svg';
import g1_red from '/src/assets/svg/g1_red.svg';
import g2 from '/src/assets/svg/g2.svg';
import g2_red from '/src/assets/svg/g2_red.svg';
import g3 from '/src/assets/svg/g3.svg';
import g3_red from '/src/assets/svg/g3_red.svg';
import g4 from '/src/assets/svg/g4.svg';
import g4_red from '/src/assets/svg/g4_red.svg';
import g5 from '/src/assets/svg/g5.svg';
import g5_red from '/src/assets/svg/g5_red.svg';
import g6 from '/src/assets/svg/g6.svg';
import g6_red from '/src/assets/svg/g6_red.svg';
import g7 from '/src/assets/svg/g7.svg';
import g7_red from '/src/assets/svg/g7_red.svg';
export default {
    name: "Popup",
    mounted() {
        this.isPopupOpened = false;
        this.themes = {
            "AMBIENTE E AGRICOLTURA": "g1",
            "ASSETTO IDROGEOLOGICO": "g2",
            "PAESAGGIO - Parte III del D.Lgs 42/2004 - Art. 136 e 157": "g3",
            "PAESAGGIO - Parte III del D.Lgs 42/2004 - Art. 142 - Aree tutelate per legge": "g4",
            "PAESAGGIO - Parte III del D.Lgs 42/2004 - Art. 143 comma 1 lettera d": "g5",
            "ULTERIORI CONTESTI BENI IDENTITARI - Parte III del D.Lgs 42/2004 - Art. 143 comma 1 lettera e": "g6",
            "SITI UNESCO": "g7",
        }
        this.iconMap = {
            g1,
            g1_red,
            g2,
            g2_red,
            g3,
            g3_red,
            g4,
            g4_red,
            g5,
            g5_red,
            g6,
            g6_red,
            g7,
            g7_red,
        };

        this.mapsLink = []
        this.redIndexes = []
        for (var theme in this.themes) {
            let themeKey = this.themes[theme];
            var k = false;
            for (var vincolo in this.j) {
                console.log(theme, this.j[vincolo]['tema_di_riferimento'])
                if (this.j[vincolo]['tema_di_riferimento'] == theme) {
                    console.log('cacca')
                    this.mapsLink.push(this.iconMap[`${themeKey}_red`]);
                    this.redIndexes.push(true);

                    k = true;
                    break
                }

            }
            if (!k) {
                this.mapsLink.push(this.iconMap[themeKey]);
                this.redIndexes.push(false);

            }
        }
        console.log(this.mapsLink)
        this.$forceUpdate()
    },
    props: {
        j: {
            required: true,
        },
    },
    methods: {
        showVincoli(index,goAhead) {
            if (!goAhead){
                return
            }
            if (this.isPopupOpened & this.selectedIndex==index) {
                this.vincoli=[]
                this.isPopupOpened = false
                this.$forceUpdate()
                return;
            }
            this.selectedIndex = index
            console.log(index)
            console.log(Object.keys(this.themes)[index])
            this.vincoliClassificati = []
            var targetClasseVincolo = Object.keys(this.themes)[index]

            for (var vincolo in this.j) {
                if (this.j[vincolo]['tema_di_riferimento'] == targetClasseVincolo) {
                    this.vincoliClassificati.push(this.j[vincolo])
                }
            }
            this.vincoli = this.vincoliClassificati
            console.log(this.vincoli)
            if (this.vincoli.length > 0) {
                this.isPopupOpened = true;
                this.$forceUpdate()
            }


        }
    },
};


</script>


<style scoped>
.popup-container {
    text-align: center;
    max-width: 350px;
    max-height: 700px;
}

.image-container {
    display: flex;
    justify-content: center;
    gap: 10px;
    /* Space between images */
    flex-wrap: nowrap;
    /* Prevent wrapping */
    overflow-x: auto;
    /* Add horizontal scroll if needed */
}

.image-container img {
    max-width: 25px;
    max-height: 25px;
    object-fit: cover;
    border-radius: 2px;
}

button {
    margin-top: 10px;
    padding: 5px 10px;
    cursor: pointer;
}

.pointer {
    cursor: pointer;
}

p {
    -webkit-hyphens: auto;
    -moz-hyphens: auto;
    hyphens: auto;
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
  overflow: auto; /* Ensure content doesn't overflow improperly */
}
.openedPopup {
    width: 300px
}
</style>