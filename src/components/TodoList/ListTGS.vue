<template>
 <v-main class="list">
     <h3 class="text-h3 font-weight-medium mb-5">To Do List TGS</h3>

    <v-card>
        <v-card-title>
            <v-text-field
                v-model="search"
                append-icon="mdi-magnify"
                label="Search"
                single-line
                hide-details
            ></v-text-field>
            <v-spacer></v-spacer>

            <v-btn class="mr-2" color="purple" dark @click="finishedItem = true">
                TODO SELESAI
            </v-btn>
            
            <v-btn class="mr-2" color="success" dark @click="dialog = true">
                Tambah
            </v-btn>
        </v-card-title>

        <v-data-table :headers="headers" :items="todos" :search="search">
            <template v-slot:[`item.actions`]="{ item }">
                
                <v-icon small class="pencil mr-2" 
                        @click="editItem(item)"
                        color="blue">  {{ icons.mdiPencil }} </v-icon>
                <v-icon small class="bin mr-2" 
                        @click="deleteItem(item)"
                        color="red">   {{ icons.mdiDelete }}</v-icon>
            </template>
            
            <template v-slot:[`item.checkbox`]="{ item }">
                    <v-checkbox v-model="item.isSelected"></v-checkbox>
            </template>

            <template v-slot:[`item.priority`]="{ item } ">
                        <v-chip v-if="item.priority == 'Penting'"  color="red" label outlined>
                            {{ item.priority }}
                        </v-chip>
                        <v-chip v-if="item.priority == 'Biasa'"  color="blue" label outlined>
                            {{ item.priority }}
                        </v-chip>
                        <v-chip v-if="item.priority == 'Tidak penting'" color="green" label outlined>
                            {{ item.priority }}
                        </v-chip>
            </template>

            
        </v-data-table>
    </v-card>

    <v-card v-if="todos.filter((todo) => todo.isSelected).length > 0" class="mt-5">
            <v-card-title>
                <span class="font-weight-bold">Delete Multiple</span>
            </v-card-title>
            <v-card-text>
                <span class="font-weight-bold ml-1">Todo terpilih:</span>
                <ul v-for="(todo, index) in todos.filter((todo) => todo.isSelected)" :key="index">
                    <li>{{ todo.task }}</li>
                </ul>
                <v-btn class="mt-6" color="red" dark @click="deleteChecklist = true">
                    Hapus Semua
                </v-btn>
            </v-card-text>
        </v-card>

    <v-dialog v-model="deleteChecklist" persistent max-width="450px">
            <v-card>
                <v-card-title>
                    <span class="headline font-weight-bold"
                        >Yakin ingin menghapus
                        {{ todos.filter((todo) => todo.isSelected).length }} todo?</span
                    >
                </v-card-title>
                <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-spacer></v-spacer>
                    <v-btn color="green darken-1" text @click="deleteChecklist = false">
                        Tidak
                    </v-btn>
                    <v-btn color="red darken-1" text @click="confirmDeleteChecklist">
                        Ya
                    </v-btn>
                </v-card-actions>
            </v-card>
        </v-dialog>


    <v-dialog v-model="dialog" persistent max-width="600px">
        <v-card>
            <v-card-title>
                    <span class="headline" 
                        v-if="tambah == true" >
                           Form Todo - Add
                    </span>
                    <span class="headline" 
                        v-else>
                           Form Todo - Edit
                    </span>
                </v-card-title>

            <v-card-text>
                <v-container>
                    <v-text-field
                         v-model="formTodo.task"
                         label="Task"
                         required
                    ></v-text-field>

                    <v-select
                         v-model="formTodo.priority"
                         :items="['Penting', 'Biasa', 'Tidak penting']"
                         label="Priority"
                         required
                    ></v-select>

                    <v-textarea
                         v-model="formTodo.note"
                         label="Note"
                         required
                    ></v-textarea>
                </v-container>
            </v-card-text>

            <v-card-actions>
            <v-spacer></v-spacer>
                <v-btn color="blue darken-1" 
                    text @click="cancel">
                        Cancel
                </v-btn>

                

                <v-btn v-if="tambah == true" 
                        color="blue darken-1" 
                        text 
                        @click="save">
                        Save
                </v-btn>

                <v-btn v-else 
                        color="blue darken-1" 
                        text 
                        @click="edit(formTodo)">
                        Save
                </v-btn>
            </v-card-actions>
        </v-card>
    </v-dialog>

    <v-dialog v-model="finishedItem" persistent max-width="1000px">
            <v-card>
                <v-card-title>
                    <span class="headline font-weight-bold">Finished To do</span>
                </v-card-title>
                <v-card-text>
                    <v-container>
                        <v-data-table :headers="headersFinished" :items="finishedTodos">
                            <template v-slot:[`item.priority`]="{ item } ">
                                        <v-chip v-if="item.priority == 'Penting'"  color="red" label outlined>
                                            {{ item.priority }}
                                        </v-chip>
                                        <v-chip v-if="item.priority == 'Biasa'"  color="blue" label outlined>
                                            {{ item.priority }}
                                        </v-chip>
                                        <v-chip v-if="item.priority == 'Tidak penting'" color="green" label outlined>
                                            {{ item.priority }}
                                        </v-chip>
                            </template>
                        </v-data-table>
                    </v-container>
                </v-card-text>
                <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn color="blue darken-1" text @click="finishedItem = false">
                        Close
                    </v-btn>
                </v-card-actions>
            </v-card>
        </v-dialog>

        <v-dialog v-model="deleteitem" persistent max-width="400px">
            <v-card>
                <v-card-title>
                    <span class="headline">
                        Yakin ingin menghapus?
                    </span>
                </v-card-title>

                <v-card-actions>
                    <v-spacer></v-spacer>
                    
                    <v-btn color="green darken-1" text @click="cancel">
                        Tidak
                    </v-btn>

                    <v-btn color="red darken-1" text @click="confirmdelete">
                        Ya
                    </v-btn>

                </v-card-actions>

            </v-card>
        </v-dialog>

    

 </v-main>
</template>


<script>
export default {
    name: "List",
data() {
     return {
     search: null,
     tambah: true,
     edititem: -1,
     dialog: false,
     deleteitem: false,
     deleteChecklist: false,
     finishedItem: false,
     finishedTodos: [],
     
     headers: [
         {
         text: "Task",
         align: "start",
         sortable: true,
         value: "task",
         },
         { text: "Priority", value: "priority" },
         { text: "Note", value: "note" },
         { text: "Actions", value: "actions" },
         { text: "", value: "checkbox" },
     ],

     headersFinished: [
         {
         text: "Task",
         align: "start",
         sortable: true,
         value: "task",
         },
         { text: "Priority", value: "priority" },
         { text: "Note", value: "note" },
     ],

     filters: {
                search: '',
                priority: '',
            },

     todos: [
         {
         task: "bernafas",
         priority: "Penting",
         note: "huffttt",
         isSelected: false,
         },
         {
         task: "nongkrong",
         priority: "Tidak penting",
         note: "bersama tman2",
         isSelected: false,
         },
         {
         task: "masak",
         priority: "Biasa",
         note: "masak air 500ml",
         isSelected: false,
         },
     ],
     formTodo: {
             task: null,
             priority: null,
             note: null,
         },

     icons: {
                mdiPencil,
                mdiDelete,
                mdiTextBoxSearchOutline,
            },

    detail: {
                task: null,
                priority: null,
                note: null,
            }
     };
},
methods: {
    save() {
         this.todos.push(this.formTodo);
         this.resetForm();
         this.dialog = false;
        },

    cancel() {
         this.resetForm();
         this.dialog = false;
         this.edititem = -1;
         this.tambah = true;
         this.deleteitem = false;
        },

    deleteItem(item) {
         this.deleteitem = true;

         this.edititem = this.todos.indexOf(item);
        },

    confirmdelete() {
         this.finishedTodos.push(this.todos[this.edititem])
         this.todos.splice((this.edititem), 1);
         this.deleteitem = false;
         this.edititem = -1;
         
        },

    editItem(item) {
         this.tambah = false;
         this.formTodo = {
             task: item.task,
             priority: item.priority,
             note: item.note,
             isSelected: item.isSelected,
            };
            this.dialog = true;
            this.edititem = item;
        },
   

    edit(formTodo) {
         this.edititem.task = formTodo.task;
         this.edititem.priority = formTodo.priority;
         this.edititem.note = formTodo.note;
         this.cancel();
        },

    resetForm() {
         this.formTodo = {
         task: null,
         priority: null,
         note: null,
         isSelected: false,
        };
     },
     confirmDeleteChecklist() {
            this.finishedTodos.push(...this.todos.filter((todo) => todo.isSelected));
            this.todos = this.todos.filter((todo) => !todo.isSelected);
            this.deleteChecklist = false;
        },
   },
};

import {
    mdiTextBoxSearchOutline,
    mdiPencil,
    mdiDelete,
} from '@mdi/js'
</script>


