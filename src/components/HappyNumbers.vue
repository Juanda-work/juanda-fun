<template>
    <v-app>
        <v-app-bar v-if="$vuetify.breakpoint.smAndDown" app color="primary" dark>
            <v-toolbar-title>Números Felices</v-toolbar-title>
        </v-app-bar>
  
        <v-container>
            <v-progress-linear
                v-if="loading"
                indeterminate
                color="black"
            ></v-progress-linear>

            
            <h1 v-if="$vuetify.breakpoint.mdAndUp" class="text-center display-2 mb-5">Números Felices</h1>
            
            <v-row justify="center">
                <v-col cols="12" sm="8" md="4">
                    <v-form ref="happyNumberForm" @submit.prevent :valid="valid" class="mb-3" >
                        <v-text-field
                            label="Introduce un número"
                            placeholder="Número"
                            v-model="numberToSearch"
                            required
                            :rules="numberRules"
                            @keyup.enter="checkNumber"
                            outlined
                            dense
                        ></v-text-field>
                    </v-form>
    
                    <v-row dense>
                        <v-col cols="6">
                            <v-btn elevation="2" outlined block 
                                color="green"
                                :loading="loading"
                                @click="checkNumber"
                            >
                            Comprobar
                            </v-btn>
                        </v-col>
                        <v-col cols="6">
                            <v-btn elevation="2" outlined block
                                color="red"
                                @click="deleteNumber"
                            >
                            Borrar
                            </v-btn>
                        </v-col>
                    </v-row>
    
                    <v-row dense class="mt-3">
                        <v-col cols="6">
                            <v-btn elevation="2" outlined block
                                color="#f99a00"
                                @click="randomNumber"
                            >
                            Aleatorio
                            </v-btn>
                        </v-col>
                        <v-col cols="6">
                            <v-btn elevation="2" outlined block
                                color="gray"
                                @click="checkAttemps"
                            >
                            Ver intentos
                            </v-btn>
                        </v-col>
                    </v-row>
                    <v-row>
                        <v-col cols="12">
                            <a href="https://youtu.be/R2D8SuRjtnQ?si=ex98wJWb0RwZx9Ka" target="_blank">¿Qué son los numeros felices? </a>
                        </v-col>
                    </v-row>
                </v-col>
    
                <v-col cols="12" sm="8" md="4">
                    <v-card elevation="2" class="py-2">
                        <v-card-text>
                            <div class="font-weight-bold">
                                <h2>Resultados:</h2>
                            </div>
            
                            <v-timeline align-top dense>
                                <v-timeline-item
                                    :color="isHappy ? 'green' : 'red'"
                                >
                                    <div class="subtitle-1">
                                        Iteraciones:
                                        <span class="title">{{ iterationQuantity }}</span>
                                    </div>
                                    <v-card elevation="2">
                                        <v-card-text class="subtitle-1">
                                            {{ iterationNumbers }}
                                        </v-card-text>
                                    </v-card>
                                </v-timeline-item>
            
                                <v-timeline-item
                                    :color="isHappy ? 'green' : 'red'"
                                >
                                    <div class="subtitle-1">
                                    Número repetido:
                                    <span class="title">{{ numberRepeated || 'ninguno' }}</span>
                                    </div>
                                </v-timeline-item>
                            </v-timeline>
                        </v-card-text>
                    </v-card>
                </v-col>
            </v-row>
  
            <v-dialog
                v-model="showAttemps"
                width="90%"
                max-width="500px"
                scrollable
            >
                <v-card>
                    <v-card-title class="text-h4 grey lighten-2">
                        Intentos
                    </v-card-title>
        
                    <v-card-text>
                        <div class="title">
                            <span class="green--text">Acertados: {{ attempsOk }}</span>
                        </div>
                        <div class="title">
                            <span class="red--text">Fallados: {{ attempsFailed }}</span>
                        </div>
                    </v-card-text>
        
                    <v-divider></v-divider>
        
                    <v-card-actions>
                        <v-btn color="red" text @click="resetAttemps">Resetear</v-btn>
                        <v-spacer></v-spacer>
                        <v-btn
                            color="primary"
                            text
                            @click="showAttemps = false"
                        >
                            Cerrar
                        </v-btn>
                    </v-card-actions>
                </v-card>
            </v-dialog>

            <v-snackbar
                v-model="snackbar"
                :timeout="snackbarTimeout"
                outlined
                :color="snackbarColor"
            >
            <div v-html="snackbarText"></div>
            <template v-slot:action="{ attrs }">
                <v-btn
                    text
                    v-bind="attrs"
                    @click="snackbar = false"
                >
                Cerrar
                </v-btn>
            </template>
            </v-snackbar>
        </v-container>
    </v-app>
</template>

<script>
export default {
    data() {
        return {
            loading : false,
            numberToSearch : null,
            numberRules: [
                value => !!value || 'El número es obligatorio',
                value => (value && /^\d+$/.test(value)) || 'Debe ser un número válido',
            ],
            iterationQuantity : null,
            numberRepeated : null,
            iterationNumbers : [],
            isHappy : false,
            valid: false,
            attempsOk : 0,
            attempsFailed : 0,
            showAttemps : false,
            snackbar : false,
            snackbarText: null,
            snackbarColor: null,
            snackbarBtnColor: null,
            snackbarTimeout: 5000,
        }
    },
    methods: {
        checkNumber(){
            if (!this.$refs.happyNumberForm.validate()) {
                return;
            }
            this.resetFields();
            this.loading = true;
            let number = this.numberToSearch;
            
            setTimeout(() => {
                this.iterationNumbers = this.process(number);
                this.loading = false;                
            }, 250);
        },
        resetFields(){
            this.iterationQuantity = null;
            this.numberRepeated = null;
            this.iterationNumbers = [];
            this.isHappy = false;
            this.loading = false;
            this.snackbar = false;
        },
        process(number){
            const MAX_ITERATION_LIMIT = 100;
            let counter = 0;
            let aux = number.toString();
            let results = [];

            while (counter != MAX_ITERATION_LIMIT) {
                if (aux.length == 1 && counter == 0) {
                    aux = this.singlePow(number);
                }else if(aux.length == 1){
                    aux = this.singlePow(aux);
                }else{
                    const splitedNumber = aux.split('');
                    aux = this.getPowFromArray( splitedNumber );
                }
                
                if ( results.includes(aux) ) {
                    results.push(aux);
                    this.numberRepeated = aux;
                    this.iterationQuantity = results.length;
                    this.attempsFailed++;
                    this.showUnhappyMsg();
                    return results;
                }else if( aux == 1 ){
                    results.push(aux);
                    this.iterationQuantity = results.length;
                    this.attempsOk++;
                    this.isHappy = true;
                    this.showHappyMsg();
                    return results;
                }
                counter++;
                results.push(aux);
                this.iterationQuantity = results.length;
            }
        },
        singlePow(n){
            return Math.pow(n, 2).toString();
        },
        getPowFromArray(array){
            let res = 0;
            for (let i=0; i<array.length; i++) {
                res += Number.parseInt( this.singlePow(array[i]) );
            }
            return res.toString();
        },
        deleteNumber(){
            this.$refs.happyNumberForm.reset();
            this.resetFields();
        },
        randomNumber(){
            let randomNumber = Math.floor(Math.random()*1000);
            this.numberToSearch = randomNumber;
            this.$nextTick(() => { 
                this.checkNumber();
            });
        },
        showHappyMsg(){
            if (this.isHappy && this.numberToSearch) {
                this.snackbar = true;
                this.snackbarColor = "success";
                this.snackbarText = ` <div class="subtitle-1">
                    El número <span class="title">${this.numberToSearch}</span> es feliz! ;)
                </div>`;
            }
        },
        showUnhappyMsg(){
            if (!this.isHappy && this.iterationQuantity && this.numberToSearch) {
                this.snackbar = true;
                this.snackbarColor = "blue";
                this.snackbarText = ` <div class="subtitle-1">
                    El número <span class="title">${this.numberToSearch}</span> no es feliz... :(
                </div>`;
            }
        },
        checkAttemps(){
            this.showAttemps = true;
            this.snackbar = false;
        },
        resetAttemps(){
            this.attempsOk = 0;
            this.attempsFailed = 0;
        },
    },
    watch:{
        number(value){
            if(!value){
                this.resetFields();
            }
        },
    }
}
</script>
<style scoped>
    .finalMessage{
        margin-top: 50px;
    }
    .green{
        color: green;
    }
    .red{
        color: red;
    }
</style>