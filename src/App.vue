<template>
    <div id="app">
        <div class="section section--sticky">
            <h3>Статус запроса:</h3>
            <p>{{ requestStatus }}</p>
            <div class="section__row">
                <div class="section__col">
                    <button type="button" @click="getArray()">Отправить запрос</button>
                </div>
                <div class="section__col" v-if="arrayNotEmpty(testArr)">
                    <button type="button" @click="reset()" class="section__reset">Сброс</button>
                </div>
            </div>

            <template v-if="arrayNotEmpty(states)">
                <h4>Состояния:</h4>
                <div class="section__row">
                    <div class="section__stateButtons">
                        <button type="button" class="section__stateButton" v-for="(item, index) of states" @click="setState(item)">
                            State {{ returnButtonCounter(index, states.length) }}
                        </button>
                    </div>
                </div>
            </template>
        </div>

        <div class="section" v-if="arrayNotEmpty(testArr)">
            <h3>Исходный массив:</h3>
            <pre>{{ testArr }}</pre>
        </div>

        <div class="section" v-if="arrayNotEmpty(floatArr)">
            <h3>Сглаженный массив:</h3>
            <pre>{{ floatArr }}</pre>
        </div>

        <div class="section" v-if="arrayNotEmpty(divideArray)">
            <h3>Разделенный по примитивам:</h3>
            <pre>{{ divideArray }}</pre>
        </div>

        <div class="section" v-if="arrayNotEmpty(divideArray)">
            <div class="section__row">
                <template v-for="(item, index) of divideArray">
                    <div class="section__col">
                        <multiselect
                            class="select"
                            v-model="selectedItems[index]"
                            :options="item"
                            :multiple="true"
                            :searchable="false"
                            :close-on-select="false"
                            :show-labels="false"
                            @input="addState(selectedItems)"
                            placeholder="Pick a value"
                        />

                        <list-items :items="selectedItems[index]" v-if="arrayNotEmpty(selectedItems[index])" class="section__listItems" />
                    </div>
                </template>
            </div>
        </div>
    </div>
</template>

<script>
    import axios from 'axios';
    import Multiselect from 'vue-multiselect';
    import 'vue-multiselect/dist/vue-multiselect.min.css';
    import ListItems from '@/components/listItems';

    export default {
        name: 'App',

        components: {
            ListItems,
            Multiselect,
        },

        data() {
            return {
                testArr: [],
                requestStatus: '—',
                selectedItems: [],
                states: [],
            };
        },

        methods: {
            async getArray() {
                this.requestStatus = `Загрузка ...`;
                const requestStartTime = performance.now();

                await axios
                    .get('https://raw.githubusercontent.com/WilliamRu/TestAPI/master/db.json')
                    .then(res => {
                        const requestDelta = parseInt(performance.now() - requestStartTime);
                        this.requestStatus = `Загрузка завершена. Примерное время выполнения запроса: ${requestDelta} мс`;
                        this.testArr = res.data?.testArr;
                    })
                    .catch(err => {
                        this.requestStatus = `Ошибка выполнения запроса. ${err}`;
                        console.error(err);
                    });
            },

            arrayNotEmpty(arr) {
                return Array.isArray(arr) && arr.length > 0;
            },

            addState() {
                const current = [...this.selectedItems];
                this.states.push(current);

                if (this.states.length > 10) {
                    this.states.shift();
                }
            },

            setState(state) {
                this.selectedItems = state;
            },

            returnButtonCounter(index, length) {
                return length - 1 === index ? 'last' : `-${length - 1 - index}`;
            },

            reset() {
                this.testArr = [];
                this.requestStatus = '—';
                this.selectedItems = [];
                this.states = [];
            },
        },

        computed: {
            floatArr() {
                if (this.arrayNotEmpty(this.testArr)) {
                    const tempArr = [];
                    const inner = arr => {
                        for (const item of arr) {
                            Array.isArray(item) ? inner(item) : tempArr.push(item);
                        }
                    };

                    inner(this.testArr);

                    return tempArr;
                }
            },

            divideArray() {
                const arr = this.floatArr;
                // const arr = [`someStirng`, `foo`, 1, 2, 3, `bar`, { someProperty: `test` }, null, {}];
                if (!arr || this.arrayNotEmpty(arr) === false) return;

                const tempObj = {};

                const validationItem = item => {
                    return item && (typeof item !== 'object' || (typeof item === 'object' && Object.keys(item).length > 0));
                };

                for (const item of arr) {
                    if (validationItem(item)) {
                        const currentType = tempObj[typeof item];
                        Array.isArray(currentType) ? currentType.push(item) : (tempObj[typeof item] = [item]);
                    }
                }

                return Object.values(tempObj);
            },
        },
    };
</script>

<style lang="scss">
    body {
        display: flex;
        justify-content: center;
        margin: 0;
        padding: 30px;

        background: #eee;
        color: #2c3e50;
        font-family: Avenir, Helvetica, Arial, sans-serif;
        overflow-y: scroll;

        #app {
            flex: 0 1 600px;
        }

        h3 {
            margin-top: 0;
        }

        button {
            padding: 8px 20px;
        }

        pre {
            padding: 10px;
            max-height: 200px;

            border: 1px solid #ddd;
            border-radius: 2px;
            overflow: auto;
        }

        * {
            box-sizing: border-box;
        }
    }

    .section {
        width: 100%;
        max-width: 600px;
        padding: 25px;
        margin: 0 0 20px;

        background: #ffffff;
        border: 1px solid #ddd;
        border-radius: 3px;
        text-align: left;

        &--sticky {
            position: sticky;
            top: 10px;
            box-shadow: 0 1px 5px rgba(0, 0, 0, 0.1);
        }

        &__row {
            display: flex;
            justify-content: flex-start;
            align-items: flex-start;
            flex-wrap: wrap;
            margin: 0 -10px;
        }

        &__col {
            display: flex;
            flex-direction: column;
            justify-content: flex-start;
            align-items: flex-start;
            flex: 1 0 calc(100% / 2 - 20px);
            margin: 0 10px 20px;
        }

        &__listItems {
            width: 100%;
            margin-top: 5px;
        }

        &__reset {
            margin-left: auto;
        }

        &__stateButtons {
            display: flex;
            justify-content: flex-start;
            align-items: flex-start;
            flex-wrap: wrap;
            margin: 0 7px;
            width: 100%;
        }

        &__stateButton {
            flex: 0 0 calc(100% / 5 - 6px);
            margin: 3px;
        }
    }
</style>
