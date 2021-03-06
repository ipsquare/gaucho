<template>
    <section class="section settings-section" @contextmenu.stop="">
        <a class="back-button" @click="saveSettings">
            <span class="icon is-small">
                <i class="fas fa-arrow-left" />
            </span>
            Go Back
        </a>
        <div class="columns is-mobile is-centered">
            <div class="column is-two-thirds settings-menu">
                <h1 class="title settings-title">Settings</h1>
                <h3 class="settings-subtitle">Display Settings</h3>
                <checkbox-item v-model="bottomBar" label="Bottom bar"/>
                <checkbox-item v-model="showTimer" label="Show timer"/>
                <div class="level is-mobile">
                    <div class="level-left">
                        <p class="level-item">Theme</p>
                    </div>
                    <div class="level-right">
                        <div class="control">
                            <div class="select">
                                <select v-model="theme">
                                    <option value="light">Light</option>
                                    <option value="dark">Dark</option>
                                    <option value="classic">Classic</option>
                                </select>
                            </div>
                        </div>
                    </div>
                </div>
                <hr>
                <div class="columns is-mobile is-centered">
                    <div class="column is-two-thirds">
                        <button-item @select="resetConfig">Reset Settings</button-item>
                        <button-item @select="exportTasks">Export Tasks</button-item>
                        <button-item @select="importTasks" label="Warning: This will override your previous tasks">Import Tasks</button-item>
                        <button-item @select="clearTasks" label="Warning: This will remove all your suites and tasks">Clear Tasks</button-item>
                    </div>
                </div>
                <h3 class="settings-subtitle">Shortcuts</h3>
                <shortcuts-cheatsheet />
            </div>
        </div>
    </section>
</template>


<script>
"use strict";
const os = require('os');
const path = require('path');
const app = require('electron').remote;
const dialog = app.dialog;

const DeleteConfirmationAlert = require('../api/app_alerts').DeleteConfirmationAlert;

const components = {
    "checkbox-item": require('./settings/checkbox_item.vue'),
    "button-item": require('./settings/button_item.vue'),
    "shortcuts-cheatsheet": require('./settings/shortcuts_cheatsheet.vue')
};

module.exports = {
    data() {
        return {
            bottomBar: this.$store.state.userConfig.bottomBar,
            showTimer: this.$store.state.userConfig.showTimer,
            theme: this.$store.state.userConfig.theme
        };
    },
    components: components,
    watch: {
        theme() {
            this.$store.commit("setTheme", this.theme);
        }
    },
    methods: {
        saveSettings() {
            this.$store.commit("setBottomBar", this.bottomBar);
            this.$store.commit("setShowTimer", this.showTimer);
            this.$store.commit("setTheme", this.theme);

            this._close();
        },
        resetConfig() {
            this.bottomBar = true;
            this.showTimer = true;
        },
        importTasks() {
            dialog.showOpenDialog({
                filters: [{
                    name: 'json',
                    extensions: ['json']
                }]
            }, (filenames) => {
                if (filenames && filenames[0]) {
                    const filename = filenames[0];
                    const confirmationAlert = new DeleteConfirmationAlert("Importing tasks will remove all current tasks.", {
                        confirmButtonText: "Yes, import tasks",
                        cancelButtonText: "No, cancel import"
                    });
                    confirmationAlert.toggle().then(() => {
                        return this.$store.dispatch("importTasks", filename).then(() => {
                            this._close();
                        });
                    }).catch((err) => {
                        console.warn(err);
                    });
                }
            });
        },
        exportTasks() {
            dialog.showSaveDialog({
                defaultPath: path.join(os.homedir(), "gtask.json"),
                filters: [{
                    extensions: ['json']
                }]
            }, (filename) => {
                if (filename) {
                    this._close();
                    this.$store.dispatch("exportTasks", filename).catch((err) => {
                        console.warn(err);
                    });
                }
            });
        },
        clearTasks() {
            const confirmationAlert = new DeleteConfirmationAlert("You will not be able to recover these tasks after deletion!", {
                confirmButtonText: "Yes, clear them!",
                cancelButtonText: "No, keep them"
            });
            confirmationAlert.toggle().then(() => {
                return this.$store.dispatch("clearTasks").then(() => {
                    this.$store.commit("toggleActiveSuite", 0);
                    this._close();
                });
            }, () => {});
        },
        _close() {
            this.$store.commit("toggleSettings");
        }
    }
};

</script>


<style lang="scss" scoped>
.settings-menu{
    max-width: 650px;
}
.back-button{
    .icon{
        vertical-align: text-top;
    }
}

.settings-section{
    padding-top:30px;
}

.settings-title{
    margin-top: 15px;
}

.settings-subtitle {
    font-size: 15px;
    color: #9e9e9e;
    margin-top: 25px;
}
</style>
