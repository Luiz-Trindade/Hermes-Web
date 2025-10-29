<template>
    <v-navigation-drawer expand-on-hover permanent rail>
        <!-- Button to create a new conversation -->
        <div class="d-flex justify-center ma-4">
            <v-btn color="primary" @click="createNewConversation">
                <v-icon class="me-2">mdi-plus</v-icon>
                New Conversation
            </v-btn>
        </div>

        <v-divider></v-divider>

        <v-list density="compact" nav>
            <div v-for="key in filteredKeys" :key="key" class="d-flex align-center ma-2">
                <v-btn text @click="handleKeyClick(key)" class="flex-grow-1 mr-2" style="max-width: 120px;">
                    <v-icon class="me-2">mdi-message</v-icon>
                    {{ key.slice(0, 8) }}
                </v-btn>
                <v-btn icon color="blue" @click="openRenameDialog(key)">
                    <v-icon>mdi-pencil</v-icon>
                </v-btn>
                <v-btn icon color="red" @click="handleDeleteKey(key)">
                    <v-icon>mdi-delete</v-icon>
                </v-btn>
            </div>
        </v-list>

        <!-- Export and Import Buttons -->
        <div class="d-flex justify-center ma-2">
            <v-row>
                <v-col>
                    <v-btn color="green" @click="exportConversation">
                        <v-icon class="ma-2">mdi-export"></v-icon>
                        Export Conversation
                    </v-btn>
                    <v-btn color="orange" @click="showImportDialog = true">
                        <v-icon class="ma-2">mdi-import"></v-icon>
                        Import Conversation
                    </v-btn>
                </v-col>
            </v-row>
        </div>

        <!-- Button fixed at the bottom -->
        <div class="d-flex justify-center ma-4">
            <v-btn color="red" @click="showResetDialog = true">
                <v-icon class="me-2">mdi-delete-sweep</v-icon>
                Reset All
            </v-btn>
        </div>

        <!-- Confirmation dialog -->
        <v-dialog v-model="showResetDialog" max-width="400">
            <v-card>
                <v-card-title class="text-h5">Confirm Reset</v-card-title>
                <v-card-text>Are you sure you want to delete all keys? This action cannot be undone.</v-card-text>
                <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn text @click="showResetDialog = false">Cancel</v-btn>
                    <v-btn color="red" text @click="resetAllKeys">Confirm</v-btn>
                </v-card-actions>
            </v-card>
        </v-dialog>

        <!-- Import dialog -->
        <v-dialog v-model="showImportDialog" max-width="500">
            <v-card>
                <v-card-title class="text-h5">Import Conversation</v-card-title>
                <v-card-text>
                    <v-text-field v-model="importedKey" label="Conversation Name"></v-text-field>
                    <v-textarea v-model="importedValue" label="Conversation JSON"></v-textarea>
                </v-card-text>
                <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn text @click="showImportDialog = false">Cancel</v-btn>
                    <v-btn color="green" text @click="importConversation">Import</v-btn>
                </v-card-actions>
            </v-card>
        </v-dialog>

        <!-- Rename dialog -->
        <v-dialog v-model="showRenameDialog" max-width="400">
            <v-card>
                <v-card-title class="text-h5">Rename Conversation</v-card-title>
                <v-card-text>
                    <v-text-field v-model="newKeyName" label="New Name"></v-text-field>
                </v-card-text>
                <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn text @click="showRenameDialog = false">Cancel</v-btn>
                    <v-btn color="blue" text @click="renameConversation">Rename</v-btn>
                </v-card-actions>
            </v-card>
        </v-dialog>
    </v-navigation-drawer>
</template>

<script>
import localforage from 'localforage';

export default {
    data() {
        return {
            keys: [],
            showResetDialog: false,
            showImportDialog: false,
            showRenameDialog: false,
            importedKey: '',
            importedValue: '',
            keyToRename: '',
            newKeyName: ''
        };
    },
    computed: {
        filteredKeys() {
            return this.keys.filter(key => key !== 'actual_chat_history');
        }
    },
    mounted() {
        const listLocalForageKeys = async () => {
            const keys = [];
            await localforage.iterate((value, key) => {
                keys.push(key);
            });
            this.keys = keys;
        };

        listLocalForageKeys();
    },
    methods: {
        async handleKeyClick(key) {
            try {
                const value = await localforage.getItem(key);
                await localforage.setItem('actual_chat_history', value);
                window.location.reload();
            } catch (error) {
                console.error('Error updating the "actual_chat_history" key:', error);
            }
        },
        async handleDeleteKey(key) {
            try {
                await localforage.removeItem(key);
                this.keys = this.keys.filter(k => k !== key);
            } catch (error) {
                console.error(`Error removing the key "${key}":`, error);
            }
        },
        async createNewConversation() {
            try {
                await localforage.setItem('actual_chat_history', null);
                window.location.reload();
            } catch (error) {
                console.error('Error creating a new conversation:', error);
            }
        },
        async resetAllKeys() {
            try {
                await localforage.clear();
                this.keys = [];
                this.showResetDialog = false;
            } catch (error) {
                console.error('Error resetting all keys:', error);
            }
        },
        async exportConversation() {
            try {
                const value = await localforage.getItem('actual_chat_history');
                const blob = new Blob([JSON.stringify(value, null, 2)], { type: 'application/json' });
                const url = URL.createObjectURL(blob);
                const a = document.createElement('a');
                a.href = url;
                a.download = 'conversation.json';
                a.click();
                URL.revokeObjectURL(url);
            } catch (error) {
                console.error('Error exporting conversation:', error);
            }
        },
        async importConversation() {
            try {
                const parsedValue = JSON.parse(this.importedValue);
                await localforage.setItem(this.importedKey, parsedValue);
                this.keys.push(this.importedKey);
                this.showImportDialog = false;
                this.importedKey = '';
                this.importedValue = '';
            } catch (error) {
                console.error('Error importing conversation:', error);
            }
        },
        openRenameDialog(key) {
            this.keyToRename = key;
            this.newKeyName = key;
            this.showRenameDialog = true;
        },
        async renameConversation() {
            try {
                const value = await localforage.getItem(this.keyToRename);
                await localforage.setItem(this.newKeyName, value);
                await localforage.removeItem(this.keyToRename);
                this.keys = this.keys.map(key => (key === this.keyToRename ? this.newKeyName : key));
                this.showRenameDialog = false;
                this.keyToRename = '';
                this.newKeyName = '';
            } catch (error) {
                console.error('Error renaming conversation:', error);
            }
        }
    }
};
</script>
