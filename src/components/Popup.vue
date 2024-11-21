<template>
    <div class="popup-container" style="width: 260px;">
        <div class="image-container" style="padding-bottom: 10px;">
            <img v-for="(image, index) in this.mapsLink" :src="image" :alt="'Image ' + (image + 1)"
                :class="String(image).includes('red') ? 'pointer' : ''" @click="showVincoli(index)" />
        </div>
        <div style="height:600px;overflow: auto;" >
            <ul v-for="vincolo in this.vincoli" >
                <p style="text-align:justify; color: white;" v-text="vincolo['tema_di_riferimento']"></p>
                <p style="text-align:justify;color: white;" v-text="vincolo['tipologie_area']"></p>
                <p style="text-align:justify;color: white;" v-text="vincolo['riferimento_normativo_area']"></p>
                <p style="text-align:justify;color: white;" v-text="vincolo['codice']"></p>
                <p style="text-align:justify;color: white;" v-text="vincolo['elementi_considerati']"></p>
                <p style="text-align:justify;color: white;" v-text="vincolo['disposizioni_tutela']"></p>
                <hr>
            </ul>
        </div>
    </div>
</template>
<script>
export default {
    name: "Popup",
    mounted() {
        this.themes = {
            "AMBIENTE E AGRICOLTURA": "g1",
            "ASSETTO IDROGEOLOGICO": "g2",
            "PAESAGGIO - Parte III del D.Lgs 42/2004 - Art. 136 e 157": "g3",
            "PAESAGGIO - Parte III del D.Lgs 42/2004 - Art. 142 - Aree tutelate per legge": "g4",
            "PAESAGGIO - Parte III del D.Lgs 42/2004 - Art. 143 comma 1 lettera d": "g5",
            "ULTERIORI CONTESTI BENI IDENTITARI - Parte III del D.Lgs 42/2004 - Art. 143 comma 1 lettera e": "g6",
            "SITI UNESCO": "g7",
        }
        this.mapsLink = []
        for (var theme in this.themes) {
            var k = false;
            for (var vincolo in this.j) {
                console.log(theme, this.j[vincolo]['tema_di_riferimento'])
                if (this.j[vincolo]['tema_di_riferimento'] == theme) {
                    console.log('cacca')
                    this.mapsLink.push(`/src/assets/svg/${this.themes[theme]}_red.svg`)
                    k = true;
                    break
                }

            }
            if (!k) {
                this.mapsLink.push(`/src/assets/svg/${this.themes[theme]}.svg`)
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
        showVincoli(index) {

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
            this.$forceUpdate()
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
</style>