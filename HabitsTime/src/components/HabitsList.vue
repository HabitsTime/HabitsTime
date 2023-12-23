<script lang="ts">
import Airtable, { Record } from 'airtable';
import PlusIcon from '@/assets/icons/PlusIcon.vue';
import DeleteIcon from '@/assets/icons/DeleteIcon.vue';
import * as relativeTime from 'dayjs/plugin/relativeTime';
// @ts-ignore
import dayjs from 'dayjs';
dayjs.extend(relativeTime);

const AIRTABLE_API_KEY = 'patMtyHTX78FSOzP8.cb5f2d6a899eed97709134d7d08347cafdaa013bf0923cfd3a8884f6c201b83e'
const AIRTABLE_BASE_ID = 'app1wUc5fIsAE3v2x'
const base = new Airtable({ apiKey: AIRTABLE_API_KEY }).base(AIRTABLE_BASE_ID);

type Habit = {
    id: string; // airtable generated id
    name: string;
    lastDone: dayjs.Dayjs;
};

export default {
    components: {
        PlusIcon: PlusIcon, DeleteIcon: DeleteIcon,
    },
    created() {
        this.fetchHabits();
    },
    data() {
        return {
            locallyChangedName: '',
            changed: false,
            habits: [] as Array<Habit>,
            doneLoading: false,
            longPressTimer: 0,
        };
    },
    methods: {
        pushToHabits(records: Array<Record<Habit>>) {
            records.forEach((record: Record<Habit>) => {
                const habit: Habit = {
                    id: record.id,
                    name: record.get('name'),
                    lastDone: dayjs(record.get('lastDone'))
                };
                this.habits.push(habit);
            });
            this.sortHabits();
        },
        // sort habits by last done from newest to oldest
        sortHabits() {
            this.habits.sort((a, b) => {
                if (a.lastDone.isBefore(b.lastDone)) {
                    return 1;
                } else if (a.lastDone.isAfter(b.lastDone)) {
                    return -1;
                } else {
                    return 0;
                }
            });
        },
        fetchHabits() {
            this.doneLoading = false;
            this.habits = [];
            base('Habits').select({
                maxRecords: 10,
                view: 'Grid view'
            }).eachPage((records: any, fetchNextPage: Function) => {
                this.pushToHabits(records);
                fetchNextPage();
            });
            this.doneLoading = true;
        },
        getLocalHabitById(id: string): Habit | undefined {
            return this.habits.find(habit => habit.id === id);
        },
        addHabitToAirtable() {
            // add a new habit to airtable
            base('Habits').create([
                {
                    // @ts-ignore
                    fields: {
                        name: 'New Habit',
                        lastDone: dayjs(),
                    }
                },
            ], (err: any, records: any) => {
                if (err) {
                    console.error(err);
                    return;
                }
                // add new habit locally
                this.pushToHabits(records);
            });
        },
        storeLocalChanges(event: any) {
            if (event.target.value === this.locallyChangedName)
                this.locallyChangedName = event.target.value;
            this.changed = true;
        },
        updateHabitToAirtable(id: string) {
            // get local habit
            const localHabit = this.getLocalHabitById(id);
            if (!this.changed || localHabit === undefined)
                return;
            this.changed = false;
            // update a habit in airtable
            base('Habits').update([
                {
                    id: localHabit.id,
                    // @ts-ignore
                    fields: {
                        name: localHabit.name,
                        lastDone: localHabit.lastDone,
                    }
                },
            ], (err: any, records: any) => {
                if (err) {
                    console.error(err);
                    return;
                }
            });
        },
        deleteHabit(id: string) {
            // delete a habit in airtable
            base('Habits').destroy([
                id,
            ], (err: any, deletedRecords: any) => {
                if (err) {
                    console.error(err);
                    return;
                }
                console.log('Deleted', deletedRecords.length, 'records');
            });
            // remove deleted habit locally
            this.habits = this.habits.filter(habit => habit.id !== id);
        },
        startLongPress(id: string) {
            this.longPressTimer = setTimeout(() => {
                let localHabit: Habit | undefined = this.getLocalHabitById(id);
                if (localHabit === undefined)
                    return;
                localHabit.lastDone = dayjs();
                this.changed = true;
                this.updateHabitToAirtable(id);
            }, 0.5 * 1000);
        },
        endLongPress() {
            clearTimeout(this.longPressTimer);
        },
    },
};
</script>

<template>
    <div class="d-flex justify-content-center">
        <div v-if="doneLoading" class="list-group">
            <div v-for="habit in habits">
                <a href="#" class="list-group-item list-group-item-action" aria-current="true"
                    @mousedown="startLongPress(habit.id)" @mouseup="endLongPress()" @mouseleave="endLongPress()">
                    <div class="d-flex w-100 justify-content-between">
                        <input class="name" type="text" v-model="habit.name" @change="storeLocalChanges"
                            @blur="updateHabitToAirtable(habit.id)" />
                        <small class="last-done">{{ habit.lastDone.fromNow() }}</small>
                        <button class="delete-button" @click="deleteHabit(habit.id)">
                            <DeleteIcon />
                        </button>
                    </div>
                </a>
            </div>
            <button class="add-button" @click="addHabitToAirtable()">
                <PlusIcon />
            </button>
        </div>
    </div>
</template>

<style scoped>
.list-group-item {
    margin: 0.5rem 0 0 0 !important;
    width: 400px;
    border-radius: 40px !important;

    @media(max-width: 750px) {
        margin: 0.5rem 1rem 0 1rem !important;
        width: 90vw;
    }
}

.add-button {
    height: 50px;
    width: 50px;
    border-radius: 50px;
    background-color: #d3d3d3;
    color: white;
    display: flex;
    align-items: center;
    justify-content: center;
    box-shadow: 0 2px 5px rgba(0, 0, 0, 0.2);
    border: none;
    cursor: pointer;
    padding: 0.7rem;

    margin-top: 0.5rem;
    margin-left: 180px;

    @media(max-width: 750px) {
        position: fixed;
        bottom: 20px;
        right: 20px;
    }
}

.add-button:active {
    box-shadow: 0 0 0 rgba(0, 0, 0, 0.2);
}

.delete-button {
    border: none;
    background-color: transparent;
    padding-top: -0.2rem;
}

.name {
    border: none;
    background-color: transparent;
}

.name:focus {
    outline: none;
}

.last-done {
    margin-top: 0.2rem;
}
</style>
