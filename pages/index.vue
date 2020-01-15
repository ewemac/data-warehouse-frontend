<template>
    <v-layout
            column
            justify-center
            align-left
    >
        <v-flex
                xs12
                sm8
                md6
        >
            <v-row>
                <v-col cols="12" md="12">
                    <v-card>
                        <v-card-title class="headline text-center">
                            ETL PROCESS
                        </v-card-title>
                        <v-card-text>
                            <form @submit="addCity">
                                <v-text-field
                                        label="Add new city weather to database"
                                        placeholder="city"
                                        outlined
                                        v-model="city"
                                ></v-text-field>
                                <p>ETL actions</p>
                                <v-btn
                                        color="primary"
                                        nuxt
                                        :disabled="isEprocessButtonDisabled"
                                        type="submit"
                                >
                                    Submit ETL process
                                </v-btn>
                            </form>
                            <div class="buttons-wrapper">
                                <v-btn
                                        color="primary"
                                        nuxt
                                        :disabled="isEprocessButtonDisabled"
                                        @click="eProcess"
                                        type="submit"
                                >
                                    Submit E process
                                </v-btn>
                                <v-btn
                                        color="primary"
                                        nuxt
                                        :disabled="isTProcessButtonDisabled"
                                        @click="tProcess"
                                        type="submit"
                                >
                                    Submit T process
                                </v-btn>
                                <v-btn
                                        color="primary"
                                        nuxt
                                        :disabled="isLProcessButtonDisabled"
                                        @click="addCity"
                                        type="submit"
                                >
                                    Submit L process
                                </v-btn>
                            </div>
                        </v-card-text>
                        <v-card-text>
                            <p>Aditional actions</p>
                            <v-btn
                                    color="primary"
                                    nuxt
                                    @click="loadDatabase"
                            >
                                Show database
                            </v-btn>
                            <v-btn
                                    color="error"
                                    nuxt
                                    @click="resetDatabase"
                            >
                                Reset database
                            </v-btn>
                        </v-card-text>
                    </v-card>
                </v-col>
                <v-col cols="12" md="12" lg="12"
                       v-show="weather.length">
                    <v-data-table
                            :headers="headers"
                            :items="weather"
                            :items-per-page="10"
                            class="elevation-1"
                    ></v-data-table>
                    <v-btn  class="json-button"
                            color="primary"
                            nuxt
                            @click="exportJson"
                    >
                        JSON FILE EXPORT
                    </v-btn>
                </v-col>
                <v-col cols="12" md="6">
                    <v-textarea
                            class="textarea"
                            v-show="eProcessData"
                            :value="JSON.stringify(eProcessData, undefined, 4)"
                            label="E process Data"
                    ></v-textarea>
                </v-col>
                <v-col cols="12" md="6">
                    <v-textarea
                            class="textarea"
                            v-show="tProcessData"
                            :value="JSON.stringify(tProcessData, undefined, 4)"
                            label="T process Data"
                    ></v-textarea>
                </v-col>

            </v-row>

        </v-flex>
        <v-alert type="success" v-show="messages.success">
            {{ messages.success }}
        </v-alert>

        <v-alert type="error" v-show="messages.error">
            {{ messages.error }}
        </v-alert>
        <a id="downloadAnchorElem" style="display:none"></a>
    </v-layout>
</template>

<script>
    import axios from 'axios';

    export default {
        data() {
            return {
                apiUrl: 'http://localhost:3000/weather',
                weather: [],
                city: '',
                eProcessData: null,
                tProcessData: null,
                messages: {
                    error: false,
                    success: false
                },
                messagesText: {
                    error: 'Something went wrong, try again later',
                    inputError: 'You must fill the city input before submit',
                    databaseReset: 'Database was reset successfully',
                    databaseLoaded: 'Database was loaded successfully',
                    databaseEmpty: 'Database is empty, add something first',
                    transformProcess: 'The process Transform was successful',
                    extractProcess: 'The process Extract was successful'
                },
                timeout: null,
                headers: [
                    {
                        text: 'Id',
                        value: 'id'
                    },
                    {
                        text: 'City',
                        value: 'city'
                    },
                    {
                        text: 'Lon',
                        value: 'lon'
                    },
                    {
                        text: 'Lat',
                        value: 'lat'
                    },
                    {
                        text: 'Country',
                        value: 'country'
                    },
                    {
                        text: 'Clouds',
                        value: 'clouds'
                    },
                    {
                        text: 'Temp',
                        value: 'temp'
                    },
                    {
                        text: 'Visibility',
                        value: 'visibility'
                    },
                    {
                        text: 'Wind',
                        value: 'wind'
                    }
                ]
            }
        },
        computed: {
            isEprocessButtonDisabled() {
                return !!(this.tProcessData || this.eProcessData);
            },
            isTProcessButtonDisabled() {
                return !!(!this.eProcessData || this.tProcessData);
            },
            isLProcessButtonDisabled() {
                return !!(!this.tProcessData);
            }
        },
        methods: {
            clearProcessData() {
                this.tProcessData = null;
                this.eProcessData = null;
            },
            async loadDatabase(refresh) {
                try {
                    const weather = await axios.get(this.apiUrl);
                    this.weather = weather.data;
                    if (weather.data.length) {
                        if (refresh !== true) {
                            this.showMessage('success', this.messagesText.databaseLoaded);
                        }
                    } else {
                        this.showMessage('error', this.messagesText.databaseEmpty);
                    }
                } catch (error) {
                    this.showMessage('error', this.messagesText.error);
                }
            },
            async eProcess() {
                if (this.city !== '') {
                    try {
                        const eProcessData = await axios.get(this.apiUrl, {params: {city: this.city}});

                        if (eProcessData.data.message && eProcessData.data.message.errors || eProcessData.data.error) {
                            this.showMessage('error', eProcessData.data.message._message || eProcessData.data.message);
                        } else {
                            this.showMessage('success', this.messagesText.extractProcess);
                            this.eProcessData = eProcessData;
                        }
                    } catch (error) {
                        this.showMessage('error', this.messagesText.error);
                    }
                } else {
                    this.showMessage('error', this.messagesText.inputError);
                }
            },
            async tProcess() {
                try {
                    const tProcessData = await axios.get(this.apiUrl, {params: {object: this.eProcessData}});
                    this.tProcessData = tProcessData.data;
                    this.showMessage('success', this.messagesText.transformProcess);
                } catch (error) {
                    this.showMessage('error', this.messagesText.error);
                }
            },
            async resetDatabase() {
                try {
                    await axios.delete(this.apiUrl);
                    this.weather = [];
                    this.showMessage('success', this.messagesText.databaseReset);
                } catch (error) {
                    this.showMessage('error', this.messagesText.error);
                }

            },
            async addCity(e) {
                e.preventDefault();
                if (this.city !== '') {
                    try {
                        const weather = await axios.post(this.apiUrl, {
                            city: this.city
                        });

                        if (weather.data.message && weather.data.message.errors || weather.data.error) {
                            this.showMessage('error', weather.data.message._message || weather.data.message);
                        } else {
                            this.showMessage('success', weather.data.message);
                            this.loadDatabase(true);
                            this.clearProcessData();

                        }
                    } catch (error) {
                        this.showMessage('error', this.messagesText.error);
                    }
                } else {
                    this.showMessage('error', this.messagesText.inputError);
                }
            },
            exportJson() {
                const dataStr = "data:text/json;charset=utf-8," + encodeURIComponent(JSON.stringify(this.weather));
                const dlAnchorElem = document.getElementById('downloadAnchorElem');
                dlAnchorElem.setAttribute("href", dataStr);
                dlAnchorElem.setAttribute("download", "database.json");
                dlAnchorElem.click();
            },
            showMessage(type, message) {
                clearTimeout(this.timeout);
                this.messages[type] = message;
                this.timeout = setTimeout(() => {
                    this.messages['success'] = false;
                    this.messages['error'] = false;
                }, 5000);
            }
        }
    }
</script>

<style lang="scss">
    .my-column {
        flex-grow: unset !important;

        & > div {
            min-width: 200px;
        }
    }

    .buttons-wrapper {
        margin-top: 24px;
    }

    .textarea textarea {
        min-height: 500px;
    }

    .json-button {
        float: right;
        margin-top: 24px;
    }
</style>
