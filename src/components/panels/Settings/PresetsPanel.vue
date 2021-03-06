
<template>
    <div>
        <v-card>
            <v-toolbar flat dense >
                <v-toolbar-title>
                    <span class="subheading"><v-icon left>mdi-fire</v-icon>{{ $t('Settings.PresetsPanel.PreheatPresets') }}</span>
                </v-toolbar-title>
            </v-toolbar>
            <v-card-text class="py-3">
                <v-container>
                    <v-row v-for="(preset, index) in this['gui/getPreheatPresets']" v-bind:key="index">
                        <v-col class="rounded transition-swing secondary py-2 px-2 mb-3" style="cursor: pointer;">
                            <v-row align="center">
                                <v-col class="pl-6">
                                    <strong>{{ preset.name }}</strong>
                                </v-col>
                                <v-col class="col-6">
                                    <div v-for="[key, value] in Object.entries(preset.values)" v-bind:key="key">
                                        <span class="text-no-wrap" v-if="value.bool" >{{ convertPresetName(key, value) }}: {{ value.value }}°C</span>
                                    </div>
                                </v-col>
                                <v-col class="col-auto text-right"><v-btn small class="minwidth-0 float-right" v-on:click.stop.prevent="editPreset(preset)"><v-icon small>mdi-pencil</v-icon></v-btn></v-col>
                            </v-row>
                        </v-col>
                    </v-row>
                    <v-row>
                        <v-col class="rounded transition-swing secondary py-2 px-2 mb-2" style="cursor: pointer;">
                            <v-row align="center">
                                <v-col class="pl-6">
                                    <strong>{{ $t('Settings.PresetsPanel.Cooldown')}}</strong>
                                </v-col>
                                <v-col class="col-auto text-right"><v-btn small class="minwidth-0 float-right" v-on:click.stop.prevent="editCooldown"><v-icon small>mdi-pencil</v-icon></v-btn></v-col>
                            </v-row>
                        </v-col>
                    </v-row>
                    <v-row>
                        <v-col class="text-center mt-0">
                            <v-btn @click="createPreset">{{ $t('Settings.PresetsPanel.AddPreset')}}</v-btn>
                        </v-col>
                    </v-row>
                </v-container>
            </v-card-text>
        </v-card>
        <v-dialog v-model="dialog.bool" persistent :width="600">
            <v-card dark>
                <v-toolbar flat dense color="primary">
                    <v-toolbar-title>
                    <span class="subheading">
                        <v-icon class="mdi mdi-fire" left></v-icon>
                        {{ dialog.index === null ? $t('Settings.PresetsPanel.Create') : $t('Settings.PresetsPanel.Edit') }} {{ $t('Settings.PresetsPanel.Preset') }}
                    </span>
                    </v-toolbar-title>
                    <v-spacer></v-spacer>
                    <v-btn small class="minwidth-0" @click="dialog.bool = false"><v-icon small>mdi-close-thick</v-icon></v-btn>
                </v-toolbar>
                <v-card-text class="pt-3">
                    <v-container class="pb-0">
                        <v-form
                            v-model="dialog.valid"
                            @submit.prevent="savePreset"
                        >
                            <template v-if="dialog.bool">
                                <v-row>
                                    <v-col class="col-12 col-sm-6">
                                        <v-row>
                                            <v-col class="col-12">
                                                <v-text-field
                                                    v-model="dialog.name"
                                                    :label="$t('Settings.PresetsPanel.Name')"
                                                    hide-details="auto"
                                                    :rules="[rules.required, rules.unique]"
                                                    dense
                                                ></v-text-field>
                                            </v-col>
                                        </v-row>
                                        <v-row class="mt-2 mx-0 mb-2" v-for="(heater) of this['printer/getHeaters']" v-bind:key="heater.name" align="center">
                                            <v-checkbox
                                                v-model="dialog.values[heater.name].bool"
                                                hide-details
                                                class="shrink mt-0"
                                            ></v-checkbox>
                                            <v-text-field
                                                v-model="dialog.values[heater.name].value"
                                                :label="convertName(heater.name)"
                                                hide-details="auto"
                                                type="number"
                                                suffix="°C"
                                            ></v-text-field>
                                        </v-row>
                                        <v-row class="mt-2 mx-0 mb-2" v-for="(fan) of this['printer/getTemperatureFans']" v-bind:key="'temperature_fan '+fan.name" align="center">
                                            <v-checkbox
                                                v-model="dialog.values['temperature_fan '+fan.name].bool"
                                                hide-details
                                                class="shrink mt-0"
                                            ></v-checkbox>
                                            <v-text-field
                                                v-model="dialog.values['temperature_fan '+fan.name].value"
                                                :label="convertName(fan.name)"
                                                hide-details="auto"
                                                type="number"
                                                suffix="°C"
                                            ></v-text-field>
                                        </v-row>
                                    </v-col>
                                    <v-col class="col-12 col-sm-6">
                                        <v-textarea
                                            outlined
                                            name="input-7-4"
                                            :label="$t('Settings.PresetsPanel.CustomGCode')"
                                            v-model="dialog.gcode"
                                            hide-details="auto"
                                        ></v-textarea>
                                    </v-col>
                                </v-row>
                                <v-row class="mt-3" v-if="dialog.boolInvalidMin">
                                    <v-col class="py-0">
                                        <v-alert dense text type="error">{{ $t('Settings.PresetsPanel.PresetInfo')}}</v-alert>
                                    </v-col>
                                </v-row>
                                <v-row class="mt-3">
                                    <v-col class="text-center">
                                        <v-btn
                                            color="red"
                                            outlined
                                            class="float-left minwidth-0"
                                            @click="deletePreset"
                                            v-if="this.dialog.index !== null"
                                        >
                                            <v-icon>mdi-delete</v-icon>
                                        </v-btn>
                                        <v-btn
                                            color="white"
                                            outlined
                                            :class="dialog.index !== null ? 'float-right' : '' "
                                            type="submit"
                                        >
                                            {{ dialog.index === null ? $t('Settings.PresetsPanel.Store') : $t('Settings.PresetsPanel.Update') }} {{ $t('Settings.PresetsPanel.Preset') }}
                                        </v-btn>
                                    </v-col>
                                </v-row>
                            </template>
                        </v-form>
                    </v-container>
                </v-card-text>
            </v-card>
        </v-dialog>
        <v-dialog v-model="cooldownDialog.bool" persistent :width="300">
            <v-card dark>
                <v-toolbar flat dense color="primary">
                    <v-toolbar-title>
                    <span class="subheading">
                        <v-icon class="mdi mdi-fire" left></v-icon> {{ $t('Settings.PresetsPanel.EditCooldown')}}
                    </span>
                    </v-toolbar-title>
                    <v-spacer></v-spacer>
                    <v-btn small class="minwidth-0" @click="cooldownDialog.bool = false"><v-icon small>mdi-close-thick</v-icon></v-btn>
                </v-toolbar>
                <v-card-text class="pt-3">
                    <v-container class="pb-0">
                        <v-form
                            v-model="cooldownDialog.valid"
                            @submit.prevent="saveCooldown"
                        >
                            <v-row>
                                <v-col class="col-12">
                                    <v-textarea
                                        outlined
                                        name="input-7-4"
                                        :label="$t('Settings.PresetsPanel.CustomGCode')"
                                        v-model="cooldownDialog.gcode"
                                        :rules="[rules.required]"
                                        hide-details="auto"
                                    ></v-textarea>
                                </v-col>
                            </v-row>
                            <v-row>
                                <v-col class="text-center">
                                    <v-btn
                                        color="white"
                                        outlined
                                        type="submit"
                                    >
                                        {{ $t('Settings.PresetsPanel.UpdateCooldown')}}
                                    </v-btn>
                                </v-col>
                            </v-row>
                        </v-form>
                    </v-container>
                </v-card-text>
            </v-card>
        </v-dialog>
    </div>
</template>

<script>
    import { mapState, mapGetters } from 'vuex';
    import {convertName} from "@/plugins/helpers";

    export default {
        components: {

        },
        data: function() {
            return {
                dialog: {
                    bool: false,
                    valid: false,
                    name: "",
                    gcode: "",
                    index: null,
                    boolInvalidMin: false,
                    values: {},
                },
                rules: {
                    required: (value) => value !== '' || 'required',
                    unique: (value) => !this.existsPresetName(value) || 'Name already exists',
                },
                cooldownDialog: {
                    bool: false,
                    gcode: "",
                    valid: false,
                }
            }
        },
        computed: {
            ...mapState({
                orgState: state => state.gui.presets,
                cooldownGcode: state => state.gui.cooldown_gcode,
            }),
            ...mapGetters([
                'printer/getHeaters',
                'printer/getTemperatureFans',
                'gui/getPreheatPresets',
            ])
        },
        mounted() {
            this.clearDialog()
        },
        methods: {
            convertName: convertName,
            convertPresetName(name, value) {
                if (value.type === "temperature_fan") name = name.replace("temperature_fan ", "")

                return this.convertName(name)
            },
            existsPresetName(name) {
                return (this["gui/getPreheatPresets"].findIndex((preset) => preset.name === name && preset.index !== this.dialog.index) >= 0)
            },
            createPreset() {
                this.clearDialog()
                this.dialog.bool = true
            },
            clearDialog() {
                this.dialog.bool = false
                this.dialog.index = null
                this.dialog.name = ""
                this.dialog.gcode = ""
                this.dialog.boolInvalidMin = false
                this.dialog.values = {}

                for(const heater of this["printer/getHeaters"]) {
                    this.dialog.values[heater.name] = {
                        bool: true,
                        value: 0,
                        type: 'heater',
                    }
                }

                for(const fan of this["printer/getTemperatureFans"]) {
                    this.dialog.values['temperature_fan '+fan.name] = {
                        bool: true,
                        value: 0,
                        type: 'temperature_fan',
                    }
                }
            },
            editPreset(preset) {
                this.dialog.name = preset.name
                this.dialog.index = preset.index
                this.dialog.gcode = preset.gcode
                this.dialog.values = {}

                for(const heater of this["printer/getHeaters"]) {
                    if (heater.name in preset.values) {
                        this.dialog.values[heater.name] = {...preset.values[heater.name]}
                    } else {
                        this.dialog.values[heater.name] = {
                            bool: false,
                            value: 0,
                            type: 'heater',
                        }
                    }
                }

                for(const fan of this["printer/getTemperatureFans"]) {
                    if ('temperature_fan '+fan.name in preset.values) {
                        this.dialog.values['temperature_fan '+fan.name] = {...preset.values['temperature_fan '+fan.name]}
                    } else {
                        this.dialog.values['temperature_fan '+fan.name] = {
                            bool: false,
                            value: 0,
                            type: 'temperature_fan',
                        }
                    }
                }

                this.dialog.bool = true
            },
            savePreset() {
                let setValues = 0
                for (const key of Object.keys(this.dialog.values)) {
                    if (this.dialog.values[key].bool) setValues++
                }
                if (this.dialog.gcode.length) setValues++

                if (setValues === 0) this.dialog.boolInvalidMin = true
                else if (this.dialog.valid) {
                    for (const key of Object.keys(this.dialog.values)) {
                        this.dialog.values[key].value = parseInt(this.dialog.values[key].value)
                    }

                    if (this.dialog.index) {
                        this.$store.dispatch('gui/updatePreset',  this.dialog )
                    } else this.$store.dispatch('gui/addPreset',  this.dialog )

                    this.clearDialog()
                }
            },
            deletePreset() {
                this.$store.dispatch('gui/deletePreset',  this.dialog )
                this.clearDialog()
            },
            editCooldown() {
                this.cooldownDialog.gcode = this.cooldownGcode
                this.cooldownDialog.bool = true
            },
            saveCooldown() {
                if (this.cooldownDialog.valid) {
                    this.$store.dispatch("gui/setSettings", { cooldown_gcode: this.cooldownDialog.gcode })
                    this.cooldownDialog.bool = false
                }
            },
        }
    }
</script>