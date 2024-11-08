<template>
    <v-app>
        <v-progress-linear v-if="loading" indeterminate color="black" ></v-progress-linear>
        <v-container>
            <h1 class="text-center display-2 mb-5">Números Felices</h1>
            <v-row>
                <v-spacer></v-spacer>
                <v-col cols="4">
                    <v-form ref="happyNumberForm" class="mb-3" :valid="valid">
                        <v-row>
                            <v-col cols="6">
                                <v-text-field 
                                    label="Introduce un número" 
                                    placeholder="Número" 
                                    v-model="numberToSearch"
                                    required
                                    :rules="numberRules"
                                >
                                </v-text-field>
                            </v-col>
                        </v-row>
                    </v-form>
                    <v-btn 
                        elevation="2"
                        outlined
                        color="green"
                        :loading="loading"
                        @click="checkNumber">
                        Comprobar
                    </v-btn>
                    <v-btn
                        class="ml-5"
                        elevation="2"
                        outlined
                        color="red"
                        @click="deleteNumber">
                        Borrar
                    </v-btn>
                    <v-spacer class="my-5"></v-spacer>
                    <v-btn
                        elevation="2"
                        outlined
                        color="#f99a00"
                        @click="randomNumber">
                        Aleatorio
                    </v-btn>
                    <v-btn
                        class="ml-5"
                        elevation="2"
                        outlined
                        color="gray"
                        @click="checkAttemps">
                        Ver intentos
                    </v-btn>
                </v-col>

                <v-col cols="3" class="finalMessage">
                    <div> 
                        <v-fab-transition v-show="isHappy"> 
                            <v-alert v-show="showHappyMsg()" type="success"> El número es feliz </v-alert> 
                        </v-fab-transition> 
                        <v-fade-transition v-show="!isHappy"> 
                            <v-alert v-show="showUnhappyMsg()" type="error"> No es feliz... </v-alert> 
                        </v-fade-transition> 
                    </div>
                </v-col>

                <v-col cols="5">
                    <v-card-text>
                        <div class="font-weight-bold ml-8 mb-2">
                            <h2>Resultados:</h2>
                        </div>
                        <v-timeline align-top dense>
                            <v-timeline-item v-if="iterationQuantity" :color="isHappy ? 'green' : 'red'">
                                <div>
                                    <div>
                                        <div class="subtitle-1">Iteraciones: <span class="title">{{iterationQuantity}}</span> </div>
                                    </div>
                                    <v-card elevation="2"> 
                                        <v-card-text class="subtitle-1"> {{ iterationNumbers }} </v-card-text>
                                    </v-card>
                                </div>
                            </v-timeline-item>
                            <v-timeline-item v-if="showUnhappyMsg()" :color="isHappy ? 'green' : 'red'">
                                <div>
                                    <div>
                                        <div class="subtitle-1">Número repetido: <span class="title">{{numberRepeated}}</span> </div>
                                    </div>
                                </div>
                            </v-timeline-item>
                        </v-timeline>
                    </v-card-text>
                </v-col>
            </v-row>

            <v-dialog
                v-model="showAttemps"
                width="500"
                >
                <v-card>
                    <v-card-title class="text-h4 grey lighten-2">
                     Intentos
                    </v-card-title>

                    <v-card-text>
                        <div class="title">
                            <span color="green"> Acertados: {{ attempsOk }} </span>
                        </div>
                        <div class="title">
                            <span color="red"> Fallados: {{ attempsFailed }} </span>
                        </div>
                    </v-card-text>

                    <v-divider></v-divider>

                    <v-card-actions>
                        <v-btn
                            color="red"
                            text
                            @click="resetAttemps"
                        >
                            Resetear intentos
                        </v-btn>
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
                    return results;
                }else if( aux == 1 ){
                    results.push(aux);
                    this.iterationQuantity = results.length;
                    this.attempsOk++;
                    this.isHappy = true;
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
            return this.isHappy && this.numberToSearch;
        },
        showUnhappyMsg(){
            return !this.isHappy && this.iterationQuantity && this.numberToSearch;
        },
        checkAttemps(){
            this.showAttemps = true;
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
        }
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