<template>

    <div>
        <v-toolbar v-if="!iframe">
            <v-toolbar-title class="headline">
                <span> Vue Sorting Visualizations </span>
                <span class="font-weight-light"> Demo App </span>
            </v-toolbar-title>
            <v-spacer></v-spacer>
            <v-btn flat href="https://github.com" target="_blank">
                <span class="mr-2">Latest Release</span>
            </v-btn>
        </v-toolbar>
        <v-container>
            <v-card class="pa-3">
                <v-card-title primary-title>
                    <div>
                        <h3 class="headline mb-0 text-capitalize">{{ sortType }} Sort</h3>
                    </div>
                </v-card-title>
                <v-container fill-height fluid>
                    <v-layout fill-height>
                        <div class="visualisation-container">
                            <div class="visualisation-item" v-for="(item, index) in array" :title="item"
                                 :class="{ 'primary': visualisation.highlights.indexOf(index) !== -1 }"
                                 :style="{ height: (Math.floor(item/array.length * 400)) + 'px' }">
                            </div>
                        </div>
                    </v-layout>
                </v-container>
                <v-card-actions class="text-right">
                    <v-layout row wrap>
                        <v-flex xs12>
                            <v-subheader class="pl-0">Array Size</v-subheader>
                            <v-slider v-model="size" min="10" max="100" thumb-label="always"></v-slider>
                        </v-flex>
                        <v-flex xs12>
                            <v-subheader class="pl-0">Animation Speed</v-subheader>
                            <v-slider v-model="speed" min="1" max="10" thumb-label="always"></v-slider>
                        </v-flex>
                        <v-flex xs-3>
                            <v-select color="info" height="25" dense v-model="sortType" class="sort-select"
                                      :items="sortTypes"
                                      label="Sort Type"
                            ></v-select>
                        </v-flex>
                        <v-flex xs-9 class="text-xs-right">
                            <v-btn light color="info" @click="randomize()">
                                Random
                                <v-icon right light>cached</v-icon>
                            </v-btn>
                            <v-btn light color="primary" @click="sort()">
                                Sort
                                <v-icon right light>sort</v-icon>
                            </v-btn>
                        </v-flex>
                    </v-layout>

                </v-card-actions>
            </v-card>
        </v-container>
    </div>

</template>

<style>
    .visualisation-container {
        width: 100%;
        height: 404px;
        display: flex;
        align-items: flex-end;
        justify-content: space-evenly;
        background: linear-gradient(#f8f8f8, #f0f0f0);
        border: 1px solid #dedede;
    }
    .visualisation-item {
        background: #888;
        margin: 1px;
        width: auto;
        align-self: flex-end;
        flex-grow:1;
        border-radius: 4px;
        background: linear-gradient(#a0a0a0, #787878);

    }
    .visualisation-item.primary {
        background: linear-gradient(#2196f3, #1976d2);
    }
    .sort-select {
        width: 200px;
    }
</style>

<script>
    export default {
        data: () => ({
            query: {},
            iframe: false,
            array: [],
            size: 50,
            speed: 5,
            sortType: 'bubble',
            sortTypes: [
                {
                    'value': 'quick',
                    'text': 'Quick Sort'
                },
                {
                    'value': 'bubble',
                    'text': 'Bubble Sort'
                }
            ],
            visualisation: {
                step: 0,
                callbacks: [],
                highlights: []
            }
        }),

        mounted() {
            let self = this;
            self.randomize();
            self.init();
        },

        methods: {

            init() {
                let self = this;
                let queryString = window.location.href.split('?')[1];
                if (queryString) {
                    queryString.split('&').forEach((q) => {
                        if (q.split('=').length) {
                            self.query[q.split('=')[0]] = q.split('=')[1];
                        }
                    })
                }
                if (self.query.hasOwnProperty('sort')) {
                    self.sortType = (self.sortTypes.filter((s) => { return s.value === self.query.sort })[0] || { 'value': 'bubble'}).value;
                }
                if (self.query.hasOwnProperty('iframe')) {
                    self.iframe = self.query.iframe === 'true';
                }
            },

            randomize() {
                let self = this;
                self.resetVisualization();
                self.array = [];
                for (let i=0; i<self.size; i++) {
                    self.array.push(i+1);
                }
                let currentIndex = self.array.length, aux, randomIndex;

                while (0 !== currentIndex) {

                    randomIndex = Math.floor(Math.random() * currentIndex);
                    currentIndex -= 1;

                    aux = self.array[currentIndex];
                    self.array[currentIndex] = self.array[randomIndex];
                    self.array[randomIndex] = aux;
                }
            },

            sort() {
                let self = this;
                self.resetVisualization();
                switch (self.sortType) {
                    case 'bubble': {
                        self.bubbleSort();
                        break;
                    }
                    case 'quick': {
                        self.quickSort();
                        break;
                    }
                    default: {
                        self.bubbleSort();
                    }
                }
            },

            resetVisualization() {
                let self = this;
                self.visualisation.step = 0;
                self.visualisation.highlights = [];
                self.visualisation.callbacks.forEach((c) => {
                    clearTimeout(c);
                });
                self.visualisation.callbacks = [];
            },

            addVisualization(highlights, params, callback) {
                let self = this;
                self.visualisation.callbacks.push(
                    setTimeout(() => {
                        callback(params);
                        self.visualisation.highlights = highlights;
                    }, self.visualisation.step *  Math.floor(1000 / Math.pow(self.speed, 2)))
                );
                self.visualisation.step += 1;
            },

            finishVisualization() {
                let self = this;
                self.addVisualization([], [], () => {});
            },

            bubbleSort() {
                let self = this;
                let array = self.array.slice();
                let n = array.length;
                let swapped;
                do {
                    swapped = false;
                    for (let i = 1; i < n; i++) {
                        if (array[i-1] > array[i]) {
                            self.swap(array, i-1, i);
                            swapped = true;
                            self.addVisualization([i, i-1], [self, i, i-1], (params) => {
                                let self = params[0];
                                let i = params[1];
                                let j = params[2];
                                self.swap(self.array, i, j);
                                self.array = self.array.slice();
                            });
                        }
                    }
                } while (swapped);
                self.finishVisualization();
            },

            quickSort() {
                let self = this;
                let array = self.array.slice();
                let partition = (A, lo, hi) => {
                    let pivot = A[lo];
                    let i = lo - 1;
                    let j = hi + 1;
                    while (true) {
                        do {
                            i += 1;
                        } while (A[i] < pivot);
                        do {
                            j -= 1;
                        } while (A[j] > pivot);
                        if (i >= j) {
                            return j;
                        }
                        self.swap(A, i, j);
                        self.addVisualization([i, j], [], () => {});
                        self.addVisualization([i, j], [self, i, j], (params) => {
                            let self = params[0];
                            let i = params[1];
                            let j = params[2];
                            self.swap(self.array, i, j);
                        });
                    }
                };

                let quicksort = (A, lo, hi) => {
                    if (lo < hi) {
                        let p = partition(A, lo, hi);
                        quicksort(A, lo, p);
                        quicksort(A, p+1, hi);
                    }
                };

                quicksort(array, 0, array.length-1);
                self.finishVisualization();
                // self.array = array.slice();
                // console.log('done');
            },

            swap(array, i, j) {
                let aux = array[i];
                array[i] = array[j];
                array[j] = aux;
            }
        },

        watch: {
            size: function (val) {
                let self = this;
                self.randomize();
            }
        }
    }
</script>

<style>

</style>
