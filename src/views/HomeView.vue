<template>
	<div class="home">
		<h2>Тестовое задание FrontEnd</h2>
		<div>
			
			<h3>1. Поле Link</h3>
			<div v-if="!editing">
				<div>
					<a :href="url" target="_blank">{{ title ? title : url }}</a>
				</div>
				<Button @click="startEditing()">Edit</Button>
			</div>
			<div v-else>
				<InputText type="text" v-model="editedURL"/>
				<Button @click="saveEdit()">Save</Button>
				<Button @click="cancelEdit()">Cancel</Button>
			</div>
		</div>
		<div>
			
			<h3>2. DateTime picker </h3>
			<div>
				<Calendar id="calendar-timeonly"
						  v-model="time"
						  timeOnly
						  format="HH:mm"/>
				<p v-if="time">{{ formattedTime() }}</p>
			</div>
		</div>
		
		<div>
			<h3>3. Сброс значений в Select</h3>
			<MultiSelect v-model="selectedItems"
						 :options="options"
						 optionLabel="name"
						 placeholder="Select Cities"
						 :maxSelectedLabels="3"
						 class="w-full md:w-20rem"/>
		</div>
		
		<div><h3>4. Многострочный текст в ячейке</h3>
			<div>
				<h3>Массив действий:</h3>
				
				<DataTable>
					<Column v-for="col of data" :key="col.time">{{col.time}}</Column>
				</DataTable>
				
			</div>
		</div>
	</div>
</template>

<script setup lang="ts">
import { Ref, ref } from "vue";
import InputText from "primevue/inputtext";
import axios from "axios";
import Button from "primevue/button";
import Calendar from "primevue/calendar";
import MultiSelect from "primevue/multiselect";
import DataTable from "primevue/datatable";
import Column from "primevue/datatable";
import { onMounted, computed } from "vue";

interface Action {
	text: string;
}

const actions = ref<Action[]>([]);
let url: Ref<string> = ref("");
let title: Ref<string> = ref("");
let editedURL: Ref<string> = ref("");
let editing: Ref<boolean> = ref(false);
let time: Ref<string> = ref("");
let selectedItems = ref([]);
const options = ref([
	{
		name: "New York",
		code: "NY"
	},
	{
		name: "Rome",
		code: "RM"
	},
	{
		name: "London",
		code: "LDN"
	},
	{
		name: "Istanbul",
		code: "IST"
	},
	{
		name: "Paris",
		code: "PRS"
	}
]);
const apiString = "[13:36:53] Расчетное время: 9 мин\n[13:36:58] Открыть клапан откачки К1\n[13:36:58] Включить вакуумный насос\n[13:36:58] Закрыть клапан К5\n[13:36:58] Закрыть клапан дистилляции К2\n[13:36:58] Ожидание: 8 с\n[13:37:06] Заливка 2.2мл. в испаритель\n[13:37:06] Заданно 26.50602409638554 шагов\n[13:37:09] Заливка перекиси завершена\n[13:37:09] Открыть клапан дистилляции К2\n[13:42:09] Включить нагрев испарителя\n[13:42:09] Закрыть клапан дистилляции К2\n[13:42:09] конечное давление1.0960040758227925 торр\n[13:42:09] Выпаривание через К2\n[13:42:09] Выпаривание длилось5 мин\n[13:42:09] Откачка до 1 торр\n[13:42:15] Закрыть клапан откачки К1\n[13:43:09] Открыть клапан откачки К1\n[13:43:14] Аппаратное смещение 0 денсит. = -0.313683180809021\n[13:43:14] Закрыть клапан дистилляции К2";


const splitString = () =>{
	const lines = apiString.split('\n');
	
	for(const line of lines) {
		const timestamp = line.substring(0, line.indexOf(']') + 1);
		const message = line.substring(line.indexOf(']') + 2);
		console.log(`${timestamp} ${message}`);
		return `${timestamp} ${message}`;
	}
}

const parseData = (input: string) => {
	const lines = input.split('\n');
	const data = lines.map((line) => {
		const values = line.split(' , ');
		return {
			time: values[0],
			action: values[1],
			step: values[2],
			message: values[3],
		};
	});
	return data;
}
const data = parseData(apiString)

const getTitle = () => {
	axios.get(url.value).then((response) => {
		const parser = new DOMParser();
		const htmlDoc = parser.parseFromString(response.data, "text/html");
		title.value = htmlDoc.getElementsByTagName("title")[0].innerText;
	}).catch(() => {
		title.value = "";
	});
};

const startEditing = () => {
	editing.value = true;
	editedURL = url;
};

const saveEdit = () => {
	url = editedURL;
	editing.value = false;
	getTitle();
};

const cancelEdit = () => {
	editing.value = false;
};

const formattedTime = (): string => {
	if(!time) {
		return "";
	}
	const date = new Date(time.value);
	const hours = date.getHours().toString().padStart(2, "0");
	const minutes = date.getMinutes().toString().padStart(2, "0");
	return `${hours}:${minutes}`;
};

computed(() => {
	formattedTime();
});

onMounted(() => {
	getTitle();
});

</script>
