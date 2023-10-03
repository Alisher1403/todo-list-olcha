<template>
    <div class="todo">
        <div class="todo-title-wrapper">
            <input class="todo-title" type="text" placeholder="Title" v-if="titleEdit" v-model="title">
            <div class="todo-title show unactive" v-if="!titleEdit && title.length == 0">Title</div>
            <div class="todo-title show" v-if="!titleEdit && title.length !== 0" v-html="title"></div>
            <button class="todo-title-button" @click="titleEdit = !titleEdit">{{ titleEdit ? 'Done' : 'Edit' }}</button>
        </div>
        <div class="todo-add flex">
            <input class="add-input" type="text" v-model="input">
            <button class="add-button" @click="addItem()">Add</button>
        </div>
        <div class="search-wrapper">
            <span>search</span>
            <input class="search-input" type="text" v-model="searchInput" @input="filterList()">
        </div>
        <ul class="todo-list">
            <li class="todo-list-item" :class="{ done: element.done }" v-for="(element, index) in searchList"
                :key="element.id">
                <input class="element-done" type="checkbox" v-model="element.done">
                <strong class="element-index">{{ index + 1 }}</strong>
                <p class="element-text" v-if="editingItemId !== element.id">{{ element.text }}</p>
                <input class="element-text" type="text" v-if="editingItemId === element.id" v-model="editingItem.text">
                <div>
                    <div class="flex" v-if="editingItemId !== element.id">
                        <button class="element-button" @click="editClick(element)">Edit</button>
                        <button class="element-button" @click="deleteItem(element.id)">Delete</button>
                    </div>

                    <div class="flex" v-if="editingItemId === element.id">
                        <button class="element-button" @click="updateItem(element)">Save</button>
                        <button class="element-button" @click="cancelEdit()">Cancel</button>
                    </div>
                </div>
            </li>
        </ul>
        <div class="flex">
            <button @click="importList()">Import</button>
            <button @click="exportList()">Export</button>
        </div>
    </div>
</template>

<script lang="ts">
import { Vue } from 'vue-class-component';
import { Watch } from 'vue-property-decorator';
// Please press Allow in clipboard permission, it's important for importing data

interface iTodoItem {
    id: number,
    text: string,
    done: boolean,
}

export default class TodoList extends Vue {
    public title: string = '';
    public titleEdit: boolean = false;
    public searchInput: string = '';
    public searchList: any = [];
    public input: string = '';
    public list: iTodoItem[] = [];
    public editingItemId: number = 0;
    public editingItem: iTodoItem = { id: 0, text: '', done: false };
    public importFile: any = null;

    public addItem(): void {
        this.list.push({ id: Date.now(), text: this.input, done: false })
        this.input = ''
    }

    public deleteItem(id: number): void {
        this.list = this.list.filter(el => el.id !== id);
    }

    public updateItem(element: iTodoItem): void {
        Object.assign(element, this.editingItem)
        this.editingItem = { id: 0, text: '', done: false };
        this.editingItemId = 0;
    }

    public cancelEdit(): void {
        this.editingItemId = 0;
        this.editingItem = { id: 0, text: '', done: false };
    }

    public editClick(element: iTodoItem) {
        this.editingItemId = element.id;
        this.editingItem = { ...element };
    }

    public filterList() {
        this.searchList = [...this.list].filter((item) => item['text'].toLowerCase().includes(this.searchInput.toLowerCase()))
    }

    public exportList() {
        const data = JSON.stringify({ title: this.title, list: this.list });
        navigator.clipboard.writeText(data)
            .then(() => {
                alert('Data copied to clipboard, press import in another project to import data')
            })
            .catch((error) => {
                console.error("Failed to copy data to clipboard:", error);
            });
    }

    public importList() {
        navigator.clipboard.readText()
            .then((clipboardData) => {
                const parsedData = JSON.parse(clipboardData);
                if (Array.isArray(parsedData.list)) {
                    this.list = parsedData.list;
                }
                if (parsedData.title) {
                    this.title = parsedData.title;
                    sessionStorage.todoTitle = JSON.stringify(parsedData.title);
                    alert(`Project ${parsedData.title} is imported successfully`);
                }
            })
            .catch((error) => {
                console.error("Error to get data:", error);
            });
    }

    mounted() {
        if (sessionStorage.todoList) {
            this.list = JSON.parse(sessionStorage.todoList);
        }
        if (sessionStorage.todoTitle) {
            this.title = JSON.parse(sessionStorage.todoTitle);
        }
        if (sessionStorage.search) {
            this.searchInput = JSON.parse(sessionStorage.search)
            this.filterList();
        }
    }

    @Watch('searchInput')
    onsearchChange() {
        sessionStorage.setItem('search', JSON.stringify(this.searchInput))
    }

    @Watch('titleEdit')
    onTitleChange(currentVal: boolean) {
        if (!currentVal) {
            sessionStorage.setItem('todoTitle', JSON.stringify(this.title))
        }
    }

    @Watch('list', { deep: true })
    onListChange() {
        sessionStorage.setItem('todoList', JSON.stringify(this.list))
        this.filterList();
    }
}

</script>

<style>
:root {
    --border: 2px solid rgb(0, 105, 63);
}

.todo {
    padding: 20px 0;
    margin: 0 auto;
    max-width: 800px;
    width: 100%;
}

.todo-title-wrapper {
    display: flex;
    margin-bottom: 30px;
}

.todo-title {
    font-size: 40px;
    color: rgb(58, 58, 58);
    margin: 0;
    outline: none;
    border: var(--border);
    border-width: 1px;
    width: 100%;
    min-height: 50px;
}

.todo-title.show {
    border-color: transparent;
    text-align: center;
}

.todo-title.unactive {
    color: rgb(171, 171, 171);
}

.todo-title::placeholder {
    color: rgb(210, 210, 210);
}

.todo-title-button {
    width: 70px;
}

.todo-add {
    column-gap: 5px;
}

.add-input {
    border: var(--border);
    width: 100%;
    border-radius: 5px;
    font-size: 16px;
    padding: 5px 10px;
}

.add-button {
    border: var(--border);
    background: none;
    border-radius: 5px;
    font-size: 16px;
    cursor: pointer;
    min-width: 70px;
}

.add-button:hover {
    background: rgba(0, 105, 63, 0.157);
}

.search-wrapper {
    display: flex;
    justify-content: flex-end;
    align-items: center;
    column-gap: 10px;
    margin-top: 20px;
}

.search-input {
    border: var(--border);
    border-radius: 5px;
    font-size: 16px;
    padding: 5px 10px;
    outline: none;
}

.todo-list {
    list-style: none;
    padding: 0;
}

.todo-list-item {
    display: grid;
    border: var(--border);
    align-items: center;
    grid-template-columns: 40px 40px auto 150px;
    margin: 10px 0;
    border-radius: 7px;
    transition: 0.2s;
}

.todo-list-item.done {
    background: rgb(211, 211, 211);
}

.element-done {
    cursor: pointer;
}

.element-text {
    text-align: start;
    margin: 0;
    padding: 5px 10px;
    margin: 10px 5px;
    outline: none;
    border: none;
    font-size: 16px;
    border: 1px solid transparent;
    background: transparent;
}

.element-text[type="text"] {
    border: 1px solid gray;
}

.element-button {
    border: var(--border);
    background: none;
    margin-right: 5px;
    cursor: pointer;
    border-radius: 30px;
    padding: 5px 10px;
    width: 100%;
}

.element-button:hover {
    background: rgba(0, 105, 63, 0.157);
}

.flex {
    display: flex;
}
</style>